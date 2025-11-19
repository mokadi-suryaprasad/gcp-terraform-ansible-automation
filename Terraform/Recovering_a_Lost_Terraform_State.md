# Recovering a Lost Terraform State (GCP)

This guide shows **simple steps** to recover when your `terraform.tfstate` file was deleted or lost and your state was only local (not in GCS). Follow in order. Do **not** run `terraform apply` until you finish recovery.

---

## 1) Stay calm and stop

* **Do not run** `terraform apply` or change infrastructure yet. Without state, Terraform can destroy or recreate resources.

---

## 2) Look for backups (fastest fix)

1. Go to the folder where you run Terraform.
2. Run:

```bash
ls -la
```

3. Look for files like:

* `terraform.tfstate.backup`
* any `*.tfstate` files
* `.terraform/` directory

4. If you find `terraform.tfstate.backup`, restore it:

```bash
cp terraform.tfstate.backup terraform.tfstate
terraform init
terraform state list
terraform plan
```

If resources show up, you are mostly done — check `terraform plan` carefully.

---

## 3) If no local backup — check cloud (are resources still there?)

1. Use `gcloud` to list main resources. Examples:

```bash
gcloud compute instances list --project=MYPROJECT
gcloud compute networks list --project=MYPROJECT
gsutil ls -p MYPROJECT
gcloud sql instances list --project=MYPROJECT
```

2. If resources exist in GCP, we can re-import them into a new state.
3. If resources are missing (deleted), look for snapshots/backups in GCP (disks, Cloud SQL backups, GCS object versions). If no backups, you may not be able to recover data.

---

## 4) Reconstruct state by importing (easy steps)

### A. Prepare terraform config files

* Create `.tf` files that **match** the real resources (use same names, zones, project). Example: if you have a VM named `prod-vm-1`, add a resource block for it.

**Example for a VM (main.tf):**

```hcl
resource "google_compute_instance" "web" {
  name = "prod-vm-1"
  zone = "asia-south1-a"
  project = "my-gcp-project"
  # Add only the fields that are essential and won't differ.
}
```

**Example for a GCS bucket:**

```hcl
resource "google_storage_bucket" "logs" {
  name = "my-gcp-logs"
  project = "my-gcp-project"
}
```

### B. Initialize terraform

```bash
terraform init
```

### C. Import each resource

Use the Terraform import command: `terraform import <address> <id>`

**GCE instance example:**

```bash
terraform import google_compute_instance.web "projects/my-gcp-project/zones/asia-south1-a/instances/prod-vm-1"
```

**GCS bucket example:**

```bash
terraform import google_storage_bucket.logs my-gcp-logs
```

After each import, run:

```bash
terraform state list
terraform plan
```

Repeat for every resource.

---

## 5) If you have many resources (faster ways)

* **Use Terraformer** (tool) — it scans GCP and generates Terraform files + state automatically. It saves a lot of time for many resources.
* Or write a small shell script that lists resources via `gcloud`, builds import commands, and runs them in a loop.

**Simple import loop example (pseudo):**

```bash
# create matching resource blocks first
for name in prod-vm-1 prod-vm-2; do
  terraform import google_compute_instance.${name} "projects/my-gcp-project/zones/asia-south1-a/instances/${name}"
done
```

---

## 6) Fix plan differences

* After import, `terraform plan` may show changes. This is normal.
* Edit your `.tf` files to match the real resource attributes (labels, machine type, disk sizes, metadata) until `terraform plan` shows no unintended changes.

---

## 7) If resources were deleted in GCP

* Check for backups and snapshots:

  * Disk snapshots (Compute Engine)
  * Cloud SQL backups
  * GCS object versions
* Restore from backups if possible, then import restored resources as above.
* If no backups exist, you may have to recreate resources and accept data loss.

---

## 8) After recovery — do these immediately

1. Move your state to a **remote backend** (GCS) so this never happens again.

**Example backend (backend.tf):**

```hcl
terraform {
  backend "gcs" {
    bucket = "my-terraform-state-bucket"
    prefix = "project-name/terraform.tfstate"
  }
}
```

2. Enable **versioning** on the GCS bucket:

```bash
gsutil versioning set on gs://my-terraform-state-bucket
```

3. Use IAM to limit who can delete state files.
4. Consider using **Terraform Cloud** or another backend with locking and history.

---

## 9) Quick commands checklist (copy-paste)

Restore backup if present:

```bash
cp terraform.tfstate.backup terraform.tfstate
terraform init
terraform state list
terraform plan
```

Import a GCE instance:

```bash
terraform import google_compute_instance.web "projects/my-gcp-project/zones/asia-south1-a/instances/prod-vm-1"
```

Import a GCS bucket:

```bash
terraform import google_storage_bucket.logs my-gcp-logs
```

Enable GCS versioning:

```bash
gsutil versioning set on gs://my-terraform-state-bucket
```

---

## 10) Need help? What I can do next

* I can generate the exact `resource` blocks for specific resource types you have (GCE, GKE, SQL, Storage, VPC). Tell me the resource names, zones/regions, and project.
* Or I can write a shell script to import a list of resources you provide.

---

**You are not alone — follow the steps slowly and recover your state.**
