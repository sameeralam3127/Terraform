#  Terraform: Zero → Hero — 1-Month Hands-On Bootcamp (30 days)

**Goal:** Learn Terraform end-to-end: core concepts, state, modules, workspaces, provisioning resources on a cloud provider, testing, CI/CD, security, and production best practices — finishing with a capstone production-grade project.


**Primary cloud used in labs:** AWS (most examples use AWS). Optional notes mention Azure/GCP where appropriate.

**Tools you will use:**

* Terraform CLI (recommended recent stable release 1.x)
* Cloud CLI: AWS CLI (or Azure CLI / gcloud for optional modules)
* Local terminal (Linux / macOS / Windows WSL)
* A text editor or IDE with Terraform support (VS Code recommended)
* Remote state options: Terraform Cloud, S3 + DynamoDB (AWS), Azure Storage + Lock (Azure), GCS + Cloud KMS (GCP)
* Optional: Terraform Enterprise, GitLab CI / Jenkins / Azure DevOps for CI/CD (avoid GitHub per request)
* Optional testing tools: Terratest (Go), tflint, checkov, terraform-compliance

**Prerequisites:**

* Basic command line usage
* Basic networking and cloud concepts (VMs, VPC/subnet, SSH, security groups)
* Free/paid AWS account (or equivalent cloud account) with programmatic access configured

---

# Getting started (before Day 1)

1. Install Terraform CLI (1.x).
2. Install AWS CLI and configure `aws configure` with an IAM user that has permissions for the resources you will create.
3. Create a local workspace folder structure:

   ```
   terraform-bootcamp/
     ├─ day01_basics/
     ├─ modules/
     ├─ projects/
     └─ utils/
   ```
4. Create an IAM user with programmatic access for labs (limited permissions ok, but be mindful of required permissions for creating resources).
5. Optional: create an account on Terraform Cloud for remote runs and state.

---

# Week 1 — Fundamentals & Basic Resource Management

### Day 1 — Introduction & Core Concepts

**Goal:** Understand what Terraform is and core concepts: providers, resources, HCL, state.

* Theory: Infrastructure as Code (IaC) vs imperative provisioning, Terraform lifecycle.
* Lab:

  * `terraform init`
  * Write a simple `main.tf` to create an AWS S3 bucket (or equivalent).
  * Commands: `terraform plan`, `terraform apply`, `terraform destroy`.
* Deliverable: `day01_basics/main.tf` with working S3 bucket and README explaining commands used.

**Code snippet**

```hcl
provider "aws" {
  region = "us-east-1"
}

resource "aws_s3_bucket" "example" {
  bucket = "tf-bootcamp-example-<unique-suffix>"
  acl    = "private"
}
```

---

### Day 2 — Variables, Outputs, and State

**Goal:** Learn input variables, outputs, and inspect Terraform state.

* Theory: variable types, default values, sensitive flag, `terraform state` commands.
* Lab:

  * Refactor Day 1 to accept region and bucket name as variables.
  * Add outputs showing bucket ARN.
  * Inspect state with `terraform show` and `terraform state list`.
* Deliverable: `variables.tf`, `outputs.tf`, updated README.

**Code snippet**

```hcl
variable "region" {
  type    = string
  default = "us-east-1"
}

variable "bucket_name" {
  type = string
}

output "bucket_arn" {
  value = aws_s3_bucket.example.arn
}
```

---

### Day 3 — Resource Graph, Dependencies, and Meta-Arguments

**Goal:** Understand implicit/explicit dependencies, `depends_on`, lifecycle meta-arguments.

* Theory: Resource graph, `lifecycle` block (`create_before_destroy`, `prevent_destroy`).
* Lab:

  * Create an EC2 instance that depends on a security group; illustrate implicit dependency.
  * Add `prevent_destroy` to a critical resource and test plan/apply.
* Deliverable: small project demonstrating dependency and lifecycle.

---

### Day 4 — Locals, Terraform fmt, and Validation

