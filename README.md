# Fleet bundles

This repository contains Rancher Fleet bundles and Kubernetes manifests for:

- `kured/`
- `rancher-inventory/`
- `rancher-monitoring/`

## Proxy support for rancher-inventory

Proxy settings are optional and applied only to clusters labeled with
`use-inventory-proxy=true`. The base Deployment stays proxy-free.

### 1) Label the target clusters

Apply the label to only the clusters that require a proxy.

```sh
kubectl label clusters.management.cattle.io <cluster-name> use-inventory-proxy=true
```

### 2) Create the proxy Secret on those clusters

Create the Secret in the `rancher-inventory` namespace on each proxy cluster.
Do not store this Secret in Git.

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: inventory-proxy-env
  namespace: rancher-inventory
type: Opaque
stringData:
  HTTP_PROXY: http://proxy.example.com:8080
  HTTPS_PROXY: http://proxy.example.com:8443
  NO_PROXY: 10.0.0.0/8,.svc,.cluster.local,localhost
```

### Notes

- The Fleet patch injects `envFrom` pointing to `inventory-proxy-env` only for
  labeled clusters.
- Clusters without the label do not require the Secret.
