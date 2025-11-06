
# 🧱 Terraform Interview Questions (GCP Focused)

## ❓ What are the different blocks used in Terraform (in GCP)?

When writing Terraform code for Google Cloud (GCP), we commonly use the following blocks:

---

### 0) Terraform Settings Block
terraform {
  required_version = ">= 1.8"
  required_providers {
    google = {
      source = "hashicorp/google"
      version = ">= 5.32.0"
    }
  }
}

### 1) **provider** Block
This block tells Terraform which cloud provider to use and how to connect to it.

```hcl
provider "google" {
  project = "my-gcp-project"
  region  = "us-central1"
  zone    = "us-central1-a"
}
```

---

### 2) **resource** Block
Used to **create actual cloud resources** such as VMs, VPC networks, storage buckets, etc.

```hcl
resource "google_compute_instance" "vm1" {
  name         = "my-vm"
  machine_type = "e2-medium"
  zone         = "us-central1-a"

  boot_disk {
    initialize_params {
      image = "debian-cloud/debian-11"
    }
  }

  network_interface {
    network = "default"
  }
}
```

---

### 3) **variable** Block
Used to define input values, making configurations reusable and dynamic.

```hcl
variable "instance_name" {
  type    = string
  default = "web-server"
}
```

---

### 4) **output** Block
Displays useful information after running `terraform apply` (example: IP addresses, URLs).

```hcl
output "public_ip" {
  value = google_compute_instance.vm1.network_interface[0].access_config[0].nat_ip
}
```

---

### 5) **locals** Block
Used to create temporary or derived values for internal use.

```hcl
locals {
  machine_type = "e2-small"
}
```

---

### 6) **data** Block
Used to **read existing resources** in GCP instead of creating new ones.

```hcl
data "google_compute_network" "default" {
  name = "default"
}
```

---

### 7) **module** Block
Helps reuse Terraform code by grouping resources into reusable components.

```hcl
module "vpc" {
  source       = "./modules/vpc"
  network_name = "my-custom-vpc"
}
```

---

## 🧠 Quick Summary

| Block | Purpose |
|------|---------|
| provider | Connect Terraform to GCP |
| resource | Creates cloud infrastructure |
| variable | Input values |
| output | Display values after deployment |
| locals | Internal helper values |
| data | Read existing infrastructure |
| module | Reuse and organize Terraform code |

---

## ⭐ One-Line Interview Answer
**Terraform uses blocks like provider, resource, variable, output, locals, data, and modules. Provider connects to GCP, resource creates infrastructure, and modules help reuse code.**


## 📦 Terraform Modules Approach

## What Are Terraform Modules?
Terraform **modules** are reusable blocks of infrastructure code.  
They help us **avoid duplication**, **standardize deployments**, and **keep Terraform clean**.

---

## Module Structure We Followed in Organization

We used a **Root Module + Child (Reusable) Modules** approach:

### 1) Root Modules
These were **environment-specific** folders such as:
```
dev/
stage/
prod/
```
Each environment had its own:
```
main.tf
variables.tf
outputs.tf
```

### 2) Child Modules (Reusable)
These were stored under the `modules/` directory and reused across multiple environments.

Example reusable modules:
```
modules/
├── vpc/
├── gke/
├── iam/
├── cloudsql/
└── compute_instance/
```

---

## 🗂️ Example Folder Structure

```
terraform/
│
├── modules/
│   ├── vpc/
│   ├── gke/
│   ├── compute_instance/
│   └── cloudsql/
│
├── dev/
│   └── main.tf
├── stage/
│   └── main.tf
└── prod/
    └── main.tf
```

---

## 🎯 Why We Followed This Approach

| Benefit | Description |
|--------|-------------|
| Reusability | Same modules used across Dev, Stage, Prod |
| Standardization | Consistent infrastructure everywhere |
| Cleaner Code | No duplicate configuration |
| Easy Maintenance | Update logic once → reused everywhere |

---

## ⭐ One-Line Interview Answer

**We used a Root Module + Reusable Child Modules approach, where shared resources were defined as modules and consumed by Dev, Stage, and Prod environments.**



## ⚙️ Common Terraform Functions Used in GCP

Terraform provides built-in functions that help transform data and simplify configurations.  
These functions are not only for GCP, but are commonly used in GCP infrastructure deployments.

