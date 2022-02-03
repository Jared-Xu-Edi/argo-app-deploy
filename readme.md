# A Demo of ArgoCD Deployment

## Prerequisite
Make sure you have your k8s cluster ready and kubectl installed locally.

## App Being Deployed in This Demo
This is a microservice app I wrote a while back but in this demo, I only deploy the frontend to simply things.
The repo is here: https://github.com/Jared-Xu-Edi/baas-frontend
Assume there are already a CI pipeline which do all the testing, security scan, and finally published the docker image here: https://hub.docker.com/repository/docker/jaredxu/baas-frontend/general
There're 2 versions:
* v1: the nav-bar is dark color
* v2: the nav-bar is light color
You can run and check the effect as following:
```
docker run -p 7070:8080 jaredxu/baas-frontend:v1
```
you should see dark theme web app, or
```
docker run -p 7070:8080 jaredxu/baas-frontend:v2
```
you should see light theme.

## Step1: Prepare ArgoCD Environment
You should be able to install with the following commands:
* Installation
```
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```
However, if you are curious about the content of install.yaml, you can find it in argoCD/install.yaml
* Expose the Service
You should run the following to expose the service:
```
kubectl port-forward svc/argocd-server -n argocd 8080:443
```

## Step2: App Deploy Folder/Branch Structure
You can see 3 folders relevant to app deployment: `dev`, `staging` and `prod`, in which you should define your deployment.yml and service.yml files.

## Step3: Prepare ArgoCD deploy manifest
In application.yaml, you can see how the source from a github repo is defined and how this is mapped to a namespace in k8s as a destination.

## Step4: Deploy the App from the argoCD manifest
This is a one-off execution, as from now on, the live deployment will be changed only from the github repo, run command:
```
kubectl apply -f .\application.yaml
```

## Step5: Access The App
As I didn't make nodePort service work in Kind, so in this case, we need to expose the pod with:
(You need to find out the pod name with `kubectl get pods -n baas`)
```
kubectl port-forward baas-frontend-7689997c8c-k9jf4 7777:8080 -n baas
```
By default, it's running v1, you should see dark theme web app.

## Step6: Change code in the repo
Go to repo https://github.com/Jared-Xu-Edi/argo-app-deploy.git and change the app version to be v2.
Now you should see argoCD is synchrozing this change and roll out the old pods automatically.

## Step7: Try change deployment in k8s
Run the following and try to change the replica number to be 3 in the deployment.
```
kubectl edit deployment baas-frontend -n baas
```
You should see nothing changes as everything is synced from the code in github repo.