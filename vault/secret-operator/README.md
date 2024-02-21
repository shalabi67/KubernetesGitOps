# Vault with kind
In this example we will install vault, secret operator on kind cluster and add secrets and display them

for more information see [secret operator](https://developer.hashicorp.com/vault/tutorials/kubernetes/vault-secrets-operator)

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

## Install the Vault Secrets Operator
```shell
helm install vault-secrets-operator hashicorp/vault-secrets-operator -n vault-secrets-operator-system --create-namespace --values vault-operator-values.yaml

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
vault secrets enable -path=kvv2 kv-v2
vault kv put kvv2/webapp/config username="static-user" password="static-password"
```

### Configure Kubernetes authentication
```shell
vault auth enable -path demo-auth-mount kubernetes

vault write auth/demo-auth-mount/config \
   kubernetes_host="https://$KUBERNETES_PORT_443_TCP_ADDR:443"

    
vault policy write dev - <<EOF
path "kvv2/*" {
   capabilities = ["read"]
}
EOF


vault write auth/demo-auth-mount/role/role1 \
   bound_service_account_names=default \
   bound_service_account_namespaces=app \
   policies=dev \
   audience=vault \
   ttl=24h


```

### Launch a web application
This is a web application that reads secrets from vault
```shell
kubectl create ns app
kubectl apply -f vault-auth-static.yaml
kubectl apply -f static-secret.yaml

```

run the app
```shell
curl http://localhost:8080
echo "you should see <<password:static-password username:static-user>>"
```