---

## 1) `file()`
Reads the contents of a local file.
```hcl
credentials = file("service-account.json")
```
Used to load **GCP Service Account Keys**.

---

## 2) `lookup()`
Safely retrieves a value from a map.
```hcl
machine_type = lookup(var.vm_types, "prod", "e2-medium")
```

---

## 3) `join()`
Combines list elements into a single string.
```hcl
allowed_ips = join(",", var.whitelist_ips)
```

---

## 4) `split()`
Splits a string into a list.
```hcl
ip_list = split(",", var.csv_ips)
```

---

## 5) `length()`
Returns the number of items in a list.
```hcl
count = length(var.instances)
```
Helpful when deploying **multiple VMs**.

---

## 6) `format()`
Formats a value into a string.
```hcl
name = format("vm-%s", var.env)
```

---

## 7) `cidrsubnet()`
Used to create subnet CIDRs inside a VPC.
```hcl
subnet_cidr = cidrsubnet(var.vpc_cidr, 8, 1)
```
**Very common in VPC designs.**

---

## 8) `toset()` and `tolist()`
Converts data types.
```hcl
subnets = toset(var.subnet_names)
```

---

## 9) `for_each` Loop (Important for Scaling)
Used to create multiple resources dynamically.
```hcl
resource "google_compute_firewall" "rules" {
  for_each = var.ports
  name     = "allow-${each.key}"
  allow {
    protocol = "tcp"
    ports    = [each.value]
  }
}
```

---

## 10) `templatefile()`
Loads and renders a template file.
```hcl
startup_script = templatefile("startup.sh", {
  app_name = var.app_name
})
```

---

## ⭐ Interview Summary
Common Terraform functions used in GCP include:  
`file()`, `lookup()`, `join()`, `split()`, `length()`, `format()`, `cidrsubnet()`, `toset()`, and `templatefile()` — usually combined with `for_each` to create scalable resources.


## 🔗 Terraform Dependencies (Implicit & Explicit) — With Real-Time Examples

## 1) What are Dependencies in Terraform?
Dependencies define **which resource should be created first**.
Terraform builds a **resource graph** and decides the order of execution.

There are **two types** of dependencies:

---

## ✅ Implicit Dependency
Implicit dependency happens **automatically** when one resource **references another**.
Terraform understands the relationship by itself.

### **Real-Time Example:** GCP VPC → Subnet

```hcl
resource "google_compute_network" "vpc" {
  name = "prod-vpc"
  auto_create_subnetworks = false
}

resource "google_compute_subnetwork" "subnet" {
  name          = "prod-subnet"
  ip_cidr_range = "10.0.1.0/24"
  network       = google_compute_network.vpc.id   # reference creates dependency
}
```

Here:
- Subnet **depends on** VPC
- Terraform **automatically** creates VPC first ✅

---

## 📝 Explicit Dependency
Explicit dependency is used when **no direct reference** exists, but **order matters**.
We use `depends_on`.

### **Real-Time Example:** VM must be created **after** Firewall Rule

```hcl
resource "google_compute_firewall" "allow_http" {
  name    = "allow-http"
  network = google_compute_network.vpc.name
  allow {
    protocol = "tcp"
    ports    = ["80"]
  }
}

resource "google_compute_instance" "vm" {
  name         = "web-server"
  machine_type = "e2-small"
  zone         = "us-central1-a"

  depends_on = [
    google_compute_firewall.allow_http   # manually declaring dependency
  ]
}
```

Here:
- The VM **does not reference** the firewall resource directly
- But we **must create firewall before VM**
- So `depends_on` ensures correct order ✅

---

## 🆚 Quick Comparison

| Feature | Implicit Dependency | Explicit Dependency |
|--------|---------------------|--------------------|
| How Created | Automatically by references | Manually using `depends_on` |
| Use Case | Resources naturally connected | Need to force order |
| Example | Subnet depends on VPC | VM depends on Firewall |

---

## ⭐ One-Line Interview Answer
**Implicit dependencies are created automatically when a resource references another, while explicit dependencies use `depends_on` when Terraform needs manual order control.**


## 🧰 Common Terraform Commands (Simple & Real-Time Use)