**Goal:** Use `locals`, `terraform fmt`, `terraform validate`, and `terraform console`.

* Theory: Locals for computed values, style and validation commands.
* Lab:

  * Clean up files with `terraform fmt`.
  * Use `locals` to compute subnet CIDRs for a VPC.
  * Validate configuration.
* Deliverable: `locals` example and formatted files.

---

### Day 5 — State Backends (local vs remote) and Locking

**Goal:** Learn about state storage, risks, remote backends, and locking.

* Theory: Local state file issues, S3 backend + DynamoDB locking (AWS), Terraform Cloud backend.
* Lab:

  * Configure S3 backend with DynamoDB state lock.
  * Migrate existing local state to the remote backend (`terraform init -migrate-state`).
* Deliverable: `backend.tf` example documentation for S3 backend.

**S3 backend example**

```hcl
terraform {
  backend "s3" {
    bucket         = "tf-bootcamp-state-bucket"
    key            = "envs/dev/terraform.tfstate"
    region         = "us-east-1"
    dynamodb_table = "tf-state-lock"
    encrypt        = true
  }
}
```

---

### Day 6 — Workspaces

**Goal:** Understand workspaces for environment separation.

* Theory: Workspaces vs separate state files vs separate configs.
* Lab:

  * Create `dev` and `prod` workspaces and deploy lightweight resources showing different outputs.
* Deliverable: README explaining workspace usage and tradeoffs.

---

### Day 7 — Review & Mini Challenge

**Goal:** Reinforce first week concepts.

* Mini challenge: Build a VPC with two subnets, an SG, and an EC2 instance — all configurable via variables, push state to S3 backend.
* Deliverable: `day07_challenge/` folder with files and a short document describing the design and commands used.

---

# Week 2 — Modularization, Inputs/Outputs, and Reusability

### Day 8 — Modules: Basics

**Goal:** Learn module structure and how to call modules.

* Theory: Root module vs child modules, inputs/outputs.
* Lab:

  * Create an EC2 module (`modules/ec2`) that accepts AMI, instance type, SG IDs.
  * Use it from root.
* Deliverable: Reusable `modules/ec2`.

---

### Day 9 — Module Best Practices & Versioning

**Goal:** Module design patterns, versioning, and registry structure.

* Theory: Keep modules small, single responsibility, use variables and outputs, semantic versioning.
* Lab:

  * Split VPC, subnet, and EC2 into separate modules; wire them together in a root project.
* Deliverable: `modules/vpc`, `modules/subnet`, `modules/ec2`.

---

### Day 10 — Module Composition & Outputs

**Goal:** Recompose modules, return useful outputs, avoid leaks.

* Theory: Output hygiene, exposing only necessary info, using module outputs in root.
* Lab:

  * Build root config that uses module outputs to create a load balancer pointing at module-created instances.
* Deliverable: architecture diagram (simple ASCII or markdown) and code.

---

### Day 11 — Remote Module Sources (Registry, Git, Local)

**Goal:** Use remote modules and best practices for sourcing modules.

* Theory: module sources (`git::`, registry, local path), version pinning.
* Lab:

  * Create a module and reference it locally and via a `git::` source (use a local git repo if avoiding remote hosting).
* Deliverable: examples of `module` blocks with different sources.

---

### Day 12 — State Management in Modules & Data Sources

**Goal:** Understand `data` sources and reading remote state.

* Theory: `terraform_remote_state` data source, avoiding data access anti-patterns.
* Lab:

  * Use `terraform_remote_state` (or remote state data) to reference VPC outputs from another workspace.
* Deliverable: `remote_state.tf` demonstration.

---

### Day 13 — Templating, Provisioners and Cloud-Init

**Goal:** Use user data, templates, and reduce reliance on `provisioner` where possible.

* Theory: `templatefile()`, `cloud-init`, and why remote-exec/local-exec are last resort.
* Lab:

  * Create an instance that uses `user_data` with `templatefile()` to bootstrap Nginx.
