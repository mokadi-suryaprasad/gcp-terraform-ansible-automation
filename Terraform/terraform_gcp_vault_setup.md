
Real-time, **step-by-step** setup to integrate **Terraform ⇄ HashiCorp Vault ⇄ GCP**.  
Copy this file into your repo and follow it top to bottom.

> Covers: local test with AppRole, production-ready with GitHub OIDC (JWT), KV & Dynamic secrets, verification at every step.

---

## 0) Checklist (what you need)
- **OS**: Linux/macOS terminal (Windows WSL ok)
- **CLI tools**: `vault` `terraform` `gcloud` `jq` `curl`
- **Vault**: reachable URL (dev or prod); you have an **admin token** for initial setup
- **GCP**: a project you control, billing enabled
- **Service Accounts**:
  - `vault-gcp-admin` (lets Vault mint tokens/keys) → download JSON `vault-gcp-admin.json`
  - *(optional for KV)* `terraform-runner` → download JSON `sa-key.json`

> Tip: Put all temp files in a new working dir:
```bash
mkdir -p ~/tf-vault-gcp && cd ~/tf-vault-gcp
```

---

## 1) GCP Prep (one-time)
Authenticate and set your project.
```bash
gcloud auth login
gcloud config set project <YOUR_PROJECT_ID>
export GCP_PROJECT_ID=$(gcloud config get-value project)
```

### 1.1 Create the **Vault admin** SA and grant rights (if you don’t have it yet)
This SA allows Vault to issue access tokens or ephemeral keys.

```bash
gcloud iam service-accounts create vault-gcp-admin \
  --display-name="Vault GCP Admin"

# Grant roles needed to mint tokens/keys and manage bindings for generated SAs
gcloud projects add-iam-policy-binding "$GCP_PROJECT_ID" \
  --member="serviceAccount:vault-gcp-admin@${GCP_PROJECT_ID}.iam.gserviceaccount.com" \
  --role="roles/iam.serviceAccountTokenCreator"

gcloud projects add-iam-policy-binding "$GCP_PROJECT_ID" \
  --member="serviceAccount:vault-gcp-admin@${GCP_PROJECT_ID}.iam.gserviceaccount.com" \
  --role="roles/iam.serviceAccountAdmin"

# (Optional) For storage demo, give admin so generated identities can operate buckets
gcloud projects add-iam-policy-binding "$GCP_PROJECT_ID" \
  --member="serviceAccount:vault-gcp-admin@${GCP_PROJECT_ID}.iam.gserviceaccount.com" \
  --role="roles/storage.admin"

# Create and download key for Vault to use
gcloud iam service-accounts keys create vault-gcp-admin.json \
  --iam-account="vault-gcp-admin@${GCP_PROJECT_ID}.iam.gserviceaccount.com"
```

### 1.2 *(Optional)* Create **terraform-runner** SA (used only for KV static secret flow)
```bash
gcloud iam service-accounts create terraform-runner \
  --display-name="Terraform Runner"

# Grant least privilege (for demo we use storage.admin — tighten in prod)
gcloud projects add-iam-policy-binding "$GCP_PROJECT_ID" \
  --member="serviceAccount:terraform-runner@${GCP_PROJECT_ID}.iam.gserviceaccount.com" \
  --role="roles/storage.admin"

gcloud iam service-accounts keys create sa-key.json \
  --iam-account="terraform-runner@${GCP_PROJECT_ID}.iam.gserviceaccount.com"
```

---

## 2) Vault Engines (admin token required only here)
Point your shell to Vault:
```bash
export VAULT_ADDR="https://vault.example.com"   # or http://127.0.0.1:8200
export VAULT_TOKEN="hvs.xxxxx"                  # admin token for setup only
vault status
```

Enable engines:
```bash
vault secrets enable -path=secret kv-v2 || true
vault secrets enable gcp || true
```

Configure GCP engine with the admin SA JSON:
```bash
vault write gcp/config credentials=@vault-gcp-admin.json
```

Create **rolesets** for dynamic creds:
```bash
# Short-lived OAuth access tokens
vault write gcp/roleset/tf-access-token \
  project="$GCP_PROJECT_ID" \
  secret_type="access_token" \
  token_scopes='["https://www.googleapis.com/auth/cloud-platform"]'

# Ephemeral Service Account keys (prebind roles to generated identities)
vault write gcp/roleset/tf-ephemeral-key \
  project="$GCP_PROJECT_ID" \
  secret_type="service_account_key" \
  bindings=@- <<'JSON'
{
  "roles/storage.admin": [
    "serviceAccount:__service_account_email__"
  ]
}
JSON
```

**Verify**:
```bash
vault read gcp/roleset/tf-access-token | jq
vault read gcp/roleset/tf-ephemeral-key | jq
```

---

## 3) KV Static Secret (optional)
Store the pre-created terraform-runner key as a KV secret.
```bash
vault kv put secret/gcp/sa-key creds=@sa-key.json
vault kv get -format=json secret/gcp/sa-key | jq .data.data
```

---

## 4) Policies (least privilege)
Create two files:

