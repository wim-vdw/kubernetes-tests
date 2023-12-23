# Argo CD tests

Create a new project and two applications in Argo CD:

```shell
kubectl apply -f projects.yaml
kubectl apply -f applications.yaml  
```

Cleanup Argo CD applications and project:

```shell
kubectl delete -f applications.yaml  
kubectl delete -f projects.yaml
```

The application's Kubernetes resources that were under control by Argo CD will be deleted by Argo CD.
