<div align="center">

# Advanced Container Security Platform

**Defense-in-depth container security, from a signed build to a killed pod, built to be defended, not just demoed.**

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Kubernetes](https://img.shields.io/badge/Kubernetes-k3s-326CE5?logo=kubernetes)](https://k3s.io)
[![Falco](https://img.shields.io/badge/Runtime-Falco%20%7C%20eBPF-00AEC7)](https://falco.org)
[![Kyverno](https://img.shields.io/badge/Admission-Kyverno-blue)](https://kyverno.io)
[![Trivy](https://img.shields.io/badge/Scanner-Trivy-1904DA)](https://trivy.dev)
[![Cosign](https://img.shields.io/badge/Signing-Cosign-4A90D9)](https://sigstore.dev)


[![Supply Chain](https://img.shields.io/badge/Supply%20Chain-SLSA%20L2%20%7C%20SBOM-4A90D9)](https://slsa.dev)
[![Compliance](https://img.shields.io/badge/Compliance-CIS%20%7C%20kube--bench%20%7C%20Polaris-green)](https://www.cisecurity.org/)
[![Observability](https://img.shields.io/badge/Observability-Prometheus%20%7C%20Grafana%20%7C%20Loki-E6522C)](#observability)
[![ML Detection](https://img.shields.io/badge/ML-Isolation%20Forest-purple)](#ml-anomaly-detection)
[![Automated Response](https://img.shields.io/badge/Response-Python%20Engine-orange)](#automated-response)



</div>

---

## The Problem

A container image passes through more hands than most systems its size ever will: a build step, a registry, a scheduler, and finally a kernel that runs it. Most security programs defend one or two of those stages well and treat the rest as someone else's problem. A vulnerability scan at build time says nothing about whether the image reaching production is the one that was actually scanned. A policy blocking a bad manifest at deploy time says nothing about a process spawning a shell three hours after the pod is already running. Point solutions solve point problems. The attacks that work live in the gaps between them.

The harder problem shows up after something is actually detected. Plenty of tooling can raise an alert. Far less of it can act on that alert consistently, without a human awake to read it, in a way someone can reconstruct afterward. Fewer projects still are honest about the trade-off baked into that automation: respond too aggressively and a false positive takes down a legitimate workload, respond too conservatively and a real compromise sits there quietly while someone sleeps. Most teams end up on one side of that trade-off by accident, and never write down which one.

---

## Why This Exists

This began as an EduQual capstone project and outgrew the assignment. Most container security demos prove a tool runs: a scanner produces a report, an intrusion detector fires on a default rule. Proving the pieces hold together as a system is a different, harder problem, and so is surviving the question of what happens when one layer fails. This project was built to survive that question at every layer. Every tool choice is written down against the alternatives that lost. Every automated response is a deliberate trade-off, not a default. Every known gap is documented instead of buried in a footnote.

---

## Key Features

- **Keyless image signing**: every build signed via Sigstore, recorded to a public transparency log, no private key to protect or leak
- **SBOM and vulnerability gating** before an image is even eligible to be signed
- **Policy-as-code admission control**: seven enforced Kyverno policies plus Pod Security Standards, rejecting non-compliant workloads before they run
- **Kernel-level runtime detection** via eBPF, with 11 custom rules mapped to MITRE ATT&CK
- **Sub-second automated response**: compromised pods terminated or quarantined with no human in the loop
- **ML anomaly detection, advisory by design**: a second, pattern-agnostic signal that never gets to act on its own
- **A full observability stack**: metrics, logs, and a live audit trail correlated across every layer
- **Automated compliance evidence**: CIS Kubernetes Benchmark scoring plus per-workload configuration audits
- **Default-deny network segmentation** across every namespace
- **A framework mapping that's actually filled in**, cross-walked against NIST 800-190, NIST CSF 2.0, DORA, and HIPAA

---

## Architecture

![Platform Architecture](assets/diagrams/main-architecture-diagram.jpeg)

Five independent layers run across the container lifecycle: supply chain signing and admission control keep untrusted workloads from ever starting, kernel-level runtime detection watches what happens once they do, an automated response engine acts on what it finds, and an observability and compliance layer records all of it. A sixth layer which is ML-based anomaly detection runs alongside the rest in an advisory-only capacity. Component breakdowns, data flow, and trust boundaries live in [`docs/architecture/`](docs/architecture/).

---

## Tech Stack

| Layer | Technology |
|---|---|
| Runtime | K3s · containerd · Ubuntu 24.04 |
| Supply Chain | Cosign · Sigstore · Rekor · Syft · SLSA L2 |
| Admission | Kyverno · Pod Security Standards |
| Detection | Falco 0.43.1 · modern eBPF |
| Response | Python · Flask · kubernetes-client |
| ML | scikit-learn · Isolation Forest |
| Observability | Prometheus · Grafana · Loki · Promtail |
| Compliance | kube-bench · Polaris |
| Network | Kubernetes NetworkPolicy (default-deny) |

---

## Repository Structure

```text
├── app/ # Hardened demo application
├── falco/ # Runtime detection rules + Helm values
├── kyverno/ # Admission control policies
├── response-engine/ # Automated response service
├── ml-service/ # ML anomaly detection service
├── monitoring/ # Prometheus · Grafana · Loki · Promtail
├── compliance/ # kube-bench · Polaris · framework mapping
├── network-policies/ # Default-deny namespace isolation
├── k8s/ # Namespaces, audit policy, demo manifests
├── docs/ # Architecture, design decisions, ADRs, production & future-work notes
├── scripts/ # demo · verify · cleanup
└── .github/workflows/ # CI/CD supply chain pipeline
```


---

## Documentation

This repository treats documentation as a deliverable, not an afterthought, more than 40 markdown files recording not just what was built, but why, what alternatives lost, and where it still falls short.

| Folder | What's inside |
|---|---|
| [`docs/architecture/`](docs/architecture/) | System overview, per-component breakdown, data flow, runtime behavior, threat model, trust boundaries |
| [`docs/design-decisions/`](docs/design-decisions/) | Why each component looks the way it does. The context, alternatives considered, trade-offs |
| [`docs/adr/`](docs/adr/) | Chronological architecture decision records |
| [`docs/production/`](docs/production/) | Scaling, high availability, secrets management, monitoring & alerting, backup & recovery |
| [`docs/limitations/`](docs/limitations/) | Known gaps, deliberate scope exclusions, and the assumptions every control depends on |
| [`docs/future-work/`](docs/future-work/) | Roadmap plus deep-dive plans for what's next |

Every component documents itself in place, too. [`falco/README.md`](falco/README.md), [`kyverno/README.md`](kyverno/README.md), [`response-engine/README.md`](response-engine/README.md), and others cover exact configuration and known gaps down to the line of code. [`compliance/README.md`](compliance/README.md) includes a full control-by-control mapping against NIST SP 800-190, NIST CSF 2.0, DORA, and HIPAA.

Start with [`docs/architecture/overview.md`](docs/architecture/overview.md) if you only read one thing.

---

## Screenshots

| | |
|---|---|
| ![CI/CD pipeline](assets/screenshots/pipeline-run.png)<br>*Build → scan → sign → attest, gated on a Trivy CRITICAL finding* | ![Kyverno rejection](assets/screenshots/kyverno-rejection.png)<br>*A non-compliant pod rejected at admission, reason included* |
| ![Falcosidekick alert stream](assets/screenshots/falcosidekick-ui.png)<br>*Live alerts, seconds after a shell spawns in a monitored container* | ![Grafana dashboard](assets/screenshots/grafana-dashboard.png)<br>*Cross-layer metrics, ML anomaly scores, and the audit trail* |

---

## Current Limitations

- Single-node K3s: no high availability, no disaster recovery, one node is the whole cluster
- Admission-time signature checking is a string match today, not full cryptographic verification against the signing chain
- 24-hour log and metric retention, no backup strategy. Fine for a lab, not for production
- Automated response has to pick a side of the false-positive-vs-dwell-time trade-off; it's tuned, not solved
- Alerts land on a dashboard, not in someone's pocket. No paging or notification routing yet

Full list, including the assumptions every control depends on: [`docs/limitations/`](docs/limitations/).

---

## Future Work

- Expand Falco's rule set. Container escape via the runtime socket, service account token theft, kernel module loads
- Turn quarantine into a real containment boundary. Network isolation and a forensic hold, not just a label
- Give the ML anomaly service a narrow, corroborated path into automated response
- Add GitOps (ArgoCD/Flux) and continuous in-cluster scanning with the Trivy Operator
- Push the supply chain to SLSA Level 3 with a full in-toto attestation chain
- **Next, separately:** an eBPF enforcement platform built on Tetragon. [`docs/future-work/tetragon-ebpf-enforcement.md`](docs/future-work/tetragon-ebpf-enforcement.md)

Full roadmap and reasoning: [`docs/future-work/roadmap.md`](docs/future-work/roadmap.md).

---

## License

Released under the MIT License.

---

<div align="center">

Built by **Ali Afnan**

</div>
