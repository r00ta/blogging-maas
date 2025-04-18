+++
title = "Vault"
type = "chapter"
weight = 4
+++

## How to integrate Vault with MAAS

Use this magic script!

```bash
#!/bin/bash

set -e

# CONFIGURATION
VAULT_ADDR="http://127.0.0.1:8200"
POLICY_NAME="maas"
SECRET_PATH="maas"
TOKEN_TTL="5m"

echo "🚀 Installing Vault..."
sudo snap install vault
sudo snap start vault.vaultd

echo "⏳ Waiting for Vault to be ready..."
until curl -s "$VAULT_ADDR/v1/sys/health" | grep -q '"initialized":false'; do
  sleep 2
done
echo "✅ Vault is ready to initialize."

echo "🔐 Setting up environment variables..."
export VAULT_SKIP_VERIFY=true
export VAULT_ADDR=$VAULT_ADDR

echo "🔑 Initializing Vault..."
INIT_OUTPUT=$(vault operator init -key-shares=1 -key-threshold=1 -format=json)
UNSEAL_KEY=$(echo "$INIT_OUTPUT" | jq -r '.unseal_keys_b64[0]')
ROOT_TOKEN=$(echo "$INIT_OUTPUT" | jq -r '.root_token')

echo "📝 Saving unseal key and root token..."
echo "Unseal Key: $UNSEAL_KEY"
echo "Root Token: $ROOT_TOKEN"

echo "🔓 Unsealing Vault..."
vault operator unseal "$UNSEAL_KEY"

echo "🔐 Logging into Vault..."
vault login "$ROOT_TOKEN"

echo "🔐 Enabling AppRole and secret engine..."
vault auth enable approle
vault secrets enable -path=$SECRET_PATH kv-v2

echo "📜 Creating policy..."
cat <<EOF > ${POLICY_NAME}-policy.hcl
path "${SECRET_PATH}/metadata/secrets/" {
  capabilities = ["list"]
}

path "${SECRET_PATH}/metadata/secrets/*" {
  capabilities = ["read", "update", "delete", "list"]
}

path "${SECRET_PATH}/data/secrets/*" {
  capabilities = ["read", "create", "update", "delete"]
}
EOF

vault policy write $POLICY_NAME ${POLICY_NAME}-policy.hcl

echo "🔧 Creating AppRole..."
vault write auth/approle/role/$POLICY_NAME policies=$POLICY_NAME token_ttl=$TOKEN_TTL

ROLE_ID=$(vault read -format=json auth/approle/role/$POLICY_NAME/role-id | jq -r '.data.role_id')
WRAPPED_SECRET=$(vault write -format=json -wrap-ttl=$TOKEN_TTL auth/approle/role/$POLICY_NAME/secret-id role_id="$ROLE_ID")
SECRET_ID=$(echo "$WRAPPED_SECRET" | jq -r '.wrap_info.token')

echo "🤝 Integrating with MAAS..."
echo "Run the following command on your MAAS host:"
echo ""
echo "sudo maas config-vault configure http://<Vault-IP>:8200 $ROLE_ID $SECRET_ID secrets --mount $SECRET_PATH"
```
