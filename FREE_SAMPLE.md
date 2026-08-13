# DevOps AI Prompt Pack — Free Sample

> **5 complete prompts** from the full 37-prompt pack.
> This file is a free demonstration. The complete pack is available at [phoenixplatform.gumroad.com](https://phoenixplatform.gumroad.com/).

---

## How to use these prompts

1. **Collect the Required Inputs** listed under each prompt — do not skip this step
2. **Replace every `{{placeholder}}`** with your actual diagnostic data
3. **Paste the completed prompt** into your AI tool (Claude, ChatGPT, Gemini, or similar)
4. **Verify the output** using the Verification Steps before applying any fix
5. **Read the Production Warning** before making changes to a live system

> **Disclaimer**: AI output must be reviewed by a qualified engineer before executing any commands or making configuration changes in a production environment.

---

## Prompts in this sample

| # | Title | Domain |
|---|-------|--------|
| 1 | CrashLoopBackOff: The Application Keeps Dying | Deployment |
| 2 | DNS Resolution Failing Inside the Cluster | Networking |
| 3 | etcd Slow: The Entire Control Plane Is Sluggish | Performance |
| 4 | SLO Breach: Diagnosing an Error Budget Burn Rate Alert | Observability |
| 5 | OLM Operator Installation Failure (OpenShift) | Platform |

---

---

## Sample 1 — CrashLoopBackOff: The Application Keeps Dying

### Scenario
A pod enters `CrashLoopBackOff` after a new deployment. The container starts, runs for 2–15 seconds, then dies. Kubernetes keeps restarting it with exponential backoff. You do not know if it is a misconfigured liveness probe, a missing secret, a bad entrypoint, or an application-level panic. Every second the pod is down is a second your service is degraded.

### Symptoms
- `kubectl get pods` shows `CrashLoopBackOff` or `Error` status
- Restart count increasing (3+)
- `kubectl logs` shows partial output or nothing (container dies too fast)
- No changes to the container image visible in the deployment history

### Required Inputs
- Last 100 lines from `kubectl logs <pod> --previous`
- Full `kubectl describe pod <pod-name>` (focus on Last State section)
- Liveness/readiness probe config from deployment YAML
- List of environment variable names injected (names only, not values)
- Names of mounted ConfigMaps and Secrets

### Prompt
```
A pod is in CrashLoopBackOff. Here is all available diagnostic data:

Application: {{service-name}}
Language/Runtime: {{e.g. Java 17 / Node 20 / Python 3.11}}
Deployment type: {{Deployment / StatefulSet / DaemonSet}}

Last 100 lines of container logs (kubectl logs {{pod}} --previous):
{{paste logs here}}

kubectl describe pod {{pod-name}}:
{{paste full describe — pay attention to Last State section}}

Current liveness and readiness probe config:
{{paste probe section from deployment YAML}}

Environment variables injected (names only, not values):
{{list env var names}}

ConfigMaps and Secrets mounted:
{{list names}}

Based on this data:
1. Identify the crash cause: application panic, missing dependency, misconfigured probe, OOMKill, signal handling issue, or init container failure
2. If the logs are empty or cut off because the container dies too fast, suggest the exact command to capture logs before death (init container with sleep, or ephemeral debug container)
3. Provide the exact remediation: probe adjustment, missing secret, entrypoint fix, or resource limit change
4. If the issue is a liveness probe killing a healthy-but-slow startup, provide the corrected probe config with startupProbe separation
5. Estimate blast radius: is this pod the only instance or does this affect all replicas?
```

### Expected Output
- Root cause of the crash with line reference from logs
- If logs are empty: exact command to attach an ephemeral debug container
- Corrected YAML section (probe, env, resource limits) — ready to apply
- Blast radius assessment

### Verification Steps
1. Confirm pod is Running with restart count 0 for 5+ minutes
2. Verify no `Liveness probe failed` events in pod events
3. Confirm application logs show clean startup sequence without panic/error

### ⚠️ Production Warning
Never force-delete a CrashLoopBackOff pod on a StatefulSet without verifying no write operations are in flight. If crash is caused by a missing Secret, the application may have partially initialised state that requires cleanup.

### Senior Insight
> The most common CrashLoopBackOff causes in production, in order of frequency: (1) missing Secret/ConfigMap, (2) misconfigured liveness probe with too-short `initialDelaySeconds`, (3) OOMKill with no container memory limit set, (4) application-level panic on startup. This prompt identifies all four in one pass. If the container dies before logging anything, use an ephemeral debug container (`kubectl debug -it <pod> --image=busybox --target=<container>`) or add a `sleep 3600` init container to inspect the filesystem before the main container starts.

---

---

## Sample 2 — DNS Resolution Failing Inside the Cluster

### Scenario
Applications inside the cluster cannot resolve internal service names or external hostnames. DNS intermittently fails or completely stops working. CoreDNS is running. The cluster is operational. But pods are getting `NXDOMAIN`, `SERVFAIL`, or just timing out on DNS queries. This often looks like an application bug until you realise every pod in the cluster is affected.

### Symptoms
- `nslookup kubernetes.default` fails from inside pods
- External domains also fail or are intermittent
- CoreDNS pods are Running but logs show errors
- Issue is cluster-wide or namespace-scoped

### Required Inputs
- CoreDNS version and replica count
- `kubectl get pods -n kube-system -l k8s-app=kube-dns -o wide`
- CoreDNS logs: `kubectl logs -n kube-system <coredns-pod> --tail=100`
- Corefile: `kubectl describe configmap coredns -n kube-system`
- DNS test results from affected pod (nslookup internal, external, /etc/resolv.conf)
- Whether NodeLocal DNSCache is enabled

### Prompt
```
DNS resolution is broken or intermittent inside the Kubernetes cluster. CoreDNS is running but queries are failing.

Cluster DNS setup:
- DNS provider: {{CoreDNS / kube-dns}}
- CoreDNS version: {{version}}
- Number of CoreDNS replicas: {{N}}

kubectl get pods -n kube-system -l k8s-app=kube-dns -o wide:
{{paste — include NODE placement}}

kubectl logs -n kube-system {{coredns-pod-1}} --tail=100:
{{paste}}

kubectl describe configmap coredns -n kube-system:
{{paste Corefile content}}

Test results from an affected pod:
- nslookup kubernetes.default.svc.cluster.local: {{output}}
- nslookup {{external-domain}}: {{output}}
- cat /etc/resolv.conf: {{output}}

Is NodeLocal DNSCache enabled? {{yes/no}}
Network plugin: {{Calico/Cilium/other}}

Diagnose:
1. Identify the failure mode: NXDOMAIN, SERVFAIL, timeout, or loop (CoreDNS forwarding to itself)
2. If CoreDNS logs show errors: parse them and identify the root cause (upstream forwarder unreachable, Corefile misconfiguration, OOM, or conntrack table exhaustion)
3. If the issue is conntrack table exhaustion: explain the fix and the kernel parameter to change
4. Assess whether NodeLocal DNSCache would eliminate this class of problem for this cluster
5. Provide the immediate mitigation (restart, config patch, or kernel tuning) and the permanent fix
```

### Expected Output
- DNS failure mode classified (NXDOMAIN / SERVFAIL / timeout / loop)
- Root cause from CoreDNS logs with line reference
- Conntrack table exhaustion diagnosis and fix if applicable
- NodeLocal DNSCache recommendation with implementation steps
- Immediate mitigation command + permanent fix

### Verification Steps
1. `kubectl exec <pod> -- nslookup kubernetes.default.svc.cluster.local` must succeed
2. `kubectl exec <pod> -- nslookup google.com` must succeed
3. Verify CoreDNS Prometheus metrics show zero SERVFAIL responses

### ⚠️ Production Warning
Restarting CoreDNS pods during active conntrack exhaustion may cause a brief DNS blackout for all pods in the cluster. Coordinate with on-call before restarting. If enabling NodeLocal DNSCache for the first time, test on one non-production node before cluster-wide rollout.

### Senior Insight
> `conntrack` table exhaustion is the most underdiagnosed DNS failure in production clusters. The symptoms are deceptive: DNS works perfectly under low load, randomly fails under traffic spikes, and CoreDNS logs show no errors at all — because the packets never reach CoreDNS. They are dropped at the kernel level when the conntrack table overflows. Fix: `sysctl -w net.netfilter.nf_conntrack_max=1048576` on each node. Permanent fix: enable NodeLocal DNSCache to route DNS traffic via a local cache that bypasses conntrack entirely.

---

---

## Sample 3 — etcd Slow: The Entire Control Plane Is Sluggish

### Scenario
The Kubernetes control plane is slow. `kubectl` commands take 5–30 seconds. New pods take minutes to schedule. The API server is responding but sluggishly. The root cause is almost always etcd: high write latency, disk I/O saturation, large key-value objects, or leader election churn. This is a severity-1 platform issue — the entire cluster is operating in degraded mode.

### Symptoms
- `kubectl` commands respond slowly (more than 3 seconds)
- Deployments take much longer than normal to roll out
- etcd metrics show high `backend_commit_duration` or `wal_fsync_duration`
- API server logs show timeout waiting for etcd responses

### Required Inputs
- etcd metrics from Prometheus or etcd /metrics endpoint:
  - `etcd_disk_wal_fsync_duration_seconds` p99
  - `etcd_disk_backend_commit_duration_seconds` p99
  - `etcd_server_leader_changes_seen_total`
  - `etcd_mvcc_db_total_size_in_bytes`
- `etcdctl member list` output
- `etcdctl endpoint health --cluster` and `etcdctl endpoint status --cluster`
- API server logs filtered for etcd errors
- Disk type on etcd nodes and auto-compaction configuration

### Prompt
```
The Kubernetes control plane is slow. All kubectl operations are sluggish. I suspect etcd is the root cause.

etcd version: {{version}}
Number of etcd members: {{3 / 5}}
Disk type on etcd nodes: {{SSD / HDD / NVMe / cloud disk type and IOPS spec}}

etcd metrics (from Prometheus or etcd /metrics endpoint):
- etcd_disk_backend_commit_duration_seconds (p99, last 1h): {{value — alert threshold is >25ms}}
- etcd_disk_wal_fsync_duration_seconds (p99, last 1h): {{value — alert threshold is >10ms}}
- etcd_server_leader_changes_seen_total (last 1h): {{value — >2 in 1h is concerning}}
- etcd_mvcc_db_total_size_in_bytes: {{value — alert if >8GB}}
- etcd_server_proposals_failed_total: {{value}}

etcd member list (etcdctl member list):
{{paste}}

etcd endpoint health and latency:
{{paste etcdctl endpoint health --cluster and etcdctl endpoint status --cluster}}

API server logs showing etcd-related errors:
{{paste kubectl logs -n kube-system kube-apiserver-* | grep -i etcd | tail -50}}

Is compaction running?
{{value — e.g. "auto-compaction-retention=1h" or "not configured"}}

Diagnose:
1. Identify the primary etcd bottleneck from the metrics: disk I/O latency (wal_fsync), DB size (compaction needed), leader churn (network instability), or proposal failures (quorum issues)
2. For disk latency: assess whether the disk type meets etcd requirements (NVMe/SSD with < 10ms fsync P99 required)
3. For DB size: calculate if defragmentation is needed (db_size vs. db_size_in_use ratio) and provide the safe online defrag procedure
4. For leader churn: identify network latency or resource contention causes and how to detect them
5. Provide the immediate remediation sequence ordered by impact, and the permanent architectural fix
```

### Expected Output
- Primary bottleneck identified from metrics with threshold comparison
- Disk I/O assessment and whether current storage meets etcd requirements
- Defragmentation procedure if DB size is the issue (online, safe)
- Leader churn diagnosis and network fix
- Immediate + permanent remediation sequence

### Verification Steps
1. wal_fsync p99 below 10ms and backend_commit p99 below 25ms
2. `kubectl` response time below 1 second
3. After defrag: db_size_in_use/db_size ratio above 0.9
4. Leader changes rate is 0: `etcd_server_leader_changes_seen_total`

### ⚠️ Production Warning
Never defragment all etcd members simultaneously — defrag briefly takes the member offline. Defragment one member at a time, waiting for it to rejoin and sync before proceeding. Start with followers, never the leader first.

### Senior Insight
> etcd's number one enemy is disk latency. The `wal_fsync_duration_seconds` P99 must stay below 10ms — etcd calls `fdatasync()` on every write and blocks until the disk confirms. Cloud-managed disks (gp2 EBS, standard persistent disk) frequently burst above this threshold under load. This is a misconfiguration, not a workload problem: cloud disks have burst IOPS that deplete under sustained load and then throttle. Production etcd nodes require local NVMe SSDs or premium cloud disk tiers with guaranteed IOPS. A sluggish control plane that recovers overnight is almost always an IOPS burst depletion pattern.

---

---

## Sample 4 — SLO Breach: Diagnosing an Error Budget Burn Rate Alert

### Scenario
Your error budget burn rate alert fired. You are burning error budget 14x faster than sustainable. You have a 30-day SLO and you have consumed 40% of the budget in 2 hours. The alert tells you something is wrong — but it does not tell you what, where, or how severe. You need to go from "burn rate alert" to "root cause identified" in under 15 minutes.

### Symptoms
- `ErrorBudgetBurnRate > 14` alert firing (fast burn — 1h + 5m windows both above threshold)
- SLO dashboard shows error budget dropping steeply
- No single obvious spike in the error rate dashboard
- Multiple services in the dependency chain

### Required Inputs
- SLO definition: metric, target %, window
- Current burn rate value and error budget remaining
- SLI PromQL query
- Current vs baseline error rate
- Error breakdown by type (last 30 minutes)
- Recent changes in the last 2 hours
- Service dependency map

### Prompt
```
An SLO error budget burn rate alert fired. I need to identify the root cause fast.

Service: {{service-name}}
SLO definition:
- Metric: {{availability (successful requests / total requests) / latency (requests < 200ms / total requests)}}
- Target: {{e.g. 99.9%}}
- Window: {{30 days}}

Current burn rate: {{value — e.g. 14.2x}}
Error budget remaining: {{value — e.g. 61% before this event}}
Estimated time to budget exhaustion at current rate: {{value}}

SLO metric query (the PromQL that defines your SLI):
{{paste}}

Current error rate (rate of bad requests, last 5m): {{value}}
Baseline error rate (last 30d, p50): {{value}}

Service dependency map:
{{describe or paste}}

Recent changes in the last 2 hours:
{{deployments, config changes, traffic spikes, external events}}

Error breakdown by type (HTTP status codes, gRPC codes, or error classes — last 30 minutes):
{{paste from logs or metrics}}

Diagnose this burn event:
1. Calculate: at the current burn rate, what is the exact SLO compliance at the end of the 30-day window if nothing changes? Show the math
2. From the error breakdown: identify the dominant error type and the most likely root cause — distinguish between client errors (4xx), infrastructure errors (5xx, timeout), and dependency errors (upstream returning errors)
3. Triage the dependency chain: is this service generating errors or propagating them from a downstream failure? Provide the exact query to distinguish
4. Define the immediate mitigation threshold: at what error rate must you stabilize to stop burning budget faster than it regenerates?
5. Write the post-burn action: once stabilized, calculate the recovery time and whether the SLO window should be reset
```

### Expected Output
- End-of-window SLO projection with math
- Dominant error type and root cause hypothesis
- Origin vs. propagation diagnosis query
- Maximum sustainable error rate calculation
- Recovery timeline and budget replenishment math

### Verification Steps
1. Burn rate drops below 1x (sustainable) on SLO dashboard
2. Dominant error type identified by prompt matches application logs
3. Maximum sustainable error rate calculation verified by back-calculation against SLO target
4. After stabilization, error budget recovery follows the projected timeline

### ⚠️ Production Warning
Burn rate above 14x means your monthly error budget exhausts in under 2 hours. Do NOT spend time diagnosing root cause before stabilizing: rollback, circuit break, or shed load first. Stabilize, then diagnose. The order matters.

### Senior Insight
> The Google SRE burn rate alert thresholds: fast burn = 14x for 1h window + 5m window (consumes 2% budget in 1h); slow burn = 1x sustained over 3d window (consumes 10% budget in 3 days). Both windows must fire simultaneously to reduce false positives. The key formula: maximum sustainable error rate = (1 - SLO_target) × 100. For a 99.9% SLO, you can sustain at most 0.1% error rate without burning budget. Anything above that is burning. The prompt forces the AI to do this math explicitly, so you arrive at the incident bridge with a number, not a feeling.

---

---

## Sample 5 — OLM Operator Installation Failure (OpenShift-Native)

### Scenario
An Operator installation via OLM (Operator Lifecycle Manager) is stuck or failing. The Subscription exists, but no CSV (ClusterServiceVersion) reaches the Succeeded phase. The operator pods never appear, or they appear and fail. This is one of the most common OpenShift administrative problems — and it is almost always diagnosed incorrectly because engineers look at the wrong resource in the wrong namespace.

### Symptoms
- `oc get subscription -n <namespace>` shows the Subscription but no InstallPlan is created
- InstallPlan exists but stays in `RequiresApproval` or `Failed`
- CSV exists but stays in `Pending` or `Installing`
- Operator pod never appears or enters CrashLoopBackOff

### Required Inputs
- Subscription YAML: `oc get subscription <name> -n <namespace> -o yaml`
- InstallPlan status: `oc get installplan -n <namespace>`
- CSV status: `oc get csv -n <namespace>`
- CatalogSource status: `oc get catalogsource -n openshift-marketplace`
- OLM operator pod logs: `oc logs -n openshift-operator-lifecycle-manager deploy/olm-operator --tail=50`
- Catalog operator logs: `oc logs -n openshift-operator-lifecycle-manager deploy/catalog-operator --tail=50`

### Prompt
```
My OLM operator installation on OpenShift is failing. Here is the diagnostic data:

Subscription (oc get subscription <name> -n <namespace> -o yaml):
{{paste}}

InstallPlan list (oc get installplan -n <namespace> -o yaml):
{{paste}}

CSV status (oc get csv -n <namespace>):
{{paste}}

CatalogSource status (oc get catalogsource -n openshift-marketplace):
{{paste}}

OLM operator logs (last 50 lines):
{{paste oc logs -n openshift-operator-lifecycle-manager deploy/olm-operator --tail=50}}

Catalog operator logs (last 50 lines):
{{paste oc logs -n openshift-operator-lifecycle-manager deploy/catalog-operator --tail=50}}

OpenShift version: {{e.g. 4.14.5}}
Operator being installed: {{name and version}}
Target namespace type: {{all-namespaces / single-namespace / own-namespace}}

Diagnose:
1. Identify the failure layer: CatalogSource unreachable (image pull failure, network), InstallPlan approval required (manual approval mode), CSV dependency resolution failure (required API not available), or RBAC preventing OLM from creating resources
2. For CatalogSource failures: identify if the catalog pod is running and the image is accessible from the cluster (air-gapped environments require a mirrored catalog)
3. For CSV dependency failures: identify which required CRD or API is missing and why (missing prerequisite operator, wrong channel)
4. Provide the exact sequence of commands to advance the installation from current state to Succeeded
5. Identify if this is a supported operator for this OpenShift version (check the operator's supported OCP version range in the CSV)
```

### Expected Output
- Failure layer identified (CatalogSource / InstallPlan / CSV dependency / RBAC)
- Specific cause with evidence from the pasted data
- Ordered command sequence to advance to Succeeded state
- Version compatibility assessment

### Verification Steps
1. `oc get csv -n <namespace>` shows the CSV in `Succeeded` phase
2. Operator pod(s) are Running: `oc get pods -n <namespace>`
3. CRDs introduced by the operator are present: `oc get crd | grep <operator-domain>`
4. A test instance of the operator's CRD can be created successfully

### ⚠️ Production Warning
Never approve an InstallPlan for a major version upgrade without reading the operator's upgrade documentation. Major operator upgrades may migrate CRD schemas in ways that break existing custom resources. Always test on non-production first.

### Senior Insight
> When diagnosing OLM issues, the correct inspection order is: CatalogSource → InstallPlan → CSV → operator pod. Engineers almost always start with the operator pod and miss the upstream failure. A CatalogSource that cannot pull its image will silently block all operators in the channel — one failure can prevent multiple operators from installing. In air-gapped OpenShift environments, every CatalogSource image must be mirrored to the internal registry. Check `oc get catalogsource -n openshift-marketplace` for pods in `CrashLoopBackOff` or `ImagePullBackOff` before looking at anything else.

---

---

## Want all 37 prompts?

The full pack includes:

- All **25 Core Prompts** covering Deployment, Networking, Performance, Security, and Observability
- **7 Bonus Prompts** (post-deployment regression, JVM GC, cluster autoscaler, RBAC, Prometheus cardinality, and more)
- **5 OpenShift-Native Prompts** (OLM, MachineConfig, Routes, OAuth, ClusterOperators)
- Complete Required Inputs, Verification Steps, and Production Warnings for every prompt
- Index, Diagnostic Safety Rules, and Troubleshooting Methodology

**[Get the full pack on Gumroad](https://phoenixplatform.gumroad.com/)**

---

*Phoenix Labs · LAB-001 · DevOps AI Prompt Pack v1.0 · Free Sample*
