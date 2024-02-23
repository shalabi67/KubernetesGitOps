# Helm-Secret plugin
This plugin will decrypt your encrypted value.yaml files.
see [ArgoCD-Integration with helm-secret plugin](https://github.com/jkroepke/helm-secrets/wiki/ArgoCD-Integration)

More information on helm secrets without argocd:
* [helm secerst](https://lyz-code.github.io/blue-book/devops/helm/helm_secrets/)
* [youtube helm secrets](https://www.youtube.com/watch?v=hRSlKRvYe1A)

## install helm secret plugin
```shell
helm plugin install https://github.com/jkroepke/helm-secrets
helm secrets --version

```

## encrypt secret file
```shell
gpg --full-generate-key --rfc4880

```
## setup argocd
```shell
helm repo add argo https://argoproj.github.io/argo-helm
helm install argocd argo/argo-cd -n argocd -f config.yaml 
```


