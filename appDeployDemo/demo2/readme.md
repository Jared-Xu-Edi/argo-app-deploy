# Demo2
In this demo, you will see the manifest files are organized by kustomize.

You can see that Kustomize is converting them into kubernetes manifest files:
Look at what kustomize builds your base resource like:

```
kustomize build .\appDeployDemo\demo2\base\
```

Then look at how the diff pacth is built based on the base:

```
kustomize build .\appDeployDemo\demo2\overlays\dev
```