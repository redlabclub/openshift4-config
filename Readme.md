# OpenShift 4 config with GitOps

## Deploy GitOps on prod cluster

`# oc apply -f conf-openshift-gitops/openshift-gitops-operator-subscription.yml`

## Config cluster authentication on ArgoCD

```
# oc login --insecure-skip-tls-verify https://api.prod.ocp.redlabclub.eu:6443

# prod cluster
oc login --insecure-skip-tls-verify https://api.prod.ocp.redlabclub.eu:6443
oc config rename-context $(oc config current-context) prod

# lab1 cluster
oc login --insecure-skip-tls-verify  https://api.lab1.dev.ocp.redlabclub.eu:6443
oc config rename-context $(oc config current-context) lab1

# switch context prod
oc config use-context prod

ARGO_PWD=$(oc -n openshift-gitops get secret openshift-gitops-cluster -o jsonpath='{.data.admin\.password}' | base64 -d)
ARGO_ROUTE=$(oc get route openshift-gitops-server -o jsonpath='{.spec.host}' -n openshift-gitops)

argocd --insecure --grpc-web login $ARGO_ROUTE:443  --username admin --password $ARGO_PWD

argocd --insecure --grpc-web cluster add prod

argocd --insecure --grpc-web cluster add lab1

argocd cluster list
```

## Deploy banner on prod cluster

`# oc apply -f gitops-app/banner-prod.yml`

## Deploy banner on lab1 cluster

`# oc apply -f gitops-app/banner-dev.yml`
