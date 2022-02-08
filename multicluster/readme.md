# Create a KIND cluster and Add into ArgoCD
This is an instruction about how to create a(multiple) cluster and add that into your argoCD cluster list.

## Create Cluster
* find out your local ip address and modify dev-cluster.yml accordingly
* run command and create it.
```
kind create cluster --name dev-cluster --config .\multicluster\dev-cluster.yml
```

## Add Cluster into ArgoCD
* find all available cluster context list by running `argocd cluster add`:
```
C:\Users\xuj1>argocd cluster add
time="2022-02-08T14:53:07Z" level=error msg="Choose a context name from:"
CURRENT  NAME                       CLUSTER                    SERVER
         AAD-WXSDIAKS               AAD-WXSDIAKS               https://aad-wxsdiaks-dns-d761d03e.hcp.northeurope.azmk8s.io:443
         i17-int-ctop-eus-aks       i17-int-ctop-eus-aks       https://i17-int-ctop-eus-f0f0f558.hcp.eastus.azmk8s.io:443
         i17-int-dev-jared-aks      i17-int-dev-jared-aks      https://i17-int-dev-jared-03ab77c8.hcp.northeurope.azmk8s.io:443
*        kind-dev-cluster           kind-dev-cluster           https://192.168.1.139:8443
         kind-kind-jared            kind-kind-jared            https://127.0.0.1:53034
```

* add cluster into your argoCD cluster list:

```
argocd cluster add kind-dev-cluster
```
* Check your ArgoCD cluster by running `argocd cluster list`


```
C:\Users\xuj1>argocd cluster list
SERVER                          NAME              VERSION  STATUS      MESSAGE  PROJECT
https://192.168.1.139:8443      kind-dev-cluster  1.21     Successful
https://kubernetes.default.svc  in-cluster        1.21     Successful
```
