# Workshop Roadmap

## Exercise Map

| # | Exercise | Type | Est. Time | Status |
|---|----------|------|-----------|--------|
| 01 | Bootstrap ArgoCD | Core | 30 min | ✅ Implemented |
| 02 | Deploy podinfo via GitOps | Core | 30 min | ✅ Implemented |
| 03 | MetalLB + Ingress-Nginx (LAN exposure) | Core | 45 min | ✅ Implemented |
| 03b | Cloudflare Tunnel voor webhooks | Bonus | 30–45 min | ✅ Implemented |
| 04 | Tekton pipeline (image tag bump → GitOps loop) | Core | 45 min | ✅ Implemented |
| 05 | App upgrade via GitOps | Core | 15 min | ✅ Implemented |
| 06 | Monitoring: Prometheus + Grafana | Bonus | 60 min | ✅ Implemented |

**Total core: ~2.5–3h. Beginners may stop after Exercise 03 (~1h45m).**

---

## Solution Branches

Model: solution branches are **standalone per exercise** (not cumulative).

| Branch | State |
|--------|-------|
| `solution/01-argocd-bootstrap` | ArgoCD running, root app applied |
| `solution/02-deploy-podinfo` | podinfo synced via ArgoCD |
| `solution/03-metallb-ingress` | MetalLB + Ingress-Nginx + podinfo reachable on LAN; CRD `caBundle` drift handling included |
| `solution/03b-cloudflare-tunnel` | Cloudflared tunnel connector manifests met token placeholders |
| `solution/04-tekton-pipeline` | Full Tekton GitOps loop working |
| `solution/05-app-upgrade` | deployment.yaml bumped to 6.7.0 |
| `solution/06-monitoring` | Prometheus + Grafana running |

---

## Verification Status

| Exercise | Smoke-tested |
|----------|-------------|
| 01 | ✅ Validated (clean VM + bootstrap + root sync) |
| 02 | ✅ Validated (podinfo app deploy + healthy) |
| 03 | ✅ Validated (MetalLB + ingress + podinfo URL reachable) |
| 04 | ✅ Validated after hardening fixes (PSA patch + pipeline runtime fixes) |
| 05 | ✅ Validated (upgrade/drift workflow over working 04 stack) |
| 06 | ✅ Validated (Prometheus/Grafana app healthy + Grafana ingress reachable) |

Full end-to-end test: completed on `ops-demo-tryout` from clean baseline through 01–06.

---

## Recent Changes (2026-03-01)

- End-to-end smoke test executed in clean tryout environment (`vagrant destroy && vagrant up`).
- Exercise 04 hardening to make tutorial reproducible:
  - Tekton namespace PodSecurity patch (`pod-security.kubernetes.io/enforce=privileged`)
  - pipeline validate step switched to pure client-side `kubectl create --dry-run=client`
  - clone task now ensures workspace writeability for later task images (`chmod -R a+rwX .`)
  - git clone/push switched to HTTP auth header flow (no URL credential embedding)
- Exercise 04 docs clarified with explicit PSA semantics and workshop trade-offs.
- Assignment clarity improvements across docs/01..06:
  - every shell snippet clearly marked as `VM` or `HOST`
  - removed large per-page top callout blocks; context now lives at snippet level
- Exercise 03 docs expanded with practical explanation around MetalLB manifests and key Kubernetes terms.
- Exercise 04 docs expanded with:
  - explicit mandatory credential step before PipelineRun
  - clear distinction between Argo wrapper manifest vs full Tekton pipeline manifest
  - Tekton Dashboard + ingress walkthrough
- `scripts/vm/set-git-credentials.sh` now prints a context-correct PipelineRun path (`/vagrant/...` fallback included).
- Earlier branch-level fixes remain in place:
  - root recursive discovery
  - MetalLB CRD `caBundle` drift handling
  - Tekton empty `kustomize` drift fix in solution flow

## Recent Changes (2026-03-03)

- `ops-demo-tryout` reset for continuation from Exercise 03:
  - VM recreated from scratch
  - local `main` reset to `upstream/main`
  - solution 01 and 02 merged in sequence
  - tags created: `tryout-after-solution-01`, `tryout-after-solution-02`, `tryout-ready-assignment-03`
- Tryout manifests updated to fork URL `https://github.com/paulharkink/ops-demo-tryout.git`
  for `root`, Argo values source, and `podinfo` app.
- Verified runtime handoff state in tryout cluster:
  - `root` and `podinfo` are `Synced/Healthy`
  - `podinfo` deployment reaches `1/1` ready