## Basic Commands
| Command | Purpose |
|--------|---------|
| `terraform init` | Initializes Terraform and downloads provider plugins. |
| `terraform validate` | Checks if the Terraform code syntax is correct. |
| `terraform fmt` | Formats Terraform files to standard style. |

---

## Plan & Apply
| Command | Purpose |
|--------|---------|
| `terraform plan` | Shows what changes Terraform will make. |
| `terraform apply` | Applies the changes and creates/updates resources. |
| `terraform destroy` | Deletes resources created by Terraform. |

---

## State Inspection
| Command | Purpose |
|--------|---------|
| `terraform show` | Displays current Terraform state details. |
| `terraform output` | Shows output values defined in outputs.tf. |
| `terraform state list` | Lists resources tracked in the Terraform state. |
| `terraform state show <resource>` | Shows details of a specific resource. |

---

## Fixing / Managing Resources
| Command | Purpose |
|--------|---------|
| `terraform taint <resource>` | Marks a resource to be recreated on next apply. |
| `terraform import <resource> <id>` | Brings existing infrastructure under Terraform control. |

---

## Multi-Environment (DEV / STAGE / PROD)
| Command | Purpose |
|--------|---------|
| `terraform workspace new <env>` | Creates a new environment workspace. |
| `terraform workspace select <env>` | Switches to the selected environment. |

---

## 🏗️ Real-Time Workflow Example

```
terraform init
terraform fmt
terraform validate
terraform plan -out=tfplan
terraform apply tfplan
terraform output
```

To delete infrastructure:
```
terraform destroy
```

To manage environments:
```
terraform workspace new dev
terraform workspace select dev
```

---

## ⭐ One-Line Interview Answer
I regularly use `init`, `plan`, `apply`, and `destroy` for provisioning,  
`fmt` and `validate` for code quality,  
and `workspace`, `taint`, and `import` for environment and lifecycle management.


## 🗄️ Terraform Backend (GCS) and State Locking

## What is a Backend?
The **backend** in Terraform defines where the **state file** (`terraform.tfstate`) is stored.
Instead of keeping state locally, we store it in **Google Cloud Storage (GCS)** for:
- Team collaboration
- Backup & recovery
- Centralized infrastructure tracking

---

## GCS Backend Example
```hcl
terraform {
  backend "gcs" {
    bucket  = "my-terraform-state-bucket"
    prefix  = "env/dev"
  }
}
```

### Explanation:
| Key     | Meaning |
|--------|---------|
| bucket | GCS bucket that stores the state file |
| prefix | Folder path inside bucket (separates environments) |

---

## 🔐 State Locking
When multiple people run Terraform at the same time — conflicts may occur.
So Terraform uses **Google Cloud Storage + Locking (via GCS metadata)** to:
- Avoid race conditions
- Ensure only one person can modify state at a time

✅ This prevents broken infrastructure deployments.



## 🌍 Terraform Workspaces (Managing dev/stage/prod)

## Why Workspaces?
Workspaces allow us to use the **same Terraform code** for multiple environments:
- dev
- stage
- prod

Each workspace maintains a **separate state file**, so environments stay isolated.

---

## Common Workspace Commands
| Command | Purpose |
|--------|---------|
| `terraform workspace list` | View all workspaces |
| `terraform workspace new dev` | Create `dev` workspace |
| `terraform workspace select dev` | Switch to `dev` workspace |
| `terraform workspace select prod` | Deploy to `prod` |

---

## Example Use
```
terraform workspace new dev
terraform apply
terraform workspace new prod
terraform apply
```

Same code → Different environments → Separate state files ✅


## 🔄 Terraform Lifecycle Rules (create_before_destroy & prevent_destroy)

Terraform allows us to control how resources behave during updates.

---

## 1) `create_before_destroy`
Used to **create new resource first**, then destroy the old one.
Prevents downtime.

### Example:
```hcl
resource "google_compute_instance" "vm" {
  name = "app-vm"

  lifecycle {
    create_before_destroy = true
  }
}
```

✅ Useful for production resources.

---

## 2) `prevent_destroy`
Used to **protect critical resources** from accidental deletion.

### Example:
```hcl
resource "google_compute_network" "vpc" {
  name = "core-vpc"

  lifecycle {
    prevent_destroy = true
  }
}
```

🛑 Terraform will **block destroy** unless removed manually.

---

