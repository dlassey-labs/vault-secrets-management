# DL Labs - HashiCorp Vault Secrets Management (KV v2)

## Scope

This document covers only Vault Secrets Management (KV v2).
PKI, Root CA, Intermediate CA, and TLS certificate issuance are documented separately.

## Environment

- Vault URL: https://vault.dl.lab:8200
- TLS Enabled
- Internal DNS: vault.dl.lab
- KV Engine: secret/

## Installation

```bash
sudo apt update
sudo apt install -y gpg wget

wget -O- https://apt.releases.hashicorp.com/gpg \
| sudo gpg --dearmor \
-o /usr/share/keyrings/hashicorp-archive-keyring.gpg

echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] \
https://apt.releases.hashicorp.com $(lsb_release -cs) main" \
| sudo tee /etc/apt/sources.list.d/hashicorp.list

sudo apt update
sudo apt install vault -y

vault version
```

## TLS Configuration

Vault was configured to use TLS.

File: `/etc/vault.d/vault.hcl`

```hcl
ui = true

storage "file" {
  path = "/opt/vault/data"
}

listener "tcp" {
  address = "0.0.0.0:8200"
  tls_disable = 0

  tls_cert_file = "/opt/vault/tls/vault.crt"
  tls_key_file  = "/opt/vault/tls/vault.key"
}

api_addr     = "https://vault.dl.lab:8200"
cluster_addr = "https://vault.dl.lab:8201"
```

## Start Vault

```bash
sudo systemctl enable vault
sudo systemctl start vault
sudo systemctl status vault
```

## Initialize Vault

```bash
export VAULT_ADDR="https://vault.dl.lab:8200"

vault operator init
```

Store:
- Unseal Keys
- Root Token

Securely.

## Unseal Vault

```bash
vault operator unseal
vault operator unseal
vault operator unseal
```

Verify:

```bash
vault status
```

Expected:

```text
Sealed false
```

## Login

```bash
vault login
```

## Enable KV v2

Verify existing engines:

```bash
vault secrets list
```

Enable KV v2:

```bash
vault secrets enable -path=secret kv-v2
```

Verify:

```bash
vault secrets list
```

Expected:

```text
secret/ kv
```

## Recommended Structure

```text
secret/

├── infra/
│   ├── ansible
│   ├── terraform
│   ├── proxmox
│   ├── opnsense
│   ├── pbs
│   └── truenas
│
├── api/
│   ├── openai
│   ├── openrouter
│   ├── tavily
│   ├── gitlab
│   └── n8n
│
├── ssh/
│   ├── proxmox
│   └── putty-mobaxterm
│
└── apps/
    ├── keycloak
    ├── rancher
    ├── vault
    └── odoo
```

## Example Secret - Ansible

Path:

```text
secret/infra/ansible
```

Store:

```bash
vault kv put secret/infra/ansible \
proxmox_token_id="ansible@pve!ansible" \
proxmox_token_secret="xxxxxxxxxxxx"
```

Read:

```bash
vault kv get secret/infra/ansible
```

Read single field:

```bash
vault kv get -field=proxmox_token_secret secret/infra/ansible
```

## Secret Versioning

Updating a secret automatically creates a new version.

```bash
vault kv put secret/infra/ansible \
proxmox_token_id="ansible@pve!ansible" \
proxmox_token_secret="NEW_SECRET"
```

## Delete Secret

```bash
vault kv delete secret/infra/ansible
```

## Policies

Example policy:

```hcl
path "secret/data/infra/ansible" {
  capabilities = ["read"]
}
```

Load policy:

```bash
vault policy write ansible ansible-policy.hcl
```

## Dedicated Token

Create token:

```bash
vault token create -policy=ansible
```

Use:

```bash
export VAULT_TOKEN="TOKEN"
```


