---
type: project
name: io.space.infra
path: /Users/mova/code/io.space.infra
stack: [terraform, argocd, gitops]
tags: [project, infra]
---
# io.space.infra

**Terraform + ArgoCD GitOps for isohub.space.** Owns all cloud infrastructure provisioning and deployment for the [[isohub]] platform.

- **Path:** `/Users/mova/code/io.space.infra`
- **Stack:** Terraform · ArgoCD (GitOps)
- **Entry points:** `README.md`
- **Relation:** deploys [[isohub]]. Changes here are production-affecting — treat as
  outward-facing, confirm before apply.

## Notes
- Wiki notes: `#project/infra`. Runbooks belong in [[memory/wiki/_index|wiki/runbooks]].
