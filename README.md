# DevOps AI Prompt Pack

**Structured AI prompts for production Kubernetes and OpenShift troubleshooting.**

Maintained by **Serdhub**.

---

## The problem this solves

You are on-call at 2am. Something is broken. You paste a vague description into an AI tool and get a generic answer that does not match your environment, does not reference the actual data you have, and does not tell you what to verify before applying the fix.

This pack is different. Every prompt is a structured diagnostic template that:

- Requires you to collect **specific diagnostic inputs** before running the prompt
- Directs the AI to produce **evidence-backed, specific output** — not generic advice
- Includes **concrete verification steps** to confirm the fix is working
- Includes a **production warning** identifying the primary operational risk

The result: faster root cause identification, fewer blind guesses, and a clear verification checklist before you change anything in production.

---

## Target audience

- **DevOps Engineers** managing Kubernetes deployments and CI/CD pipelines
- **SRE / Platform Engineers** responsible for cluster reliability and on-call response
- **Kubernetes and OpenShift Administrators** dealing with control plane, networking, and security
- **Security Engineers** handling Kubernetes RBAC, secrets management, and runtime security

This is not a beginner resource. The prompts assume production environments, real diagnostic data, and operational familiarity with Kubernetes internals.

---

## What's included

The full pack contains **37 production-grade prompts** across five domains:

| Domain | Core Prompts | Topics covered |
|--------|-------------|----------------|
| Deployment & Rollout | 5 | Pod Pending, CrashLoopBackOff, stalled rollouts, Helm recovery, canary decisions |
| Networking & Connectivity | 5 | Pod-to-pod failures, DNS/conntrack, NetworkPolicy drops, mTLS certs, port exhaustion |
| Performance & Resource Pressure | 5 | CPU throttling, node eviction, slow DB queries, etcd latency, thundering herd |
| Security & Compliance | 5 | Secret rotation, CVE triage, secrets migration, Falco forensics, supply chain |
| Observability & Incident Response | 5 | Alert fatigue, SLO burn rate, distributed tracing gaps, log cost, capacity planning |
| **Bonus Prompts** | 7 | Post-deployment regression, namespace migration, GC pauses, cluster autoscaler, RBAC, Prometheus cardinality |
| **OpenShift-Native** | 5 | OLM operators, MachineConfig, Routes, OAuth/IdP, ClusterOperator degradation |

**Total: 37 prompts** — each with Scenario, Symptoms, Required Inputs, Prompt block, Expected Output, Verification Steps, Production Warning, and Notes.

---

## Free Sample

The [`FREE_SAMPLE.md`](./FREE_SAMPLE.md) file in this repository contains **5 complete prompts** selected to demonstrate the depth and style of the full pack:

1. **CrashLoopBackOff** — Diagnose a crashing container including the ephemeral debug container technique for containers that die before logging
2. **DNS Resolution Failing** — Identify `conntrack` table exhaustion as the silent DNS killer in high-throughput clusters
3. **etcd Slow: Control Plane Sluggish** — Expert-level diagnosis of etcd disk latency, DB fragmentation, and leader churn
4. **SLO Breach: Error Budget Burn Rate** — Apply SRE burn rate math to identify root cause and calculate the maximum sustainable error rate
5. **OLM Operator Installation Failure** — OpenShift-native: diagnose CatalogSource, InstallPlan, and CSV failures in the OLM pipeline

These five prompts are representative of the full pack's approach: they require real diagnostic data, produce specific output, and include verification steps you can run before declaring the incident resolved.

---

## Why this is different

Most "AI prompts for DevOps" collections are:
- Generic enough that the AI produces the same answer regardless of your actual data
- Missing the verification step — you apply a fix and hope it worked
- Missing the production warning — you learn about the risk after something goes wrong

This pack is built around a different philosophy:

**Garbage in, garbage out — so the prompt forces you to collect the right inputs first.**

Every prompt template requires specific diagnostic outputs (kubectl describe, Prometheus metrics, log extracts, etcd status) before the AI can produce a useful diagnosis. The quality of the AI output scales directly with the quality of the inputs you provide.

**The prompt is not the answer — it is the diagnostic framework.**

The AI produces a ranked analysis, a specific fix, and a verification checklist. You verify the fix before applying it. The production warning tells you the specific risk for that class of problem, not a generic "test in staging" boilerplate.

---

## OpenShift

The full pack includes **5 OpenShift-native prompts** covering problems that have no direct upstream Kubernetes equivalent:

- **OLM Operator Installation Failure** — Diagnosing CatalogSource, InstallPlan, and CSV lifecycle failures
- **MachineConfig Not Applied** — Debugging `machine-config-daemon` degradation and node rollout issues
- **OpenShift Route Not Exposed** — HAProxy router diagnostics, TLS termination, and IngressController issues
- **OAuth / Identity Provider Broken** — LDAP, HTPASSWD, and OIDC login failures via the OpenShift OAuth server
- **Cluster Operator Degraded** — Parsing `oc get co` Conditions messages to diagnose control plane component failures

These prompts reference OpenShift-specific resources: `oc`, Routes, SCCs, OLM, MachineConfig, ClusterOperators, and the OpenShift authentication stack. They are not reworded Kubernetes prompts.

---

## How to use a prompt

1. **Find your scenario** in the index or the FREE_SAMPLE
2. **Read the Required Inputs** — collect every item listed before you run the prompt
3. **Fill in the placeholders** — replace every `{{value}}` with your actual data
4. **Paste the completed prompt** into your AI tool (Claude, ChatGPT, Gemini, or similar)
5. **Read the output** — verify the suggested fix against the Verification Steps before applying it
6. **Check the Production Warning** — understand the specific risk before making changes

> **Disclaimer**: AI output must be reviewed by a qualified engineer before executing any commands or making configuration changes in a production environment. The prompts structure the diagnostic process — they do not replace engineering judgment.

---

## Get the full pack

The full 37-prompt pack remains available on the existing storefront during the brand transition:

**Full pack distribution is being migrated to **Serdhub**.**

The pack includes:
- All 25 Core Prompts, 7 Bonus Prompts, and 5 OpenShift-Native Prompts
- Complete Required Inputs, Verification Steps, and Production Warnings for every prompt
- Index, Diagnostic Safety Rules, and Troubleshooting Methodology
- Changelog and versioning for future updates

---

## Repository structure

```
.
├── README.md          — This file
├── FREE_SAMPLE.md     — 5 complete prompts from the full pack
├── CONTRIBUTING.md    — Contribution guidelines
├── LICENSE.md         — License and usage terms
└── .gitignore
```

---

## License

This repository is licensed under **CC BY-NC-ND 4.0** (Creative Commons Attribution-NonCommercial-NoDerivatives 4.0 International).

See [`LICENSE.md`](./LICENSE.md) for full terms. The sample prompts are published as demonstration material. The full product is a commercial release available on Gumroad.

---

*Serdhub · LAB-001 · DevOps AI Prompt Pack v1.0*
