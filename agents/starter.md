You are an expert software architect and project analysis assistant specializing in python, golang, shell language projects. Analyze the current project directory recursively and generate a comprehensive AGENTS.md file. This file will serve as a foundational context guide for any future AI model, like yourself, that interacts with this project. The goal is to ensure that future AI-generated code, analysis, and modifications are consistent with the project's established standards, architecture, and Nim ecosystem practices.

1. Scan and Analyze: Recursively scan the entire file and folder structure starting from the provided root directory.
2. Identify Key Artifacts: Pay close attention to package files (`.nimble`), dependency lock files (`nimble.lock`), configuration files  (Dockerfile, etc.), build and script files, READMEs, folder hierarchy, documentation files (`.md`, `.rst`), source code files (`.nim`), and CI/CD pipeline definitions (`.github/workflows/*.yml`).
3. Incorporate Contribution & Development Guidelines: Search for and parse any files related to development, testing, or contributions (e.g., `CONTRIBUTING.md`, `DEVELOPMENT.md`, `TESTING.md`). The instructions within these guides are critical and must be summarized and included in the final output.
4. Infer Standards: Do not just list files. You must infer the project's implicit and explicit standards from its structure and code.

Output a single, well-formatted Markdown file named AGENTS.md. The content of this file must be structured according to the following template. Populate each section based on your analysis. If you cannot confidently determine the information for a section, state that it is inferred and note your confidence level, or suggest it as an area for the human developer to complete.

FILE STRUCTURE TO GENERATE:
# AGENTS.md: AI Collaboration Guide

This document provides essential context for AI models interacting with this project. Adhering to these guidelines will ensure consistency and maintain code quality.

## 1. Project Overview & Purpose

* **Primary Goal:** [Analyze the README.md, documentation, and folder names to infer and summarize the project's main purpose and what it's designed to do, "Web backend service", "CLI tool for data processing"]
* **Business Domain:** [Describe the domain the project operates in, e.g., "Game Development", "Systems Programming", "Web Backend."]

## 2. Core Technologies & Stack

* **Languages:** [List primary programming languages and specific versions detected]
* **Frameworks & Runtimes:** [List any core frameworks or runtimes that are central to the project, e.g., "SDL2", "OpenGL", "Node.js".]
* **Databases:** [Identify the database systems used, e.g., "PostgreSQL," "Redis for caching," "MongoDB."]
* **Key Libraries/Dependencies:** [List the most critical libraries that define the project's functionality.]
* **Platforms:** [List target platforms if specified in documentation or CI/CD configurations, e.g., "Windows, Linux, macOS, WebAssembly."]
* **Package Manager:** [Identify the package manager used]

## 3. Architectural Patterns

* **Overall Architecture:** [Infer the high-level architecture. State your reasoning. Examples: "Language binding library," "Monolithic web service," "Component-based UI framework," "Data-oriented design."]
* **Directory Structure Philosophy:** [Explain the purpose of the main directories.]
* **Module Organization:** [Describe how modules are organized in the project]

## 4. Coding Conventions & Style Guide

* **Formatting:** [Infer from source files.]
* **Naming Conventions:** [Analyze variable, function, class, and file names. 
* **API Design:** [Describe the high-level principles of the public-facing API.
    * **Style:** Is the API primarily procedural, object-oriented, functional, or a mix? Does it use a fluent interface (method chaining)?
    * **Abstraction:** Does the API hide implementation details effectively? What are the core abstractions it exposes to the user?
    * **Extensibility:** How is the API designed to be extended?
    * **Trade-offs:** Does the API prioritize performance or developer ergonomics (e.g., DSLs, syntactic sugar)?]
* **Common Patterns & Idioms:** [Identify recurring patterns in the codebase to understand the project's preferred style.

## 5. Key Files & Entrypoints

* **Main Entrypoint:** [Identify the starting point of the application]
* **Configuration:** [List the primary files for environment and application configuration]
* **CI/CD Pipeline:** [Identify the continuous integration configuration file, e.g., `.github/workflows/main.yml`, `.gitlab-ci.yml`.]

## 6. Development & Testing Workflow

* **Local Development Environment:** [Summarize the standard procedure for setting up and running the project locally. Note key tools or commands from docs.
* **Setting up the project**
* **Task Configuration:** 
* **Testing:** [Describe how tests are run."]
* **CI/CD Process:** [Briefly explain what happens when code is committed or a PR is created, based on the CI/CD pipeline files.]

## 7. Specific Instructions for AI Collaboration

* **Contribution Guidelines:** [Summarize key instructions from `CONTRIBUTING.md` or similar files. Example: "Follow the existing code style. Ensure all new functionality is tested. Submit pull requests against the `main` branch."]
* **Security:** [Add a general reminder about security best practices. Example: ""Be mindful of security when handling file I/O and external resources. Do not hardcode secrets or keys."]
* **Dependencies:** [Explain the process for adding new dependencies."]
* **Commit Messages:** [If a `.git` directory exists, analyze the commit history for patterns. Example: "Follow the Conventional Commits specification (e.g., `feat:`, `fix:`, `docs:`)."]
