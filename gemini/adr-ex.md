ADR-001: Secrets Broker Service Architecture for Hashicorp Vault Integration

Date: 2025-11-14

Status: Proposed (Updated)

1. Context

We are deploying a suite of microservices (TOKENIZER Control Plane, Data Plane, BFF, Crypto Gen, Config API) via separate Helm charts. These services require a secure and auditable mechanism to read and, in some cases, write secrets to a central Hashicorp Vault instance.

This ADR explores options for a "secrets-broker" service to facilitate this, acting as the single point of contact for all services. This broker must enforce security, map Kubernetes-native identities (ServiceAccounts) to Vault permissions, and be manageable within our Kubernetes environment.

Key Requirements (Updated):

Secure Tenancy: Isolate permissions. The TOKENIZER Data Plane must not be able to access TOKENIZER Control Plane secrets.

Mandatory Read/Write Operations: The solution must support both read and write operations. Writing secrets (e.g., by TOKENIZER Crypto Gen Tool) is a non-negotiable requirement.

HTTP/S API Client: All TOKENIZER services will be updated to communicate with the secrets-broker via an internal HTTP/S API.

K8s-Native: Leverage Kubernetes constructs like Namespaces, ServiceAccounts (SAs), and RBAC.

Auditability: All access to Vault should be auditable, ideally tied back to the original K8s service identity.

Centralized Policy: The mapping of K8s identities to Vault permissions should be managed centrally.

2. Decision Drivers

Handling Writes: (Critical) Securely managing write operations is a primary challenge and a mandatory requirement.

Security (Least Privilege): The broker must not possess "god-mode" permissions. The access granted to a service should be the minimum required.

Application Impact: (Resolved) The decision has been made to update all applications to use an HTTP/S API. This is no longer a blocker and is now a prerequisite implementation task.

Operational Overhead: How complex is the solution to deploy, configure (both in K8s and Vault), and maintain?

Scalability & Performance: The solution must not become a significant bottleneck.

3. Considered Options

Option 1: K8s Auth Passthrough Proxy

Summary: The secrets-broker acts as a smart proxy. It receives API requests from services, validates the service's identity, and then uses the service's own ServiceAccount token to authenticate to Vault via the Vault Kubernetes Auth Method. All permissions are enforced directly by Vault policies mapped to K8s SAs.

Architecture & Flow:

sequenceDiagram
    participant Pod as Tokenizer Pod (NS: tokenizer-cg)
    participant BrokerSvc as secrets-broker (NS: broker)
    participant BrokerPod as Broker Pod
    participant Vault as Hashicorp Vault

    Note over Pod,BrokerPod: Pod mounts a projected, audience-bound SA token.
    
    Pod->>+BrokerSvc: POST /api/v1/write/kv/crypto-gen/new<br/>(Header: Auth: Bearer <pod-sa-token>)
    BrokerSvc->>+BrokerPod: Forward request
    
    BrokerPod->>BrokerPod: 1. Inspect token (get 'ns' and 'sa').<br/>2. Check internal policy mapping:<br/>('ns: tokenizer-cg', 'sa: crypto-gen-sa') -> 'vault-role: crypto-gen'
    
    BrokerPod->>+Vault: POST /v1/auth/kubernetes/login<br/>(role: 'crypto-gen', jwt: <pod-sa-token>)
    Vault-->>-BrokerPod: { auth: { client_token: 'vault-pod-token', policies: ['crypto-gen-policy'], ... } }
    
    BrokerPod->>+Vault: POST /v1/kv/crypto-gen/new<br/>(Header: X-Vault-Token: 'vault-pod-token')<br/>(Body: { data: ... })
    
    Note over Vault: Vault policy 'crypto-gen-policy' (tied to role) permits this write.
    
    Vault-->>-BrokerPod: { 200 OK }
    BrokerPod-->>-Pod: { status: "written", path: "..." }


Analysis:

Pros:

Excellent Security: The broker itself is not privileged in Vault. It only proxies auth. All permissions are enforced by Vault policies, which are mapped 1:1 with K8s ServiceAccounts. This is true "least privilege."

Handles Writes Perfectly: (Meets Requirement) Write operations are secured by the same mechanism. The Crypto Gen Tool's SA is mapped to a Vault policy that allows write on its specific path, while the Data Plane's SA policy only allows read.

Auditable: Vault audit logs will show the specific K8s SA (via the kubernetes auth method) that performed every action.

No Secret Storage in etcd: Secrets are fetched "just-in-time" and are never stored in K8s Secret objects.

Cons:

Performance: Every secret request involves at least two API calls (1 to broker, 1 from broker to Vault, plus the initial auth). This is acceptable for boot-time config but could be slow for frequent, "just-in-time" secret requests. Caching the vault-pod-token in the broker (keyed by pod-sa-token hash) could mitigate this.

Complexity: Requires configuration in three places:

K8s: ServiceAccount for each pod, NetworkPolicy.

Vault: kubernetes auth backend, Vault policies, and Vault roles (to map K8s SA to Vault policy).

Broker: The internal policy mapping (ns, sa) to a vault-role.

Option 2: CRD-based "Secret Sync" Operator (Read-Only)

Summary: The secrets-broker is a Kubernetes Operator. To get a secret, a service's Helm chart deploys a Custom Resource (e.g., kind: TokenizerSecretRequest). The operator sees this CR, fetches the secret from Vault using its own privileged identity, and creates a native Kubernetes Secret.

Architecture & Flow:

