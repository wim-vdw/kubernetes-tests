# Kubernetes tests

## Deploy ArgoCD and onboard rootapp (App Of Apps Pattern)

ArgoCD is deployed as an `Application`, this will allow us to perform future patching of ArgoCD via ArgoCD.

The `rootapp` will initialize all ArgoCD projects and applications.

Following controllers are part of the `core-apps` and will start synchronizing when `rootapp` gets applied:

* `Ingress NGINX` controller for Kubernetes
* `MetalLB` load-balancer implementation for bare metal Kubernetes clusters
* `Reloader` controller to watch changes in Kubernetes ConfigMaps and Secrets

The rest of the applications are all tests and are manually synchronized.

```bash
# Deploy ArgoCD including an "Application" to perform future patching of ArgoCD via ArgoCD.
cd ./argocd-app/overlays/prd
kubectl apply -k .

# Onboard rootapp (App Of Apps Pattern).
cd ./argocd
kubectl apply -f seeding-root-app.yaml
```

## Following fix was needed to ClusterRole argocd-server

[Application in any namespace | Synced with NO resources deployed ](https://github.com/argoproj/argo-cd/issues/11638)  
[Argo CD Security](https://argo-cd.readthedocs.io/en/stable/operator-manual/security/)

This has been implemented in the ArgoCD app
manifest: [argocd-app/overlays/prd/kustomization.yaml](argocd-app/overlays/prd/kustomization.yaml)

Manual fix if needed:

```bash
kubectl edit ClusterRole argocd-server -n argocd
- apiGroups:
  - argoproj.io
  resources:
  - applications
  - applicationsets
  - appprojects
  verbs:
  - create
  - get
  - list
  - watch
  - update
  - delete
  - patch
```

## test05

```bash
APISERVER=https://kubernetes.default.svc
SERVICEACCOUNT=/var/run/secrets/kubernetes.io/serviceaccount
TOKEN=$(cat ${SERVICEACCOUNT}/token)
CACERT=${SERVICEACCOUNT}/ca.crt
curl --cacert ${CACERT} --header "Authorization: Bearer ${TOKEN}" -X GET ${APISERVER}/api
curl --cacert ${CACERT} --header "Authorization: Bearer ${TOKEN}" -X GET ${APISERVER}/api/v1
curl --cacert ${CACERT} --header "Authorization: Bearer ${TOKEN}" -X GET ${APISERVER}/api/v1/namespaces/test05/pods
curl --cacert ${CACERT} --header "Authorization: Bearer ${TOKEN}" -X GET ${APISERVER}/api/v1/namespaces/test05/pods/alpine
```

## test08

Following has been tested:

* Helm integration with Kustomize and Argo CD.
* additionalValuesFiles feature with helmCarts in Kustomize (version >=5.0.0 needed).
* [kustomize/v5.0.0](https://github.com/kubernetes-sigs/kustomize/releases/tag/kustomize%2Fv5.0.0)
* Reloader to perform rolling upgrade when ConfigMap data changes.

## test10

Test Kustomize Strategic Merging.  
In the standard JSON merge patch, JSON objects are always merged **but lists are always replaced**. Often that isn't
what we want.  
To solve this problem, Strategic Merge Patch uses the go struct tag of the API objects to determine what lists should be
merged and which ones should not.

## test11

Test NFS subdir external provisioner using Raspberry Pi as NFS server to support dynamic provisioning of Kubernetes
Persistent Volumes via Persistent Volume Claims.

```bash
# Install NFS server on Raspberry Pi
sudo apt install nfs-common nfs-kernel-server -y

# Create NFS share
sudo mkdir -p /mnt/nfsshare

# Add to /etc/exports
sudo nano /etc/exports
/mnt/nfsshare	192.168.1.0/24(rw,all_squash,insecure,async,no_subtree_check,anonuid=1000,anongid=1000)

# Export and restart NFS server
sudo exportfs -avr
sudo systemctl restart nfs-kernel-server
sudo systemctl status nfs-kernel-server

# Mount on Mac OS (for test purposes)
sudo mount -o vers=4,resvport -t nfs 192.168.1.5:/mnt/nfsshare /Users/wim/Downloads/nfsmount
```