## ⭐ One-Line Interview Answer
Lifecycle rules help prevent accidental deletion and ensure safe replacement of resources during updates.



## 🔁 for_each vs count in Terraform (With Real-Time Examples)

## 1) `count` (Used for creating multiple identical resources)
- Best when all resources are the **same**
- Uses **index numbers**

### Example: Create 3 identical VM instances
```hcl
resource "google_compute_instance" "vm" {
  count        = 3
  name         = "app-vm-${count.index}"
  machine_type = "e2-small"
  zone         = "us-central1-a"
}
```

---

## 2) `for_each` (Used when resources are **unique** or named)
- Best when dealing with **maps** or **sets**
- Uses **key-value pairs**

### Example: Create multiple firewall rules with different ports
```hcl
variable "ports" {
  type = map(any)
  default = {
    http  = 80
    https = 443
  }
}

resource "google_compute_firewall" "rules" {
  for_each = var.ports
  name     = "allow-${each.key}"
  allow {
    protocol = "tcp"
    ports    = [each.value]
  }
}
```

---

## 🆚 Quick Comparison

| Feature | `count` | `for_each` |
|--------|--------|------------|
| Identical resources | ✅ Yes | ❌ No |
| Unique named resources | ❌ No | ✅ Yes |
| Index based | Yes | No, uses keys |

---

## ⭐ One-Line Interview Answer
Use **`count`** for identical resources, use **`for_each`** when resources have **unique values or names**.



## ⚙️ Conditions in Terraform (when, depends_on, count, expressions)

## 1) Conditional Resource Creation (`count`)
```hcl
resource "google_compute_instance" "vm" {
  count = var.enable_vm ? 1 : 0
}
```
✅ Creates the resource **only if** `enable_vm` = true.

---

## 2) Conditional Expressions
```hcl
machine = var.env == "prod" ? "e2-medium" : "e2-small"
```

---

## 3) Explicit Dependency (`depends_on`)
```hcl
resource "google_compute_instance" "app" {
  depends_on = [google_compute_firewall.allow_http]
}
```
✅ Ensures resource order even without references.

---

## 4) `when` Equivalent (Not an actual keyword)
Terraform does not use `when` like Ansible;
we use **count**, **for_each**, or **expressions** for conditional logic.

---

## ⭐ One-Line Interview Answer
Terraform uses **count**, **for_each**, and **conditional expressions** to control resource creation, and `depends_on` only when Terraform cannot detect dependencies automatically.



## 🗂️ Terraform State Management Commands

| Command | Purpose |
|--------|---------|
| `terraform state list` | Shows all resources tracked in state |
| `terraform state show <resource>` | Shows details of a resource |
| `terraform state pull` | Downloads current state content |
| `terraform state mv <src> <dest>` | Moves a resource between addresses |
| `terraform state rm <resource>` | Removes a resource from state (does not destroy it) |
| `terraform refresh` | Syncs state with real infrastructure |
| `terraform import <resource> <id>` | Adds existing infrastructure to state |

---

## Real Example: Import Existing VM into State
```hcl
terraform import google_compute_instance.myvm projects/my-proj/zones/us-central1-a/instances/instance-1
```

---

## ⭐ One-Line Interview Answer
Terraform state commands help **inspect**, **modify**, and **sync** the state file to manage real cloud resources safely.



## 🗄️ Backend (Advanced): GCS Remote State + Locking + Terraform Cloud

## Why Remote Backend?
- Team collaboration (shared state)
- Versioning & backups
- CI-friendly
- Access control

---

## Option A) Google Cloud Storage (GCS) Backend

### 1) Create bucket (one-time)
```bash
PROJECT_ID="your-project"
BUCKET_NAME="tf-state-${PROJECT_ID}"
gsutil mb -p $PROJECT_ID -l asia-south1 gs://$BUCKET_NAME
gsutil versioning set on gs://$BUCKET_NAME
```

### 2) Configure backend
```hcl
terraform {
  backend "gcs" {
    bucket = "tf-state-your-project"
    prefix = "env/dev"
  }
}
```

### 3) Initialize
```bash
terraform init
```

> **Locking:** GCS provides object preconditions that effectively prevent concurrent writes. For stricter org-wide control, pair with a workflow gate (e.g., CI required reviewers) or use Terraform Cloud below.

