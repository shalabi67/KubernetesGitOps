# Dynamic secret
Information about dynamic secret
* [Dynamic secret use case](https://developer.hashicorp.com/vault/docs/use-cases#dynamic-secrets)
* [Dynamic secret tutorial](https://developer.hashicorp.com/vault/tutorials/db-credentials/database-secrets)
* [AWS dynamic secret tutorial](https://developer.hashicorp.com/vault/tutorials/getting-started/getting-started-dynamic-secrets)

tutorial used in this lab
* [Tutorial used in this example](https://developer.hashicorp.com/vault/tutorials/kubernetes/vault-secrets-operator)
* [Tutorial code](https://github.com/hashicorp-education/learn-vault-secrets-operator/tree/main/dynamic-secrets)

Now you will create a [dynamic secret](https://developer.hashicorp.com/vault/docs/use-cases#dynamic-secrets) with the 
database secrets engine. Dynamic secrets lifecycle is managed by Vault and will be automatically rotated. The lifecycle 
management includes deleting and recreating the secrets regularly. In this section we will use the Vault Secrets 
Operator to rotate the Kubernetes secrets every 1 minute.

## Install Postgres pod
```shell
kubectl create ns postgres
helm repo add bitnami https://charts.bitnami.com/bitnami
helm upgrade --install postgres bitnami/postgresql --namespace postgres --set auth.audit.logConnections=true  --set auth.postgresPassword=secret-pass

```

## Configure vault
```shell
kubectl exec -it vault-0  -- sh
vault secrets enable -path=demo-db database
vault write demo-db/config/demo-db \
   plugin_name=postgresql-database-plugin \
   allowed_roles="dev-postgres" \
   connection_url="postgresql://{{username}}:{{password}}@postgres-postgresql.postgres.svc.cluster.local:5432/postgres?sslmode=disable" \
   username="postgres" \
   password="secret-pass"
   
vault write demo-db/roles/dev-postgres \
   db_name=demo-db \
   creation_statements="CREATE ROLE \"{{name}}\" WITH LOGIN PASSWORD '{{password}}' VALID UNTIL '{{expiration}}'; \
      GRANT ALL PRIVILEGES ON DATABASE postgres TO \"{{name}}\";" \
   backend=demo-db \
   name=dev-postgres \
   default_ttl="1m" \
   max_ttl="1m"

vault policy write demo-auth-policy-db - <<EOF
path "demo-db/creds/dev-postgres" {
   capabilities = ["read"]
}
EOF

vault secrets enable -path=demo-transit transit
# Create a encryption key.
vault write -force demo-transit/keys/vso-client-cache
# Create a policy for the operator role to access the encryption key.
vault policy write demo-auth-policy-operator - <<EOF
path "demo-transit/encrypt/vso-client-cache" {
   capabilities = ["create", "update"]
}
path "demo-transit/decrypt/vso-client-cache" {
   capabilities = ["create", "update"]
}
EOF

vault write auth/demo-auth-mount/role/auth-role-operator \
   bound_service_account_names=demo-operator \
   bound_service_account_namespaces=vault-secrets-operator-system \
   token_ttl=0 \
   token_period=120 \
   token_policies=demo-auth-policy-db \
   audience=vault

```

### Setup dynamic secrets
```shell
vault write auth/demo-auth-mount/role/auth-role \
   bound_service_account_names=default \
   bound_service_account_namespaces=demo-ns \
   token_ttl=0 \
   token_period=120 \
   token_policies=demo-auth-policy-db \
   audience=vault

```

## Demo
```shell
kubectl create ns demo-ns
kubectl apply -f dynamic-secrets/.

# see secret vso-db-demo in demo-ns namesapce
```