graph TD
    subgraph "Helm Chart (tokenizer-dp)"
        A[1. Deploy: TokenizerSecretRequest CR<br/>(spec: path='kv/data-plane/db')] --> B((tokenizer-dp NS))
        B --> C[3. Deploy: Tokenizer Pod]
        C -- mounts --> D[4. K8s Secret<br/>(name: db-creds)]
    end

    subgraph "secrets-broker NS"
        E[Broker Operator Pod<br/>(Controller)]
    end

    subgraph "Hashicorp Vault"
        F[Vault<br/>(kv/data-plane/db)]
    end

    subgraph "K8s API Server"
        G[K8s API]
    end

    A -- 1. Create --> G
    E -- 2. Watches CRs --> G
    
    E -- 2. Policy Check --> E
    E -- 2. Auth to Vault --> F
    E -- 2. Reads Secret --> F
    E -- 2. Creates K8s Secret --> G
    D -- 2. Is Created By --> G


Analysis:

Pros:

K8s-Native Workflow: Managing access is done via K8s RBAC (who can create/edit TokenizerSecretRequest CRs).

Cons:

Disqualified - Unsuitable for Writes: (Blocker) This pattern is not designed for writing secrets. A SecretWriteRequest CRD would be awkward, insecure, and non-atomic. It does not meet the mandatory write requirement.

Disqualified - Misaligned with Client: The decision to update all services to HTTP/S API clients makes this "pull" model (where services read from K8s Secrets) obsolete and contrary to the new technical direction.

Secrets Stored in etcd: The primary drawback. All secrets are replicated as K8s Secret objects, meaning they are stored (base64-encoded) in the cluster's etcd database.

Privileged Operator: The operator's SA needs broad permissions in both K8s (read/write secrets in many namespaces) and Vault (read access to all synced secrets).

4. Decision

Option 1 (K8s Auth Passthrough Proxy) is the only viable solution.

The new requirements have made the decision clear:

The mandatory write requirement disqualifies Option 2.

The decision to update all services to use an HTTP/S API negates the primary benefit of Option 2 (No Application Impact) and aligns perfectly with the architecture of Option 1.

Option 1 is now the clear and undisputed path forward. It is the only solution that securely handles both read and write operations, enforces true least-privilege, provides clear auditability, and aligns with the new technical requirement for all services to act as API clients.

The application-level changes are an accepted one-time engineering cost that provides a long-term, secure, and scalable foundation for all service-to-secret interactions.

5. Implementation Details (for Option 1)

A. K8s/Helm Chart Responsibilities

secrets-broker Chart:

Deploys the Broker Deployment, Service (ClusterIP), and ServiceAccount.

Deploys NetworkPolicy to allow Ingress from namespaces with a label (e.g., tokenizer-service: "true") on the broker's port.

Deploys a ConfigMap for the broker's policy: (ns, sa) -> vault-role.

# broker-policy-configmap.yaml
policy:
  - k8s_ns: "tokenizer-dp"
    k8s_sa: "tokenizer-dp-sa"
    vault_role: "tokenizer-dp-role"
  - k8s_ns: "tokenizer-cg"
    k8s_sa: "tokenizer-cg-sa"
    vault_role: "tokenizer-cg-role"


TOKENIZER-* Charts:

Must create a unique ServiceAccount (e.g., tokenizer-dp-sa).

Deployment: Must set serviceAccountName: tokenizer-dp-sa.

Deployment: Must mount a projected service account token:

volumes:
  - name: vault-sa-token
    projected:
      sources:
        - serviceAccountToken:
            path: token
            audience: "secrets-broker" # Binds the token for use only by the broker
            expirationSeconds: 3600
...
containers:
  - name: my-app
    volumeMounts:
      - name: vault-sa-token
        mountPath: "/var/run/secrets/broker-token"
        readOnly: true


Deployment: Must inject the broker URL and token path as env vars:

env:
  - name: SECRETS_BROKER_URL
    value: "[http://secrets-broker.secrets-broker.svc.cluster.local:8080](http://secrets-broker.secrets-broker.svc.cluster.local:8080)"
  - name: SECRETS_BROKER_TOKEN_PATH
    value: "/var/run/secrets/broker-token/token"


B. Vault Administrator Responsibilities

Enable K8s Auth:

# (One-time setup)
vault auth enable kubernetes
vault write auth/kubernetes/config \
    kubernetes_host="https$KUBERNETES_SERVICE_HOST:$KUBERNETES_SERVICE_PORT" \
    kubernetes_ca_cert=@/var/run/secrets/kubernetes.io/serviceaccount/ca.crt \
    ...


Create Vault Policies:

# policy-tokenizer-dp.hcl
path "kv/data/tokenizer/data-plane/*" {
  capabilities = ["read"]
}

# policy-tokenizer-cg.hcl
path "kv/data/tokenizer/crypto-gen/*" {
  capabilities = ["read", "write", "list"]
}


Create Vault K8s Auth Roles: (This binds K8s SA to Vault Policy)

# Role for Data Plane (read-only)
vault write auth/kubernetes/role/tokenizer-dp-role \
    bound_service_account_names="tokenizer-dp-sa" \
    bound_service_account_namespaces="tokenizer-dp" \
    policies="default,policy-tokenizer-dp" \
    token_ttl="1h"

# Role for Crypto Gen (read-write)
vault write auth/kubernetes/role/tokenizer-cg-role \
    bound_service_account_names="tokenizer-cg-sa" \
    bound_service_account_namespaces="tokenizer-cg" \
    policies="default,policy-tokenizer-cg" \
    token_ttl="1h"
