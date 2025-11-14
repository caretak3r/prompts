# Project Overview

This project provides a secure, production-ready Terraform template for deploying Kubernetes clusters on Azure AKS and AWS EKS, with nascent support for GCP GKE. It's designed with a modular approach, allowing for consistent configuration and security best practices across cloud providers.

**Main Technologies:**

*   Terraform
*   Kubernetes (AWS EKS, Azure AKS, GCP GKE)
*   AWS, Azure, GCP

**Architecture:**

The project is structured using Terraform modules to separate concerns for each cloud provider. A root `main.tf` file selects the appropriate module based on the `cloud_provider` variable. Configuration is managed through `.tfvars` files, with pre-configured examples for different environments (dev, perf, prod).

# Building and Running

**Prerequisites:**

*   Terraform >= 1.0
*   Azure CLI (for Azure deployments)
*   AWS CLI (for AWS deployments)
*   GCP CLI (for Google Cloud deployments)
*   `kubectl`

**Key Commands:**

1.  **Initialization:**
    ```bash
    terraform init
    ```

2.  **Planning:**
    ```bash
    # For a specific environment
    terraform plan -var-file="environments/dev-azure.tfvars"
    ```

3.  **Applying:**
    ```bash
    # For a specific environment
    terraform apply -var-file="environments/dev-azure.tfvars"
    ```

4.  **Destroying:**
    ```bash
    terraform destroy
    ```

**TODO:** The `outputs.tf` file is empty. It would be beneficial to output key information like the cluster endpoint, kubeconfig commands, and other relevant details.

# Development Conventions

*   **Modular Design:** The project is organized into modules for each cloud provider (`aws`, `azure`, `gcp`).
*   **Environment-specific Configurations:** Use the `environments` directory to store `.tfvars` files for different deployment environments.
*   **Variable Definitions:** All configurable options are defined in `variables.tf`.
*   **Size-based Configurations:** The `node_size_config` variable allows for deploying clusters with different resource allocations (small, medium, large).
*   ALWAYS LOOK UP THE MODULE YOU ARE ABOUT TO USE (MODULE/RESOURCE/CONFIGURATIONS) navigate to the documentation URL and make sure you implement clauses and resources correctly. Try to be as complete as possible (minimally required values for configuration) and always use their configured defaults.
*   If you make changes to the provider.tf files or required terraform version always test these changes (terraform init) PRIOR to making changes to any resource or modules code. This will keep you from breaking things further.
*   When using open source terraform modules, please get the documentation + dependencies + inputs from the terraform module registry.
* Ensure we use something consistent like name_prefix for all resources prefix standards and naming conventions.
* Keep naming conventions across all modules similar. For example, the network resource for all modules is VPC, knowing that azure calls it's VPC a VNET, and so on. We want to use consistent variables names across all modules that are familiar and similar and easy to understand. Don't prefix things like region or cluster_name with cloud specific naming or prefixes.
* Consistent module usage should look like this, where we always start with the source, and then the count (i.e the toggle to enable or disable the module)
```
module "bastion" {
  source = "./bastion"
  count  = var.enable_bastion ? 1 : 0
```
* Always update the respective cloud environments `terraform.tfvars.example` file so the user knows what they can toggle on/off, but provide them with the least or minimal amount of things they need to run the infrastructure-as-code projects.
* Ensure all cloud modules have implemented set of tags defined by whatever the user provides in their tfvars file.
* Ensure we retain the wording or description or variable for different cloud providers: GCP uses location instead of region, like AWS. AWS uses region instead of location. We want to keep these things and things like this unchanged to not confuse the end user. 

### Committing Changes

Before committing any changes, please run the pre-commit hooks to ensure your code adheres to the project's quality standards. This will automatically format and lint your code.

```bash
pre-commit run --all-files
```

### Cleaning Up
Once all testing is completed and is successful, remove any terraform artifacts and directories like `.terraform` `.terraform.lock.hcl` and any state files.
