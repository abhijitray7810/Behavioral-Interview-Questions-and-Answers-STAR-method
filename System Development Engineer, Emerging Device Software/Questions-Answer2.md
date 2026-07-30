# 21-Day Amazon Lab126 SDE Interview Prep Plan
**System Development Engineer, Emerging Device Software | Job ID: 10452432**
Prepared for: Abhijit Ray

This plan follows your requested structure: Week 1 (Leadership Principles, 50% weight), Week 2 (Technical Deep Dive, 40% weight), Week 3 (Integration & Polish). Each day lists the must-know questions for that topic with full model answers, tailored to your resume (Aegis Stack, RoutineOps, PodPlate, RT Network Solutions QA work).

---

## Table of Contents
- [Week 1 — Leadership Principles](#week-1--leadership-principles)
- [Week 2 — Technical Deep Dive](#week-2--technical-deep-dive)
- [Week 3 — Integration & Polish](#week-3--integration--polish)
- [Master Question Bank (Quick Revision)](#master-question-bank-quick-revision)

---

## WEEK 1 — LEADERSHIP PRINCIPLES

### Day 1: Amazon Culture + STAR Method

**What to internalize:**
- Amazon has 16 Leadership Principles; interviewers are each assigned 1-2 to probe per round
- **STAR:** Situation (1-2 sentences of context) → Task (your specific goal) → Action (what *you* did — use "I," not "we") → Result (quantified outcome)
- Common mistake: spending 80% of the answer on Situation/Task and rushing Action/Result. Aim for roughly 20/10/50/20 split.
- Every story should have a number attached to the Result if at all possible.

**Practice question:** "Tell me about yourself."
**Model answer:**
> "I'm a final-year Computer Science student graduating in 2026, currently working as a Project Associate at RT Network Solutions doing QA on ML annotation pipelines, where I cut mislabeling by 18%. Outside that, my focus has been build/release automation — I've built three infrastructure projects covering CI/CD security (Aegis Stack), Kubernetes deployment automation (RoutineOps), and self-healing microservices (PodPlate). I'm AWS Well-Architected certified, and this role appeals to me because it's exactly this kind of infrastructure work applied to next-gen hardware."

**Day 1 action item:** Write out your own 60-90 second intro and time yourself reading it aloud. Cut anything over 90 seconds.

---

### Day 2: Ownership + Dive Deep

**Q1 (Ownership): "Tell me about a time you took on something outside your job description."**
> **Situation:** While building Aegis Stack, I noticed the CI/CD pipeline would happily push container images with zero vulnerability scanning or provenance tracking.
> **Task:** No one assigned this to me — I decided the pipeline wasn't "done" until this gap was closed.
> **Action:** I integrated Trivy for scanning, Syft for SBOM generation, and Cosign for signing, then added OPA/Conftest gates that fail the build on high-severity CVEs or unsigned images.
> **Result:** The pipeline now enforces secure-by-default end-to-end, and I reused the same pattern in my other projects.

**Q2 (Ownership): "Describe a time you fixed a problem that would recur if you didn't address the root cause."**
> Use your QA redesign story (see Day 3, Dive Deep) — ownership and dive deep often overlap; be ready to tell the same story with either emphasis depending on which principle the interviewer names.

**Q3 (Dive Deep): "Tell me about a time you had to dig deep to find a root cause, not just a symptom."**
> **Situation:** .
> **Task:** I needed to find why errors kept slipping past review, not just flag more of them.
> **Action:** I audited a sample of mislabeled batches against the annotation spec, traced the errors to a specific set of ambiguous edge cases, and redesigned the QA review cycle with a targeted second-pass check for exactly those cases.
> **Result:** Mislabeled-object rate dropped 18%, directly improving downstream model accuracy.

**Q4 (Dive Deep): "Tell me about the most technically complex problem you've debugged."**
> Prepare a specific infrastructure debugging story from RoutineOps or PodPlate — e.g., a pipeline failure you traced to a specific root cause (config drift, resource limits, a bad manifest). If you don't have one memorized yet, write one out today using a real debugging session you've had.

**Day 2 action item:** Write both stories in full STAR format, then read them aloud and time them (~90 seconds each).

---

### Day 3: Bias for Action + Deliver Results

**Q1 (Bias for Action): "Tell me about a decision you made quickly with incomplete information."**
> **Situation:** While automating RoutineOps' deployment pipeline, a Kubernetes rollout started failing intermittently right before a demo, with no single clear error in the logs.
> **Task:** Limited time to either fix forward or roll back before the demo.
> **Action:** Rather than fully root-causing under time pressure, I checked recent changes, identified the most recent config change as the likely cause, reverted it, and validated the deployment — deferring full root-cause analysis until after the demo.
> **Result:** Demo proceeded on schedule; I later confirmed the reverted change was indeed the cause and fixed it properly, adding a validation step to the pipeline.

**Q2 (Bias for Action): "Tell me about a time you chose speed over perfect information."**
> Alternative angle on the same story, or use a QA example — e.g., deciding to flag and pause a batch immediately on suspected labeling drift rather than waiting for full statistical confirmation.

**Q3 (Deliver Results): "Tell me about a project you delivered end-to-end under real constraints."**
> **Situation:** Aegis Stack aimed to cover the full build-and-release lifecycle with defense-in-depth security — a large scope for a self-driven project.
> **Task:** Deliver a working, production-style reference implementation, not just a proof of concept.
> **Action:** Prioritized must-have security gates first (Trivy, Cosign, OPA), then layered in Vault-based secrets and Istio mTLS once the core pipeline was solid — working incrementally instead of trying to build everything at once.
> **Result:** Aegis Stack became a complete reference architecture I can walk through end-to-end, covering scanning, signing, policy gating, secrets, and zero-trust networking.

**Q4 (Deliver Results): "Tell me about a time you hit a measurable target."**
> Use the 30% manual-deployment-effort reduction from RoutineOps, or the 20% turnaround-time improvement from your QA workflow automation.

**Day 3 action item:** Draft both stories; make sure each Result line has a real number.

---

### Day 4: Earn Trust + Customer Obsession

**Q1 (Earn Trust): "Tell me about a time you had to get others to adopt something new."**
> **Situation:** Across 10+ concurrent annotation datasets/projects, teams followed inconsistent data-handling practices.
> **Task:** Get teams to actually adopt a documented standard, not just have one exist on paper.
> **Action:** Documented annotation standards and privacy/data-handling controls clearly, then worked directly with data science and engineering teams to fold the standard into their existing workflow rather than imposing a brand-new process on top.
> **Result:** Consistent enforcement across all 10+ datasets; per-batch turnaround time dropped 20% because less time was lost to rework and clarification.

**Q2 (Earn Trust): "Tell me about a time you admitted a mistake."**
> Prepare a genuine example — Amazon interviewers specifically probe for humility here. If you don't have one ready, think back to a bug you shipped or a wrong assumption in one of your projects (e.g., an early Aegis Stack iteration without proper resource limits) and how you owned and fixed it.

**Q3 (Customer Obsession): "Tell me about a time you did something because it mattered to the end user, even if it wasn't the easy path."**
> **Situation:** Sustaining 97%+ labeling accuracy directly affects the ML teams relying on that data as their "customer."
> **Task:** Balance thoroughness against turnaround-time pressure.
> **Action:** Instead of cutting review corners to hit turnaround targets, I focused the QA redesign specifically on the highest-impact error categories so accuracy improved without slowing the pipeline down.
> **Result:** Both accuracy (18% fewer errors) and turnaround time (20% faster) improved together — customer obsession didn't have to trade off against efficiency.

**Day 4 action item:** Draft the "mistake" story — this is the one candidates most often leave unprepared, and it shows up often.

---

### Day 5: Invent & Simplify + Have Backbone, Disagree and Commit

**Q1 (Invent & Simplify): "Tell me about a time you automated or simplified something manual."**
> **Situation:** Annotation workflows and tooling had manual steps slowing down per-batch turnaround.
> **Task:** Reduce turnaround time without sacrificing accuracy.
> **Action:** Partnered with data science and engineering teams to identify the highest-friction manual steps and streamlined the tooling around them.
> **Result:** Per-batch turnaround time cut by 20%.

**Q2 (Invent & Simplify): "Tell me about the most creative technical solution you've built."**
> Aegis Stack's OPA policy work is a good anchor — describe a specific non-obvious policy you wrote (e.g., blocking `:latest` image tags) and the creative parsing/logic challenge involved.

**Q3 (Have Backbone, Disagree and Commit): "Tell me about a time you disagreed with a technical decision."**
> This one needs a real example from your experience — think about a design choice in one of your projects where you initially favored a different tool/approach (e.g., choosing Istio over Linkerd, or GitOps over manual `kubectl apply`) and had to make and defend that call, even if only to yourself or a small team. Write this out fully today; it's commonly asked and easy to be caught flat-footed on.

**Day 5 action item:** All 8 stories (Ownership, Dive Deep, Bias for Action, Deliver Results, Earn Trust, Customer Obsession, Invent & Simplify, Have Backbone) should now exist in full written STAR form. Read all 8 aloud once today.

---

### Day 6: First Behavioral Mock Interview

**How to self-run this:**
1. Pick 4 of your 8 stories at random (don't choose — use a die roll or random.org so you practice retrieval under uncertainty, the way a real interview works)
2. Set a timer: 2 minutes per answer, no notes
3. Record yourself (phone voice memo is enough)
4. After each answer, ask yourself: Did I lead with Action, or did I ramble through Situation/Task? Did I include a number in the Result?

**Common follow-up questions to expect after any story:**
- "What would you do differently if you did this again?"
- "What was the hardest part of that?"
- "How did others react to your decision?"
- "What did you learn from it?"

Prepare a one-line answer to each of these four follow-ups for at least your top 4 stories — interviewers almost always dig one layer deeper.

---

### Day 7: Review + Rest

- Re-read all 8 stories once, silently
- Do **not** cram new material today — spaced review works better than late cramming
- Get proper sleep; Week 2 is technical and needs a clear head

---

## WEEK 2 — TECHNICAL DEEP DIVE

### Day 8: CI/CD Pipeline Design (Yocto, GitOps, Embedded Firmware)

**Q1: "Design a CI/CD pipeline for an embedded Linux device that builds firmware, tests it, and performs OTA updates."**
> Structure the answer in stages: (1) source validation — lint, pre-commit hooks; (2) build — cross-compile for target SoC, cache build state; (3) test — unit tests, static analysis, hardware-in-loop or QEMU emulation; (4) security — SBOM generation, signing, policy gates *(this is where you can speak with direct authority from Aegis Stack)*; (5) artifact management — versioned, signed artifacts with retention policies; (6) OTA deployment — canary rollout (5%→25%→100%) with automatic rollback on failure spikes.
> **Bridge line if asked about Yocto specifically:** "I haven't worked with Yocto directly, but the caching, reproducibility, and artifact-signing patterns I built in Aegis Stack apply directly — Yocto builds are just slower and more dependency-heavy, so build-state caching matters even more."

**Q2: "How do you handle flaky tests in a pipeline?"**
> Track per-test failure rate; quarantine anything intermittent (e.g., >5% failures over 2 weeks) to a non-blocking stage while root-causing. Common causes: race conditions, shared resource contention, unmocked external dependencies. Reintegrate only after a consistent run of passes.

**Q3: "GitOps vs. traditional CI/CD — when do you use each?"**
> **GitOps** (Git as source of truth, auto-sync via ArgoCD/Flux): ideal for Kubernetes-native tooling infra — *this is what you did in PodPlate.* **Traditional CI/CD:** better for imperative processes like device flashing, where steps don't map cleanly to declarative state. **Hybrid (best fit for Lab126):** CI for build/test/artifact creation, GitOps for K8s tooling, traditional pipelines for device flashing/OTA orchestration.

**Q4: "How would you reduce a slow embedded build time?"**
> Shared build-state caching, parallelizing independent build steps, using faster storage (NVMe/SSD) for the cache directory, and mirroring external dependency downloads to local/internal storage to avoid network bottlenecks.

---

### Day 9: Linux Systems & Troubleshooting

**Q1: "A build agent is experiencing extremely slow I/O. How do you diagnose it?"**
> `iostat -x` for disk utilization/queueing, `iotop`/`pidstat -d` to find the offending process, `df -i` to check inode exhaustion (common with many small build files), `dmesg` for disk errors. Fixes depend on the cause: move caches to faster storage, increase inode limits, add RAM to reduce swap thrashing, mirror slow network fetches locally.

**Q2: "How would you secure a Linux build host used for firmware compilation?"**
> Host hardening (minimal install, no GUI, SELinux/AppArmor enforcing), no root SSH, key-based auth with centralized IAM and session logging, ephemeral/isolated build environments (containers or microVMs) so no state persists between builds, checksum/signature verification on all downloaded sources, and runtime monitoring for unexpected outbound connections during builds.

**Q3: "Monolithic vs. microkernel — why does Android modify Linux?"**
> Monolithic (Linux): faster, all services in kernel space, but a driver crash can take the system down. Microkernel (QNX, seL4): stronger isolation, more IPC overhead, common in safety-critical systems. Android's modifications: wakelocks for power management, a tuned low-memory killer, SELinux enforcement, and hardware abstraction to keep vendor drivers in user space — directly relevant to wearables, where power management and responsiveness matter.

**Q4: "Walk me through what happens when you run a command in a Linux shell."**
> Shell parses/tokenizes the command → checks if it's a builtin or external binary → for external, searches `$PATH` → `fork()`s a child process → child `exec()`s the binary, replacing its memory image → parent shell `wait()`s for the child → exit status returned.

---

### Day 10: Distributed Systems & Scalability

**Q1: "Design a build system that handles 1000+ concurrent builds."**
> Queue/orchestrator with priority scheduling → elastic build farms (spot instances with checkpointing to survive interruption) → shared storage layered by access pattern (S3 for artifacts, shared network storage for build cache, local fast disk for scratch space) → idempotent build steps so failures retry cleanly on any agent.

**Q2: "How do you ensure high availability for a critical build/release service?"**
> Multi-AZ deployment, load-balanced with health checks, N+1 capacity planning, graceful degradation (queue rather than fail when storage briefly unavailable), and a defined recovery point/time objective for disaster recovery.

**Q3: "How do you handle spot instance interruptions in a build farm without losing work?"**
> Checkpoint build state periodically, listen for the interruption warning, save in-progress state to persistent storage on warning, and requeue the job to a fresh agent rather than losing all progress.

**Q4: "How would you design a system to detect and prevent one team's builds from starving another's?"**
> Fair scheduling with per-team/per-priority queues, resource quotas, and monitoring on queue wait time per team with alerting if one group's wait time spikes.

---

### Day 11: AWS & Infrastructure as Code

**Q1: "Design the VPC/security model for an isolated build environment on AWS."**
> Private subnets only, no public IPs for build hosts; bastion host with MFA for any interactive access; VPC endpoints for S3/ECR so AWS API traffic stays off the public internet; least-privilege IAM via IRSA for EKS-based build agents; KMS encryption everywhere. *This maps directly onto what you built in Aegis Stack — say so explicitly.*

**Q2: "Terraform state management best practices for a 20-engineer team?"**
> Remote state in S3 with versioning, state locking via a lock table, state segmented per environment and per component to limit blast radius, mandatory `terraform plan` on every PR with gated `apply` in production, OIDC auth instead of long-lived credentials, nightly drift detection.

**Q3: "How do you manage secrets in your infrastructure — walk me through your approach."**
> From your Aegis Stack work: HA Vault for dynamic secrets with TTLs, sidecar injection so applications read secrets from a file mount with zero code changes, and no secrets ever committed to Git or stored in Terraform state (mark sensitive outputs, keep actual values in Secrets Manager/Vault).

**Q4: "Explain the trade-off between using managed services (RDS, EKS) versus self-managed (Kubernetes on EC2)."**
> Managed services reduce operational burden (patching, HA, backups handled for you) at the cost of some flexibility and vendor lock-in. Self-managed gives full control and can be cheaper at scale but requires the team to own reliability engineering. For a build farm, managed (EKS) usually wins unless there's a very specific customization need.

---

### Day 12: Kubernetes, Security & Monitoring

**Q1: "How would you implement runtime security monitoring for a Kubernetes-based build farm?"**
> Runtime threat detection watching for anomalous syscalls/outbound connections from build pods (Falco), policy enforcement blocking privileged containers and requiring resource limits/read-only filesystems (Kyverno/OPA), and a full observability stack (Prometheus/Grafana) with severity-routed alerting. *This is close to exactly what you implemented in Aegis Stack and PodPlate — use your own components as the concrete answer.*

**Q2: "A production build pipeline has been compromised. Walk me through your incident response."**
> **Containment:** isolate compromised agents, revoke active credentials, disable the pipeline.
> **Eradication:** trace the attack vector via logs/audit trail, rebuild infra from known-good IaC state, rotate all secrets.
> **Recovery:** rebuild artifacts from clean source, verify integrity against backups, restore with heightened monitoring.
> **Post-incident:** blameless postmortem, concrete guardrails added (MFA everywhere, signed commits, binary authorization).

**Q3: "How did Istio's mTLS improve security in your PodPlate/Aegis Stack projects?"**
> Enforces mutual TLS between all services by default (zero-trust), so even if the network perimeter is breached, unauthorized services can't establish connections without valid certificates — encrypts traffic in transit and authenticates service identity, not just network location.

**Q4: "What's the difference between Kyverno and OPA/Gatekeeper?"**
> Both do policy enforcement in Kubernetes. OPA/Gatekeeper uses Rego, a general-purpose policy language, which is powerful but has a steeper learning curve. Kyverno uses native Kubernetes YAML for policies, which is more approachable for teams already fluent in K8s manifests. Choice often comes down to team familiarity and whether policies need Rego's more complex logic.

---

### Day 13: System Design — OTA & Device Management

**Q1: "Design a system for managing firmware releases to millions of IoT devices."**
> Walk the full chain: developer commit → CI/CD build/test/sign → artifact + release metadata store → OTA update service (API layer devices poll, orchestration, push notification) → device-side update agent with A/B partitioning and bootloader signature verification → observability with automatic rollback if failure rate crosses a threshold on a phased rollout (canary → 10% → 50% → 100%).
> Name these specifically: mutual TLS/device certificates for authentication, HSM-backed firmware signing, delta updates (binary diff) to cut bandwidth, and an SBOM per firmware version for compliance.

**Q2: "How do you design safe rollback for a failed OTA update?"**
> A/B partitioning on-device: the new firmware writes to the inactive partition; the bootloader only switches the "active" flag after boot verification succeeds. If the new firmware fails to boot or crashes repeatedly, the bootloader falls back to the last-known-good partition automatically.

**Q3: "How would you reduce OTA bandwidth costs for millions of devices?"**
> Delta/binary-diff updates instead of full images, staged rollout scheduling during low-usage hours, and CDN/edge caching for update payloads close to device populations.

**Q4: "How do you monitor the health of an OTA rollout in real time?"**
> Per-version success/failure rate dashboards, automatic pause/rollback triggers if failure rate crosses a threshold (e.g., >1%), and device check-in telemetry (crash rate, connectivity) tracked against the rollout percentage.

---

### Day 14: Technical Review + Mock

**Self-mock structure (60-90 min):**
1. Pick 2 questions from Day 8-13 at random
2. Answer out loud, timed, on a whiteboard/paper if possible (sketch the architecture, don't just talk)
3. Check: did you state assumptions? Did you mention trade-offs, not just one "correct" design?
4. Re-read the questions you found hardest and rewrite your answer in your own words

**Common technical follow-ups to prepare for:**
- "What would you change if this needed to handle 10x the scale?"
- "What's the biggest single point of failure in what you just designed?"
- "How would you test this before rolling it out?"

---

## WEEK 3 — INTEGRATION & POLISH

### Day 15: Project Deep-Dive Prep (10 Follow-Ups Each)

**Aegis Stack — be ready for:**
1. Why Istio over Linkerd or Cilium? → mature mTLS and traffic management, native Vault integration; trade-off is higher resource overhead
2. How did you handle Vault secret rotation without downtime? → dynamic secrets with TTLs, sidecar injection, apps read from file mount
3. What was the hardest OPA policy to write? → have one specific example ready (e.g., blocking `:latest` tags, parsing private registry URLs in Rego)
4. How did you test your OPA policies before enforcing them? → run policies in "audit/dry-run" mode first, review violations before switching to enforce
5. What happens if Vault itself goes down? → describe your HA Vault setup and failover behavior
6. How do you handle key rotation for Cosign signing? → describe your key management approach
7. What's your blast radius if IRSA is misconfigured? → over-permissioned service account risk; mitigated by least-privilege review
8. How would you extend this pipeline for embedded firmware signing? → connect to HSM-backed signing, bootloader verification
9. What was the SBOM format you used and why? → SPDX/CycloneDX, chosen for tool compatibility
10. What would you change if you rebuilt this today? → have a genuine answer — shows self-awareness

**RoutineOps — be ready for:**
1. How did you handle Jenkins master availability? → persistent storage, configuration-as-code, backups
2. What metrics did you alert on? → queue depth, failure rate, resource saturation, severity-routed alerts
3. How did you reduce manual deployment effort by 30%? → be specific about which steps were automated
4. What was your rollback strategy? → describe it concretely
5. How did you handle secrets in Jenkins pipelines? → avoid hardcoding, use a secrets backend
6. What would break first if load doubled? → identify the actual bottleneck
7. How did you validate infrastructure changes before applying them? → `terraform plan` review process
8. What monitoring gaps did you have, if any? → honest answer shows maturity
9. How did you structure your Terraform modules? → per-environment/component segmentation
10. What's one incident (real or simulated) you handled in this project?

**PodPlate Platform — be ready for:**
1. How did Istio improve fault tolerance? → circuit breaking, retries with jitter, mTLS
2. What was your GitOps workflow? → Git as source of truth, auto-sync, image automation, manual prod approval
3. How does horizontal autoscaling decide when to scale? → CPU/memory metrics or custom metrics via HPA
4. What happens during a node failure? → pod rescheduling, PodDisruptionBudget behavior
5. How did you instrument observability? → Prometheus/Grafana stack, what specifically you tracked
6. How do you prevent cascading failures? → circuit breakers, retries with backoff, bulkheading
7. What's your service mesh's performance overhead? → be honest about the mTLS/sidecar cost trade-off
8. How do you test self-healing behavior? → chaos-style pod termination testing
9. What's your canary/blue-green strategy, if any? → describe or acknowledge it's a gap
10. What would you change for production readiness?

**Day 15 action item:** Write short (2-3 sentence) answers to all 30 follow-ups above. You won't get all of them, but having them written removes the anxiety of being surprised.

---

### Day 16: Day-in-the-Life Scenarios

**Scenario 1 — On-call, build farm down at 2 AM:**
> Acknowledge the page → check whether it's a recent code/infra change first → check autoscaling group/node health → roll back to the last known-good image if it's an agent issue → scale out temporarily if it's pure load → communicate status in the team channel → update the runbook afterward.

**Scenario 2 — Security incident, hardcoded credentials found in a deployed artifact:**
> Rotate the exposed credentials immediately → check how far the artifact propagated → purge it from all repositories/caches → trace whether it was committed to Git → add secret-scanning to the pipeline going forward → run a blameless postmortem.

**Scenario 3 — Cross-team ask, hardware team needs a new compiler toolchain:**
> Containerize the new toolchain alongside the old one → run both in parallel for a defined period comparing outputs → migrate one target at a time → keep a rollback path available for a month after full migration.

**Day 16 action item:** Practice narrating these out loud in under 90 seconds each — on-call scenarios are timed to test decisiveness, not just correctness.

---

### Day 17–19: Timed System Design Whiteboard Sessions

Run one full system design question per day, 45 minutes each, structured as:
- **First 5 min:** clarify requirements and scale (how many devices? what's the failure tolerance?)
- **Next 25 min:** draw the architecture, narrating trade-offs as you go
- **Last 15 min:** discuss failure modes, monitoring, and what you'd change at 10x scale

**Day 17:** Distributed build system for 1000+ concurrent builds (Day 10, Q1)
**Day 18:** Full CI/CD pipeline for embedded firmware with OTA (Day 8, Q1 + Day 13, Q1 combined)
**Day 19:** Runtime security monitoring + incident response for a compromised pipeline (Day 12, Q1 + Q2 combined)

---

### Day 20: Full 3-Hour Loop Simulation

Simulate the real loop back-to-back:
1. Recruiter screen (15 min) — your intro + "why this role"
2. Phone screen (45 min) — 1 LP story + 1 technical question
3. Loop round 1 (45 min) — 2 LP stories
4. Loop round 2 (45 min) — 1 system design question
5. Bar Raiser round (45 min) — 2 LP stories, likely probing "Have Backbone" or "Earn Trust"

Ask a friend to ask you questions cold (from the Master Question Bank below) so you practice without knowing exactly what's coming.

---

### Day 21–24: Weak Area Targeting, Final Polish, Rest

- Re-review whichever of the 8 LP stories felt weakest in the Day 20 simulation
- Re-review whichever technical topic (Days 8-13) you stumbled on
- Day 23-24: no new material — light review only, prioritize sleep
- Reconfirm interview logistics: time zone, video link, ID requirements if any

---

## MASTER QUESTION BANK (QUICK REVISION)

**Leadership Principles (pick any, answer in <2 min):**
1. Tell me about a time you took ownership beyond your role.
2. Tell me about a time you had to dive deep to find a root cause.
3. Tell me about a decision made with incomplete information.
4. Tell me about hitting a measurable, quantified result.
5. Tell me about earning trust from a skeptical team.
6. Tell me about a mistake you made and how you handled it.
7. Tell me about simplifying or automating something manual.
8. Tell me about disagreeing with a technical decision.

**Technical (pick any, answer in <5 min, with a diagram if possible):**
1. Design a CI/CD pipeline for embedded firmware with OTA.
2. Diagnose a build agent with slow I/O.
3. Design a build system for 1000+ concurrent builds.
4. Design the VPC/security model for an isolated build environment.
5. Explain Terraform state management for a large team.
6. Design runtime security monitoring for a Kubernetes build farm.
7. Walk through incident response for a compromised pipeline.
8. Design an OTA/firmware release system for millions of devices.

**Project-specific (expect at least one deep-dive):**
- Any of the 10 follow-ups per project listed in Day 15.

---

*Your existing projects already map closely onto nearly every technical topic in this plan. The one gap — deep embedded/Yocto/RTOS experience — is worth naming honestly and bridging to your cloud/CI-CD depth rather than avoiding. That combination of real depth plus honest self-assessment is exactly what the Bar Raiser round is designed to surface.*