---

## Option B) Terraform Cloud/Enterprise (State + Locking + RBAC)

### 1) Configure backend
```hcl
terraform {
  backend "remote" {
    hostname     = "app.terraform.io"
    organization = "your-org"

    workspaces {
      name = "gcp-prod"
    }
  }
}
```

### 2) Login and init
```bash
terraform login
terraform init
```

**Benefits**
- Built-in state **locking**
- **RBAC**, audit logs
- **Run tasks** & **policy as code** (Sentinel)
- VCS integration (PR plans)

---

## Security Tips
- Restrict who can read/write state (GCS IAM or TFC RBAC).
- Enable **bucket versioning** and lifecycle rules.
- Keep **secrets** out of state (use Secret Manager, not outputs).
- Prefer **Workload Identity Federation** in CI (no static keys).



## ⤴️ Importing Existing GCP Resources into Terraform State

## Why Import?
Bring resources **created outside Terraform** under Terraform control **without recreating** them.

---

## General Syntax
```bash
terraform import <resource.address> <resource-id>
```

---

## 1) Compute Engine VM
**Resource block (minimal)**
```hcl
resource "google_compute_instance" "vm" {
  name         = "instance-1"
  machine_type = "e2-small"
  zone         = "us-central1-a"
  boot_disk {}
  network_interface {}
}
```

**Import command**
```bash
terraform import   google_compute_instance.vm   projects/PROJECT_ID/zones/us-central1-a/instances/instance-1
```

---

## 2) VPC Network
```hcl
resource "google_compute_network" "vpc" {
  name                    = "main-vpc"
  auto_create_subnetworks = false
}
```

```bash
terraform import   google_compute_network.vpc   projects/PROJECT_ID/global/networks/main-vpc
```

---

## 3) Subnet
```hcl
resource "google_compute_subnetwork" "subnet" {
  name          = "app-subnet"
  region        = "asia-south1"
  ip_cidr_range = "10.10.0.0/24"
  network       = "main-vpc"
}
```

```bash
terraform import   google_compute_subnetwork.subnet   projects/PROJECT_ID/regions/asia-south1/subnetworks/app-subnet
```

---

## 4) Firewall Rule
```hcl
resource "google_compute_firewall" "allow-http" {
  name    = "allow-http"
  network = "main-vpc"
}
```

```bash
terraform import   google_compute_firewall.allow-http   projects/PROJECT_ID/global/firewalls/allow-http
```

---

## 5) Cloud Storage Bucket
```hcl
resource "google_storage_bucket" "bucket" {
  name     = "my-bucket-123"
  location = "ASIA-SOUTH1"
}
```

```bash
terraform import   google_storage_bucket.bucket   projects/PROJECT_ID/buckets/my-bucket-123
```

---

## Tips
- Start with a **minimal** resource block → `terraform import` → then **run `terraform plan`** to see drift and fill remaining arguments.
- Keep naming identical to existing resources.
- For complex resources (e.g., GKE), consider importing **child resources** (node pools) separately if needed.



## Terraform Best Practices (GCP-Oriented)

## Structure & Modules
- Use **root modules per environment** (`env/dev`, `env/stage`, `env/prod`).
- Keep reusable code in `modules/` (vpc, gke, iam, cloudsql).
- One **resource per file** for large modules (optional but readable).

## State & Backend
- Use **remote state** (GCS or Terraform Cloud).
- Enable **GCS versioning** and least-privilege IAM.
- Never commit `.tfstate` or `.tfvars` with secrets.

## Variables & Secrets
- Keep defaults in `variables.tf`, env overrides in `*.tfvars`.
- Use **Google Secret Manager** for secrets (not outputs/state).
- Validate inputs with `validation` blocks.

## Naming & Tagging
- Standardize names: `${var.project}-${var.env}-${var.component}`.
- Apply labels/tags for cost & ownership (`env`, `team`, `owner`).

## Quality Gates
- Run `terraform fmt -check` and `terraform validate` in CI.
- Add **tfsec/Checkov** for policy scanning.
- Use `pre-commit` hooks for Terraform.

## Workspaces & Promotion
- Separate **state per environment** (workspaces or directories).
- Promote via CI: dev → stage → prod with manual approvals.

