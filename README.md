# gitops-cluster-kind-observe

Cluster config for `kind-observe` - the original platform-cicd dev cluster, predating
`idp` entirely. Companion to
[`idp-service-catalog`](https://github.com/jfillman/idp-service-catalog),
[`gitops-cluster-dev`](https://github.com/jfillman/gitops-cluster-dev), and
[`gitops-cluster-kind-prod`](https://github.com/jfillman/gitops-cluster-kind-prod),
following the per-cluster repo shape `idp/docs/gitops-strategy.md` §1 establishes.

## Repo boundary only, not a migration

This repo exists to give `kind-observe` the same enforced per-cluster boundary every
other cluster in the fleet has - `gitops-cluster-dev`'s own root app-of-apps recurses
over its whole repo with no cluster gating, so nothing structurally stopped a future
change there from being pointed at `kind-observe` by mistake. Standing up this repo
closes that gap the same way `AppProject.sourceRepos` closes it for apps (§1's own
argument for per-cluster repos).

**It deliberately carries no content of its own yet.** `kind-observe`'s real platform-
cicd instance - the one actually running `nodejs-demo-app`'s and `cicd-flow-test-app`'s
live build/test/deploy/release pipelines - stays exactly as it is today: installed
imperatively via `platform-cicd/hack/bootstrap.sh`, not GitOps-managed. Migrating it to
GitOps is real, separate future work; this repo is just its future home once that
happens, not a claim that it's happened.

## Not a blank cluster

`kind-observe` already runs a live ArgoCD instance predating this repo by weeks, plus
several other tenants unrelated to platform-cicd entirely (`holmesgpt`,
`order-processing-service`, `payment-service`). This repo's `root-app-of-apps.yaml`
reuses that instance rather than standing up a second one - same single-instance-for-now
precedent `gitops-cluster-dev` and `gitops-cluster-kind-prod` both already use (the
`argocd-platform`/`argocd-apps` split is being built on `kind-dev` first, see
`idp/docs/gitops-strategy.md` §2; nothing here is blocked on it).

Real live state as of 2026-08-16 (`kubectl --context kind-observe get appproject,
applicationset -n argocd`), for reference - none of this is owned or tracked by this
repo, it's documented so a future migration knows what it's inheriting:

- **AppProjects**: `default`; `platform-onboarding` (Helm-managed by the
  `platform-cicd-control-plane` release - installs/manages the `platform-cicd-app`
  chart for every tenant under `tenants/*/identity.yaml` in `platform-cicd`'s own repo);
  `app-cicd-flow-test-app-cicd` and `platform-cicd-demo` (per-tenant projects for the
  two real onboarded apps).
- **ApplicationSets**: `tenant-onboarding` (platform-cicd's own, generates the per-app
  Applications above from `tenants/*/identity.yaml`); `nodejs-demo-app-pr-envs`
  (`pullRequest` generator, ephemeral preview environments).

No `10-crds-operators`/`20-service-catalog` content here, and none planned while this
repo stays boundary-only - `kind-observe` doesn't run Crossplane at all. Bootstrap-tier
XRDs (`NodeJSApplication`/`ApplicationEnvironment`) centralize on `kind-dev` permanently
per `idp/docs/service-catalog-design.md` §0, regardless of fleet size.

## Cleanup done alongside standing this repo up

Found live, 2026-08-16: a stray `idp-onboarding` AppProject in `kind-observe`'s `argocd`
namespace (created `2026-08-15T18:13:05Z`, `kubectl apply`-shaped annotations, tracking-
id `xr-requests-appproject:...` - an object that belongs to `kind-dev`'s
`xr-requests-appproject` Application, not anything on `kind-observe`). No Application on
`kind-observe` owned or referenced it (`kubectl get application -n argocd` returns zero
results there) - looks like a stray `kubectl apply` against the wrong context during the
`xr-requests` build session. Deleted as dead weight, not left in place.

## Status

Repo shell stood up 2026-08-16 as the first piece of the declarative `50-platform-cicd/`
work on `kind-dev` (see `idp/docs/gitops-strategy.md` §2/§7 and
`platform_cicd_session_platform_cicd_kind_dev` / `feedback_cluster_agnostic_idp`
memory for the full context). No live cluster change beyond the AppProject cleanup
above.
