# Vault with kind
In this example we will install vault on kind cluster and add secrets and display them

## setup
```shell
helm repo add hashicorp https://helm.releases.hashicorp.com
helm repo update
helm search repo hashicorp/vault
helm install vault hashicorp/vault  
#\
 #   --values helm-vault-raft-values.yml \
  #  --dry-run

 #   --namespace vault \
 #   --set "server.ha.enabled=true" \
 #   --set "server.ha.replicas=5" \

```

init vault you can do this from the ui also
```shell
kubectl exec -ti vault-0  -- sh
vault operator init -key-shares=1 -key-threshold=1 >cluster-keys.json
VAULT_UNSEAL_KEY=$(jq -r ".keys_base64[]" cluster-keys.json)
vault operator unseal $VAULT_UNSEAL_KEY
```

## Demo
### configure vault secret
login to vault
```shell
kubectl exec -ti vault-0  -- sh
vault login

```

create secret
```shell
vault secrets enable -path=secret kv-v2
vault kv put secret/webapp/config username="static-user" password="static-password"
```

### Configure Kubernetes authentication
```shell
vault auth enable kubernetes

vault write auth/kubernetes/config \
    kubernetes_host="https://$KUBERNETES_PORT_443_TCP_ADDR:443"
    
vault policy write webapp - <<EOF
path "secret/data/webapp/config" {
  capabilities = ["read"]
}
EOF

vault write auth/kubernetes/role/webapp \
        bound_service_account_names=vault \
        bound_service_account_namespaces=default \
        policies=webapp \
        ttl=24h

```

### Launch a web application
This is a web application that reads secrets from vault
```shell
kubectl apply -f webapp.yml

kubectl port-forward \
    $(kubectl get pod -l app=webapp -o jsonpath="{.items[0].metadata.name}") \
    8080:8080
```

run the app
```shell
curl http://localhost:8080
echo "you should see <<password:static-password username:static-user>>"
```