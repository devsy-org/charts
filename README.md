# charts

Helm charts for Devsy. Charts are published as OCI artifacts to `oci://ghcr.io/devsy-org/charts/<chart>`.

## Charts

| Chart | Path | Status |
|-------|------|--------|
| platform | `charts/platform` | Scaffold (0.1.0). Default image is a placeholder. |

## Install

```sh
helm install platform oci://ghcr.io/devsy-org/charts/platform --version 0.1.0 \
  -n devsy-platform --create-namespace
```

Without an Ingress, reach the service with a port-forward (the NOTES printed by Helm show the exact command).

## Placeholder image (read first)

The default `image.repository` is `ghcr.io/devsy-org/devsy-pro`, a transitional placeholder for a future Devsy Platform server image.

- It is a rebadged upstream build. It is not the Devsy Platform server.
- It is published for **arm64 only**. amd64 nodes will fail to pull it.
- The container port (`service.containerPort`, default 80) and the admin password env var (`ADMIN_PASSWORD`) are **unverified** against the server.

Replace `image.repository` and `image.tag` when the Devsy Platform server image exists.

## Values

Full documentation is in `charts/platform/values.yaml`.

| Key | Default | Description |
|-----|---------|-------------|
| `image.repository` | `ghcr.io/devsy-org/devsy-pro` | Placeholder image (arm64 only). |
| `image.tag` | `latest` | Image tag. |
| `image.pullPolicy` | `IfNotPresent` | Image pull policy. |
| `imagePullSecrets` | `[]` | Pull secrets for private registries. |
| `product` | `platform` | Rendered as env `PRODUCT` (unverified). |
| `command` | `[]` | Container command override. Empty uses the image entrypoint/CMD. |
| `args` | `[]` | Container args override. Empty uses the image CMD. |
| `admin.password` | `""` | When set: Secret `<fullname>-admin`, env `ADMIN_PASSWORD` (unverified). When empty: nothing injected. |
| `env.DISABLE_DEVSY_ROUTER` | `""` | Rendered as env when non-empty. Use `--set-string`. |
| `env.DEVSY_HOST` | `""` | Rendered as env when non-empty. |
| `env.DEVSY_SUBDOMAIN` | `""` | Rendered as env when non-empty. |
| `replicaCount` | `1` | Keep at 1 (RWO volume, Recreate strategy). |
| `service.type` | `ClusterIP` | Service type. |
| `service.port` | `80` | Service port (unverified). |
| `service.containerPort` | `80` | Container port (unverified). |
| `ingress.enabled` | `false` | Create Ingress for the platform. Requires `ingress.host`. |
| `ingress.host` | `""` | Platform host, e.g. `platform.example.com`. |
| `ingress.className` | `""` | IngressClass name. |
| `ingress.annotations` | `{}` | Ingress annotations. |
| `ingress.tls` | `[]` | Ingress TLS entries. |
| `devsyIngress.enabled` | `false` | Second Ingress for workspace subdomains, same backend. Requires `devsyIngress.host`. |
| `devsyIngress.host` | `""` | Wildcard host, e.g. `*.example.com`. |
| `devsyIngress.className` | `""` | IngressClass name. |
| `devsyIngress.annotations` | `{}` | Ingress annotations. |
| `devsyIngress.tls` | `[]` | Ingress TLS entries. |
| `persistence.enabled` | `true` | PVC mounted at `/var/lib/postgresql/data`. When false: emptyDir (data lost with pod). |
| `persistence.storageClass` | `""` | StorageClass name. Empty uses cluster default. |
| `persistence.accessModes` | `[ReadWriteOnce]` | PVC access modes. |
| `persistence.size` | `10Gi` | PVC size. |
| `serviceAccount.create` | `true` | Create a ServiceAccount. |
| `serviceAccount.name` | `""` | ServiceAccount name override. |
| `serviceAccount.annotations` | `{}` | ServiceAccount annotations. |
| `podSecurityContext` | `runAsUser: 998`, `fsGroup: 998`, `runAsNonRoot: true` | Pod security context. |
| `securityContext` | no privilege escalation, not privileged, drop `ALL` | Container security context. |
| `livenessProbe` / `readinessProbe` | `{}` | Probes. Empty = none (server health path unverified). |
| `resources` | `{}` | Container resources. |
| `podAnnotations` / `podLabels` | `{}` | Extra pod metadata. |
| `nodeSelector` / `tolerations` / `affinity` | `{}` / `[]` / `{}` | Scheduling. |

## Operational notes

- Uninstalling the release deletes the PVC and the database data.
- Changing `admin.password` does not restart pods. Run `kubectl -n <ns> rollout restart deployment/platform`.
- Set `persistence.enabled=false` only for throwaway installs.

## Development

```sh
helm lint charts/platform --strict
helm template platform charts/platform --namespace devsy-platform
```

CI (`.github/workflows/ci.yml`) lints, renders default and CLI-contract values, and runs kubeconform on pull requests.

Chart releases: push a tag `platform-v<chart version>` (for example `platform-v0.1.0`). `.github/workflows/release.yml` verifies the tag matches `Chart.yaml` and pushes the packaged chart to GHCR.
