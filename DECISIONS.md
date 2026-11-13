# Decisions

Architecture is frozen as of Task 1. Any change must be recorded here **before** it is applied.

| # | Decision | Why | Rejected |
|---|---|---|---|
| D1 | Build and test on local `kind`; use GKE Autopilot only for a time-boxed final run (2h, hard limit 3h) | Total GCP budget is 30 MAD; the real cost risk is a forgotten cluster | Developing directly on GKE |
| D2 | GitOps with **Argo CD** (pinned v3.6.1, official `install.yaml`) | UI makes sync and drift visible for evidence | Flux |
| D3 | Policy as Code with **Kyverno** | Policies are plain YAML; same policies run in CI (CLI) and at admission | OPA Gatekeeper (Rego) |
| D4 | Intent -> Desired State with a **Helm chart** (`platform/golden-path`); the Intent is a values file | Declarative, native to Argo CD, no custom code | Crossplane, KRO, custom script |
| D5 | Agents run in **GitHub Actions** on Pull Requests, read-only | Free on a public repo; zero cluster or cloud credentials | Agents as Pods in the cluster |
| D6 | LLM through **GitHub Models** with the workflow `GITHUB_TOKEN` | No API key to store, no cost. Quota and model to be confirmed in Task 4 | Paid API with a stored key |
| D7 | Orchestrator is one small Python script | A framework is not justified for four LLM calls | LangGraph, CrewAI |
| D8 | Demo app is the public image `ghcr.io/stefanprodan/podinfo` | Has health endpoints; nothing to build or host | Custom app |
| D9 | No Load Balancer, Ingress, NAT, registry or persistent disk | Cost; `kubectl port-forward` is enough | - |
| D10 | Golden Path sets `requests == limits` including `ephemeral-storage` | Predictable cost; prevents GKE Autopilot from rewriting resources and causing false drift in Argo CD | Leaving resources to defaults |
| D11 | App-of-apps: `gitops/bootstrap/root-app.yaml` is the only manifest applied by hand | Everything else, including the platform tools, comes from Git | Applying each Application manually |
| D12 | Observability agent reviews manifests only (probes, labels, resources) | No Prometheus within budget and time | Live telemetry |
