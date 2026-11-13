# NEXUS - Architecture

NEXUS is a small Intent-Driven platform. A developer writes a short **Intent** file; the platform
turns it into a Kubernetes **Desired State**, checks it against policies, has four AI agents review
it in a Pull Request, waits for a **human approval**, and lets GitOps apply and protect it.

## Overview

```mermaid
flowchart TB
  subgraph DEV["Developer"]
    I["intents/shop-api.yaml<br/>Intent"]
  end
  subgraph GH["GitHub - public repo"]
    PR["Pull Request"]
    subgraph CI["GitHub Actions"]
      R["helm template<br/>Intent to Desired State"]
      K1["kyverno CLI<br/>policy check"]
      O["Orchestrator"]
      A1["Architect Agent"]
      A2["Observability Agent"]
      A3["Governance Agent"]
      A4["FinOps Agent"]
    end
    H["Human Approval<br/>review + merge"]
    M["main branch<br/>source of truth"]
  end
  subgraph K8S["Cluster - kind, then GKE Autopilot"]
    ARGO["Argo CD"]
    KYV["Kyverno admission"]
    APP["shop-api Pods"]
  end
  I --> PR --> R --> K1 --> O
  O --> A1 & A2 & A3 & A4
  A1 & A2 & A3 & A4 --> O
  O -->|"PR comment"| H
  H -->|merge| M
  ARGO -->|"pull"| M
  ARGO -->|apply| KYV --> APP
  ARGO -.->|"drift? self-heal"| APP
```

## Layers

| Layer | Role | Implementation |
|---|---|---|
| Intent | Developer interface: ~10 lines of YAML | `intents/*.yaml` |
| Translation | Golden Path: Intent -> manifests | Helm chart `platform/golden-path` |
| Governance | Rules as code, checked twice (CI + admission) | Kyverno, `policies/` |
| Intelligence | Four specialised reviews + synthesis | `agents/` (Python + LLM) |
| Approval | A human decides | GitHub PR + branch protection |
| Delivery | Git -> cluster, continuously | Argo CD, `gitops/` |
| Runtime | Where Pods run | kind (dev), GKE Autopilot (demo) |

## Data flow

```mermaid
sequenceDiagram
  actor Dev
  participant Git as GitHub PR
  participant CI as GitHub Actions
  participant Ag as Agents
  actor Hum as Human
  participant Argo as Argo CD
  participant Kyv as Kyverno
  participant K8s as Kubernetes
  Dev->>Git: push intent + open PR
  Git->>CI: trigger
  CI->>CI: helm template (Desired State)
  CI->>CI: kyverno apply (pass/fail)
  CI->>Ag: intent + manifests + results
  Ag-->>Git: comment: 4 reviews + verdict
  Hum->>Git: approve + merge
  Argo->>Git: pull main
  Argo->>K8s: apply
  K8s->>Kyv: admission review
  Kyv-->>K8s: allow / deny
  Note over Argo,K8s: loop: compare Git vs cluster, fix drift
```

## Two reconciliation loops

- **Kubernetes reconciliation**: controllers compare the object stored in the API (e.g. a
  Deployment asking for 2 replicas) with reality (running Pods) and fix reality.
- **GitOps reconciliation**: Argo CD compares Git with the objects stored in the API and fixes the
  API objects.

Git -> API (Argo CD), then API -> reality (Kubernetes).

## Agents

```mermaid
flowchart LR
  IN["Inputs:<br/>intent.yaml<br/>rendered manifests<br/>Kyverno report<br/>cost calculation"] --> ORC["Orchestrator<br/>Python"]
  ORC --> AR["Architect"]
  ORC --> OB["Observability"]
  ORC --> GO["Governance"]
  ORC --> FI["FinOps"]
  AR & OB & GO & FI --> SYN["Synthesis:<br/>APPROVE / NEEDS CHANGES"]
  SYN --> PRC["PR comment"]
  PRC --> HU["Human decides"]
```

These are role agents inside an orchestrated workflow. They advise; they never act on the cluster.

## Security boundaries

- Agents and CI hold only the workflow `GITHUB_TOKEN`. No kubeconfig, no cloud credentials.
- Argo CD uses a **pull** model: no cluster credential ever leaves the cluster.
- Kyverno admission is the last line of defence, even against a direct `kubectl apply`.
- `main` is protected: PR, one approval and green CI are required.
- No secrets in this repository. The repo is public and Argo CD reads it anonymously.

## Network

- No Load Balancer, no Ingress, no static IP, no Cloud NAT.
- Services are `ClusterIP`; access is through `kubectl port-forward` only.

## IAM (GCP)

- A dedicated project, owned by the student account. No service account keys.
- In the cluster: Argo CD default RBAC; application Pods have no API permissions.