**policy-kv-read.hcl**
```hcl
path "secret/data/gcp/sa-key" {
  capabilities = ["read"]
}
```

**policy-gcp-access.hcl**
```hcl
path "gcp/token/tf-access-token" {
  capabilities = ["read"]
}

path "gcp/key/tf-ephemeral-key" {
  capabilities = ["read"]
}
```

Apply:
```bash
vault policy write terraform-kv-read policy-kv-read.hcl
vault policy write terraform-gcp-access policy-gcp-access.hcl
vault policy list
```

---

## 5) Auth for Automation — Choose ONE
### 5A) **AppRole** (simple for local/CI)
Enable + create role:
```bash
vault auth enable approle || true

vault write auth/approle/role/terraform \
  token_policies="terraform-kv-read,terraform-gcp-access" \
  token_ttl="1h" token_max_ttl="4h" \
  secret_id_ttl="1h" secret_id_num_uses="1"
```

Fetch credentials:
```bash
ROLE_ID=$(vault read -field=role_id auth/approle/role/terraform/role-id)
SECRET_ID=$(vault write -field=secret_id -f auth/approle/role/terraform/secret-id)
echo "ROLE_ID=$ROLE_ID"
echo "SECRET_ID=$SECRET_ID"
```

Test login:
```bash
LOGIN=$(curl -sSf -H "Content-Type: application/json" \
  -X POST -d "{\"role_id\":\"$ROLE_ID\",\"secret_id\":\"$SECRET_ID\"}" \
  "$VAULT_ADDR/v1/auth/approle/login")
echo "$LOGIN" | jq .auth.client_token
```

### 5B) **GitHub OIDC (JWT)** (recommended for GitHub Actions)
Configure JWT auth (admin token):
```bash
vault auth enable jwt || true

vault write auth/jwt/config \
  oidc_discovery_url="https://token.actions.githubusercontent.com" \
  bound_issuer="https://token.actions.githubusercontent.com"

vault write auth/jwt/role/gh-terraform \
  role_type="jwt" \
  user_claim="actor" \
  bound_audiences="vault" \
  bound_claims="{\"repository\":\"owner/repo\",\"repository_owner\":\"owner\"}" \
  token_policies="terraform-kv-read,terraform-gcp-access" \
  token_ttl="1h"
```

In GitHub → Repo **Settings**:
- **Secret**: `VAULT_ADDR=https://vault.example.com`
- **Variables**: `GCP_PROJECT_ID`, `GCP_REGION=us-central1`, `VAULT_JWT_ROLE=gh-terraform`, `VAULT_JWT_AUDIENCE=vault`

Drop this workflow as `.github/workflows/terraform_vault_gcp.yml`:
```yaml
name: Terraform via Vault (GCP)
on:
  workflow_dispatch:
    inputs:
      action:
        type: choice
        options: [plan, apply]
        default: plan
  push:
    branches: [ "main" ]
    paths: [ "terraform/**", ".github/workflows/terraform_vault_gcp.yml" ]
permissions:
  id-token: write
  contents: read
env:
  TF_IN_AUTOMATION: "true"
  TF_INPUT: "false"
  VAULT_ADDR: ${{ secrets.VAULT_ADDR }}
  VAULT_ROLE: ${{ vars.VAULT_JWT_ROLE }}
  TF_VAR_project_id: ${{ vars.GCP_PROJECT_ID }}
  TF_VAR_region: ${{ vars.GCP_REGION || 'us-central1' }}

jobs:
  terraform:
    runs-on: ubuntu-latest
    defaults:
      run:
        working-directory: terraform
    steps:
      - uses: actions/checkout@v4
      - uses: hashicorp/setup-terraform@v3
        with: { terraform_version: 1.7.5 }
      - name: Install jq curl
        run: sudo apt-get update -y && sudo apt-get install -y jq curl
      - name: GitHub OIDC → Vault login
        id: vault
        run: |
          AUD=${{ vars.VAULT_JWT_AUDIENCE || 'vault' }}
          TOK_JSON=$(curl -sSf -H "Authorization: Bearer $ACTIONS_ID_TOKEN_REQUEST_TOKEN" "${ACTIONS_ID_TOKEN_REQUEST_URL}&audience=$AUD")
          OIDC=$(echo "$TOK_JSON" | jq -r .value)
          LOGIN=$(curl -sSf -H "Content-Type: application/json" -X POST \
            -d "{\"role\":\"${VAULT_ROLE}\",\"jwt\":\"$OIDC\"}" \
            "$VAULT_ADDR/v1/auth/jwt/login")
          echo "$LOGIN" | jq . >/tmp/vlogin.json
          echo "VAULT_TOKEN=$(jq -r .auth.client_token /tmp/vlogin.json)" >> $GITHUB_ENV
      - run: terraform init -input=false
      - run: terraform validate
      - if: inputs.action == 'plan' || github.event_name == 'push'
        run: terraform plan -no-color
      - if: inputs.action == 'apply'
        run: terraform apply -auto-approve
```

---

## 6) Terraform Code (choose ONE auth path to test)
Create `terraform/`:

