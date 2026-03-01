# Sessions Log

Per-session progress notes. Newest entry first.

---

## 2026-03-01 — Full end-to-end validation + out-of-box hardening (SESSION 5)

**Session goal**: Run the complete workshop flow in a clean tryout repo/VM and close all blockers until 01→06 works out of the box.

**Validated flow**:
- Fresh baseline in `ops-demo-tryout`:
  - force-reset to `upstream/main`
  - `vagrant destroy -f && vagrant up`
  - bootstrap + Argo repo registration + root commit
- Progressive exercise validation completed through 01→06.
- Final runtime state confirmed:
  - all Argo apps `Synced/Healthy`
  - podinfo image at `ghcr.io/stefanprodan/podinfo:6.7.0`
  - URLs responding: podinfo `200`, tekton dashboard `200`, grafana `302` (login redirect)

**Critical blockers found and fixed**:
1. **Tekton TaskRuns rejected by Pod Security Admission**  
   Symptom: `PodAdmissionFailed` in `tekton-pipelines` namespace.  
   Fix:
   - `manifests/ci/tekton/kustomization.yaml` now patches existing Namespace
   - new `manifests/ci/tekton/namespace-podsecurity-patch.yaml`
   - docs/04 updated with explicit rationale (what PSA means and why this trade-off is used in workshop)

2. **Pipeline validate step required unintended RBAC**
   Symptom: `validate` task failed with `Forbidden` on reads in `podinfo` namespace.  
   Fix:
   - switched validate command from `kubectl apply --dry-run=client` to
     `kubectl create --dry-run=client` (pure client-side validation)

3. **Workspace file ownership/mode mismatch between task images**
   Symptom: `bump-image-tag` failed with permission denied writing `deployment.yaml`.  
   Fix:
   - clone task now runs `chmod -R a+rwX .` so subsequent task images/users can write.

4. **Git push URL credential embedding failed**
   Symptom: `git-commit-push` failed with URL parse error (`Port number was not a decimal number...`).  
   Fix:
   - clone/push now use `http.extraHeader=Authorization: Basic ...`
     instead of embedding credentials in remote URL.

**Docs hardened**:
- `docs/04-tekton-pipeline.md` on `main` expanded with practical explanations:
  - clear PSA meaning (`enforce=privileged` does **not** mean pods must be privileged)
  - why namespace patch is needed in this workshop
  - task-level explanation and stronger troubleshooting guidance
- removed obsolete troubleshooting about `validate Forbidden` after validate-step fix.

**Branches updated**:
- `main`:
  - `f7a54b6` docs(ex04): clarify PodSecurity patch meaning and rationale
  - `2ef3bae` docs(ex04): align validate explanation with client-side check
- `solution/04-tekton-pipeline`:
  - `acf6be0` fix(ex04): patch Tekton namespace pod-security label
  - `09262dc` docs(ex04): clarify PodSecurity patch meaning and rationale
  - includes validated pipeline runtime fixes (validate mode, workspace perms, auth header clone/push)

**Notes**:
- Tryout required repoURL substitutions to its fork URL where solution manifests referenced `ops-demo`.
- No unresolved runtime blockers remained at end of session.

---

## 2026-03-01 — Assignment clarity pass + Tekton docs hardening (SESSION 4)

**Session goal**: Remove ambiguity in exercise instructions and align docs with real execution flow.

**Completed this session**:
- Exercise 03 expanded with explanatory text around key manifests:
  - MetalLB speaker/tolerations explanation
  - IPAddressPool + L2Advertisement purpose
  - Argo app split and sync-wave reasoning
  - Ingress intent for podinfo and ArgoCD
- Exercise 04 clarified and hardened:
  - Explicitly states `apps/ci/pipeline.yaml` is only an Argo wrapper
  - Makes `set-git-credentials.sh` a mandatory pre-step
  - Added Tekton Dashboard + ingress walkthrough in assignment text
  - Added troubleshooting for common Tekton/root drift
- Command-context UX improved across assignments:
  - Shell snippets now clearly labeled `VM` or `HOST` in quote-style blocks
  - Removed oversized top callout blocks from exercise pages per user preference
- `scripts/vm/set-git-credentials.sh` improved:
  - Next-step output now prints a usable PipelineRun manifest path (`manifests/...` or `/vagrant/...`)
    depending on where the script is run.

**Key commits pushed (main)**:
- `83d227a` docs(ex04): document tekton kustomize drift fix
- `a2c15d6` docs(ex04): add Tekton Dashboard UI + ingress walkthrough
- `0212f4b` docs: clarify command context and workshop flow

**Open follow-up**:
- If dashboard setup should be mandatory in `solution/04`, validate in tryout and backport explicitly to that branch.

---

## 2026-02-28 — Workflow hardening + docs alignment (SESSION 3)

**Session goal**: Fix operator-facing workflow issues, prevent wrong-cluster mistakes, and align docs/solutions with real usage.

**Completed this session**:
- Host/VM script split is now the working model in docs and flow:
  - host: `scripts/host/bootstrap-from-host.*`, `scripts/host/argocd-ui-tunnel.*`
  - vm: `scripts/vm/bootstrap.sh`, `scripts/vm/set-git-credentials.sh`, `scripts/vm/argocd-port-forward.sh`
- Bootstrap safety improved:
  - cluster target checks enforced in `scripts/vm/bootstrap.sh`
  - recursive app discovery fix merged (`cc0d36b`)
