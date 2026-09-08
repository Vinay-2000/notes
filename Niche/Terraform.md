
## 1. What is Terraform?

- **Definition:** An open-source **Infrastructure as Code (IaC)** tool used to automate, provision, and manage infrastructure platforms and services.
    
- **Declarative Approach:** You define the _desired end-state_ of your infrastructure. Terraform automatically determines the steps and dependency graph required to achieve that state.
    
- **Core Purpose:** Mainly focused on **infrastructure provisioning** (e.g., creating VPCs, spinning up EC2 instances, configuring firewalls, setting up network routes, creating IAM users/permissions).
---

## 2. Declarative vs. Imperative Paradigm

- **Declarative (Terraform):**
    
    - You specify **"WHAT"** the final state should look like (e.g., _"I want 7 servers and these firewall rules"_).
        
    - When updating infrastructure, you update the target file to reflect the new desired state. Terraform calculates the _delta_ (what to add, modify, or destroy).
        
    - Keeps configuration files clean, concise, and self-documenting.
        
- **Imperative (e.g., traditional scripts):**
    
    - You specify **"HOW"** to perform every individual step (e.g., _"Step 1: Create network, Step 2: Add 2 instances, Step 3: Configure firewall"_).
        
    - Harder to maintain over time because you must manually track and write the differential steps.
        
---
## 3. Terraform vs. Ansible (Key Differences)

|**Feature**|**Terraform**|**Ansible**|
|---|---|---|
|**Primary Role**|**Infrastructure Provisioning** (creates VPCs, EC2, subnets)|**Configuration Management** (installs software, updates packages, deploys code)|
|**Strengths**|Advanced cloud orchestration, multi-cloud lifecycle management|Application deployment, OS-level package configuration|
|**Common Best Practice**|Use **Terraform** to provision the base infrastructure, then hand over to **Ansible** to configure software inside those instances.||

---
## 4. Terraform Architecture

Terraform relies on two core architectural components:

1. **Terraform Core:**
    
    - Takes **2 Inputs**:
        
        1. **Terraform Configuration File (`.tf`):** Defines the desired end-state written by the user.(there are more like variables.tf, tfvars)
            
        2. **Terraform State File (`terraform.tfstate`):** Tracks the real-world current state of the provisioned infrastructure.
            
    - **Role:** Compares the config file against the state file to calculate the execution plan (what needs to be created, modified, or destroyed).
        
2. **Providers:**
    
    - Plugins for specific cloud or software technologies (e.g., AWS, Azure, GCP, Kubernetes).
        
    - Translates execution plans into API calls to interface with cloud platforms and manage target resources.

```
When Terraform runs in a CI/CD pipeline, the terraform.tfstate file is kept separate from your .tf code and managed remotely—most commonly using GitLab Managed Terraform State (stored securely under Operate/Infrastructure $\rightarrow$ Terraform states) or an external cloud bucket like AWS S3 with DynamoDB state locking.
During terraform apply, the pipeline job authenticates to the remote backend, acquires a state lock to prevent conflicting deployments, updates the live infrastructure along with the remote state file, and releases the lock. The state file is never committed to Git because it contains unencrypted sensitive secrets (such as credentials and private keys) and requires real-time locking to prevent state corruption across parallel pipeline runs.
```

### Key File Roles

- **`main.tf`** $\rightarrow$ Infrastructure resource definitions (EC2, VPC, S3).
    
- **`variables.tf`** $\rightarrow$ Variable declarations (names, types, defaults, descriptions).
    
- **`terraform.tfvars`** $\rightarrow$ Actual variable value assignments (automatically loaded).
    

### Example Code

**1. Declaration (`variables.tf`):**

Terraform

```
variable "instance_type" {
  type        = string
  description = "EC2 instance size"
  default = "t3.micro" //if no assignment this will be used
}

variable "environment" {
  type = string
}
```

**2. Assignment (`terraform.tfvars`):**

Terraform

```
instance_type = "t3.micro"
environment   = "production"
```

### Loading Rules

- **Auto-Loaded:** `terraform.tfvars`, `terraform.tfvars.json`, and `*.auto.tfvars`.
    
- **Custom Names (Manual):** Any custom file name (e.g., `dev.tfvars`, `example.tfvars`) requires the explicit flag:
    
    Bash
    
    ```
    terraform plan -var-file="example.tfvars"
    ```
    
---
## 5. Key CLI Lifecycle Commands

- **`terraform refresh`:** Queries the cloud provider to fetch the current live state of managed resources and update the state file.
    
- **`terraform plan`:** A dry-run preview command. Compares the live state vs. `.tf` code to construct and show the execution plan without making actual changes.
    
- **`terraform apply`:** Executes the actual deployment/plan against the cloud platform through provider APIs _(runs `refresh` and `plan` implicitly)_.
    
- **`terraform destroy`:** Reverts and deletes all infrastructure resources declared in the configuration file in the correct dependency order.

`terraform init` is the mandatory first command you run in any Terraform working directory to set up the execution environment.

```
It performs three core tasks:

Downloads Provider Plugins: Inspects your .tf files and downloads the required provider binaries (such as AWS, Azure, or Kubernetes) into a local .terraform directory so Terraform knows how to talk to those cloud APIs.

2. Initializes the Remote Backend: Connects to your configured backend (such as GitLab Managed State, AWS S3, or Terraform Cloud) to prepare state management and state locking.

3. Downloads External Modules: Fetches any external or reusable modules referenced in your code from Git repositories, local paths, or the Terraform Registry.

It creates a .terraform.lock.hcl file to lock specific provider versions, ensuring consistent runs across your team and CI/CD pipelines.
```

---
## 6. Primary Use Cases

1. **Initial Provisioning:** Spin up new environments from scratch automatically.
    
2. **Infrastructure Management & Drift:** Add, modify, or scale existing production infrastructure cleanly using version-controlled code.
    
3. **Environment Replication:** Easily replicate staging/dev environments identically into production by reusing the same code.
    
4. **Ephemeral Environments:** Quickly spin up a demo/test environment and tear it down cleanly using `terraform destroy`.