```
terraform/
├─ versions.tf
├─ providers.tf
├─ variables.tf
├─ main_kv_static.tf
├─ main_dynamic_access_token.tf
├─ main_dynamic_key.tf
├─ outputs.tf
└─ terraform.tfvars  (gitignore this)
```

**versions.tf**
```hcl
terraform {
  required_version = ">= 1.5"
  required_providers {
    vault = { source = "hashicorp/vault",  version = ">= 4.3.0" }
    google = { source = "hashicorp/google", version = ">= 5.32.0" }
  }
}
```

**providers.tf**
```hcl
provider "vault" {}
# Uses VAULT_ADDR + one of: VAULT_TOKEN (JWT login) or VAULT_APPROLE_* (AppRole login)
```

**variables.tf**
```hcl
variable "project_id" { type = string }
variable "region"     { type = string default = "us-central1" }
```

**Option A – KV static** (`main_kv_static.tf`)
```hcl
data "vault_kv_secret_v2" "gcp_sa" {
  mount = "secret"
  name  = "gcp/sa-key"
}

provider "google" {
  project     = var.project_id
  region      = var.region
  credentials = data.vault_kv_secret_v2.gcp_sa.data["creds"]
}

resource "google_storage_bucket" "kv_demo" {
  name          = "tf-kv-demo-${var.project_id}"
  location      = "US"
  force_destroy = true
}
```

**Option B1 – Dynamic OAuth token** (`main_dynamic_access_token.tf`)
```hcl
data "vault_gcp_access_token" "tf" {
  role = "tf-access-token"
}

provider "google" {
  project      = var.project_id
  region       = var.region
  access_token = data.vault_gcp_access_token.tf.token
}

resource "google_storage_bucket" "dyn_token_demo" {
  name          = "tf-dyntoken-demo-${var.project_id}"
  location      = "US"
  force_destroy = true
}
```

**Option B2 – Dynamic ephemeral key** (`main_dynamic_key.tf`)
```hcl
data "vault_gcp_service_account_key" "tf" {
  role = "tf-ephemeral-key"
}

locals {
  sa_key_json = base64decode(data.vault_gcp_service_account_key.tf.private_key_data)
}

provider "google" {
  project     = var.project_id
  region      = var.region
  credentials = local.sa_key_json
}

resource "google_storage_bucket" "dyn_key_demo" {
  name          = "tf-dynkey-demo-${var.project_id}"
  location      = "US"
  force_destroy = true
}
```

**outputs.tf**
```hcl
output "bucket_names" {
  value = compact([
    try(google_storage_bucket.kv_demo.name, null),
    try(google_storage_bucket.dyn_token_demo.name, null),
    try(google_storage_bucket.dyn_key_demo.name, null)
  ])
}
```

**terraform.tfvars**
```hcl
project_id = "<YOUR_PROJECT_ID>"
region     = "us-central1"
```

Add `.gitignore` entries:
```
.terraform/
*.tfstate
*.tfstate.backup
*.tfvars
**/*key.json
vault-gcp-admin.json
sa-key.json
```

---

## 7) Run Locally (AppRole) — end-to-end
Open a **new shell** (no admin token!).
```bash
export VAULT_ADDR="https://vault.example.com"

# If using AppRole:
export VAULT_APPROLE_ROLE_ID="<ROLE_ID>"
export VAULT_APPROLE_SECRET_ID="<SECRET_ID>"

# If using JWT (e.g., local OIDC not typical) set VAULT_TOKEN instead.
```

Choose ONE of the main files (comment out the other two). Then:
```bash
cd terraform
terraform init
terraform plan -var="project_id=${GCP_PROJECT_ID}"
terraform apply -auto-approve -var="project_id=${GCP_PROJECT_ID}"
```

**Verify bucket**:
```bash
gsutil ls | grep tf-
```

---

## 8) Rotate & Revoke
- **Dynamic**: auto-expire via leases; revoke immediately if needed:
```bash
vault lease revoke <LEASE_ID>
```
- **KV static**: rotate by pushing a new JSON:
```bash
vault kv put secret/gcp/sa-key creds=@new-sa-key.json
```

---

## 9) Troubleshooting (fast checks)
- Vault login failing in CI:
  - Confirm `VAULT_ADDR`, network, and auth method (AppRole vs JWT).
  - For JWT: audience in workflow must equal Vault role `bound_audiences` (default `vault`).  
- 403 from GCP:
  - Check IAM roles in roleset `bindings` (for ephemeral key) or token `scopes`.
- `no such path`:
  - Engine/role paths must match (`gcp/token/<role>`, `gcp/key/<role>`, `secret/data/...`).
- Google provider errors (`no default creds`):
  - Ensure **either** `access_token` **or** `credentials` is configured (not both empty).
- Policy denied:
  - `vault read sys/policy` and `vault read auth/token/lookup-self` to inspect effective policies.

---

## 10) Security Notes
- Prefer **dynamic** secrets in CI/CD.
- Scope Vault policies to exact paths; avoid `list`/`update` unless needed.
- Avoid committing any JSON keys; rely on Vault data sources at runtime.
- Use remote state with encryption (GCS + CMEK, Terraform Cloud, etc.).

---


