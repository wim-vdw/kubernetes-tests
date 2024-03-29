# Kubernetes tests

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
