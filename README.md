# Demo Cluster

Flux driven kubernetes cluster.

```mermaid
mindmap
  root(clusters/local)
    apps
      demo
        emojivoto
        online boutique
    charts
      redpanda
      sealed secrets
      zilla
    manifests
      GatewayClass
      Gateway
      RuntimeClass
      StorageClass
    notifications
      webhook receiver
```

## Getting Started

### Kubernetes (via k0s)

An open-source system for automating deployment, scaling, and management of containerized applications.

<https://k0sproject.io/>

### Flux

Used to **pull** repository changes into kubernetes clusters.

<https://fluxcd.io/>

### Brew

The Missing Package Manager for macOS (or Linux).

<https://brew.sh>

This repo includes a collection of dependencies to install:

```sh
brew bundle
```

## Usage

### Bootstrap

When spinning up the cluster for the first time, there are 3 primary steps.

1. Install `k0s`

    <https://docs.k0sproject.io/v1.28.2+k0s.0/k0sctl-install/>

    ```sh
    k0sctl apply --config ./clusters/overlays/local/k0s.yaml
    k0sctl kubeconfig --config ./clusters/overlays/local/k0s.yaml
    # add the output of this to ~/.kube/config
    ```

2. Bootstrap `flux`

    <https://fluxcd.io/flux/installation/bootstrap/github/>

    ```sh
    flux bootstrap github \
      --components-extra=image-reflector-controller,image-automation-controller \
      --owner=dudo \
      --repository=turing-pi \
      --private=false \
      --personal=true \
      --path=clusters/overlays/local
    ```

### kubectl

<https://kubernetes.io/docs/reference/kubectl/cheatsheet/>

```sh
kubectl get GitRepository -n flux-system
kubectl get Kustomization -n flux-system
kubectl get HelmRelease -n blue
kubectl logs -n flux-system deploy/image-automation-controller

kubectl run curl --image=curlimages/curl --restart=Never --rm -it -- sh
kubectl run busybox --image=busybox --restart=Never --rm -it -- sh
```

### flux

<https://fluxcd.io/flux/cmd/>

```sh
flux get all -A

flux suspend image update my-service
flux resume image update my-service

flux reconcile source git flux-system
flux reconcile kustomization flux-system
flux reconcile kustomization charts
```

### kubeseal

<https://github.com/bitnami-labs/sealed-secrets>

```sh
encoded_string=$(echo -n "This is a string" | base64)
encoded_string=$(base64 <<EOF
  This is a
  multi-line string
  that I want to encode.
EOF
)

kubeseal --format=yaml <<EOF
apiVersion: v1
kind: Secret
metadata:
  name: mysecret
  namespace: whatever
data:
  my.file: ${encoded_string}
EOF
```

### Reset

Tearing down the cluster is a 1 step process.

1. Reset the cluster

   <https://docs.k0sproject.io/v1.28.2+k0s.0/reset/>

   ```sh
   k0sctl reset --config ./clusters/overlays/local/k0s.yaml
   ```

## Pertinent Sections

- [Apps](./apps)
- [Charts](./charts)
- [Clusters](./clusters)
- [Manifests](./manifests)
- [Notifications](./notifications)
