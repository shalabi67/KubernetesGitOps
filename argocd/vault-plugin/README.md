# Vault plugin
The resources created by argocd will fetch values from vault.

## configure vault
```shell
# create secret
vault secrets enable -path=dbaas kv-v2
vault kv put dbaas/app/config username="static-user" password="static-password"
vault policy write dbaas-app - <<EOF
path "dbaas/data/app/config" {
  capabilities = ["read"]
}
EOF

# create token you will use this token in vault-secret.yaml file
vault token create
echo -n  generated token | base64
```

## install argocd and vault plugin
```shell
kubectl apply -f plugin/argocd-cm.yaml
kubectl apply -f plugin/vault-secret.yaml
helm repo add argo https://argoproj.github.io/argo-helm
helm install argocd argo/argo-cd -n argocd -f plugin/config.yaml 
```

