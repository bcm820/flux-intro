# NOTES

This repo contains Flux component manifests bootstrapped by the Flux CLI. Refer to [https://fluxcd.io/docs/get-started/](https://fluxcd.io/docs/get-started/).

## What was run

1. `flux bootstrap github`: Creates the repository `flux-intro` in your GitHub account and adds Flux component manifests to it in the `flux-system` directory. Also deploys Flux components to your current K8s cluster and configures Flux components to track changes to the repo.

Note that I did not specify a subdirectory via the `--path` flag (i.e. `--path=./clusters/my-cluster`), which would bootstrap all components in that subdirectory and instruct Flux to only track changes in that subdirectory.

```bash
flux bootstrap github \
  --owner=$GITHUB_USER \
  --repository=flux-intro \
  --personal
```

_(From here on out, Git commit and push all changes to the repo following each step.)_

2. `flux create source git`: Creates a `GitRepository` manifest pointing to [another repo, podinfo](https://github.com/stefanprodan/podinfo) for reconciling manifests from that repo.

Note that this doesn't yet reconcile any manifests. It just registers a new source repository to reconcile from and checks it for updates every 30 seconds.

```bash
flux create source git podinfo \
  --url=https://github.com/stefanprodan/podinfo \
  --branch=master \
  --interval=30s \
  --export > podinfo-source.yaml
```

3. `flux create kustomization`: Creates a `Kustomization` manifest that specifies a directory in the registered source repository to reconcile manifests from. Flux will reconcile every 5 minutes.

Kustomize is a major dependency for Flux. Flux expects the directory to have a single `kustomization.yaml` file that declares the manifests to reconcile. [Plain YAML manifests are acceptable](https://fluxcd.io/docs/faq/#can-i-use-repositories-with-plain-yamls), but Flux will still create a `kustomization.yaml` internally.

```bash
flux create kustomization podinfo \
  --target-namespace=default \
  --source=podinfo \
  --path="./kustomize" \
  --prune=true \
  --interval=5m \
  --export > podinfo-kustomization.yaml
```

4. `flux get kustomizations --watch`: Watches for events around each defined Kustomization. This helps us to know that our `podinfo` Kustomization is being applied.

## Customizing manifests

Manifests from repos you don't own can be customized by adding `patches` to the `spec` of a `Kustomization`.

See the update made in [podinfo-kustomization.yaml](podinfo-kustomization.yaml) for an example.
