# Cost

**Hard ceiling: 30 MAD total** for all Google Cloud usage. Budget alerts are not a spending cap.

## Estimate (to be re-checked against the official pricing page before creating anything)

Assumptions: region `us-central1`, on-demand, 1 USD ~ 10 MAD (rounded up). Unit prices below are
from memory and **not yet verified**; they are verified in Task 5 before any paid resource exists.

| Item | Calculation | USD/h |
|---|---|---|
| GKE cluster fee (free tier not assumed) | 0.10 | 0.100 |
| Pod vCPU | 3 vCPU x ~0.0445 | 0.134 |
| Pod memory | 6 GiB x ~0.0049 | 0.029 |
| Pod ephemeral storage | 8 GiB x ~0.00005 | ~0.001 |
| Load Balancer, NAT, static IP, disks, registry, AI API | not used | 0 |
| **Total** | | **~0.264 USD/h ~ 2.6 MAD/h** |

3 vCPU / 6 GiB is a deliberately high ceiling for 8 Pods.

| Scenario | Cost |
|---|---|
| Planned: 2h | ~5.3 MAD |
| Hard limit: 3h | ~7.9 MAD |
| 3h and prices wrong by x2 | ~16 MAD |
| Cluster forgotten for 24h | ~63 MAD - over budget |

## Actual

| Phase | Paid resources | Cost |
|---|---|---|
| Tasks 1-4 (local kind, GitHub) | none | 0 MAD |
| Task 5 (GKE Autopilot) | pending | pending |

Final figure and the billing report go here and in `evidence/06-cleanup/`.
