+++
title = "Vault"
type = "chapter"
weight = 4
+++

## How to integrate Vault with MAAS

### Install Vault

```
sudo snap install vault
sudo snap start vault.vaultd
```

### Setup Vault

Setup a couple of env variables

```
export VAULT_SKIP_VERIFY=true
export VAULT_ADDR=http://127.0.0.1:8200
```

Initialize Vault

```
vault operator init -key-shares=1 -key-threshold=1
```

and write down the root token and the unseal keys.

Now unseal the vault with the unseal keys.

```
vault operator unseal
```

Now login with the root token

```
vault login
```

### Setup the approle

Enable the approle

```
vault auth enable approle
```

enable the `maas` path

```
vault secrets enable -path maas kv-v2
```

Now write down the policy 

```
cat <<EOF > maas-policy.hcl
path "maas/metadata/secrets/" {
	capabilities = ["list"]
}

path "maas/metadata/secrets/*" {
	capabilities = ["read", "update", "delete", "list"]
}

path "maas/data/secrets/*" {
	capabilities = ["read", "create", "update", "delete"]
}
EOF
```

and create the policy

```
vault policy write maas maas-policy.hcl
vault write auth/approle/role/maas policies=maas token_ttl=5m
```

Extract the role id

```
vault read auth/approle/role/maas/role-id
```

and finally 

```
vault write -wrap-ttl=5m auth/approle/role/maas/secret-id role_id=<the role id>
```

### Integrate with MAAS

```
sudo maas config-vault configure http://<the IP where you installed vault>:8200 <the role id> <the secret token> secrets --mount maas
```
