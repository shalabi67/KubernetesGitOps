# Vault setup
* [Install](https://developer.hashicorp.com/vault/install)
* [vault setup using helm](https://developer.hashicorp.com/vault/docs/platform/k8s/helm)

## Ubuntu
```shell
wget -O- https://apt.releases.hashicorp.com/gpg | sudo gpg --dearmor -o /usr/share/keyrings/hashicorp-archive-keyring.gpg
echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/hashicorp.list
sudo apt update && sudo apt install vault
sudo systemctl start vault
```

## Helm setup
Add hashicorp repo
```shell
helm repo add hashicorp https://helm.releases.hashicorp.com
helm search repo hashicorp/vault

```

Install latest release
```shell
kubectl create namespace vault
helm install --namespace vault vault hashicorp/vault
```

init vault
```shell
kubectl exec -ti vault-0 -n vault -- sh
vault operator init
vault operator unseal
```

### To install a specific version
```shell
# List the available releases
$ helm search repo hashicorp/vault -l

# Install version 0.27.0
$ helm install vault hashicorp/vault --version 0.27.0

```