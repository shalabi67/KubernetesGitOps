# commands
vault login -tls-skip-verify hvs.PBp3H03XMKC3bXciCJ4foEQK
vault token create -tls-skip-verify -id=vault-kms-k8s-plugin-token
vault secrets enable -tls-skip-verify transit
vault write -tls-skip-verify -f transit/keys/my-key
vault secrets list -tls-skip-verify
vault kv put -tls-skip-verify cubbyhole/my-secret my-value=yea

vault kv put secret/devwebapp/config username='giraffe' password='salsa'
