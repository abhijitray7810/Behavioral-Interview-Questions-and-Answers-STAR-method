### self-introduction
Hi, I’m Abhijit Ray from West Bengal, India
I started my journey in software development with frontend development and UI/UX design, but as I worked on projects, I became increasingly interested in backend systems, Linux, automation, and how large-scale systems stay reliable. That curiosity gradually led me toward DevOps and Site Reliability Engineering.

Currently, I’m focused on building my skills in areas like automation, Kubernetes, monitoring, and distributed systems, while also improving my software engineering fundamentals such as coding and problem-solving. I really enjoy solving system-level problems and finding ways to improve reliability through engineering.

Looking ahead, I want to grow into an SRE-SWE, DevOps, FDE role where I can work on large-scale infrastructure, improve system reliability, and contribute through automation. That’s one of the reasons I’m excited about Google, because of the scale of the systems, the engineering culture, and the strong focus on reliability through software.
# 🚀 SRE + SWE Mock Interview Guide — Abhijit Ray
 
> **Google SRE Book Aligned · 55 Questions · STAR Method · Fresher → Advanced**
> Based on [Google SRE Book](https://sre.google/sre-book/table-of-contents/) | Projects: Aegis Stack · PodPlate · RoutineOps

---

## 📋 Table of Contents

- [Overview](#overview)
- [Topic Coverage](#topic-coverage)
- [Part 1 — SRE Principles](#part-1--sre-principles)
- [Part 2 — SWE / Coding](#part-2--swe--coding)
- [Part 3 — System Design](#part-3--system-design)
- [Part 4 — Security / DevSecOps](#part-4--security--devsecops)
- [Part 5 — Behavioural (STAR)](#part-5--behavioural-star)
- [How to Use This Guide](#how-to-use-this-guide) 
- [Quick Reference — Key Formulas & Commands](#quick-reference--key-formulas--commands)

---

## Overview

| Stat | Value |
|------|-------|
| Total Questions | 55 |
| Difficulty Levels | Fresher · Mid-level · Advanced |
| Answer Format | Model Answer + STAR Method |
| SRE Book Chapters Covered | Ch. 3–15, 21–24, 32 |
| Resume Projects Referenced | Aegis Stack · PodPlate · RoutineOps |

---

## Topic Coverage

| # | Category | Questions | SRE Book Chapter |
|---|----------|-----------|-----------------|
| 1 | SRE Principles | 15 | Ch. 3, 4, 5, 6, 7, 8, 11, 12, 14, 15, 22 |
| 2 | SWE / Coding | 10 | — |
| 3 | System Design | 10 | Ch. 19, 21, 22, 23, 24 |
| 4 | Security / DevSecOps | 8 | Ch. 17 |
| 5 | Behavioural | 7 | Ch. 29, 31 |
| 6 | Advanced / Mixed | 5 | Ch. 32 |

---

## Part 1 — SRE Principles

---

### Q1 · Fresher · What is SRE and how does it differ from traditional DevOps?

> **SRE Book Reference:** Chapter 1 — Introduction

**Model Answer:**

SRE (Site Reliability Engineering), coined at Google, applies software engineering principles to operations. The core difference: DevOps is a cultural philosophy bridging dev and ops; SRE is a concrete job function with specific practices — error budgets, SLOs, toil elimination. SREs write code to automate away operational work rather than doing it manually.

**STAR Answer:**

| | |
|--|--|
| **Situation** | Every SRE interview opens here — baseline understanding is assumed. |
| **Task** | Define SRE, contrast with DevOps, tie to Google SRE Book. |
| **Action** | In my Aegis Stack and PodPlate projects I operated as a one-person SRE — defining SLIs for latency/error rate, automating deployments with ArgoCD (GitOps), and eliminating manual toil via Tekton pipelines. |
| **Result** | Deployments automated end-to-end with zero manual gate-keeping — the SRE ideal. |

> 💡 **Interviewer Tip:** Always cite the 50% cap rule — Google mandates SREs spend ≤50% time on ops/toil.

---

### Q2 · Fresher · Explain SLI, SLO, and SLA with concrete examples.

> **SRE Book Reference:** Chapter 4 — Service Level Objectives

**Model Answer:**

- **SLI** (Service Level Indicator) = a carefully defined quantitative measure, e.g. request success rate.
- **SLO** (Service Level Objective) = a target for the SLI, e.g. 99.9% success rate over a 30-day rolling window.
- **SLA** (Service Level Agreement) = contractual commitment to customers with financial consequences if breached.

SLOs are always tighter than SLAs.

**STAR Answer:**

| | |
|--|--|
| **Situation** | Needed to prove service reliability on PodPlate. |
| **Task** | Define and instrument SLIs → set SLOs → alert before burn-rate crosses threshold. |
| **Action** | Deployed Prometheus to track 15+ custom SLIs including order-latency p99, error rate, and pod restart count. Set SLOs of 99.9% uptime and p99 < 200ms. Grafana dashboards surfaced burn-rate alerts. |
| **Result** | MTTR reduced by 50%, demonstrated by Grafana incident timelines. |

> 💡 **Interviewer Tip:** Name a specific SLI from your project, not just a textbook definition.

---

### Q3 · Mid-level · What is an error budget and how would you use it to make a deployment decision?

> **SRE Book Reference:** Chapter 3 — Embracing Risk

**Model Answer:**

Error budget = `1 – SLO`. If SLO is 99.9%, the error budget is 0.1% of requests allowed to fail per window (~43 min/month).

- ✅ Budget healthy → dev teams can push risky releases.
- ❌ Budget exhausted → all changes freeze until reliability recovers.

It aligns dev and ops incentives mathematically.

**STAR Answer:**

| | |
|--|--|
| **Situation** | PodPlate had a hard 99.9% uptime SLO commitment. |
| **Task** | Define error budget, wire it to a deployment gate. |
| **Action** | Calculated monthly error budget from Prometheus data. Integrated a pre-deployment check in Tekton that queried current burn rate — if budget consumption exceeded 80%, the pipeline halted new deployments automatically. |
| **Result** | Eliminated the heated debate between "we need to ship" and "prod is fragile" — the data decided. |

> 💡 **Interviewer Tip:** Error budgets resolve the dev vs ops conflict without politics — interviewers love hearing this.

---

### Q4 · Fresher · Define 'toil' in SRE terms. Give an example and explain how you eliminated it.

> **SRE Book Reference:** Chapter 5 — Eliminating Toil

**Model Answer:**

**Toil** = manual, repetitive, automatable operational work that scales with service growth and produces no enduring value.

Examples of toil:
- Manually restarting crashed pods
- Rotating secrets by hand
- Running the same `kubectl` commands for every deploy

Toil is capped at **50% of SRE time** at Google.

**STAR Answer:**

| | |
|--|--|
| **Situation** | RoutineOps environment provisioning took days of manual AWS console clicking. |
| **Task** | Eliminate provisioning toil via IaC. |
| **Action** | Wrote Terraform modules for VPC, EKS, IAM, subnets. Environment now spins up with a single `terraform apply`. |
| **Result** | Provisioning time dropped from **days → 20 minutes** — a 98%+ reduction in toil. |

> 💡 **Interviewer Tip:** Always quantify the before/after. "Days to 20 minutes" is far more impactful than "I automated it."

---

### Q5 · Mid-level · Walk me through the four golden signals of monitoring.

> **SRE Book Reference:** Chapter 6 — Monitoring Distributed Systems

**Model Answer:**

| Signal | Definition | Example |
|--------|-----------|---------|
| **Latency** | Time to serve a request | p99 request duration; track success vs error latency separately |
| **Traffic** | Demand on the system | Requests per second (RPS / QPS) |
| **Errors** | Rate of failed requests | HTTP 5xx rate; implicit errors (200 with wrong body) |
| **Saturation** | How "full" the system is | CPU %, memory %, queue depth — predicts problems before errors |

**STAR Answer:**

| | |
|--|--|
| **Situation** | PodPlate microservices had opaque failure modes. |
| **Task** | Instrument all four golden signals across every service. |
| **Action** | Configured Prometheus scrape configs, wrote custom PromQL for each signal, built Grafana dashboards with per-service panels and Alertmanager rules triggering on error spikes OR saturation crossing 80%. |
| **Result** | Reduced unplanned outages by **60%** through proactive alerting before user-visible impact. |

> 💡 **Interviewer Tip:** In system design rounds, always mention all four signals when asked about observability.

---

### Q6 · Advanced · How do you handle cascading failures in a Kubernetes microservices platform?

> **SRE Book Reference:** Chapter 22 — Addressing Cascading Failures

**Model Answer:**

Cascading failures happen when a struggling downstream service causes upstream callers to queue, exhaust thread pools, and fail too.

**Prevention strategies:**

```
Circuit Breaker    → open on error threshold, stop sending traffic to failing service
Timeout Budgets    → every RPC has a deadline; never wait indefinitely
Load Shedding      → reject requests with HTTP 429 when overloaded
Bulkheads          → isolate thread pools per dependency
Retry with Jitter  → exponential backoff + randomness prevents thundering herd
```

**STAR Answer:**

| | |
|--|--|
| **Situation** | PodPlate's order service could be hammered during peak traffic, threatening the whole mesh. |
| **Task** | Implement cascade failure protection at the service mesh layer. |
| **Action** | Configured Istio circuit breakers (outlier detection), set request timeouts on all VirtualService routes, deployed HPA so pods scaled before saturation hit. Added Grafana dashboards for circuit-open events. |
| **Result** | Reduced inter-service latency by **35%** and achieved zero-downtime deployments under variable load. |

> 💡 **Interviewer Tip:** Mention the "thundering herd" variant — cached data expiry causing simultaneous stampede — and how jitter fixes it.

---

### Q7 · Advanced · Describe how you would design and execute a postmortem after a production outage.

> **SRE Book Reference:** Chapter 15 — Postmortem Culture: Learning from Failure

**Model Answer:**

A **blameless postmortem** focuses on system/process failure, not people.

**Structure:**
1. Incident timeline with precise timestamps
2. Root cause analysis (5 Whys or Fishbone diagram)
3. Impact assessment — users affected, SLO burn consumed
4. Action items — each with an owner and deadline
5. Publish internally so all teams benefit from the learning

**STAR Answer:**

| | |
|--|--|
| **Situation** | A Falco rule misconfiguration caused alert storms, exhausting on-call capacity. |
| **Task** | Run a structured blameless postmortem to prevent recurrence. |
| **Action** | Documented the 30-minute incident timeline from Loki logs. Used 5-Whys: misconfigured Falco rule → no review gate in CI → no OPA policy for Falco configs. Action items: add Conftest policy for Falco rules, add staging canary for security config changes. |
| **Result** | Zero recurrence of that class of misconfiguration in 6+ months. |

> 💡 **Interviewer Tip:** Say "blameless" and state specific action items with owners — never vague "we'll do better."

---

### Q8 · Mid-level · What is the difference between black-box and white-box monitoring?

> **SRE Book Reference:** Chapter 6 — Monitoring Distributed Systems

**Model Answer:**

| Type | What it monitors | Best for |
|------|-----------------|----------|
| **Black-box** | External behaviour (probing endpoints, synthetic transactions) | User-visible symptoms, SLO compliance |
| **White-box** | Internal signals (JVM heap, DB pool usage, goroutine count) | Predicting problems, root cause analysis |

**Rule:** Alert on symptoms (black-box). Diagnose with internals (white-box).

**STAR Answer:**

| | |
|--|--|
| **Situation** | PodPlate needed both user-facing SLO assurance and internal capacity planning. |
| **Task** | Combine Prometheus (white-box) with external health probes (black-box). |
| **Action** | Kubernetes readiness/liveness probes as black-box checks. Prometheus scraped internal metrics (queue depth, DB pool usage) as white-box. Alertmanager triggered on error rate (black-box symptom) while dashboards showed internal saturation for RCA. |
| **Result** | On-call could see both the symptom and the likely cause simultaneously — faster MTTR. |

> 💡 **Interviewer Tip:** Classic mistake — alerting on CPU (white-box cause) instead of error rate (black-box symptom). Interviewers test this.

---

### Q9 · Fresher · What is a runbook and why is it important in SRE?

> **SRE Book Reference:** Chapter 12 — Effective Troubleshooting

**Model Answer:**

A **runbook** is a documented, step-by-step procedure for responding to a specific alert or incident.

**A good runbook contains:**
- Alert description and impact
- Diagnostic commands to run
- Remediation steps
- Escalation path

**STAR Answer:**

| | |
|--|--|
| **Situation** | New engineers joining RoutineOps on-call had no documented procedures. |
| **Task** | Create runbooks linked to every Prometheus alert. |
| **Action** | Wrote runbooks in Markdown (stored in Git) for every alert rule — pod crash-loop, high memory, certificate expiry. Linked each Alertmanager rule's `runbook_url` annotation to the corresponding doc. |
| **Result** | On-call handover time dropped; new engineers handled P2 incidents without escalation on first rotation. |

> 💡 **Interviewer Tip:** Runbooks you run weekly should become scripts — that's the SRE automation ladder.

---

### Q10 · Advanced · How does distributed consensus (Raft/Paxos) affect reliability in systems you've operated?

> **SRE Book Reference:** Chapter 23 — Managing Critical State: Distributed Consensus

**Model Answer:**

Distributed consensus algorithms (Raft, Paxos) ensure a quorum of nodes agree on state before committing.

```
etcd (Kubernetes control plane) uses Raft:
  3-node cluster → tolerates 1 failure
  5-node cluster → tolerates 2 failures

Risks:
  - Network partitions → split-brain or leader election storms
  - etcd quorum loss → Kubernetes API server unavailable
```

**STAR Answer:**

| | |
|--|--|
| **Situation** | EKS control plane in Aegis Stack relied on etcd for all cluster state. |
| **Task** | Understand and monitor consensus layer health. |
| **Action** | Added Prometheus metrics for etcd leader elections, proposal failures, and fsync latency. Set Alertmanager rules for etcd quorum loss. Documented recovery runbook for etcd snapshot restore. |
| **Result** | Avoided a potential split-brain situation during AZ instability by detecting etcd heartbeat timeouts early. |

> 💡 **Interviewer Tip:** Even if you haven't operated raw etcd, mentioning Kubernetes' dependency on it shows depth.

---

### Q37 · Mid-level · What is the difference between latency percentiles (p50, p95, p99)?

**Model Answer:**

| Percentile | Meaning |
|-----------|---------|
| **p50** | Median — 50% of requests are faster |
| **p95** | 95% of requests are faster; worst 5% at or above this |
| **p99** | 99% of requests are faster; only 1% at or above this |

> **"Averages lie. Percentiles tell the truth."**

Average hides outliers — a service with 99% fast requests and 1% 30-second timeouts might show an "average" of 400ms while users experience 30s hangs.

**STAR Answer:**

| | |
|--|--|
| **Situation** | PodPlate's average order API latency looked healthy at 180ms but users complained of slowness. |
| **Task** | Switch SLOs from average to p99 percentile tracking. |
| **Action** | Changed Prometheus queries from `avg()` to `histogram_quantile(0.99, ...)`. Discovered p99 was 3.2 seconds — database connection pool exhaustion under load. Fixed by tuning pool size and adding circuit breaker. |
| **Result** | p99 dropped from **3.2s → 190ms**. User complaints stopped. Average alone would have hidden this completely. |

---

### Q42 · Mid-level · What is a canary deployment and how does it reduce production risk?

> **SRE Book Reference:** Chapter 8 — Release Engineering

**Model Answer:**

**Canary deployment:** route a small percentage of real traffic (e.g. 5%) to the new version while 95% stays on stable.

```
Traffic routing:
  95% → stable (v1)
   5% → canary (v2)

Monitor for:
  - Error rate increase
  - p99 latency degradation

Decision:
  ✅ Healthy → increment traffic 5% → 20% → 50% → 100%
  ❌ Degraded → route all traffic back (30-second rollback)
```

**STAR Answer:**

| | |
|--|--|
| **Situation** | PodPlate needed to ship new checkout logic without risking all users on a potential regression. |
| **Task** | Implement canary via Istio traffic management. |
| **Action** | Used Istio VirtualService with weight-based routing: 95% stable / 5% canary. Prometheus compared error rate and p99 latency across both versions. ArgoCD Rollouts automated weight increment (5%→20%→50%→100%) based on Prometheus analysis template. |
| **Result** | Caught a DB query regression in canary at 5% traffic before it reached all users — rollback in 30 seconds. |

> 💡 **Interviewer Tip:** ArgoCD Rollouts with Prometheus analysis is the modern automated canary pattern.

---

### Q35 · Advanced · Explain exponential backoff with jitter. Why is plain exponential backoff insufficient?

> **SRE Book Reference:** Chapter 22 — Addressing Cascading Failures

**Model Answer:**

```python
# Plain exponential backoff — PROBLEM: all clients retry simultaneously
sleep = base * 2^attempt   # 1s, 2s, 4s, 8s...

# Full jitter — SOLUTION: randomized to break synchronization
sleep = random(0, min(cap, base * 2^attempt))
```

**Why jitter matters:** If N clients all hit the same error simultaneously (server restart), they all backoff identically and retry at the same moment — **thundering herd** — creating a traffic spike that kills the recovering server.

**STAR Answer:**

| | |
|--|--|
| **Situation** | PodPlate's order service retries caused a 10× traffic spike when the payment gateway restarted. |
| **Task** | Implement full jitter in retry logic. |
| **Action** | Implemented full jitter in Python requests retry logic using a custom `HTTPAdapter` subclass with `random.uniform(0, min(cap, base * 2**attempt))`. |
| **Result** | Retry-induced traffic spikes eliminated — payment gateway recovery time improved by 60%. |

---

### Q45 · Advanced · How would you design a Prometheus alerting system that avoids alert fatigue?

> **SRE Book Reference:** Chapter 10 — Practical Alerting

**Model Answer:**

**Principles:**
1. Alert on **symptoms**, not causes (error rate, not CPU)
2. Every alert must be **actionable** — if no one knows what to do, delete it
3. Use **multi-window multi-burn-rate** alerts (Google SRE Workbook):
   - Fast burn: 1h window, 14× budget → catches acute outages
   - Slow burn: 6h window, 6× budget → catches slow leaks
4. **Inhibition rules:** suppress downstream alerts when root cause alert fires
5. **Dead man's snitch** for monitoring-itself-is-down detection

**STAR Answer:**

| | |
|--|--|
| **Situation** | PodPlate's initial Alertmanager config had 200+ rules — on-call receiving 50+ alerts/day, most noise. |
| **Task** | Redesign alerting to be actionable and minimize noise. |
| **Action** | Audited all 200 rules. Deleted 120 with no runbook. Converted remaining to symptom-based (error rate, SLO burn rate). Implemented multi-window burn rate alerts from Google SRE Workbook. Added inhibition to suppress pod-level alerts when node-level alert fired. |
| **Result** | Alert volume dropped **85%**. MTTR improved 50%. On-call went from exhaustion to confident response. |

---

### Q50 · Mid-level · How does Kubernetes HPA work? What are its limitations?

**Model Answer:**

HPA (Horizontal Pod Autoscaler) scales pod replicas based on observed metrics vs target.

```yaml
# CPU-based HPA (basic)
metrics:
- type: Resource
  resource:
    name: cpu
    target:
      type: Utilization
      averageUtilization: 70

# Custom metric HPA (advanced — preferred)
metrics:
- type: Pods
  pods:
    metric:
      name: requests_per_second
    target:
      type: AverageValue
      averageValue: "100"
```

**Limitations:**

| Limitation | Workaround |
|-----------|------------|
| Can't scale to zero | Use **KEDA** for event-driven scale-to-zero |
| Scale-down is slow (5min default) | Tune `stabilizationWindowSeconds` |
| CPU-based scaling reacts late | Scale on **RPS or queue depth** instead |

**STAR Answer:**

| | |
|--|--|
| **Situation** | PodPlate experienced 5-minute latency spikes when traffic surged before HPA added pods. |
| **Task** | Improve HPA responsiveness using custom metrics. |
| **Action** | Switched to custom metric HPA: Prometheus Adapter exposed `requests_per_second` from Istio telemetry. Added KEDA for scale-to-zero on non-peak hours. Reduced `scaleDown stabilizationWindowSeconds` from 300s → 60s. |
| **Result** | Scale-up now triggers before CPU saturates. **40% cost reduction** via overnight scale-to-zero. |

---

## Part 2 — SWE / Coding

---

### Q16 · Fresher · Write a function to find if a string has all unique characters.

**Model Answer:**

```python
# Solution 1: Hash set — O(n) time, O(min(n, charset)) space
def is_unique_set(s: str) -> bool:
    return len(s) == len(set(s))

# Solution 2: Bitmask — O(n) time, O(1) space (lowercase a-z only)
def is_unique_bitmask(s: str) -> bool:
    if len(s) > 26:          # Pigeonhole principle
        return False
    mask = 0
    for c in s:
        bit = 1 << (ord(c) - ord('a'))
        if mask & bit:
            return False
        mask |= bit
    return True

# Time: O(n) | Space: O(1) with bitmask
```

> 💡 **Always state the assumption:** lowercase only, ASCII, Unicode?

---

### Q18 · Mid-level · What is a deadlock? Write an example and explain how to prevent it.

**Model Answer:**

```python
import threading

# DEADLOCK EXAMPLE
lock_x = threading.Lock()
lock_y = threading.Lock()

def thread_a():
    lock_x.acquire()   # holds X
    lock_y.acquire()   # waits for Y → DEADLOCK

def thread_b():
    lock_y.acquire()   # holds Y
    lock_x.acquire()   # waits for X → DEADLOCK

# PREVENTION: Global lock ordering — always acquire X before Y
def thread_safe_a():
    with lock_x:
        with lock_y:
            pass  # safe

def thread_safe_b():
    with lock_x:       # same order as thread_a
        with lock_y:
            pass  # safe
```

**Prevention strategies:**
1. **Lock ordering** — always acquire locks in the same global order
2. **Trylock with timeout** — abandon and retry if lock not acquired within T ms
3. **Lock-free data structures** — atomic CAS operations
4. **Go channels** — communicate rather than share state

> 💡 **Tooling:** `go run -race` (Go) or `ThreadMXBean.findDeadlockedThreads()` (Java)

---

### Q20 · Mid-level · Write a function to reverse a linked list in-place.

**Model Answer:**

```python
class ListNode:
    def __init__(self, val=0, next=None):
        self.val = val
        self.next = next

# Iterative — O(n) time, O(1) space (PREFERRED)
def reverse_list(head: ListNode) -> ListNode:
    prev = None
    curr = head
    while curr:
        next_node = curr.next   # save next
        curr.next = prev        # reverse pointer
        prev = curr             # advance prev
        curr = next_node        # advance curr
    return prev

# Edge cases to test:
# [] → None
# [1] → [1]
# [1,2] → [2,1]
# [1,2,3] → [3,2,1]
```

> 💡 **Interview tip:** Draw the pointer arrows on a whiteboard before coding.

---

### Q21 · Advanced · Design a thread-safe LRU cache with O(1) get and put.

**Model Answer:**

```python
from collections import OrderedDict
import threading

class LRUCache:
    def __init__(self, capacity: int):
        self.capacity = capacity
        self.cache = OrderedDict()   # maintains insertion order
        self.lock = threading.Lock() # thread safety

    def get(self, key: int) -> int:
        with self.lock:
            if key not in self.cache:
                return -1
            self.cache.move_to_end(key)  # mark as recently used
            return self.cache[key]

    def put(self, key: int, value: int) -> None:
        with self.lock:
            if key in self.cache:
                self.cache.move_to_end(key)
            self.cache[key] = value
            if len(self.cache) > self.capacity:
                self.cache.popitem(last=False)  # evict LRU (first item)

# Data structures:
#   HashMap (OrderedDict) → O(1) lookup
#   Doubly Linked List (implicit in OrderedDict) → O(1) move/evict
# Thread safety: sync.RWMutex in Go, synchronized in Java
```

> 💡 **This exact problem appears in Google L4/L5 interviews.** Practice until you can code it in 15 minutes.

---

### Q47 · Mid-level · How does Git rebase differ from merge in a GitOps workflow?

**Model Answer:**

```
Merge (creates merge commit):
  main: A--B--C--M
                  \
  feature:   D--E--/
  Preserves full history ✅ | Non-destructive ✅

Rebase (replays commits, rewrites SHAs):
  main: A--B--C--D'--E'
  feature was: A--B--D--E
  Linear history ✅ | Rewrites SHAs ⚠️
```

**GitOps rule:**
- ✅ Use **merge** for PRs to main — preserves audit trail of exactly when a change entered the deployment branch
- ✅ Use **rebase** locally to clean up WIP commits before opening a PR
- ❌ Never force-push to main — ArgoCD's sync history must remain intact

> 💡 **"The Git history is the audit trail"** — say this phrase in the interview.

---

### Q44 · Advanced · What is a memory leak? How would you detect one in a containerised Go application?

**Model Answer:**

```bash
# Step 1: Watch memory growth in Grafana (container RSS growing unboundedly)

# Step 2: Enable pprof in Go code
import _ "net/http/pprof"
go http.ListenAndServe(":6060", nil)

# Step 3: Capture heap profiles
curl http://pod-ip:6060/debug/pprof/heap > heap_t0.prof
# wait 24 hours
curl http://pod-ip:6060/debug/pprof/heap > heap_t24.prof

# Step 4: Diff profiles to find leak
go tool pprof -diff_base heap_t0.prof heap_t24.prof
# look for growing allocations

# Common Go leak: goroutine never terminated
# Fix: always check ctx.Done() in goroutines
func worker(ctx context.Context) {
    for {
        select {
        case <-ctx.Done():
            return  // ← this prevents the leak
        case work := <-workChan:
            process(work)
        }
    }
}
```

> 💡 **Add Prometheus metric:** `go_goroutines` with Alertmanager rule for count > 1000.

---

## Part 3 — System Design

---

### Q11 · Fresher · Design a URL shortener like bit.ly.

**Model Answer:**

```
Architecture:

Write Path:
  Client → API Server → Generate short code → Store in DB → Return short URL
  Short code: base62(hash(longURL)) or base62(autoincrement_id)

Read Path (HOT):
  Client → CDN edge → Redis cache (LRU) → API Server → DB → 301 redirect
  Cache hit rate target: 99%

Storage estimate:
  100B URLs × 500 bytes = 50TB → use S3 + tiered archiving

Scale:
  - Read replicas for DB (read-heavy: 100:1 read:write ratio)
  - Redis cluster for hot URL caching
  - CDN edge caching for most popular 0.1% of URLs (serve 90% of traffic)
```

> 💡 **Always start with:** "Let me clarify requirements — read vs write ratio, expected QPS, URL expiry?"

---

### Q12 · Mid-level · Design a highly available Kubernetes platform on AWS.

**Model Answer:**

```
Multi-AZ Architecture:

                    ┌─────────────────────┐
                    │   Route53 (DNS)     │
                    └──────────┬──────────┘
                               │
                    ┌──────────▼──────────┐
                    │   ALB Ingress       │
                    └──────────┬──────────┘
                               │
        ┌──────────────────────┼──────────────────────┐
        │                      │                      │
   ┌────▼────┐           ┌─────▼─────┐          ┌────▼────┐
   │  AZ-1a  │           │   AZ-1b   │          │  AZ-1c  │
   │ EKS NG  │           │  EKS NG   │          │  EKS NG │
   └────┬────┘           └─────┬─────┘          └────┬────┘
        └──────────────────────┼──────────────────────┘
                               │
                    ┌──────────▼──────────┐
                    │  RDS Multi-AZ       │
                    │  ElastiCache Redis  │
                    └─────────────────────┘

Key components:
  - EKS multi-AZ node groups + Cluster Autoscaler
  - Istio service mesh (mTLS + traffic management)
  - ArgoCD GitOps (declarative cluster state)
  - PodDisruptionBudgets (min 1 replica during drain)
  - IRSA (per-pod AWS permissions — no shared IAM users)
  - cert-manager (automated TLS certificate rotation)
```

---

### Q14 · Mid-level · Explain the CAP theorem and how it applies to database selection.

**Model Answer:**

```
CAP Theorem: pick 2 of 3
  C = Consistency   (all nodes see same data)
  A = Availability  (every request gets a response)
  P = Partition tolerance (works despite network splits)

Since network partitions happen → real choice is CP vs AP:

  CP systems (consistent, may be briefly unavailable):
    etcd, ZooKeeper, HBase, Google Spanner
    Use for: secrets (Vault), distributed locks, Kubernetes state

  AP systems (available, may return stale data):
    DynamoDB, Cassandra, CouchDB
    Use for: user sessions, shopping carts, DNS
```

**STAR Answer:**

| | |
|--|--|
| **Situation** | Aegis Stack needed reliable secret storage — serving a stale/revoked secret is catastrophic. |
| **Task** | Apply CAP reasoning to HashiCorp Vault selection. |
| **Action** | Chose Vault (CP system using Raft storage backend) over Consul KV — brief unavailability during leader election is acceptable; inconsistency is not. |
| **Result** | Zero cases of stale secrets served; Vault HA with Raft maintained quorum through single-AZ failure. |

---

### Q15 · Advanced · How would you design a rate limiter? Token bucket vs leaky bucket.

**Model Answer:**

```python
# Token Bucket — allows bursts (RECOMMENDED for APIs)
# Redis atomic Lua implementation:

RATE_LIMIT_SCRIPT = """
local key = KEYS[1]
local limit = tonumber(ARGV[1])
local window = tonumber(ARGV[2])

local count = redis.call('INCR', key)
if count == 1 then
    redis.call('EXPIRE', key, window)
end
if count > limit then
    return 0  -- rejected
end
return 1  -- allowed
"""

# Call atomically — the Lua script is the key insight
# Prevents TOCTOU race between GET and INCR
result = r.eval(RATE_LIMIT_SCRIPT, 1, user_key, limit, window_seconds)

# Token Bucket:  allows bursts up to bucket capacity
# Leaky Bucket:  smooths traffic at fixed drain rate, no bursts
# Fixed Window:  simple but boundary attack (2× limit at window edge)
# Sliding Log:   accurate but high memory (store every request timestamp)
```

> 💡 **Interviewer Tip:** The Lua atomicity insight separates junior from senior candidates.

---

### Q13 · Advanced · How does consistent hashing work?

**Model Answer:**

```
Problem with naive modulo hashing (servers = N):
  key % N → when N changes, almost ALL keys remap → massive cache invalidation

Consistent Hashing solution:
  1. Map both servers and keys onto a virtual ring (hash space 0 → 2^32)
  2. Each key maps to the first server clockwise from its position
  3. Adding/removing a server → only O(K/N) keys migrate (vs O(K))

Virtual nodes:
  Each physical server gets V virtual positions on the ring
  Fixes uneven distribution when server count is small
  Redis Cluster uses 16,384 hash slots (consistent hashing variant)

Real-world usage:
  Redis Cluster, Cassandra token ring, CDN edge selection, AWS DynamoDB
```

---

## Part 4 — Security / DevSecOps

---

### Q23 · Fresher · What is the principle of least privilege and how did you implement it in AWS?

**Model Answer:**

Every identity (user, role, service) should have only the exact permissions needed and nothing more.

```json
// BAD — wildcard permissions
{
  "Effect": "Allow",
  "Action": "s3:*",
  "Resource": "*"
}

// GOOD — resource-scoped, action-specific
{
  "Effect": "Allow",
  "Action": ["s3:GetObject", "s3:PutObject"],
  "Resource": "arn:aws:s3:::my-bucket/prefix/*",
  "Condition": {
    "StringEquals": {"aws:RequestedRegion": "us-east-1"}
  }
}
```

**IRSA (IAM Roles for Service Accounts):** Each Kubernetes pod gets its own AWS IAM role via OIDC federation — no shared IAM users, no access keys in env vars.

> 💡 **Mention IAM Access Analyzer** — reads CloudTrail logs and suggests tighter policies based on actual usage.

---

### Q24 · Mid-level · Explain how OPA Gatekeeper works. Give an example policy you wrote.

**Model Answer:**

```
Flow:
  kubectl apply → API Server → Validating Webhook → OPA Gatekeeper
                                                          ↓
                                              Evaluate Rego policies
                                                          ↓
                                      ✅ Compliant → Admit to cluster
                                      ❌ Violation → Reject with error message

Key resources:
  ConstraintTemplate → defines the Rego policy logic
  Constraint          → applies the template with specific parameters
```

**Example — block privileged containers:**

```rego
package k8sdenyprivileged

violation[{"msg": msg}] {
  container := input.review.object.spec.containers[_]
  container.securityContext.privileged == true
  msg := sprintf("Privileged container not allowed: %v", [container.name])
}
```

**STAR Answer:**

| | |
|--|--|
| **Situation** | Aegis Stack needed to enforce security standards uniformly across all workloads. |
| **Task** | Write OPA Gatekeeper policies blocking non-compliant workloads at admission time. |
| **Action** | Wrote ConstraintTemplates blocking: privileged containers, root containers, untrusted registries, missing resource limits. Added `conftest test` in CI to catch violations before cluster submission. |
| **Result** | Blocked **100% of non-compliant workloads** at admission control. |

---

### Q25 · Mid-level · What is a supply chain attack? How does your CI/CD pipeline defend against it?

**Model Answer:**

```
Supply Chain Attack Surface:
  Source Code → Dependencies → Build Tools → CI Runner → Registry → Deployment

Defense — 7-Stage Pipeline (Aegis Stack):

  Stage 1: Code lint + SAST
  Stage 2: Unit tests
  Stage 3: Trivy CRITICAL CVE scan ← fail on any CRITICAL
  Stage 4: Syft SBOM generation + attestation
  Stage 5: Cosign image signing (Sigstore keyless via OIDC)
  Stage 6: Conftest policy validation
  Stage 7: ArgoCD deploy (Kyverno verifies signature at admission)

Key tools:
  Trivy  → CVE/vulnerability scanning
  Syft   → SBOM generation (SPDX / CycloneDX format)
  Cosign → image signing (Sigstore keyless signing)
  OPA    → policy validation
```

> 💡 **Sigstore keyless signing** (OIDC from GitHub Actions) is the modern approach.

---

### Q26 · Advanced · What is Falco and how does runtime threat detection complement admission control?

**Model Answer:**

```
Defense in Depth Layers:

  Layer 1 — PREVENT:  OPA Gatekeeper / Kyverno (admission control)
                       Blocks known-bad configs at deploy time
                       
  Layer 2 — DETECT:   Falco (runtime threat detection)
                       Monitors kernel syscalls via eBPF
                       Catches zero-day exploits and runtime drift
                       
  Layer 3 — PROTECT:  HashiCorp Vault (secrets management)
                       No plaintext secrets in env vars
                       
  Layer 4 — ENCRYPT:  Istio mTLS (service mesh)
                       All in-cluster traffic encrypted

Falco custom rules (Aegis Stack):
  - Shell spawned inside container
  - Unexpected outbound network connection from pod
  - Sensitive file read (/etc/passwd, /etc/shadow)
  - Package manager execution (apt, yum) in running container
```

**STAR Answer:**

| | |
|--|--|
| **Situation** | Aegis Stack needed defense in depth — admission control alone cannot catch runtime exploits. |
| **Task** | Deploy Falco with real-time alerting integration. |
| **Action** | Deployed Falco with eBPF driver on EKS. Wrote custom rules for shell-in-container and sensitive file reads. Wired Falcosidekick to Slack + Alertmanager. |
| **Result** | MTTA (Mean Time To Alert) under **30 seconds** for runtime anomalies. |

---

### Q27 · Mid-level · Explain mTLS. How does Istio implement it?

**Model Answer:**

```
Regular TLS:   Server presents cert → Client verifies server identity
mTLS:          Server presents cert → Client verifies server identity
               Client presents cert → Server verifies client identity

Istio implementation:
  1. Istio CA issues SPIFFE X.509 certs to each pod (via SPIRE)
  2. Envoy sidecar handles TLS handshake transparently
  3. App code changes: ZERO

# Enforce strict mTLS cluster-wide
apiVersion: security.istio.io/v1beta1
kind: PeerAuthentication
metadata:
  name: default
  namespace: istio-system
spec:
  mtls:
    mode: STRICT   # reject all plaintext
```

> 💡 **Zero-trust principle:** verify every connection, never trust the network. mTLS is the foundational primitive.

---

### Q40 · Mid-level · What is RBAC in Kubernetes and how would you audit it?

**Model Answer:**

```bash
# Check what a service account can do
kubectl auth can-i --list \
  --as=system:serviceaccount:default:my-service

# Most dangerous RBAC misconfiguration:
# pods/exec permission = effectively cluster-admin
kubectl auth can-i create pods/exec --as=system:serviceaccount:default:my-service

# Audit tools:
#   rbac-lookup   → show who can do what
#   kube-bench    → CIS Kubernetes benchmark compliance
#   kubectl-who-can → reverse lookup (who can verb resource?)

# Block cluster-admin bindings with OPA Gatekeeper:
violation[{"msg": msg}] {
  input.review.object.roleRef.name == "cluster-admin"
  msg := "cluster-admin ClusterRoleBinding not allowed"
}
```

---

## Part 5 — Behavioural (STAR)

---

### Q29 · Fresher · Tell me about yourself and why you want to work in SRE.

**Model Answer (90-second script):**

> "I'm Abhijit, a DevOps and Cloud Security engineer with 3 years of hands-on experience building production Kubernetes platforms on AWS. I architected Aegis Stack — a zero-trust Kubernetes security platform featuring GitOps CI/CD, OPA Gatekeeper policy-as-code, Falco runtime threat detection, and a full Prometheus/Grafana/Loki observability stack.
>
> I'm drawn to SRE because I believe reliability engineering should be a software problem — automation, error budgets, and SLOs — not a heroics problem. What excites me about this role specifically is [specific thing about company that you researched]."

> 💡 **End with a question:** "What does a typical on-call week look like here?" — shows genuine interest.

---

### Q30 · Mid-level · Describe a time you pushed back on a product team that wanted to deploy despite low reliability.

| | |
|--|--|
| **Situation** | PodPlate's product team wanted to ship a new checkout feature but error budget was at 15% remaining with 2 weeks left in the month. |
| **Task** | Present data to justify a deployment freeze while proposing an alternative path. |
| **Action** | Showed the team a Grafana error budget burn rate chart — at the current rate, the budget would be exhausted in 3 days, violating the 99.9% SLO. Proposed: delay the risky feature, deploy a reliability improvement instead to recover budget, ship the feature next window. |
| **Result** | Team agreed. Reliability recovered. Feature shipped 5 days later with zero SLO breach. Relationship strengthened because the decision was data-driven. |

> 💡 **Magic phrase:** "I brought data to the conversation, not opinions." Shows SRE maturity.

---

### Q31 · Mid-level · Tell me about a time you made a mistake that caused a production incident.

| | |
|--|--|
| **Situation** | During Aegis Stack development, pushed a Falco rule change that caused alert storms — 500+ alerts in 10 minutes — overwhelming Slack and masking a real security event. |
| **Task** | Acknowledge the mistake, mitigate quickly, prevent recurrence. |
| **Action** | Immediately rolled back the Falco rule via ArgoCD (30-second rollback). Ran postmortem: root cause was no staging test for Falco rules. Created action item: add Conftest validation for Falco rule syntax + staging canary for security config changes. |
| **Result** | Zero recurrence. CI pipeline now catches invalid Falco rules before any environment. Shared postmortem with the team — 3 engineers adopted the same Conftest check. |

> 💡 **Never say "I was lucky it wasn't worse."** Say "Here is what I built to ensure it never happens again."

---

### Q32 · Mid-level · Tell me about a time you improved a process that reduced toil for your team.

| | |
|--|--|
| **Situation** | RoutineOps environment provisioning required 2 days of manual AWS console clicking per environment — blocking developer velocity. |
| **Task** | Automate environment provisioning end-to-end. |
| **Action** | Wrote Terraform modules for VPC, subnets, EKS cluster, node groups, IAM roles, security groups. Added GitHub Actions CI to run `terraform plan` on PRs and `terraform apply` on merge. Added automated smoke tests post-apply. |
| **Result** | Provisioning time: **2 days → 20 minutes**. Estimated 40 engineer-hours/month of toil eliminated permanently. |

> 💡 **Use the word "toil" explicitly** — it signals SRE vocabulary fluency.

---

## How to Use This Guide

### Recommended Study Order

```
Week 1: SRE Principles (Q1–Q10, Q37, Q42, Q35, Q45, Q50)
         Focus: error budgets, SLOs, toil, golden signals, postmortems

Week 2: System Design (Q11–Q15)
         Focus: whiteboard one design per day; cover requirements → estimates → trade-offs

Week 3: Security (Q23–Q28, Q40)
         Focus: tie every answer back to Aegis Stack — it's your strongest differentiator

Week 4: SWE / Coding (Q16–Q21, Q44, Q47, Q51)
         Focus: code every solution by hand; verify edge cases

Week 5: Behavioural (Q29–Q34, Q53, Q55)
         Focus: practice speaking STAR answers aloud, targeting 2–3 minutes each
```

### Interview Day Checklist

- [ ] Review your resume projects (Aegis Stack, PodPlate, RoutineOps) — specific numbers
- [ ] Practice the 90-second self-introduction (Q29)
- [ ] Research the company's engineering blog — find one specific post to reference
- [ ] Prepare 3 questions to ask the interviewer
- [ ] Have your GitHub repos open to demo if asked

---

## Quick Reference — Key Formulas & Commands

### SRE Math

```
Error budget       = 1 - SLO
Monthly downtime   = (1 - 0.999) × 43,800 min = 43.8 min/month
Availability       = Uptime / (Uptime + Downtime)

Multi-window burn rate alert:
  Fast: error_rate / (1 - SLO) > 14  over 1h window
  Slow: error_rate / (1 - SLO) > 6   over 6h window
```

### Availability Table

| SLO | Downtime/month | Downtime/year |
|-----|---------------|---------------|
| 99% | 7.3 hours | 3.65 days |
| 99.9% | 43.8 minutes | 8.77 hours |
| 99.95% | 21.9 minutes | 4.38 hours |
| 99.99% | 4.4 minutes | 52.6 minutes |

### Key PromQL Snippets

```promql
# Error rate
sum(rate(http_requests_total{status=~"5.."}[5m]))
  / sum(rate(http_requests_total[5m]))

# p99 latency
histogram_quantile(0.99,
  sum(rate(http_request_duration_seconds_bucket[5m])) by (le, service))

# SLO burn rate (fast window)
(
  sum(rate(http_requests_total{status=~"5.."}[1h]))
  / sum(rate(http_requests_total[1h]))
) / (1 - 0.999) > 14
```

### Key kubectl Commands

```bash
# RBAC audit
kubectl auth can-i --list --as=system:serviceaccount:ns:sa-name
kubectl get clusterrolebindings -o json | jq '.items[] | select(.subjects[]?.name=="cluster-admin")'

# Debug DNS
kubectl exec -it <pod> -- nslookup kubernetes.default

# Check Istio mTLS
istioctl proxy-config all <pod> | grep -i mtls

# HPA status
kubectl get hpa -w

# Resource usage
kubectl top pods --sort-by=memory -A
```

---

## Resources

| Resource | Link |
|----------|------|
| Google SRE Book | https://sre.google/sre-book/table-of-contents/ |
| Google SRE Workbook | https://sre.google/workbook/table-of-contents/ |
| Aegis Stack (your project) | https://github.com/abhijitray7810/aegis-stack |
| PodPlate (your project) | https://github.com/abhijitray7810/PodPlate-Platform |
| RoutineOps (your project) | https://github.com/abhijitray7810/RoutineOps |
| CKA Curriculum | https://github.com/cncf/curriculum |
| Prometheus Alerting Best Practices | https://prometheus.io/docs/practices/alerting/ |

---

*Generated for Abhijit Ray — DevOps & Cloud Security Engineer*
*Resume: roy055659@gmail.com · github.com/abhijitray7810*