* Deliverable: template file and explanation.

---

### Day 14 — Review & Module Challenge

**Goal:** Build a reusable module library and publish examples.

* Challenge: Build a module that deploys autoscaling EC2 with LB and health checks; expose variables to control scaling policy.
* Deliverable: Module and usage example in `projects/`.

---

# Week 3 — Advanced Topics: Testing, Security, and Multi-Cloud

### Day 15 — State Security, Secrets Management, and Sensitive Data

**Goal:** Protect secrets, mark sensitive outputs, integrate KMS/Key Vault.

* Theory: `sensitive` attribute, remote state encryption, using KMS/Key Vault to encrypt secrets.
* Lab:

  * Configure S3 with encryption and demonstrate marking outputs sensitive.
* Deliverable: documentation and secure example.

---

### Day 16 — Policy as Code & Static Scanning

**Goal:** Learn policy enforcement and static checking: Sentinel/OPA, tflint, checkov.

* Theory: Preventive controls vs detective scanning.
* Lab:

  * Run `tflint` and `checkov` against your code; add a rule (example: deny public S3 buckets).
* Deliverable: scan outputs and remedial commit.

---

### Day 17 — Testing Infrastructure: Unit & Integration

**Goal:** Introduction to Terratest (Go) or alternative testing approaches.

* Theory: Unit testing modules, integration tests for full stack.
* Lab:

  * Write a simple Terratest that runs `terraform init`, `apply`, checks an endpoint, and `destroy`.
  * If you prefer not to use Go, demonstrate basic integration test scripts using `bash` + `curl`.
* Deliverable: test script(s) and explanation.

---

### Day 18 — Drift Detection & State Refresh

**Goal:** Detect drift and understand `terraform refresh` and `plan` behaviors.

* Theory: Reconciling state, read-only changes outside TF.
* Lab:

  * Manually change a resource in the cloud console and run `terraform plan` to detect drift.
* Deliverable: report showing detected drift and remediation steps.

---

### Day 19 — Workflows: Terraform Cloud / Remote Runs (No GitHub)

**Goal:** Use Terraform Cloud or remote runners for collaboration without relying on GitHub.

* Theory: Remote runs, state management, variable sets, policy checks.
* Lab:

  * Configure a workspace in Terraform Cloud and run a plan/apply manually or via CLI.
* Deliverable: Step-by-step notes for setting up remote runs.

---

### Day 20 — Multi-Cloud Basics (AWS + Azure + GCP)

**Goal:** Learn provider blocks and project structure for multi-cloud configurations.

* Theory: Multiple providers with aliasing, provider configuration best practices.
* Lab:

  * Create a sample config that provisions one resource in AWS and one in Azure using provider aliasing.
* Deliverable: `multi-cloud` example.

---

### Day 21 — Review & Advanced Challenge

**Goal:** Consolidate advanced skills.

* Challenge: Build a secure, tested, and modular three-tier architecture (VPC, autoscaling app layer, RDS or managed DB) with static scanning and a Terratest integration. Use remote backend and demonstrate drift detection.
* Deliverable: project folder with code, scan outputs, test scripts.

---

# Week 4 — Infrastructure at Scale, CI/CD, Observability, and Final Project

### Day 22 — Remote State Workflows at Scale & Locking Strategies

**Goal:** Handling multiple teams, state file organization, and partial state concerns.

* Theory: State per environment, per service, splitting state vs monolith, remote state locking.
* Lab:

  * Reorganize prior project into per-service state files (s3 keys) and test independent applies.
* Deliverable: state layout doc.

---

### Day 23 — CI/CD for Terraform (GitLab CI / Jenkins / Azure DevOps)

**Goal:** Implement automated plan and apply pipeline without GitHub.

* Theory: CI pipeline steps: init, fmt/validate, plan, manual review, apply; secrets in CI.
* Lab:

  * Create a GitLab CI YAML (or Jenkinsfile) that runs `terraform fmt`, `validate`, `plan` and stores the plan artifact. Include a manual approval stage before `apply`.
