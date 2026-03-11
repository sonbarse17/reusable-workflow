# Reusable Python DevSecOps Pipeline

This repository hosts a blazing-fast, strictly locked down, and fully parallelized **Reusable GitHub Actions DevSecOps Workflow** designed exclusively for Python and Infrastructure as Code (IaC) projects.

Because this workflow is built on GitHub Actions `workflow_call`, you do NOT need to duplicate CI code across your organization. Simply call this workflow from any downstream repository.

---

## 🚀 Features

*   **100% Python Native**: Tuned explicitly for Python builds, tests, and caching natively using the latest `actions/setup-python@v6`.
*   **Fully Parallelized**: Once the code builds successfully, it fans out up to **5 simultaneous deep-scan security layers** to eliminate pipeline bottlenecks.
*   **Bleeding-Edge Dependency Pinning**: Every single GitHub Action used is aggressively pinned to its most recent major/master tag (e.g., `actions/checkout@v6`, `build-push-action@v7`).
*   **Dynamic Concurrency Canceling**: Back-to-back commits on the same branch instantly cancel older jobs, saving CI/CD compute minutes drastically.
*   **Fail-Fast Execution**: The `secret-scan` job runs aggressively at the very beginning of the pipeline. If a developer hardcodes a cloud credential, the pipeline fails instantly before downloading heavy dependencies.
*   **Total Cloud Portability**: Out-of-the-box native support for Docker Hub (`docker.io`), GitHub Container Registry (`ghcr.io`), AWS ECR, or GCP Artifact Registry.

---

## 🛡️ The Security Matrix (Shift-Left)

This workflow implements a true enterprise-grade DevSecOps security matrix, utilizing six different industry-standard tools seamlessly.

| Stage | Tool | Description | UI Output |
| :--- | :--- | :--- | :--- |
| **Secrets Scanning** | [Gitleaks](https://github.com/gitleaks/gitleaks-action) | Runs dynamically at checkout to detect hardcoded AWS Keys, Tokens, and Passwords. | Action Logs / Pipeline Fail |
| **SAST (Static Analysis)** | [GitHub CodeQL](https://github.com/github/codeql-action) | Semantically parses Python application logic to find memory bugs and injection flaws. | GitHub Security Tab |
| **SAST (Quality Gate)** | [SonarQube](https://github.com/SonarSource/sonarqube-scan-action) | In-depth continuous inspection engine evaluating code smells, bugs, and duplicates. | SonarQube Dashboard |
| **SCA/Composition** | [OWASP Dep. Check](https://github.com/dependency-check/Dependency-Check_Action) | Scans `requirements.txt` / transitive Python packages against the NVD via API Key. | Downloadable HTML Artifact |
| **IaC Scanning** | [Checkov](https://github.com/bridgecrewio/checkov-action) & [tfsec](https://github.com/aquasecurity/tfsec-action) | Dual-layer analysis of Terraform `.tf` modules to prevent unencrypted S3, open ports, etc. | GitHub Security Tab (SARIF) |
| **Container Scanning** | [Trivy](https://github.com/aquasecurity/trivy-action) | Cracks open the freshly built Docker filesystem layer to identify OS/Linux vulnerabilities. | GitHub Security Tab (SARIF) |

---

## 🛠️ How To Use This Pipeline (Caller Example)

To integrate this DevSecOps pipeline into your application repository, create a file at `.github/workflows/ci.yml` in your downstream repository and paste the following:

```yaml
name: Project DevSecOps Pipeline

on:
  push:
    branches: [ "main", "dev" ]
  pull_request:
    branches: [ "main", "dev" ]

jobs:
  run-reusable-pipeline:
    # Point this to the repository hosting your reusable workflow
    uses: your-org/reusable-workflow/.github/workflows/reusable-ci.yml@main
    with:
      # Optional Input Overrides
      python_version: '3.14'
      docker_registry: 'docker.io'
      docker_image_name: 'your-company/your-app' # Set to your Docker image name
      
      # Optional Toggles (All default to 'true')
      enable_code_scanning: true
      enable_sonar_scan: true
      enable_owasp_scan: true
      enable_iac_scan: true
      enable_secret_scan: true
      enable_docker_build: true

    secrets:
      # The credentials required by the security tools and docker registry
      REGISTRY_USERNAME: ${{ secrets.DOCKERHUB_USERNAME }}
      REGISTRY_PASSWORD: ${{ secrets.DOCKERHUB_TOKEN }}
      
      SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}
      SONAR_HOST_URL: ${{ secrets.SONAR_HOST_URL }}
      
      NVD_API_KEY: ${{ secrets.NVD_API_KEY }}
```

## 🔑 Secret Prerequisites 

For the pipeline to execute at maximum velocity, ensure the following Repository Secrets are configured in your **Caller Downstream Repository**:
*   **`NVD_API_KEY`**: Highly recommended. Request it for free [here](https://nvd.nist.gov/developers/request-an-api-key) to prevent OWASP Dependency Check from heavily rate-limiting your workflow.
*   **`SONAR_TOKEN`** & **`SONAR_HOST_URL`**: Required if `enable_sonar_scan` is set to `true` to authenticate with your SonarQube/SonarCloud server.
*   **`REGISTRY_USERNAME`** & **`REGISTRY_PASSWORD`**: Required if `enable_docker_build` is set to `true` to authenticate with Docker Hub, AWS, or GCP.
