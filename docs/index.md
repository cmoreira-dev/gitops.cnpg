# gitops.cnpg

The [CloudNativePG](https://cloudnative-pg.io/) operator plus the per-app
Postgres `Cluster` resources it manages for the `cmoreira-dev` homelab
cluster — each `Cluster` co-located in its own consumer's namespace, not in
a shared database namespace.

## Layout

- `helm/cnpg-system/` — the CNPG operator itself (cluster-scoped CRDs +
  controller).
- `kustomize/backstage/` — the `Cluster` resource backing
  [`backstage.homelab`](https://github.com/cmoreira-dev/backstage.homelab)'s
  catalog/auth database, deployed in the `backstage` namespace.
- `kustomize/litellm/` — the `Cluster` resource backing
  [`gitops.ai-core-addons`](https://github.com/cmoreira-dev/gitops.ai-core-addons)'s
  LiteLLM database, deployed in the `litellm` namespace.
- `terraform/` — reconciled by the Burrito instance in
  [`gitops.core-addons`](https://github.com/cmoreira-dev/gitops.core-addons).

## Why per-consumer namespaces

Each app's `Cluster` lives in that app's own namespace rather than a shared
`cnpg` namespace, because Kubernetes `secretKeyRef` doesn't cross namespace
boundaries — an app's Deployment reads its database credentials Secret from
its own namespace, so the `Cluster` (and the Secret CNPG generates for it)
has to be there too. This bit LiteLLM specifically: a `cnpg-litellm` cluster
in a separate namespace would leave `litellm`'s pods unable to resolve their
DB credentials.

## Adding a new app's database

1. Add a `kustomize/<app>/` folder with a CNPG `Cluster` manifest, in the
   same namespace the consuming app deploys into.
2. Discovered automatically by the `gitops-repos` ArgoCD `ApplicationSet`
   once referenced from `argocd/`.
3. Point the consuming app's `ExternalSecret`/connection config at the
   Secret CNPG generates (`<cluster-name>-app`) in that namespace.
