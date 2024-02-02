# Lab 1
In this lab we will install a dumy app using argocd

## Lab
You should have installed the argo cd based on the setup

### verify
```shell
kubectl get all -n argocd

```

### login and connect
```shell
# user name is admin
# password you can get it from setup
argocd login localhost:9090
argocd account update-password

export ARGOCD_OPTS='--port-forward-namespace argocd'
export CLUSTER_IP=https://kubernetes.default.svc

https://localhost:9090
```

### create project
```shell
argocd app create roar-deploy-k8s --repo https://github.com/brentlaster/roar-deploy-k8s --path . --dest-server $CLUSTER_IP --dest-namespace roar

# you should see the project in the browser
```

### sync project
```shell
export EDITOR=nano
argocd app edit roar-deploy-k8s
syncPolicy:
  syncOptions:
  - CreateNamespace=true
  
argocd app sync roar-deploy-k8s  
```

### test deployed app
```shell
kubectl get svc -n roar roar-web

# Look for the port number that is between “8089:” and “/TCP”. Plug that into the URL below.
export PORT={}
kubectl port-forward -n roar svc/roar-web 8080:$PORT 2>&1 >/dev/null &

```
