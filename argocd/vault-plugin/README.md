# Vault plugin
The resources created by argocd will fetch values from vault.

## configure vault
```shell
# create secret
vault secrets enable -path=dbaas kv
vault kv put dbaas/app/config username="static-user" password="static-password"
vault policy write dbaas-app - <<EOF
path "dbaas/app/config" {
  capabilities = ["read"]
}
EOF

# create token
vault token create
echo -n  generated token | base64
```

## install argocd and vault plugin