## Safety & Lifecycle
- Use `create_before_destroy` for prod-critical resources.
- Protect with `prevent_destroy` where deletion is risky.
- Prefer **explicit dependencies** only when needed.

## Git & Reviews
- Small PRs, clear commit messages.
- Store **example tfvars** (`dev.tfvars.example`) not real secrets.
- Pin provider & Terraform versions in the **terraform** block.

## CI/CD
- Use **Workload Identity Federation** to auth to GCP in CI.
- Plan on PR, **apply only on main** with approvals.
- Export concise **plan summaries** as PR comments.

## Observability & Ops
- Output essentials: IPs, URLs, service accounts.
- Document runbooks for `import`, `state mv`, and rollbacks.
- Regularly `terraform plan` to detect drift.

---

### One-Liner
**Clean modules, remote state, policy checks, safe lifecycles, and CI-controlled applies = reliable Terraform in GCP.**



## 🎯 Destroying a Single Resource in Terraform (target)

## ❓ Q: If you created 10 resources in Terraform, how do you destroy only one resource?

There are **two correct ways** to destroy just one resource without affecting others:

---

## ✅ 1) Best Practice Method — Update Code and Apply

Remove the resource block from your Terraform code and then run:

```bash
terraform plan
terraform apply
```

Terraform will detect that this one resource is no longer defined and will **destroy only that resource**, while keeping the other 9.

**This keeps code and state consistent** ✅

---

## ✅ 2) Quick Method — Targeted Destroy (Without Editing Code)

If you want to delete only one specific resource:

```bash
terraform destroy -target=google_compute_instance.web_vm
```

### Steps:
1. Check the list of resources in state:
   ```bash
   terraform state list
   ```
2. Copy the resource name and substitute in the `-target` argument.

---

## ⚠️ Important Note

Do **not** use:
```bash
terraform state rm <resource>
```
This removes the resource from **state only** and **does not delete it in GCP**, which creates **orphaned** infrastructure.

---

## ⭐ One-Line Interview Answer

To destroy only one resource, I either remove it from code and run `terraform apply` (recommended), or use `terraform destroy -target=<resource_name>` for a quick targeted delete.



## 🔧 `terraform taint` — Full Explanation (with GCP Examples)

> **Status:** The `terraform taint` command is **deprecated** in modern Terraform (>= 0.15).  
> **Recommended replacement:** use **`terraform apply -replace=ADDRESS`** (or `terraform plan -replace=ADDRESS`).  
> This doc shows both, because many teams still say “taint” in practice.

---

## 🧠 What does “taint” mean?
To **taint** a resource is to **mark it for recreation** on the next `terraform apply`.  
Terraform will plan **Destroy → Create** for that resource, even if your code didn’t change.

Use it when a resource is:
- Misconfigured or corrupted in the cloud
- In an **unknown**/bad state
- Needs a clean rebuild without editing code
- A change should cause recreation, but Terraform didn’t detect it

---

## ✅ Modern Way (Preferred): `-replace`
Force recreation **without** changing code or state manually.

```bash
# Preview a plan that replaces only the given resource
terraform plan -replace=google_compute_instance.web1

# Apply and replace only that resource
terraform apply -replace=google_compute_instance.web1
```

### Why better than `taint`?
- It’s explicit in the **plan/apply** command
- Works well in CI (PR Plans)
- No extra “tainted state” to clean up

---

## 🧰 Legacy Way: `terraform taint` / `untaint`
Marks or unmarks a resource in state.

```bash
# Mark for recreation
terraform taint google_compute_instance.web1

# See the plan (shows Destroy/Create for web1)
terraform plan

# Execute the recreation
terraform apply

# If you change your mind before apply:
terraform untaint google_compute_instance.web1
```

> Use only if your org still relies on this workflow. Prefer `-replace` instead.

---

## 🔎 Finding the resource address
```bash
terraform state list
```
Copy the exact **address**, e.g.:
- `google_compute_instance.web1`
- `module.vpc.google_compute_network.this`
- `google_compute_instance.vm[1]` (with `count`)
- `google_compute_instance.vm["db"]` (with `for_each`)

---

## 🏗️ Real GCP Examples

### 1) Recreate a Compute Engine VM
```bash
# Preferred
terraform apply -replace=google_compute_instance.web1

# Legacy
terraform taint google_compute_instance.web1
terraform apply
```