- README and exercise docs updated multiple times for:
  - `vagrant ssh` usage
  - Argo repo registration requirement
  - GitHub PAT guidance (fine-grained token path and permissions context)
  - host/VM execution clarity and troubleshooting
- MetalLB OutOfSync drift investigated and fixed:
  - root cause: CRD webhook `caBundle` drift behavior in Argo comparison
  - validated against `pms15-cluster` behavior
  - `solution/03-metallb-ingress` updated to ignore CRD `caBundle` drift generically (not single CRD name)
  - docs/03 troubleshooting updated on main
- Formatting pass landed for markdown readability (`fc0eb1b`), then targeted wording corrections.
- `CLAUDE.md` refreshed to current architecture and branch model.

**Key commits pushed (main)**:
- `c68292e` docs: clarify VM access via vagrant ssh only
- `71c1f79` improve bootstrap safety + host-side Argo access scripts
- `4d77c82` fix host KUBECONFIG leakage in host/vm scripts
- `cb912cf` split host/vm scripts + Argo tunnel workflow fix
- `d59818d` docs refinements (workshop flow + Argo repo credentials)
- `cc0d36b` bootstrap: recursive app discovery in root app
- `fc0eb1b` markdown formatting/readability
- `0dc7062` ex03 docs: Metallb CRD drift troubleshooting

**Key commit pushed (solution branch)**:
- `solution/03-metallb-ingress`: `2e6b4fb` (ignore Metallb CRD `caBundle` drift across CRDs)

**Notes / follow-up**:
- Keep `sessions.md`/`roadmap.md` in sync after every significant change.
- Verify all solution branches still obey "standalone per exercise" constraints before next content edits.

---

## 2026-02-28 — Branching restructure + Dutch translation (SESSION 2, INCOMPLETE)

**Session goal**: Restructure branches, translate docs to Dutch, rebuild solution branches off thin main.

**Completed this session**:
- `reference-solution` branch created from old main (full working solution) ✓
- Solution files removed from `main` (staged, NOT committed) ✓
- `scripts/bootstrap.sh` rewritten: Dutch, auto-detects fork URL (SSH→HTTPS), generates apps/root.yaml ✓
- `README.md` rewritten in Dutch ✓
- `docs/vm-setup.md` rewritten in Dutch ✓
- `docs/01-argocd-bootstrap.md` through `docs/06-monitoring.md` rewritten in Dutch ✓
- `docs/presentation/final-talk.md` — **STILL EMPTY** (1 line) — NOT YET DONE
- Old solution/NN-* branches NOT yet deleted/recreated
- **NOTHING COMMITTED on new main yet**

**Git status on `main`**:
- STAGED: deletions of all solution files (apps/apps/, apps/ci/, etc., manifests/apps/, etc.)
- UNSTAGED MODIFIED: README.md, all docs/*.md, scripts/bootstrap.sh
- UNTRACKED: none relevant

**What to do next session**:
1. Write `docs/presentation/final-talk.md` in Dutch (translate from reference-solution branch, natural dev-Dutch)
2. `git add -A` + ONE commit on main (all deletions + Dutch docs + new bootstrap.sh)
3. Delete old solution branches: solution/01 through solution/06
4. Recreate solution/01-argocd-bootstrap through solution/06-monitoring cumulatively off new thin main, each with ONE commit
5. Push everything to GitHub (paulharkink/ops-demo)
6. Continue smoke-testing exercises 02–05

**Key Dutch translation rules** (user was very clear):
- Natural dev-Dutch, written as if Paul wrote it
- Technical terms stay English: "branches", "cluster", "pipeline", "deployment", "namespace", etc.
- "takken" is NEVER acceptable
- No Apple Silicon warnings
- No "Co-Authored-By: Claude" in commits

---

## 2026-02-28 — Initial implementation (SESSION 1)

**Session goal**: Full repo scaffold from implementation plan.

**Completed**:
- Phase 1: CLAUDE.md, sessions.md, roadmap.md, Vagrantfile, scripts/bootstrap.sh,
  apps/root.yaml, apps/project.yaml, apps/argocd.yaml, manifests/argocd/values.yaml
  → `solution/01-argocd-bootstrap` branch created
- Phase 2: apps/apps/podinfo.yaml, manifests/apps/podinfo/, docs/01-argocd-bootstrap.md,
  docs/02-deploy-podinfo.md
  → `solution/02-deploy-podinfo` branch created
- Phase 3: MetalLB + Ingress-Nginx apps/manifests, podinfo ingress, ArgoCD ingress,
  docs/03-metallb-ingress.md
  → `solution/03-metallb-ingress` branch created
- Phase 4: Tekton app/manifests, pipeline resources, scripts/set-git-credentials.sh,
  docs/04-tekton-pipeline.md
  → `solution/04-tekton-pipeline` branch created
- Phase 5: docs/05-app-upgrade.md → `solution/05-app-upgrade` branch (deployment at 6.7.0)
- Phase 6: Monitoring app/manifests, docs/06-monitoring.md → `solution/06-monitoring` branch
- Phase 7: docs/vm-setup.md, README.md, docs/presentation/final-talk.md

**Vagrantfile fixes applied**:
- yq arch-aware: ARCH=$(dpkg --print-architecture)
- Tekton images: ghcr.io (not gcr.io)
- Docker Hub images: docker.io/ prefix required for k3s ctr
- kubeconfig chmod 600

**Not yet verified**: Full end-to-end smoke test pending.
