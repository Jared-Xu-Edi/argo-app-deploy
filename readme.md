# A Demo of ArgoCD Deployment

## Prerequisite

Make sure you have at least 2 k8s cluster2 ready and kubectl installed locally.
You can check in multicluster folder to see how to create them.
Confirm with command:

```
kubectl config get-contexts
```

and switch to the cluster where you want to install argoCD and we call this manager cluster:

```
kubectl config use-context your_cluster_name
```

Should look like:

+-----------------+     +---------------+
| manager cluster +     +   cluster 2   +
|                 |     |               |
+-----------------+     +---------------+

## App Being Deployed in This Demo

This is a microservice app I wrote a while back but in this demo, I only deploy the frontend to simply things.
The repo is here: https://github.com/Jared-Xu-Edi/baas-frontend
Assume there are already a CI pipeline which do all the testing, security scan, and finally published the docker image here: https://hub.docker.com/repository/docker/jaredxu/baas-frontend/general
There're 2 versions:

- v1: the nav-bar is dark color
- v2: the nav-bar is light color
  You can run and check the effect as following:

```
docker run -p 7070:8080 jaredxu/baas-frontend:v1
```

you should see dark theme web app, or

```
docker run -p 7071:8080 jaredxu/baas-frontend:v2
```

you should see light theme.

## Step1: Prepare ArgoCD Environment

You should be able to install with the following commands:

- Installation

```
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

Your setup should now look like:
```
+-----------------+     +---------------+
| manager cluster |     |   cluster 2   |
|   argoCD        |     |               |
+-----------------+     +---------------+
```
However, if you are curious about the content of install.yaml, you can find it in argoCD/install.yaml

- Expose the Service
  You should run the following to expose the service:

```
kubectl port-forward svc/argocd-server -n argocd 8080:443
```

and you can access the webui now from https://localhost:8080


## Step2: App Deploy Folder/Branch Structure

You can see 3 folders in `appDeployDemo/demo1` relevant to app deployment: `dev`, `staging` and `prod`, in which you should define your deployment.yml and service.yml files.

## Step3: Prepare ArgoCD deploy manifest

In application.yaml, you can see how the source from a github repo is defined and how this is mapped to a namespace in k8s as a destination.

## Step4: Deploy the App from the argoCD manifest (Demo1: Single Cluster)

This is a one-off execution, as from now on, the live deployment will be changed only from the github repo, run command:

```
kubectl apply -f .\application.yaml
```
Now the setup should look like:

+-----------------+     +---------------+
| manager cluster |     |   cluster 2   |
|   argoCD        |     |               |
|   baas-app      |     |               |
+-----------------+     +---------------+

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

Run the following and try to change the replica number to be 3 in the deployment or change from webui.

```
kubectl edit deployment baas-frontend -n baas
```

You should see nothing changes as everything is synced from the code in github repo.


## Step8: Introduce Defects and Rollback

Change the app version from v1 to v3 in appDeployDemo/demo1/dev/deployment and commit the change.
v3 does not really exist in docker hub so this will cause the error.

Now, disable the auto-sync on ArgoCD WebUI and rollback to previous version.

Change the version back to v1 and commit the change. Re-enable auto-sync, it should be good in ArgoCD and synced to the latest good commit/version.


## Step9: Delete deployment in k8s

Run the following to delete the deployment:

```
kubectl delete -f .\application.yaml
```

## Step10: Deploy app to 2 clusters using the demo1 folder structure(Demo2) with ArgoCD applicationSet.

The ApplicationSet controller is installed alongside Argo CD (within the same namespace), and it automatically generates Argo CD Applications based on the contents of a new ApplicationSet Custom Resource (CR)

Make sure resource in namespace `baas` is already purged and run:

```
kubectl apply -f .\applicationSetDemo1.yaml
```

and you should see 2 deployment card displayed on argoCD main page.

Now the setup should look like:

+----------------------+     +--------------------+
| manager cluster(dev) |     |   cluster(prod) 2  |
|   argoCD             |     |                    |
|   baas-appv1         |     |   baas-appv2       |
+----------------------+     +--------------------+

Take a look at the applicationSetDemo1.yaml and pay attention to the template.

ArgoCD is managing deployment over multiple clusters this way.

However, in this demo, there are many boilerplate code in the manifest.

Let's delete the resource with command for now:
```
kubectl delete -f .\applicationSetDemo1.yaml
```

## Step11: Improve with Kustomize(Demo2)

Now look at how kustomize builds your base resource:

```
kustomize build .\appDeployDemo\demo2\base\
```

Then look at how the diff pacth is built based on the base:

```
kustomize build .\appDeployDemo\demo2\overlays\dev
```

and

```
kustomize build .\appDeployDemo\demo2\overlays\prod
```

You can create the resource using kubectl e.g.

```
    kustomize build .\appDeployDemo\demo2\overlays\dev | kubectl apply -f
```

But here we are creating the deployment on 2 cluster by running the argoCD applicationSet.:

```
kubectl apply -f .\applicationSetDemo2.yaml
```