### 2) Recreate a Disk that looks corrupted
```bash
terraform apply -replace=google_compute_disk.app_data
```

### 3) Recreate an Instance Template (then roll a MIG update)
```bash
terraform apply -replace=google_compute_instance_template.tpl
# Then update the MIG (depending on your module/policy) to roll out new VMs
```

### 4) Recreate only one item in a for_each map
```bash
# vm["db"] only (others untouched)
terraform apply -replace=google_compute_instance.vm["db"]
```

### 5) Recreate a module resource (e.g., a sub-module network)
```bash
terraform apply -replace=module.network.google_compute_network.this
```

---

## 🧩 Relationship to lifecycle & dependencies

- **`create_before_destroy` (lifecycle)**  
  Ensures Terraform creates the **new** resource **before** destroying the old one → minimizes downtime.  
  ```hcl
  resource "google_compute_instance" "web1" {
    lifecycle {
      create_before_destroy = true
    }
  }
  ```

- **`prevent_destroy` (lifecycle)**  
  Blocks accidental deletion of critical resources. Temporarily remove/override if you must replace.

- **`replace_triggered_by` (lifecycle)** *(newer Terraform)*  
  Automatically replace a resource when an input changes, even if Terraform wouldn’t normally force it.  
  ```hcl
  resource "google_compute_instance" "web1" {
    lifecycle {
      replace_triggered_by = [
        google_compute_disk.app_data.id
      ]
    }
  }
  ```

- **Dependencies**  
  Replacing one resource may replace dependents if required. Always **review the plan**.

---

## ⚠️ Pitfalls & Best Practices

- **Don’t edit state** for replacement. Use `-replace` / `taint` instead of `state rm`.  
  `state rm` only **forgets** a resource; it does **not** delete in GCP.

- **Always review `terraform plan`** to ensure only what you expect will change.

- **Use in CI**: Prefer `terraform plan -replace=...` in PRs; `apply` on main after approval.

- **Communicate**: Replacements can cause short downtime. Combine with `create_before_destroy` when possible.

---

## 🧪 Quick Cheatsheet

```bash
# Preferred modern flow
terraform plan   -replace=google_compute_instance.web1
terraform apply  -replace=google_compute_instance.web1

# Legacy
terraform taint    google_compute_instance.web1
terraform untaint  google_compute_instance.web1
```

**One-liner (Interview):**  
> We avoid the old `taint` and use `terraform apply -replace=ADDRESS` to force a safe, reviewable recreation of a specific resource. If needed, we combine it with `create_before_destroy` to avoid downtime.



## 🔗 Module Dependencies in Terraform (How One Module Depends on Another)

## ❓ Question:
How does one Terraform module depend on another module?

## ✅ Answer:
Modules **do not automatically depend on each other**.  
A module depends on another **only when its input uses an output** from the first module.

This is called an **implicit dependency**, and Terraform handles the correct order automatically.

---

## 🧱 Real-Time Example
We have:
- **Module 1:** Creates a VPC
- **Module 2:** Creates Subnets *inside* that VPC

So the **subnet module must wait for the VPC module**.

---

### **Module 1: VPC Module Output**
`modules/vpc/outputs.tf`
```hcl
output "vpc_id" {
  value = google_compute_network.vpc.id
}
```

---

### **Module 2: Subnet Module Input**
`modules/subnet/variables.tf`
```hcl
variable "vpc_id" {
  type = string
}
```

---

### **Root Module (Calling Both Modules)**
```hcl
module "vpc" {
  source = "./modules/vpc"
  name   = "prod-vpc"
}

module "subnet" {
  source = "./modules/subnet"
  vpc_id = module.vpc.vpc_id   # <--- Dependency (Implicit)
}
```

✅ Terraform understands:
- Create VPC **first**
- Create Subnets **after**
No need to manually define dependency.

---

## 🛑 When to Use `depends_on`
Use `depends_on` only when **no direct input/output reference exists**, but the order must still be enforced.

Example:
```hcl
module "b" {
  source = "./modules/b"

  depends_on = [
    module.a
  ]
}
```

---

## ⭐ One-Line Interview Answer
One module depends on another when the second module uses outputs from the first module as inputs. This creates an **implicit dependency**, so Terraform builds resources in the correct order automatically.
