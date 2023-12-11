# Demo Cluster

Flux driven kubernetes cluster.

```mermaid
mindmap
  root(clusters)
    apps
      demo
        emojivoto
    charts
      sealed secrets
      orbservability observer
    manifests
      storage class
```

## Setup

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

### Pixie

Open source observability tool for Kubernetes applications. Uses eBPF to automatically capture telemetry data without the need for manual instrumentation.

<https://docs.px.dev/about-pixie/what-is-pixie/>

This takes advantage of some of the features of the linux kernel, and will not work in all Kubernetes environments. See [requirements](https://docs.px.dev/installing-pixie/requirements/).

## Installation

### Bootstrap

When spinning up the cluster for the first time, it'll need to be [bootstrapped](https://fluxcd.io/flux/installation/bootstrap/github/). Make sure you have the `GITHUB_TOKEN` env set.

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
      --token-auth \
      --owner=orbservability \
      --repository=demo-cluster \
      --branch=main \
      --path=clusters/overlays/local
    ```

3. Install `cilium`

    <https://docs.cilium.io/en/stable/gettingstarted/k8s-install-default/>

    - [System Requirements](https://docs.cilium.io/en/stable/operations/system_requirements/#admin-system-reqs)
    - [Rebuilding the Linux Kernel](https://gist.github.com/dudo/7d853fd54f2d3db6e5e44b8b59ae12d5)

    ```sh
    cilium install --version 1.14.4
    cilium status --wait
    ```

## Usage

### kubectl

<https://kubernetes.io/docs/reference/kubectl/cheatsheet/>

```sh
kubectl get GitRepository -n flux-system
kubectl get Kustomization -n flux-system
kubectl get HelmRelease -n blue
kubectl logs -n flux-system deploy/image-automation-controller

kubectl run curl --image=curlimages/curl --restart=Never --rm -it -- sh
kubectl run busybox --image=busybox --restart=Never --rm -it -- sh

kubectl port-forward -n emojivoto service/web-svc 3000:80
```

### flux

<https://fluxcd.io/flux/cmd/>

```sh
flux get all -A

flux suspend image update my-service
flux suspend hr my-chart
flux resume image update my-service
flux resume hr my-chart

flux reconcile source git flux-system
flux reconcile kustomization flux-system
flux reconcile kustomization charts
```

### kubeseal

<https://github.com/bitnami-labs/sealed-secrets>

```sh
encoded_auth=$(echo -n "user:token" | base64)
json_config=$(cat <<EOF
  {
    "auths": {
      "ghcr.io": {
        "auth": "$encoded_auth"
      }
    }
  }
EOF
)
encoded_config=$(echo -n "$json_config" | base64 -w 0)

kubeseal --format=yaml <<EOF
apiVersion: v1
kind: Secret
metadata:
  name: container-registry-auth
  namespace: orbservability
type: kubernetes.io/dockerconfigjson
data:
  .dockerconfigjson: $encoded_config
EOF
```

## Pertinent Sections

- [Apps](./apps)
- [Charts](./charts)
- [Clusters](./clusters)