* Deliverable: `ci/` folder containing CI config and description.

**Example GitLab CI stage**

```yaml
stages:
  - validate
  - plan
  - apply

validate:
  stage: validate
  script:
    - terraform init -backend-config="..." 
    - terraform fmt -check
    - terraform validate

plan:
  stage: plan
  script:
    - terraform plan -out=tfplan
  artifacts:
    paths:
      - tfplan

apply:
  stage: apply
  when: manual
  script:
    - terraform apply tfplan
```

---

### Day 24 — Secrets & Credentials in CI

**Goal:** Safely handle credentials in CI pipelines.

* Theory: Use CI secret stores, environment variables, Vault, or Terraform Cloud variables.
* Lab:

  * Configure GitLab CI variables or inject credentials from a secrets manager; run a pipeline invoking `terraform plan`.
* Deliverable: Secure CI config notes.

---

### Day 25 — Drift Remediation & Rollbacks

**Goal:** Strategies for remediation and safe rollbacks.

* Theory: Immutable infrastructure, blue/green, canary, and state rollback considerations.
* Lab:

  * Simulate a failed deployment, document rollback steps and create a simple restore script using prior state backups.
* Deliverable: rollback playbook.

---

### Day 26 — Cost Estimation & Tagging Strategy

**Goal:** Estimate costs and enforce tagging for chargeback.

* Theory: Cost estimation features (e.g., `terraform plan` with cost estimation tools), consistent tagging.
* Lab:

  * Add tagging module and show how to parameterize tags per environment.
* Deliverable: tagging module and cost estimation notes.

---

### Day 27 — Observability & Monitoring Provisioning

**Goal:** Provision monitoring resources and alerts.

* Theory: Infrastructure telemetry (CloudWatch, Prometheus), alerting on failures.
* Lab:

  * Deploy a CloudWatch alarm for an EC2 instance and an SNS topic and subscription for notifications.
* Deliverable: monitoring.tf and alerting README.

---

### Day 28 — Security Review, Hardening & IAM Best Practices

**Goal:** Harden Terraform code and cloud resources.

* Theory: Least privilege, IAM roles, assume-role patterns, preventing accidental destruction.
* Lab:

  * Create an IAM role with least privilege for Terraform state operations, use `prevent_destroy` on critical resources and demonstrate IAM policy examples.
* Deliverable: IAM policy examples and hardening checklist.

---

### Day 29 — Final Capstone Project (Start)

**Goal:** Start the capstone: design and begin implementing a production-grade environment.

* Capstone summary (see below) — start implementation, set up remote state, modules, CI pipeline, tests.
* Deliverable: capstone repo structure in `projects/capstone/` and initial modules.

---

### Day 30 — Final Capstone Project (Finish) & Demo

**Goal:** Complete capstone, run tests, perform plan/apply via CI, and prepare a deployment handover.

* Deliverable: final Terraform code for capstone, test results, CI pipeline run logs, and a deployment handover doc.

---

# Final Capstone Project (Suggested)

**Project goal:** Build a production-grade, highly available web application stack in AWS using Terraform modules, remote state, CI/CD (non-GitHub), testing, policy checks and monitoring.

**Requirements:**

* VPC with public and private subnets across 2 AZs
* Autoscaling group (or managed service) for application tier behind an Application Load Balancer
* Managed database (RDS/Aurora) in private subnets
* Bastion host or Session Manager for admin access
* Secrets management for DB credentials (AWS Secrets Manager)
* Centralized logging and monitoring (CloudWatch logs and alarms)
* IAM roles with least privilege for application instances and for Terraform
* Remote state in S3 + DynamoDB locking or Terraform Cloud workspace
* Modules for VPC, autoscaling, RDS, monitoring
* CI/CD pipeline (GitLab CI / Jenkins / Azure DevOps) to run plan and manual apply
* Terratest integration and static code scans (tflint/checkov)
* Cost- and security-related tagging and policies

