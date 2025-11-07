# Vault JWT (GitHub OIDC) Setup — Quick Commands

> Run with a **Vault admin** token. This config allows GitHub Actions to authenticate to Vault via OIDC/JWT,
> map to a Vault role, and then read either KV or dynamic GCP secrets per your policies.

## 1) Enable JWT auth method
```bash
vault auth enable jwt || true
```

## 2) Configure JWT (GitHub OIDC) with issuer and JWKS
```bash
vault write auth/jwt/config \
  oidc_discovery_url="https://token.actions.githubusercontent.com" \
  bound_issuer="https://token.actions.githubusercontent.com"
```

## 3) Create a Vault role for your repo
Replace `owner/repo` with your GitHub org/repo; adjust claims for tighter scoping (ref, environment, workflow, etc.).

```bash
vault write auth/jwt/role/gh-terraform \
  role_type="jwt" \
  user_claim="actor" \
  bound_audiences="vault" \
  bound_claims='{
    "repository": "owner/repo",
    "repository_owner": "owner"
  }' \
  token_policies="terraform-kv-read,terraform-gcp-access" \
  token_ttl="1h"
```

**Notes**
- `bound_audiences` must match the audience you request in the workflow (default `vault`).
- Add additional `bound_claims` such as `"ref":"refs/heads/main"` to restrict to a branch.
- Ensure the policies referenced (`terraform-kv-read`, `terraform-gcp-access`) already exist.

## 4) GitHub Actions repo configuration
Set these in your repo:

- **Secrets**:  
  - `VAULT_ADDR` → e.g., `https://vault.example.com`

- **Repository Variables** (optional):  
  - `GCP_PROJECT_ID` → your project ID  
  - `GCP_REGION` → e.g., `us-central1`  
  - `VAULT_JWT_ROLE` → `gh-terraform`  
  - `VAULT_JWT_AUDIENCE` → `vault`

## 5) Terraform providers
Your Terraform should include the Vault provider (uses `VAULT_ADDR` and `VAULT_TOKEN` from the workflow after JWT login) and your chosen Google provider config (access_token or credentials from Vault as shown in the main guide).
