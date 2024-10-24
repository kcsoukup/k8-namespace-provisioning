# Kubernetes Namespace Provisioning Manifests
Very basic Helm Charts used for provision namespaces in Kubernetes. This is configured to be used as a dependency for ArgoCD ApplicationSet deployments, helps keep each deployment within standards.

The notes below lightly explain how to consume this repo as a Helm Chart dependency.

#### Add Chart.yaml
The repository value must match whats in index.yaml.
Also, the name must match the the parent Chart.yaml, see charts/namespaces/Chart.yaml

Chart.yaml contents:
```
apiVersion: v2
name: provision-namespace
description: A Helm chart for Kubernetes Namespaces
version: 0.1.1
dependencies:
  - name: k8-namespace-provisioning
    version: "0.1.0"
    repository: "https://raw.githubusercontent.com/kcsoukup/k8-namespace-provisioning/main/"
```

#### Add values.yaml
Also, the base reference name must match the parent Chart.yaml

values.yaml contents:
```
k8-namespace-provisioning:
  namespace:
    name: mushroomhead
```

##### Prepare Helm for Install
Add new repo that points to the raw repo path as used above

`helm repo add k8-namespace-provisioning https://raw.githubusercontent.com/kcsoukup/k8-namespace-provisioning/main/`

Update new repo cache

`helm repo update k8-namespace-provisioning`

Add dependency, this downloads the package from Git and stages it for use.

`helm dependency build`

Check package status

`helm dependency list`

Update package

`helm dependency update`

Test Helm manifest with --set

`helm install ns-mushroomhead --set k8-namespace-provisioning.namespace.name=mushroomhead . --debug --dry-run`

Output -- Can see both User Supplied and Computed, with the final manifest metadata.
```
NAME: ns-hello-world
LAST DEPLOYED: Mon Oct 21 07:59:31 2024
NAMESPACE: default
STATUS: pending-install
REVISION: 1
TEST SUITE: None
USER-SUPPLIED VALUES:
k8-namespace-provisioning:
  namespace:
    name: mushroomhead

COMPUTED VALUES:
k8-namespace-provisioning:
  global: {}
  namespace:
    name: mushroomhead

HOOKS:
MANIFEST:
---
# Source: provision-namespace/charts/k8-namespace-provisioning/templates/namespace.yaml
apiVersion: v1
kind: Namespace
metadata:
  name: mushroomhead
```

Test Helm manifest with values.yaml

`helm install ns-mushroomhead -f values.yaml . --debug --dry-run`

Output -- Can see the namespace name was taken from values.yaml
```
NAME: ns-hello-world
LAST DEPLOYED: Mon Oct 21 08:00:50 2024
NAMESPACE: default
STATUS: pending-install
REVISION: 1
TEST SUITE: None
USER-SUPPLIED VALUES:
k8-namespace-provisioning:
  namespace:
    name: mushroomhead

COMPUTED VALUES:
k8-namespace-provisioning:
  global: {}
  namespace:
    name: mushroomhead

HOOKS:
MANIFEST:
---
# Source: provision-namespace/charts/k8-namespace-provisioning/templates/namespace.yaml
apiVersion: v1
kind: Namespace
metadata:
  name: mushroomhead
```

Remove "--dry-run" to apply either of these Helm manifests

`helm install ns-hello-world --set k8-namespace-provisioning.namespace.name=mushroomhead . --debug`

Check for new namespace

`kubectl get namespaces`

Output
```
NAME                   STATUS   AGE
argocd                 Active   4d
damage-inc             Active   14d
default                Active   15d
dev                    Active   2d
ingress-nginx          Active   14d
kube-flannel           Active   15d
kube-node-lease        Active   15d
kube-public            Active   15d
kube-system            Active   15d
kubernetes-dashboard   Active   12d
metallb-system         Active   14d
mushroomhead           Active   7s      <-- New Namespace created with dependencies
prod                   Active   47h
```