**Deliverables:**

* Clear folder structure and modules
* CI pipeline config and run output
* Test reports, checkov/tflint results
* Deployment playbook and rollback steps
* Short demo script showing deploy → test → destroy

---

# Practical Patterns, File Structure & Best Practices

**Recommended file layout**

```
project/
  ├─ modules/
  │   ├─ vpc/
  │   ├─ ecs/ or ec2-autoscaling/
  │   ├─ rds/
  │   └─ monitoring/
  ├─ envs/
  │   ├─ dev/
  │   │   ├─ main.tf
  │   │   ├─ variables.tf
  │   │   └─ backend.tf
  │   └─ prod/
  ├─ ci/
  │   └─ gitlab-ci.yml
  └─ docs/
```

**Naming & tagging**

* Use consistent naming convention: `<team>-<app>-<env>-<resource>`
* Enforce tags: `project`, `owner`, `environment`, `cost_center`

**State management**

* Keep state per logical service (not a single monolith) to reduce blast radius.
* Use remote backend with locking for team collaboration.
* Encrypt state and secure backend credentials.

**Module design**

* Single responsibility modules
* Clear, documented inputs/outputs with defaults where sensible
* Avoid hardcoding account IDs/regions inside modules; use provider aliases or pass as variables.

**Security**

* Avoid embedding secrets in TF files; use data sources for secrets from Secrets Manager / Vault.
* Mark sensitive outputs with `sensitive = true`.
* Use IAM roles and assume-role patterns rather than long-lived access keys where possible.

**Testing & QA**

* Run `terraform fmt` and `terraform validate` in CI.
* Static checks: `tflint`, `checkov`, `terraform-compliance`
* Integration: Terratest or scripted end-to-end tests

---

# Useful Commands Cheat-Sheet

* `terraform init` — initialize backend and plugins
* `terraform fmt` — format HCL
* `terraform validate` — validate configs
* `terraform plan -out=tfplan` — plan and save plan artifact
* `terraform apply tfplan` — apply saved plan
* `terraform apply -auto-approve` — apply without prompt (use with caution)
* `terraform state list` — list resources in state
* `terraform show` — human readable state
* `terraform import` — import existing resource to be managed
* `terraform taint` (deprecated in 0.15; use `terraform apply -replace`) — force recreate
* `terraform workspace new <name>` / `terraform workspace select <name>`
* `terraform destroy` — destroy all resources in config

---

# Quick Reference: Example Patterns

**Provider aliasing (multi-provider)**

```hcl
provider "aws" {
  alias  = "uswest2"
  region = "us-west-2"
}

provider "aws" {
  alias  = "useast1"
  region = "us-east-1"
}
```

**Module call with version pin (if using registry)**

```hcl
module "vpc" {
  source  = "app/vpc/aws"
  version = "1.2.0"

  name = "my-vpc"
  cidr = "10.0.0.0/16"
}
```

**Marking outputs sensitive**

```hcl
output "db_password" {
  value     = aws_secretsmanager_secret_version.db_secret.secret_string
  sensitive = true
}
```

---

# Assessment & Certification (self-assessment)

At the end of the month, test yourself:

1. Can you provision a multi-AZ VPC, autoscaling app tier, and managed DB using modules?
2. Can you demonstrate state migration to S3 and locking with DynamoDB?
3. Can you automate plan and apply in a CI pipeline (non-GitHub)?
4. Can you write a Terratest that provisions and validates an endpoint?
5. Can you show static security scanning and fix a failing policy?

If yes to all — you’re at a strong “hero” level for Terraform in practice.

---

# Extras & Next Steps

* Learn advanced topics: Terraform Provider development, provider SDK, building custom providers.
* Explore Terraform Cloud/Enterprise advanced features: workspaces, run tasks, policy sets.
* Learn advanced testing patterns: unit tests for modules, mocking providers.
* Practice real-world migrations: import legacy resources into Terraform incrementally.


