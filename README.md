# kustomizer

[![e2e](https://github.com/stefanprodan/kustomizer/workflows/e2e/badge.svg)](https://github.com/stefanprodan/kustomizer/actions)
[![license](https://img.shields.io/github/license/stefanprodan/kustomizer.svg)](https://github.com/stefanprodan/kustomizer/blob/master/LICENSE)
[![release](https://img.shields.io/github/release/stefanprodan/kustomizer/all.svg)](https://github.com/stefanprodan/kustomizer/releases)

Kustomizer is command-line utility for applying kustomizations on Kubernetes clusters.
Kustomizer garbage collector keeps track of the applied resources and prunes the Kubernetes
objects that were previously applied on the cluster but are missing from the current revision.

## Install

Download the Kustomizer binary from the 
[release page](https://github.com/stefanprodan/kustomizer/releases)
or run [this script](install/README.md):

```bash
curl -s https://raw.githubusercontent.com/stefanprodan/kustomizer/master/install/kustomizer.sh | sudo bash
```

## Usage

Apply a kustomization by pointing Kustomizer to a local dir that contains Kubernetes manifests:

```bash
git clone https://github.com/stefanprodan/kustomizer
cd kustomizer

kustomizer apply testdata/plain --name=demo --revision=1.0.0
```

Kustomizer generates a `kustomization.yaml` if one doesn't exist, builds it and applies the 
resulting manifests on the cluster.

```text
$ kustomizer apply testdata/plain/ --name=demo --revision=1.0.0

namespace/kustomizer-demo created
serviceaccount/demo created
clusterrole.rbac.authorization.k8s.io/demo-read-only created
clusterrolebinding.rbac.authorization.k8s.io/demo-read-only created
service/backend created
service/frontend created
deployment.apps/backend created
deployment.apps/frontend created
horizontalpodautoscaler.autoscaling/backend created
horizontalpodautoscaler.autoscaling/frontend created
configmap/demo-snapshot created
```

After applying the resources, Kustomizer creates a ConfigMap in the format `<name>-snapshot`
used for garbage collection. You can change the ConfigMap namespace with `--gc-namespace` arg.

Remove the `frontend` and `rbac` manifests from the local dir:

```bash
rm -rf testdata/plain/frontend
rm -rf testdata/plain/common/rbac.yaml
```

Rerun the apply by changing the revision:

```text
$ kustomizer apply testdata/plain/ --name=demo --revision=2.0.0

namespace/kustomizer-demo configured
serviceaccount/demo configured
service/backend configured
deployment.apps/backend configured
horizontalpodautoscaler.autoscaling/backend configured
deployment.apps "frontend" deleted
horizontalpodautoscaler.autoscaling "frontend" deleted
service "frontend" deleted
clusterrole.rbac.authorization.k8s.io "demo-read-only" deleted
clusterrolebinding.rbac.authorization.k8s.io "demo-read-only" deleted
configmap/demo-snapshot configured
```

After applying the resources, Kustomizer removes the Kubernetes objects that are not present in the current revision.
Kustomizer garbage collector deletes the namespaced objects first then it removes the non-namspaced ones.
After the garbage collection finishes, Kustomizer update the ConfigMap snapshot with the new revision number.

Delete all the Kubernetes objects belonging to a kustomization including the ConfigMap snapshot:

```text
$ kustomizer delete --name=demo

deployment.apps "backend" deleted
horizontalpodautoscaler.autoscaling "backend" deleted
service "backend" deleted
serviceaccount "demo" deleted
namespace "kustomizer-demo" deleted
configmap "demo-snapshot" deleted
```
