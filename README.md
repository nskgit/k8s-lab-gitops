# k8s-lab-gitops — deployment source of truth

Everything Argo CD reconciles on the k8s-lab cluster lives here. A change
merged to `main` IS a deployment; `git revert` IS a rollback. Split out of
`k8s-lab-infra` on 2026-07-15 (Block 11 step 0) so that "who can deploy"
(this repo) is a separate boundary from "who can change the cloud"
(`k8s-lab-infra`: Terraform + Ansible).

## Layout

- `bootstrap/` — the hand/Ansible-applied layer: Argo CD install values,
  `root-app.yaml` (the app-of-apps), install/runbook README. Applied once
  per cluster; everything else flows from git.
- `applications/` — one Argo `Application` per component (15). The
  root-app watches this directory: membership = the directory listing.
  Sync-waves: namespaces -1 → istio-base 0 → istiod 1 → gateway charts 2
  → Gateway objects 3.
- `platform/` — namespaces, Istio values + Gateway API objects,
  observability values + hand-crafted monitors.
- `workloads/` — demo apps (podinfo, echo canary pair, httpbin, intranet)
  and the blog stack (WordPress + MariaDB + Adminer).

## Access

Argo CD reads this repo with a **read-only SSH deploy key**
(`repo-k8s-lab-gitops` Secret in the `argocd` namespace; private key in
`~/.k8s-lab-secrets/argocd/argocd-gitops-deploy.key`, never committed).
Humans and (Phase 9+) the app-CI bot get write access HERE without ever
holding cloud or cluster credentials — that separation is the point of
the split.

## Related repos

- `k8s-lab-infra` (private) — Terraform + Ansible + project docs
  (PROJECT-PLAN, LEARNING-LOG, CURRENT-STATE all live there).
- `k8s-lab-apps` (Phase 9, planned) — application source; its CI pushes
  images to OCIR and bumps image tags in `workloads/` here via PR.

Rebuild story: Terraform → Ansible (installs Argo per D4) →
`kubectl apply -f bootstrap/root-app.yaml` → the platform cascades by
sync-wave. Adoption/abort runbooks: `bootstrap/README.md`.
