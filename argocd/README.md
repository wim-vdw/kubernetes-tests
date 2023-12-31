# Argo CD tests

Create/onboard projects and applications in Argo CD:

```shell
kubectl apply -f seed-projects.yaml
kubectl apply -f seed-simple-app.yaml  
```

Remove projects and applications from Argo CD:

```shell
kubectl delete -f seed-simple-app.yaml  
kubectl delete -f seed-projects.yaml
```

The application's Kubernetes resources that were under control by Argo CD will be deleted by Argo CD.
