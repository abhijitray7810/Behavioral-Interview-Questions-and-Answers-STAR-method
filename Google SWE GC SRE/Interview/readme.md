# 🚀 Google SRE/DevOps Interview Prep Guide
## Software Engineer, Google Cloud, Site Reliability Engineering — Warsaw
### Tailored for Abhijit Ray | DevOps & Cloud Security Engineer (3+ YOE)

---

## 📋 TABLE OF CONTENTS
1. [Google Behavioral (Googleness) — STAR Format](#section-1)
2. [SRE Fundamentals & Reliability Engineering](#section-2)
3. [Linux, Networking & Systems Engineering](#section-3)
4. [Kubernetes, Containers & Service Mesh](#section-4)
5. [CI/CD, GitOps & DevSecOps](#section-5)
6. [AWS, IaC & Terraform](#section-6)
7. [Observability, Monitoring & Incident Response](#section-7)
8. [Coding, Scripting & Automation](#section-8)
9. [Google Interview Strategy & Tips](#section-9)

---

<a name="section-1"></a>
## SECTION 1: GOOGLE BEHAVIORAL (GOOGLENESS) — 15 QUESTIONS
*Google evaluates: Intellectual Humility, Comfort with Ambiguity, Collaboration, User Focus*

### Q1. Tell me about a time you had to implement a security policy that slowed down developers. How did you handle the pushback?
**STAR Answer:**
- **Situation:** At my previous role, I mandated Trivy vulnerability scanning + Cosign image signing in our Tekton/GitHub Actions pipeline, which added ~4 minutes to every build.
- **Task:** I needed to enforce supply-chain security without destroying developer velocity or creating shadow IT workarounds.
- **Action:** I scheduled 1:1s with 5 senior engineers to understand their pain points. I optimized the pipeline by parallelizing scans (image build + SBOM generation simultaneously), implemented a caching layer for Trivy DB, and created a "fast-track" for local dev using pre-signed base images. I also demo'd a real CVE-2024-XXXX that Trivy caught in a base image.
- **Result:** Build time dropped to under 90 seconds. Developer NPS improved from 3.2 to 4.6. Zero unsigned images deployed in 6 months. One engineer even contributed a Conftest policy back to the repo.

### Q2. Describe a situation where you had to debug a production outage with incomplete data. Walk me through your thought process.
**STAR Answer:**
- **Situation:** Our e-commerce platform (PodPlate) experienced intermittent 502s during peak dinner hours. Logs were incomplete due to a Loki rate-limit misconfiguration.
- **Task:** Restore service within SLO (99.9% uptime) while data was missing.
- **Action:** I used the "divide and conquer" method: (1) Checked Istio mTLS metrics — error rate spiked on checkout-service only; (2) Used `kubectl top` + Prometheus to find memory pressure; (3) SSH'd into the node and used `dmesg` to discover OOMKills; (4) Found HPA was scaling on CPU, not memory; (5) Temporarily manually scaled the deployment while fixing HPA metrics. I also created a runbook gap ticket.
- **Result:** MTTR was 18 minutes (under our 30m SLO). Post-mortem led to custom SLIs for memory-based autoscaling and a fix for Loki retention.

### Q3. Give an example of a time you disagreed with a senior engineer's architectural decision. What did you do?
**STAR Answer:**
- **Situation:** A senior architect wanted to use CloudFormation instead of Terraform for our new EKS environment, arguing it was "native AWS."
- **Task:** I believed Terraform was better for multi-cloud future state and state management, but I needed to influence without authority.
- **Action:** I didn't argue in the meeting. Instead, I built a 2-week PoC comparing both: (1) Terraform's modular reusability for EKS + VPC + IAM; (2) Drift detection capabilities; (3) Cost of maintaining 2000+ lines of YAML vs HCL. I presented objective data: Terraform reduced environment provisioning from 3 days to 20 minutes vs CloudFormation's 4 hours.
- **Result:** The team adopted Terraform. I was asked to lead the IaC guild. The senior engineer became a mentor after seeing the data-driven approach.

### Q4. Tell me about a time you had to learn a technology quickly to solve an urgent problem.
**STAR Answer:**
- **Situation:** During Aegis Stack development, we needed runtime threat detection in 1 week for a security audit. I had never used Falco before.
- **Task:** Implement Falco + Falcosidekick with <30s MTTA before the audit.
- **Action:** I spent 3 days deep-diving: read the Falco docs, deployed it on a test EKS cluster, wrote 8 custom rules for cryptomining and reverse shell detection, integrated Falcosidekick with Slack + S3 for evidence preservation, and load-tested it with `falco-event-generator`. I also shadowed a senior SRE for 2 hours on rule syntax.
- **Result:** Audit passed with zero critical findings. Falco caught a test attack in 12 seconds during the demo. I later wrote an internal wiki that became the team's onboarding guide.

### Q5. Describe a project where you had to balance security with usability. How did you decide where to draw the line?
**STAR Answer:**
- **Situation:** Implementing HashiCorp Vault for secret management in RoutineOps. Developers were storing secrets in GitHub env vars for "speed."
- **Task:** Eliminate secrets-in-env without creating a 10-step workflow that developers would bypass.
- **Action:** I implemented Vault with Kubernetes auth (service account token injection), so pods automatically received secrets at runtime. For local dev, I created a `vault-agent` Docker Compose sidecar and a simple CLI wrapper (`make vault-login`). I measured adoption via telemetry: secret-fetch API calls per day.
- **Result:** Secret-in-env incidents dropped to zero. Developer onboarding time increased by only 5 minutes. The solution was later adopted by 3 other teams.

### Q6. Tell me about a time you made a mistake in production. How did you handle it?
**STAR Answer:**
- **Situation:** I accidentally applied a Terraform change to the production VPC CIDR during a routine IAM policy update because my workspace wasn't switched from `prod` to `dev`.
- **Task:** Minimize blast radius, restore connectivity, and prevent recurrence.
- **Action:** I immediately ran `terraform state pull`, identified the exact change, and ran `terraform apply` with the previous state file to rollback (took 4 minutes). I then: (1) Communicated in #incidents channel within 60 seconds; (2) Verified all EKS node groups were healthy; (3) Added `terraform workspace show` to my pre-commit hooks; (4) Implemented OIDC-based Terraform Cloud runs with mandatory plan reviews for prod.
- **Result:** Zero customer impact (change was caught during maintenance window). The team adopted mandatory PR reviews for all IaC changes. I presented the incident in a blameless post-mortem.

### Q7. Describe a time you had to work with a difficult teammate. How did you ensure project success?
**STAR Answer:**
- **Situation:** A backend engineer consistently bypassed our GitOps workflow, manually kubectl-applying changes to "fix things faster."
- **Task:** Stop the drift without creating conflict or slowing down critical fixes.
- **Action:** I approached them privately first (not in standup). I discovered they found ArgoCD sync too slow for hotfixes. I: (1) Tuned ArgoCD's resource.inclusions to prioritize their namespace; (2) Created an "emergency sync" button with auto-rollback; (3) Added OPA Gatekeeper policy to block manual image tags. I framed it as "making their hotfixes safer," not "stopping them."
- **Result:** Manual deployments dropped 95%. The teammate became the biggest GitOps advocate and co-presented our SRE talk at a meetup.

### Q8. Give an example of how you prioritized multiple urgent tasks during an on-call shift.
**STAR Answer:**
- **Situation:** On-call night: (1) P1 alert — payment service latency >5s; (2) P2 alert — cert-manager certificate expiring in 48h; (3) Slack DM from CTO asking for cost report by morning.
- **Task:** Resolve P1 without neglecting P2 or the CTO request.
- **Action:** I used the Eisenhower Matrix: P1 was "urgent+important" — I immediately checked Istio latency metrics, found a traffic spike from a bot, applied rate-limiting via EnvoyFilter, and scaled HPA. P2 was "not urgent but important" — I created a 30-minute calendar block for the next morning to investigate cert-manager's DNS-01 challenge failure. For the CTO, I delegated the cost report to a junior engineer with a template I'd built, offering to review it.
- **Result:** Payment latency normalized in 8 minutes. Certificate renewed with 36h to spare. Cost report delivered on time. I updated the on-call runbook with the bot mitigation pattern.

### Q9. Tell me about a time you improved a process that others thought was "fine."
**STAR Answer:**
- **Situation:** Our deployment process to EKS took 45 minutes — "that's just how Kubernetes is," people said.
- **Task:** Reduce deployment lead time without compromising safety.
- **Action:** I analyzed the Tekton pipeline with `tkn taskrun logs` and found: (1) Image push to ECR was sequential; (2) Security scan ran on the full image instead of layers; (3) ArgoCD sync waited for the full cluster state. I parallelized build/push, implemented Trivy filesystem scan before image build, and enabled ArgoCD's selective sync. I A/B tested the new pipeline alongside the old one for 1 week.
- **Result:** Deployment time dropped from 45 minutes to under 8 minutes. The team shipped 3x more features that quarter. I was asked to optimize the data pipeline next.

### Q10. Describe a time you had to explain a complex technical concept to a non-technical stakeholder.
**STAR Answer:**
- **Situation:** The product manager wanted to know why we needed to spend $15K/year on Istio service mesh when "HTTP works fine."
- **Task:** Justify the cost and complexity without jargon.
- **Action:** I used the "bank vault analogy": "Right now, our services talk like postcards — anyone who intercepts them can read them. Istio is like putting each postcard in a locked box that only the receiver can open (mTLS). Plus, it gives us a security camera for every conversation (telemetry)." I showed a Grafana dashboard of inter-service traffic before/after Istio, highlighting a 40% latency reduction and the elimination of plaintext credentials in logs.
- **Result:** Budget approved in 24 hours. The PM asked me to present the same analogy to the executive team.

### Q11. Tell me about a time you had to handle an ambiguous requirement.
**STAR Answer:**
- **Situation:** Leadership said: "Make our Kubernetes platform zero-trust." No specifics, no budget, no timeline.
- **Task:** Define and deliver "zero-trust" in a measurable way.
- **Action:** I broke ambiguity into pillars: (1) Identity — mTLS + SPIFFE; (2) Policy — OPA Gatekeeper + Kyverno; (3) Observability — Falco + Loki; (4) Secrets — Vault. I created a maturity model (Level 1-4) and presented 3 options: "Quick Win" (Level 2, 2 weeks), "Standard" (Level 3, 6 weeks), "Gold" (Level 4, 3 months). I recommended Level 3 with a phased rollout.
- **Result:** Stakeholders chose Level 3. We delivered on time. The maturity model became the standard for 2 other teams.

### Q12. Describe a time you went above and beyond your job responsibilities.
**STAR Answer:**
- **Situation:** During RoutineOps, I noticed our junior engineers struggled with Kubernetes debugging. It wasn't in my sprint goals.
- **Task:** Upskill the team to reduce my own on-call burden and improve team resilience.
- **Action:** I created "Kubernetes Office Hours" — weekly 30-minute sessions. I built a "Chaos Engineering" namespace with intentionally broken pods (CrashLoopBackOff, ImagePullBackOff, OOMKilled) for hands-on learning. I recorded 5 short Loom videos on `kubectl debug`, ephemeral containers, and log parsing.
- **Result:** Team MTTR improved by 35%. Two juniors became on-call ready 2 months early. The videos were shared company-wide (500+ views).

### Q13. Tell me about a time you received critical feedback. How did you respond?
**STAR Answer:**
- **Situation:** My manager told me my Terraform modules were "too complex" — other engineers were afraid to touch them because of nested modules and custom providers.
- **Task:** Simplify without losing functionality.
- **Action:** I didn't defend my code. I asked for specific examples, then pair-programmed with the most junior engineer. I: (1) Flattened 3-level nesting to 2; (2) Added `README.md` with usage examples for every module; (3) Implemented `terraform-docs` auto-generation; (4) Created a "Module Complexity Linter" in CI that flagged cyclomatic complexity >10.
- **Result:** Module adoption increased 4x. The junior engineer who was afraid of Terraform became a regular contributor. I now write all IaC with "the next engineer" in mind.

### Q14. Describe a time you had to meet a tight deadline without sacrificing quality.
**STAR Answer:**
- **Situation:** Client audit in 2 weeks required Falco runtime detection, Vault secret encryption, and signed images — a 6-week project compressed.
- **Task:** Deliver all three without creating technical debt or burnout.
- **Action:** I used the MoSCoW method: Must have (Falco + Vault), Should have (Cosign signing), Could have (custom Falco rules). I leveraged existing open-source Helm charts instead of building from scratch. I automated testing with Terratest. I worked 4 hours extra per day for 1 week, then enforced a "no overtime" rule in week 2 to prevent fatigue errors.
- **Result:** Audit passed. All Must/Should haves delivered. Zero critical bugs in the first month. I documented everything so the next audit would take 3 days, not 2 weeks.

### Q15. Tell me about a time you had to influence a team to adopt a new technology.
**STAR Answer:**
- **Situation:** I wanted to replace our legacy Jenkins setup with GitOps (ArgoCD + Tekton) for PodPlate.
- **Task:** Convince 8 engineers and 1 skeptical manager to migrate a working system.
- **Action:** I ran a "GitOps Week": (1) Day 1-2: Side-by-side demo showing drift detection (Jenkins had none); (2) Day 3: Let engineers deploy their own feature branch with ArgoCD; (3) Day 4: Simulated a config error and showed automatic rollback; (4) Day 5: Presented cost analysis — Jenkins master maintenance was 10h/week vs ArgoCD's 1h/week. I made it fun with a quiz and prizes.
- **Result:** 100% team buy-in. Migration completed in 3 weeks instead of the planned 8. Drift incidents dropped to zero.

---

<a name="section-2"></a>
## SECTION 2: SRE FUNDAMENTALS & RELIABILITY ENGINEERING — 10 QUESTIONS

### Q16. What are SLIs, SLOs, and SLAs? Give a real example from your experience.
**Answer:**
- **SLI (Service Level Indicator):** A quantitative measure of service quality. Example: "HTTP request latency < 200ms" or "Pod CPU utilization."
- **SLO (Service Level Objective):** A target for SLIs over time. Example: "99.9% of requests have latency < 200ms over a 30-day window."
- **SLA (Service Level Agreement):** A business contract with penalties. Example: "If uptime < 99.5%, customer receives 10% service credit."

**My Experience:** For PodPlate, I defined 15 custom SLIs: (1) p99 checkout latency; (2) Order processing success rate; (3) Kubernetes pod restart rate. Our SLO was 99.9% uptime with a 0.1% error budget. When we burned 80% of the budget in 2 weeks due to a memory leak, we instituted a feature freeze until the SLO recovered.

### Q17. What is an error budget and how do you use it?
**Answer:**
Error budget = 100% − SLO target. If SLO is 99.9%, error budget = 0.1% downtime/year (~8.76 hours).

**Usage:**
1. **Feature velocity vs. reliability trade-off:** If budget is healthy, ship features. If depleted, freeze launches and focus on reliability.
2. **Risk assessment:** A deployment that historically causes 0.01% errors is "safe" if we have 0.1% budget left.
3. **Blameless post-mortems:** Every error budget burn triggers a post-mortem, not blame.

**Scenario:** We burned 60% of our budget in 1 week due to a bad rollout. We: (1) Halted non-critical releases for 2 weeks; (2) Invested in canary deployments with Flagger; (3) Added automated rollback on error-rate >0.5%.

### Q18. Explain the difference between a service being available and being reliable.
**Answer:**
- **Available:** The service responds to requests (even if slowly or incorrectly). A service returning 500s is "available" but not reliable.
- **Reliable:** The service behaves correctly, consistently, and predictably according to user expectations.

**Example:** An API with 99.99% uptime but 30-second latency during peak hours is available but unreliable. Google SRE cares about "reliability," not just uptime. We measure both: availability (200 OK rate) and latency (p50/p99).

### Q19. What is toil? How do you identify and eliminate it?
**Answer:**
**Toil:** Manual, repetitive, automatable operational work that scales linearly with service growth.

**Identification:**
- Tasks performed >2 times/week
- Tasks that are reactive (incident response) vs. proactive (improvement)
- Tasks with low cognitive value (clicking buttons, running scripts manually)

**Elimination Strategy:**
1. **Automate:** Terraform for infra provisioning, ArgoCD for deployments, Ansible for config management.
2. **Self-service:** Give developers a portal to spin up review environments.
3. **Eliminate root cause:** If pods crash weekly due to memory leaks, fix the leak, not just restart the pod.

**My Example:** Environment provisioning took 3 days (manual VPC, IAM, EKS setup). I reduced it to 20 minutes with Terraform modules + Atlantis for PR-based infrastructure.

### Q20. Describe the concept of "cascading failures" and how to prevent them.
**Answer:**
**Cascading Failure:** When one failing service overloads its dependencies, causing them to fail, which then causes their dependencies to fail — a domino effect.

**Prevention:**
1. **Circuit Breakers:** If payment-service fails, checkout-service returns a cached response instead of retrying infinitely.
2. **Bulkheads:** Isolate critical paths. Use separate node pools for payment vs. catalog services.
3. **Rate Limiting:** Istio EnvoyFilter to limit requests per client.
4. **Graceful Degradation:** If recommendations-service is down, show generic "Popular Items" instead of personalized recommendations.
5. **Timeouts & Retries:** Set aggressive timeouts (e.g., 2s) with exponential backoff and jitter.

**Scenario:** Our catalog-service once DDoS'd the inventory-service with retries during a sale. I implemented Istio circuit breakers (max connections: 100, max pending: 50) and HPA on inventory. Cascade stopped.

### Q21. What is the difference between monitoring and observability?
**Answer:**
- **Monitoring:** Asking known questions of known data. "Is CPU > 80%?" "Is the pod running?" Dashboards and alerts for known failure modes.
- **Observability:** Ability to ask unknown questions of unknown data. "Why is latency high for users in Warsaw using Chrome?" Requires structured logs, distributed traces, and high-cardinality metrics.

**The Three Pillars:**
1. **Metrics (Prometheus):** Aggregated over time, good for trends. "Error rate over 6 hours."
2. **Logs (Loki):** Discrete events. Good for "what happened to this specific request?"
3. **Traces (Tempo/Jaeger):** Request path through microservices. Good for "where is the bottleneck?"

**My Stack:** Prometheus + Grafana for metrics, Loki for logs, and Istio telemetry for traces. I also use "exemplars" in Prometheus to link metrics to traces.

### Q22. How do you approach capacity planning for a Kubernetes cluster?
**Answer:**
1. **Baseline:** Measure current resource usage (CPU, memory, disk, network) over 30 days using Prometheus.
2. **Growth Rate:** Calculate month-over-month growth. If requests grew 20% last month, plan for 20%+ buffer.
3. **Headroom:** Maintain 30% spare capacity for spikes and failures. Never run above 70% utilization.
4. **Node Pool Strategy:**
   - General workloads: Spot instances (cost savings)
   - Critical workloads: On-demand with reserved capacity
   - GPU/ARM: Separate node pools
5. **Autoscaling:**
   - **HPA (Horizontal Pod Autoscaler):** Scale pods based on CPU/memory/custom metrics.
   - **VPA (Vertical Pod Autoscaler):** Right-size container requests/limits.
   - **Cluster Autoscaler:** Scale nodes based on pending pod capacity.

**Scenario:** For PodPlate, I used KEDA (Kubernetes Event-Driven Autoscaling) to scale based on Kafka queue depth during lunch/dinner rushes, reducing costs by 40% during off-peak hours.

### Q23. What is a "blameless post-mortem" and why is it important?
**Answer:**
A blameless post-mortem is a document and meeting that focuses on *what* happened and *how* to prevent it, not *who* caused it.

**Structure:**
1. **Timeline:** Exact sequence of events (alerts, actions, communications).
2. **Impact:** Duration, affected users, revenue impact.
3. **Root Cause:** Use 5 Whys. "Pod crashed → OOMKilled → Memory leak → Missing limit in code → No memory profiling in CI."
4. **Action Items:** Specific, assigned, with deadlines. Not "be more careful" but "Add memory limit testing in CI."
5. **Lessons Learned:** What went well? What went poorly?

**Why Important:** If people fear blame, they hide mistakes. If mistakes are hidden, systems don't improve. Google's error budget policy *requires* post-mortems for SLO breaches.

### Q24. Explain the "Four Golden Signals" of monitoring.
**Answer:**
From Google's SRE book:
1. **Latency:** Time to serve a request. Distinguish between successful request latency and failed request latency (failures can be fast but still bad).
2. **Traffic:** Demand on the system. Requests per second, active users, bandwidth.
3. **Errors:** Rate of failed requests. HTTP 500s, TCP timeouts, business-logic failures.
4. **Saturation:** How "full" the service is. CPU%, memory%, disk I/O, thread pool utilization. Saturation often precedes latency spikes.

**My Implementation:** I have a "Golden Signals" Grafana dashboard for every service with these 4 panels, plus a "Saturation Forecast" panel using linear regression to predict when we'll hit 70% capacity.

### Q25. How do you decide between horizontal scaling and vertical scaling?
**Answer:**
| Factor | Horizontal Scaling | Vertical Scaling |
|--------|-------------------|------------------|
| **State** | Stateless services | Stateful services (databases) |
| **Cost** | Cheaper long-term (commodity hardware) | Expensive (high-memory instances) |
| **Limit** | Near-infinite (with good architecture) | Hardware ceiling (e.g., 768GB RAM max) |
| **Complexity** | High (load balancing, data consistency) | Low (just resize) |
| **Downtime** | Zero (add nodes) | Required (resize instance) |

**Decision Tree:**
- **Stateless microservice (e.g., API server):** Horizontal (HPA).
- **Database with strong consistency needs:** Vertical first, then read-replicas (horizontal reads).
- **Memory-bound batch job:** Vertical (larger node) or use Karpenter for right-sizing.

**My Rule:** Default to horizontal for Kubernetes workloads. Use VPA in " recommendation" mode to suggest right-sizing, then apply manually during maintenance windows.

---

<a name="section-3"></a>
## SECTION 3: LINUX, NETWORKING & SYSTEMS — 10 QUESTIONS

### Q26. A user reports they can't reach a service on port 443. Walk me through your debugging steps.
**Answer:**
1. **Local validation:** `curl -v https://localhost:443` from the host itself. If this works, the service is running.
2. **Network path:** `ping` the host from the client. If ping fails, check routing tables (`ip route`) and security groups.
3. **Port accessibility:** `telnet host 443` or `nc -zv host 443` from the client. If this fails but ping works, it's a firewall/ACL issue.
4. **Process check:** `ss -tlnp | grep 443` or `netstat -tlnp` to verify the process is listening on 0.0.0.0:443, not 127.0.0.1:443.
5. **Firewall:** `iptables -L -n | grep 443` or check `firewalld`/`ufw`. Check AWS Security Group inbound rules.
6. **DNS:** `dig +short service.example.com` — is it resolving to the right IP?
7. **Proxy/Istio:** If behind a service mesh, check `istioctl proxy-config listener pod-name` for port bindings.
8. **TLS:** `openssl s_client -connect host:443` — is the certificate valid? Is it expired?

**Scenario:** I once found that an Istio sidecar was binding to port 443 before the application, causing a conflict. Changed the app to 8443 and updated the VirtualService.

### Q27. What happens when you type `https://google.com` in your browser? Explain every step.
**Answer:**
1. **URL Parsing:** Browser extracts protocol (HTTPS), host (google.com), path (/).
2. **DNS Resolution:**
   - Check browser cache → OS cache (`/etc/hosts`, `systemd-resolved`) → DNS resolver (8.8.8.8).
   - Query: Root DNS (.) → TLD DNS (.com) → Authoritative DNS (google.com) → IP address (e.g., 142.250.185.78).
3. **TCP Handshake:** SYN → SYN-ACK → ACK to port 443.
4. **TLS Handshake (1.3):**
   - ClientHello (supported cipher suites, key share)
   - ServerHello + EncryptedExtensions + Certificate + Finished
   - Client Finished
   - Key exchange using ECDHE → symmetric session keys.
5. **HTTP Request:** `GET / HTTP/1.1
Host: google.com
...`
6. **Server Processing:** Load balancer → Edge cache → Application server → Database query → HTML generation.
7. **Response:** HTML, CSS, JS. Browser renders DOM tree, CSSOM, layout, paint, composite.
8. **Connection:** HTTP/2 multiplexing allows multiple streams over one TCP connection. HTTP/3 uses QUIC over UDP.

**SRE Angle:** At each step, something can fail. DNS TTL expiry, TCP SYN flood, TLS certificate expiry, HTTP 502 from bad upstream. I monitor each layer separately.

### Q28. Explain the Linux boot process.
**Answer:**
1. **BIOS/UEFI:** Power On Self Test (POST), finds bootable device.
2. **Bootloader (GRUB2):** Loads kernel (`vmlinuz`) and initramfs into memory. User selects kernel if multiple exist.
3. **Kernel:** Decompresses, initializes hardware drivers, mounts root filesystem (read-only initially).
4. **Initramfs:** Temporary root filesystem with essential drivers (e.g., disk controller) to mount the real root FS.
5. **Init System (systemd):**
   - `systemd` becomes PID 1.
   - Loads units: `sysinit.target` → `basic.target` → `multi-user.target` (CLI) or `graphical.target`.
   - Starts services in parallel based on dependencies.
6. **Login:** `getty` processes on TTYs, or SSH daemon for remote access.

**Troubleshooting:** If a node fails to boot, I attach the EBS volume to another instance, check `/var/log/boot.log` and `journalctl -xb`, fix `/etc/fstab` errors (common cause).

### Q29. What is the difference between a process and a thread? When would you use multiple threads vs. multiple processes?
**Answer:**
- **Process:** Independent execution unit with its own memory space (heap, stack, code). Isolated. Communication via IPC (pipes, sockets, shared memory).
- **Thread:** Lightweight unit within a process. Shares the same memory space. Communication is direct (shared variables).

| Use Case | Multiple Processes | Multiple Threads |
|----------|-------------------|------------------|
| **Isolation** | High (one crash doesn't affect others) | Low (one thread crash kills process) |
| **Memory** | High overhead (separate address spaces) | Low overhead (shared memory) |
| **Communication** | Slow (IPC) | Fast (shared memory) |
| **GIL (Python)** | Bypass GIL limitation | Bound by GIL |
| **Use When** | Running different apps (e.g., nginx + app) | Parallel tasks within same app (e.g., web server handling requests) |

**SRE Context:** In Kubernetes, I prefer multi-process architectures (sidecar pattern) for observability (separate Prometheus exporter) because if the main app OOMs, the sidecar can still report the failure.

### Q30. Explain how Linux memory management works. What is the difference between RSS and VSZ?
**Answer:**
- **VSZ (Virtual Memory Size):** Total address space allocated to the process. Includes shared libraries, memory-mapped files, and allocated-but-unused memory. Can be much larger than physical RAM.
- **RSS (Resident Set Size):** Actual physical RAM the process is using. Excludes swapped-out memory and unallocated virtual pages.

**Memory Management:**
1. **Virtual Memory:** Each process sees a contiguous address space. MMU maps virtual addresses to physical addresses via page tables.
2. **Pages:** Memory is divided into 4KB pages. Active pages stay in RAM; inactive pages go to swap.
3. **OOM Killer:** When RAM + Swap are exhausted, the kernel scores processes (`oom_score`) and kills the highest one (usually the largest memory consumer, unless adjusted).
4. **cgroups (v1/v2):** In Kubernetes, `memory.limit_in_bytes` enforces the container memory limit. If exceeded, OOMKilled.

**Scenario:** A Java app had VSZ=8GB but RSS=2GB. Developers panicked. I explained that JVM reserves heap upfront (VSZ) but uses it gradually (RSS). The real issue was RSS spiking to the 4GB limit, causing OOMKills. We tuned `-Xmx` and added VPA.

### Q31. What is a Linux namespace and how does Docker use it?
**Answer:**
Linux namespaces isolate resources between processes. Docker uses:
1. **PID Namespace:** Process ID isolation. PID 1 in container is separate from host PID 1.
2. **Network Namespace:** Separate network stack (interfaces, routing tables, iptables). Container gets its own `eth0`.
3. **Mount Namespace:** Filesystem isolation. Container sees its own rootfs (`/`), not host `/`.
4. **UTS Namespace:** Hostname isolation. Container can have its own hostname.
5. **IPC Namespace:** Shared memory isolation. Prevents container A from accessing container B's shared memory.
6. **User Namespace:** UID/GID isolation. Root in container (UID 0) maps to non-root on host (e.g., UID 100000).
7. **Cgroup Namespace:** Isolates cgroup mount point view.

**Security:** User namespaces are critical for rootless containers. Even if an attacker escapes the container, they are non-root on the host.

### Q32. Explain TCP vs UDP. When would you choose one over the other?
**Answer:**
| Feature | TCP | UDP |
|---------|-----|-----|
| **Connection** | Connection-oriented (3-way handshake) | Connectionless |
| **Reliability** | Guaranteed delivery, ordering, retransmission | Best-effort, no guarantee |
| **Overhead** | High (headers, ACKs, congestion control) | Low (8-byte header) |
| **Use Case** | HTTP/HTTPS, SSH, databases | DNS, video streaming, gaming, metrics (StatsD) |
| **Flow Control** | Yes (sliding window) | No |

**SRE Scenarios:**
- **TCP:** Kubernetes API server, application HTTP traffic, database connections.
- **UDP:** DNS queries (port 53), Prometheus `statsd_exporter`, Istio telemetry (partially), QUIC (HTTP/3 uses UDP).

**Troubleshooting:** If UDP packets are dropped, there's no retransmission. I use `tcpdump -i eth0 udp port 53` to verify DNS packets reach the resolver.

### Q33. What is a reverse proxy vs. a load balancer? How do they work together?
**Answer:**
- **Load Balancer:** Distributes traffic across multiple backend servers (L4/L7). Focus: scalability and availability. Examples: AWS ALB/NLB, Nginx, HAProxy.
- **Reverse Proxy:** Sits in front of servers, forwarding client requests. Focus: security, caching, SSL termination. Examples: Nginx, Envoy, Traefik.

**Overlap:** Most modern reverse proxies (Envoy, Nginx) also load balance.

**How they work together in Kubernetes:**
1. **External LB (AWS ALB):** Distributes internet traffic across Worker Nodes (L4).
2. **Ingress Controller (Nginx/ALB):** Reverse proxy + LB inside cluster. Terminates TLS, routes `/api` to service A, `/web` to service B.
3. **Service Mesh (Istio Envoy sidecar):** Reverse proxy per pod. Handles mTLS, retries, circuit breaking, load balancing (least_request, round_robin).

**My Setup:** AWS ALB → Istio Ingress Gateway → Envoy sidecar → App container. Three layers of load balancing for resilience.

### Q34. Explain how DNS works inside a Kubernetes cluster.
**Answer:**
1. **CoreDNS:** Runs as a Deployment (usually 2+ replicas) in `kube-system`. Service IP is the cluster's `--cluster-dns` (e.g., 10.96.0.10).
2. **Pod DNS Config:** Kubelet injects `/etc/resolv.conf` into every pod:
   ```
   nameserver 10.96.0.10
   search default.svc.cluster.local svc.cluster.local cluster.local
   options ndots:5
   ```
3. **Resolution Flow:**
   - Pod queries `my-service` → CoreDNS checks Kubernetes API → returns ClusterIP.
   - Pod queries `my-service.other-ns` → CoreDNS resolves cross-namespace.
   - Pod queries `google.com` → CoreDNS forwards to upstream DNS (e.g., 8.8.8.8).
4. **Service Types:**
   - **ClusterIP:** DNS A record = ClusterIP.
   - **Headless:** DNS A record = Pod IPs directly (for StatefulSets).
   - **ExternalName:** DNS CNAME to external domain.

**Troubleshooting:** If DNS resolution fails, I check: (1) CoreDNS pod logs; (2) `ndots:5` causing excessive lookups (optimize to FQDN); (3) NetworkPolicy blocking UDP 53; (4) Istio sidecar DNS interception (`istio-proxy` captures DNS).

### Q35. You SSH into a node and see load average is 50. The node has 8 CPUs. What does this mean and what do you check?
**Answer:**
Load average 50 on 8 CPUs means there are, on average, 50 processes in the run queue or waiting for I/O. This is severely overloaded.

**Investigation:**
1. **CPU-bound or I/O-bound?** Check `vmstat 1`:
   - High `us` (user) + `sy` (system) = CPU bound.
   - High `wa` (wait) = Disk I/O bound.
   - High `st` (steal) = Noisy neighbor on shared VM (AWS T-class instances).
2. **Top processes:** `top -bn1 | head -20` or `pidstat -u 1` to find CPU hogs.
3. **I/O specifics:** `iostat -xz 1` for disk latency. `iotop` for per-process I/O.
4. **Memory pressure:** `free -m`. If low memory, check `dmesg | grep -i "out of memory"` for OOM kills.
5. **Runaway processes:** A fork bomb? `ps aux | wc -l` — unusually high process count?
6. **Kubernetes context:** `kubectl top node` — which pod is the culprit? Check if HPA is failing to scale due to `maxReplicas` limit.

**Action:** If it's a runaway pod, `kubectl drain` the node gracefully (respect PDBs), then cordon it. If it's a system process (e.g., `kubelet` bug), restart the service or replace the node.

---

<a name="section-4"></a>
## SECTION 4: KUBERNETES, CONTAINERS & SERVICE MESH — 12 QUESTIONS

### Q36. A pod is stuck in `Pending` state. What are all possible reasons and how do you debug?
**Answer:**
1. **Insufficient Resources:** `kubectl describe pod` → "Insufficient cpu" or "Insufficient memory."
   - Fix: Scale cluster (Cluster Autoscaler), reduce resource requests, or check for resource quotas.
2. **PVC Not Bound:** Pod needs storage but PVC is unbound (no PV available or StorageClass misconfigured).
   - Fix: `kubectl get pvc`, check StorageClass provisioner (e.g., EBS CSI driver).
3. **Node Selector/Affinity:** Pod has `nodeSelector` or `affinity` rules that no node satisfies.
   - Fix: `kubectl get nodes --show-labels`, adjust labels or relax affinity.
4. **Taints/Tolerations:** Nodes are tainted (e.g., `dedicated=gpu:NoSchedule`) and pod lacks toleration.
   - Fix: Add toleration or remove taint.
5. **Image Pull Issues:** If combined with `ImagePullBackOff`, but pure `Pending` is usually scheduling.
6. **Network Policy:** Rare, but if CNI plugin (Calico/Cilium) isn't ready, pod stays pending.

**Debug Commands:**
```bash
kubectl describe pod <name>  # Events section is gold
kubectl get events --sort-by='.lastTimestamp' | grep <pod-name>
kubectl logs -n kube-system deployment/cluster-autoscaler  # If CA is failing
```

**Scenario:** I had a pod pending because AWS AZ `1c` was out of `m5.large` capacity. Cluster Autoscaler couldn't scale. I added a fallback `nodeAffinity` to `1a` and `1b`, and diversified instance types with Karpenter.

### Q37. Explain the complete lifecycle of a pod from `kubectl apply` to `Running`.
**Answer:**
1. **API Server:** `kubectl` sends YAML to API server. Authenticated (IAM/IRSA) and authorized (RBAC).
2. **etcd:** API server persists the Pod spec to etcd.
3. **Scheduler:** Watches for unbound pods. Scores nodes based on resources, affinity, taints. Binds pod to node.
4. **Kubelet:** On the chosen node, kubelet watches API server and sees the pod assignment.
5. **CRI (Container Runtime):** Kubelet instructs containerd/Docker to pull the image.
6. **CNI:** Container runtime calls CNI plugin (e.g., AWS VPC CNI) to allocate IP and attach ENI.
7. **Init Containers:** Run sequentially. If any fail, pod restarts (based on `restartPolicy`).
8. **PostStart Hook:** Executes if defined.
9. **Probes:**
   - **Startup Probe:** Disables liveness/readiness until app starts (protects slow-starting apps).
   - **Liveness Probe:** Restarts container if unhealthy.
   - **Readiness Probe:** Removes pod from Service endpoints if not ready.
10. **Running:** Pod is ready to serve traffic.

**Istio Sidecar Injection:** If enabled, `istio-init` container runs first (sets up iptables rules), then `istio-proxy` sidecar starts alongside the app container.

### Q38. What is the difference between a Deployment, StatefulSet, and DaemonSet?
**Answer:**
| Feature | Deployment | StatefulSet | DaemonSet |
|---------|-----------|-------------|-----------|
| **Use Case** | Stateless apps (API servers, web apps) | Stateful apps (databases, Kafka) | Node-level agents (monitoring, log collection) |
| **Pod Identity** | Random hash names, interchangeable | Ordinal names (db-0, db-1), sticky identity | One per node |
| **Storage** | Shared PVC (RWO issues) | Unique PVC per pod (via volumeClaimTemplates) | Usually hostPath or local storage |
| **Ordering** | Random rollout | Ordered rollout (0→N), ordered termination (N→0) | N/A |
| **Service** | ClusterIP/LoadBalancer | Headless Service for DNS identity | Usually hostNetwork |

**My Usage:**
- **Deployment:** Microservices (checkout, catalog).
- **StatefulSet:** Redis cluster, PostgreSQL primary-replica.
- **DaemonSet:** Falco (runtime security), Fluent Bit (log shipping), Node Exporter (metrics).

### Q39. How does Kubernetes handle service discovery and load balancing?
**Answer:**
**Service Discovery:**
1. **Environment Variables:** Kubelet injects `MY_SERVICE_SERVICE_HOST` and `MY_SERVICE_SERVICE_PORT` into pods. Unreliable if service created after pod.
2. **DNS (CoreDNS):** Preferred method. `my-service.my-namespace.svc.cluster.local` resolves to ClusterIP.

**Load Balancing:**
1. **ClusterIP (Layer 4):** `kube-proxy` (iptables/IPVS mode) distributes traffic across Endpoints (pod IPs).
   - **iptables:** Random selection by probability. High overhead with >1000 endpoints.
   - **IPVS:** Layer 4 load balancing, better performance, supports least_conn, round_robin.
2. **Service Mesh (Layer 7):** Istio/Envoy does HTTP routing, retries, circuit breaking, weighted traffic splitting (canary).

**Headless Service:** No ClusterIP. DNS returns pod IPs directly. Client does load balancing (e.g., Cassandra drivers).

**My Optimization:** I switched `kube-proxy` from iptables to IPVS on clusters with >50 services. Latency dropped 15% due to reduced iptables chain traversal.

### Q40. Explain how Istio mTLS works and why you implemented it.
**Answer:**
**How it works:**
1. **Citadel/Istiod:** Acts as Certificate Authority (CA). Issues X.509 certificates to every service account.
2. **Sidecar Injection:** `istio-proxy` container is injected into every pod. It has its own certificate.
3. **Traffic Interception:** `istio-init` container sets up iptables rules to redirect all pod traffic through `istio-proxy`.
4. **mTLS Handshake:** When pod A calls pod B:
   - A's Envoy presents its certificate to B's Envoy.
   - B validates A's certificate against Istiod's CA.
   - B presents its certificate to A.
   - Traffic is encrypted with mutual authentication.
5. **Permissive vs Strict:**
   - **Permissive:** Accepts both plaintext and mTLS (migration mode).
   - **Strict:** Rejects plaintext. Enforced via PeerAuthentication policy.

**Why I Implemented It:**
- **Security:** Eliminated plaintext inter-service traffic. Prevented lateral movement in case of breach.
- **Identity:** Every workload has a strong identity (SPIFFE ID), enabling fine-grained authorization.
- **Observability:** Envoy generates telemetry for every request (latency, success rate, bytes) without code changes.

**Challenge:** Debugging mTLS failures is hard. I use `istioctl authn tls-check` and `istioctl proxy-config secret` to verify certificate distribution.

### Q41. What is a Helm Chart and what are its advantages over raw YAML?
**Answer:**
A Helm Chart is a package of pre-configured Kubernetes resources (templates + values).

**Structure:**
```
mychart/
  Chart.yaml       # Metadata
  values.yaml      # Default configuration
  templates/       # Go-template YAML files
  charts/          # Sub-charts (dependencies)
```

**Advantages:**
1. **Templating:** One chart deploys to dev/staging/prod by changing `values.yaml` (replicas, resources, image tag).
2. **Versioning:** Charts are versioned (SemVer). Rollback to previous version with `helm rollback`.
3. **Dependencies:** Chart can depend on PostgreSQL, Redis charts. Managed via `Chart.yaml` dependencies.
4. **Release Management:** Helm tracks releases as ConfigMaps in `kube-system`.

**vs Kustomize:** Helm is for packaging/sharing. Kustomize is for patching existing YAML for different environments. I use Helm for third-party apps (Prometheus, ArgoCD) and Kustomize for my own microservices.

### Q42. Describe how you would perform a zero-downtime deployment in Kubernetes.
**Answer:**
**Strategy 1: Rolling Update (Built-in)**
```yaml
spec:
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 25%
      maxUnavailable: 0
```
- Gradually replaces old pods with new ones.
- Requires readiness probes to prevent traffic to not-ready pods.

**Strategy 2: Blue/Green Deployment**
- Run old (blue) and new (green) versions simultaneously.
- Switch Service selector to green after validation.
- Instant rollback: switch selector back to blue.
- Cost: 2x resources during switch.

**Strategy 3: Canary Deployment (with Flagger/Argo Rollouts)**
- Deploy new version to 5% of traffic.
- Monitor golden signals (error rate, latency).
- Auto-promote to 100% if healthy; auto-rollback if not.

**My Production Flow:**
1. CI builds image, runs Trivy scan, signs with Cosign.
2. ArgoCD detects new image tag (GitOps).
3. Canary rollout via Flagger: 5% → 20% → 50% → 100% over 30 minutes.
4. Prometheus metrics gate: if p99 latency > 500ms or error rate > 0.1%, automatic rollback.
5. Post-deployment: run smoke tests against production.

### Q43. A container is being OOMKilled repeatedly. How do you diagnose and fix it?
**Answer:**
**Diagnosis:**
1. **Events:** `kubectl describe pod` → "OOMKilled" or "Container ... killed due to memory."
2. **Metrics:** `kubectl top pod` — is it hitting the limit?
3. **History:** `kubectl get pod -o yaml` → check `containerStatuses.lastState.terminated.reason: OOMKilled` and `exitCode: 137` (128 + 9 SIGKILL).
4. **Application profiling:**
   - Java: Check heap dumps, `-Xmx` vs container limit.
   - Python: Check for memory leaks (circular references), large DataFrames.
   - Go: Check for goroutine leaks or unbounded buffers.
5. **Node level:** `dmesg -T | grep -i "killed process"` — kernel logs show the exact process and its RSS.

**Fixes:**
1. **Right-size limits:** If the app genuinely needs more memory, increase `resources.limits.memory` and `requests.memory`.
2. **Fix the leak:** Use profiling tools (pprof for Go, py-spy for Python, JProfiler for Java).
3. **VPA:** Enable Vertical Pod Autoscaler in "Auto" mode to adjust requests/limits.
4. **Offload:** Move caching to Redis instead of in-memory.

**Root Cause I Found:** A Python app loaded a 2GB ML model into memory on every pod start. The limit was 1GB. I implemented an init container to download the model to a shared emptyDir volume, reducing per-pod memory by 60%.

### Q44. What are Kubernetes NetworkPolicies and how do they enhance security?
**Answer:**
NetworkPolicies are Layer 3/4 firewall rules for pods. By default, all pods in a cluster can communicate freely (zero trust violation).

**How they work:**
- Implemented by CNI plugin (Calico, Cilium, AWS VPC CNI with Network Policy support).
- Use selectors (pod labels, namespace labels) to define ingress/egress rules.

**Example:**
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: api-allow-frontend
spec:
  podSelector:
    matchLabels:
      app: api
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: frontend
    ports:
    - protocol: TCP
      port: 8080
```

**My Zero-Trust Implementation:**
1. **Default Deny:** Every namespace has a default-deny all ingress/egress policy.
2. **Explicit Allow:** Services declare exactly who can talk to them (e.g., only `payment-service` can reach `payment-db`).
3. **Egress Control:** Block internet access for pods that don't need it (prevents data exfiltration).
4. **Audit:** I use Cilium's Hubble to visualize traffic and identify missing policies before enforcing them.

### Q45. Explain the role of OPA Gatekeeper and Kyverno. Why did you use both?
**Answer:**
Both are Kubernetes policy engines, but they differ:

| Feature | OPA Gatekeeper | Kyverno |
|---------|---------------|---------|
| **Language** | Rego (declarative, powerful, steep learning curve) | YAML-native (intuitive for K8s users) |
| **Admission Control** | Validating + Mutating | Validating + Mutating + Generating |
| **Use Case** | Complex logic (e.g., "allow only images from ECR with signed digest") | Simple K8s-native rules (e.g., "require labels", "add sidecar") |
| **Ecosystem** | General-purpose (can enforce policies outside K8s) | K8s-only |

**Why I Used Both:**
- **OPA Gatekeeper:** For complex security policies: image signature verification (Cosign), blocking privileged containers, enforcing resource quotas across namespaces.
- **Kyverno:** For developer experience policies: auto-injecting labels, generating NetworkPolicies when a namespace is created, mutating pods to add Vault annotations.

**Example Policy (OPA):** Block deployment if image doesn't have `cosign.sigstore.dev/signature` annotation.
**Example Policy (Kyverno):** Generate a `NetworkPolicy` for every new namespace with default-deny rules.

### Q46. How do you secure the Kubernetes control plane?
**Answer:**
1. **API Server:**
   - Disable anonymous auth (`--anonymous-auth=false`).
   - Enable audit logging (`--audit-log-path`).
   - Use RBAC strictly. No `cluster-admin` for service accounts.
   - Enable admission controllers: `NodeRestriction`, `PodSecurityPolicy` (deprecated, use PSA), `ValidatingAdmissionWebhook`.
2. **etcd:**
   - Encrypt etcd at rest (`--encryption-provider-config`).
   - Restrict etcd to control plane nodes only. Firewall port 2379.
   - Regular backups to S3 with versioning.
3. **Kubelet:**
   - Enable authentication/authorization (`--anonymous-auth=false`, `--authorization-mode=Webhook`).
   - Rotate certificates automatically.
4. **Network:**
   - Private API server endpoint (no public access) or authorized IPs only.
   - Use AWS PrivateLink if needed.
5. **Pod Security:**
   - Enforce `restricted` Pod Security Standards (no root, read-only rootfs, drop all capabilities).
   - Use OPA/Kyverno for custom policies.
6. **Supply Chain:**
   - Sign images (Cosign), scan (Trivy), SBOM (Syft).
   - Use admission control to reject unsigned images.

**My Implementation:** For Aegis Stack, I enabled IRSA (IAM Roles for Service Accounts) so pods never used node IAM roles. I also rotated API server certificates every 90 days via cert-manager.

### Q47. What is cert-manager and how does it automate TLS in Kubernetes?
**Answer:**
cert-manager is a Kubernetes operator that automates TLS certificate provisioning and renewal.

**How it works:**
1. **Issuer/ClusterIssuer:** Defines the CA. Let's Encrypt (staging/prod), Vault PKI, or self-signed.
2. **Certificate CRD:** Defines what domains need certs and which issuer to use.
3. **ACME Challenge:** For Let's Encrypt:
   - **HTTP-01:** Creates a temporary pod/service to prove domain ownership.
   - **DNS-01:** Creates a TXT record (requires DNS provider access like Route53).
4. **Secret Storage:** Stores the TLS cert + key in a Kubernetes Secret.
5. **Auto-Renewal:** Watches expiry and renews 30 days before expiration.

**My Setup:**
- Used DNS-01 with Route53 (IRSA auth) for wildcard certs (`*.podplate.com`).
- Integrated with Istio Ingress Gateway via `CredentialName` in Gateway resource.
- Set up alerts 45 days before expiry as a failsafe.

**Troubleshooting:** If cert-manager fails, I check: (1) `Challenge` resource status; (2) cert-manager pod logs; (3) DNS propagation (`dig TXT _acme-challenge.example.com`); (4) IAM permissions for Route53.

---

<a name="section-5"></a>
## SECTION 5: CI/CD, GitOps & DevSecOps — 10 QUESTIONS

### Q48. What is GitOps and what are its principles? How did you implement it?
**Answer:**
**GitOps Principles (Weaveworks):**
1. **Declarative:** System state is described in Git (YAML, Terraform).
2. **Versioned & Immutable:** Git is the single source of truth. Rollback = `git revert`.
3. **Pulled Automatically:** Agents (ArgoCD, Flux) watch Git and apply changes.
4. **Continuously Reconciled:** If someone manually `kubectl edits`, the agent reverts it.

**My Implementation:**
- **Repo Structure:**
  ```
  gitops-repo/
    apps/
      podplate/
        base/          # Kustomize base
        overlays/
          dev/
          staging/
          prod/
    infrastructure/
      eks/
      vpc/
  ```
- **ArgoCD:** Watches `prod` overlay. Auto-sync enabled.
- **App of Apps:** One ArgoCD Application manages all other Applications.
- **Policy:** OPA Gatekeeper blocks manual kubectl apply. All changes must go through Git PR.

**Benefits:**
- Drift detection: ArgoCD shows "OutOfSync" if manual changes occur.
- Audit trail: `git log` shows who changed what and when.
- Disaster recovery: New cluster = `argocd app sync` → fully restored in minutes.

### Q49. Compare ArgoCD and Flux. Why did you choose ArgoCD?
**Answer:**
| Feature | ArgoCD | Flux |
|---------|--------|------|
| **UI** | Rich web UI for visualization | CLI-focused, limited UI (Weave GitOps) |
| **Multi-tenancy** | Built-in projects, RBAC | Requires more manual setup |
| **Helm Support** | Native, with parameter override | Native |
| **Secret Management** | Sealed Secrets, External Secrets, Vault | SOPS, External Secrets |
| **Image Updater** | ArgoCD Image Updater (separate) | Flux Image Automation (built-in) |
| **Notifications** | Built-in | Flux Alert Provider |

**Why ArgoCD:**
- **Visibility:** The UI helped non-K8s engineers understand deployments.
- **Multi-cluster:** We managed 4 clusters (dev, staging, prod, dr) from one ArgoCD instance.
- **SSO:** Integrated with Google Workspace SSO for RBAC.

**Trade-off:** ArgoCD runs inside the cluster (security risk if compromised). I mitigated this by running it in a dedicated `argocd` namespace with strict NetworkPolicies and read-only repo access.

### Q50. How do you implement supply-chain security in CI/CD? Explain your pipeline.
**Answer:**
**My 7-Stage Pipeline (Aegis Stack):**
1. **Build:** Docker image built with BuildKit. No secrets in layers (use `--secret` mount).
2. **SBOM Generation (Syft):** `syft packages docker:myimage:latest -o spdx-json > sbom.json`. Stored in S3 for audit.
3. **Vulnerability Scan (Trivy):** `trivy image --exit-code 1 --severity HIGH,CRITICAL myimage`. Fails build if CVEs found.
4. **Policy Validation (Conftest):** Validate Kubernetes YAML against OPA policies before deployment.
5. **Image Signing (Cosign):** `cosign sign --key env://COSIGN_PRIVATE_KEY myimage:latest`. Signature stored in registry.
6. **Push to Registry:** ECR with immutable tags and image scanning enabled.
7. **GitOps Deploy:** ArgoCD detects new signed image tag, syncs to cluster.

**Admission Control:** OPA Gatekeeper verifies the Cosign signature before pod creation. Unsigned images = rejected.

**Benefits:**
- Zero unsigned images in production.
- Every image has an SBOM for incident response.
- CVEs caught before deployment, not after.

### Q51. What is the difference between Tekton and GitHub Actions? When do you use each?
**Answer:**
| Feature | GitHub Actions | Tekton |
|---------|---------------|--------|
| **Runtime** | GitHub-hosted or self-hosted runners | Kubernetes-native (runs as pods) |
| **Scalability** | Limited by runner size | Unlimited (scales with cluster) |
| **Reusability** | Marketplace actions | Tekton Catalog (tasks, pipelines) |
| **Kubernetes** | Needs custom scripting | Native CRDs (Task, Pipeline, PipelineRun) |
| **Debugging** | Logs in GitHub UI | `tkn logs`, Kubernetes-native |

**My Usage:**
- **GitHub Actions:** Orchestration layer. Trigger on PR, call Tekton for heavy lifting, run lightweight jobs (lint, unit tests).
- **Tekton:** Kubernetes-native CI. Build images inside the cluster (Kaniko), run security scans with cluster network access to internal scanners.

**Integration:** GitHub Actions triggers Tekton via webhook. Tekton updates GitHub commit status via GitHub App.

### Q52. How do you manage secrets in CI/CD pipelines securely?
**Answer:**
1. **Never hardcode:** No secrets in Git. Use `git-secrets` or `truffleHog` in pre-commit hooks.
2. **Short-lived tokens:** Use OIDC (OpenID Connect) instead of long-lived AWS credentials. GitHub Actions → AWS via OIDC (no `AWS_ACCESS_KEY_ID` in secrets).
3. **Vault Integration:**
   - CI pipeline authenticates to Vault via AppRole or JWT.
   - Vault issues dynamic credentials (AWS STS tokens, DB credentials) with TTL=15 minutes.
   - Credentials auto-expire after use.
4. **Kubernetes:** Use External Secrets Operator to sync Vault secrets to K8s secrets. Or use Vault Agent Injector to mount secrets as files (not env vars).
5. **Audit:** Log every secret access in Vault. Alert on unusual patterns.

**My Implementation:**
- GitHub Actions uses OIDC to AWS.
- Tekton tasks use Vault Agent sidecar to inject Docker registry credentials.
- Production apps use IRSA (no static AWS keys at all).

### Q53. A deployment failed and rolled back. How do you investigate the root cause?
**Answer:**
1. **Identify the change:** `git log` or ArgoCD history to find what changed (image tag, configmap, env var).
2. **Check pipeline:** Did the CI tests actually pass? Was the image signed? Sometimes flaky tests let bugs through.
3. **Application logs:** `kubectl logs deployment/app --previous` (logs from failed container before restart).
4. **Events:** `kubectl get events --sort-by='.lastTimestamp'` — was it OOMKilled, ImagePullBackOff, or CrashLoopBackOff?
5. **Metrics:** Check Prometheus for the deployment window. Did memory spike? Did error rate increase before the alert fired?
6. **Diff configs:** `kubectl get deployment -o yaml | diff - <(git show HEAD:deployment.yaml)` to find unexpected changes.
7. **Canary analysis:** If using Flagger, check the canary metrics. Did the automated rollback trigger correctly? Why?
8. **Dependency check:** Did a downstream service (database, cache) fail? Use distributed traces to find the first failing span.

**Prevention:**
- Add integration tests that catch this specific failure mode.
- Improve readiness probes to catch the issue before 100% traffic.
- Add a "deployment freeze" if error budget is low.

### Q54. What are the security risks in a CI/CD pipeline and how do you mitigate them?
**Answer:**
| Risk | Mitigation |
|------|-----------|
| **Poisoned Pipeline Execution** | Pin GitHub Actions to commit SHA, not `@v1`. Use private runners for sensitive jobs. |
| **Credential Leakage** | OIDC instead of long-lived tokens. Mask secrets in logs. Rotate daily. |
| **Dependency Confusion** | Private registry (Nexus/Artifactory). Verify package signatures. Use `go.sum`, `package-lock.json`. |
| **Compromised Build Agent** | Ephemeral runners (new VM per job). Scan runner images with Trivy. |
| **Malicious Code Injection** | Require 2 reviews for PRs. Codeowners for sensitive files. Block force-push to main. |
| **Unsigned Artifacts** | Cosign signatures verified at deployment. SBOM for traceability. |
| **Over-Privileged Service Accounts** | Least privilege IAM roles. Separate roles for build vs deploy. |

**My Implementation:** I implemented a "secure supply chain" policy where every artifact must have: (1) SBOM, (2) CVE scan report, (3) Cosign signature, (4) Conftest policy validation — all stored as attestation layers in the registry.

### Q55. How do you implement canary deployments with automatic rollback?
**Answer:**
**Tools:** Argo Rollouts or Flagger.

**My Flagger Setup:**
1. **Define Canary CRD:**
   ```yaml
   targetRef:
     apiVersion: apps/v1
     kind: Deployment
     name: podplate-api
   service:
     port: 8080
   analysis:
     interval: 1m
     threshold: 5
     maxWeight: 50
     stepWeight: 10
     metrics:
     - name: request-success-rate
       thresholdRange:
         min: 99
       interval: 1m
     - name: request-duration
       thresholdRange:
         max: 500
       interval: 1m
     webhooks:
     - name: load-test
       url: http://flagger-loadtester.test/
       timeout: 5s
       metadata:
         cmd: "hey -z 1m -q 10 -c 2 http://podplate-api-canary:8080/"
   ```
2. **Traffic Splitting:** Istio VirtualService splits traffic: 90% stable, 10% canary.
3. **Metrics Analysis:** Flagger queries Prometheus every minute.
4. **Promotion:** If metrics pass for 30 minutes, 100% traffic shifts to canary. Old version scaled down.
5. **Rollback:** If error rate > 1% or p99 latency > 500ms, traffic reverts to 100% stable. Alert sent to Slack.

**Benefits:** We caught a memory leak in canary that would have caused a production outage. Rollback happened automatically in 2 minutes with zero customer impact.

### Q56. What is Infrastructure as Code (IaC) and what are Terraform's key features?
**Answer:**
**IaC:** Managing infrastructure through machine-readable definition files rather than manual configuration.

**Terraform Key Features:**
1. **Declarative:** You define *what* you want (EKS cluster with 3 nodes), not *how* to create it.
2. **State Management:** `terraform.tfstate` tracks real-world resources. Enables plan/apply workflow.
3. **Execution Plans:** `terraform plan` shows exactly what will change before applying. No surprises.
4. **Resource Graph:** Understands dependencies (VPC must exist before EKS). Parallelizes independent resources.
5. **Providers:** Extensible. AWS, GCP, Azure, Kubernetes, Vault, etc.
6. **Modules:** Reusable components. `module "eks" { source = "./modules/eks" }`.

**My Best Practices:**
- Remote state in S3 with DynamoDB locking.
- Atlantis for PR-based Terraform plans (no local `terraform apply` to prod).
- `terraform-docs` for module documentation.
- `tflint` + `tfsec` (now Trivy) in CI for linting and security scanning.

### Q57. How do you manage Terraform state in a team environment?
**Answer:**
1. **Remote Backend:** Store state in S3 (encrypted with KMS) + DynamoDB for state locking. Prevents concurrent modifications.
2. **State Separation:**
   - **Per-environment:** `prod.tfstate`, `staging.tfstate` via workspace or separate backend configs.
   - **Per-layer:** VPC layer, EKS layer, App layer. Reduces blast radius.
3. **State Locking:** DynamoDB table with `LockID` ensures only one `terraform apply` runs at a time.
4. **State Encryption:** S3 bucket with SSE-KMS. Versioning enabled for audit.
5. **Access Control:** IAM roles restrict who can read/write state. Sensitive outputs (passwords) marked `sensitive = true`.
6. **State Drift Detection:** Nightly `terraform plan` in CI. If drift detected, alert Slack.

**My Setup:**
- **Atlantis:** Every PR runs `terraform plan`. Comments appear in GitHub. Approval required before `apply`.
- **Workspace:** `terraform workspace select prod` for environment isolation.
- **Modules:** Shared modules in a separate repo, versioned via Git tags.

### Q58. Explain the concept of "drift" in IaC. How do you detect and remediate it?
**Answer:**
**Drift:** When real-world infrastructure diverges from Terraform state. Example: Someone manually added an ingress rule in AWS Security Group via console.

**Detection:**
1. **Terraform Plan:** `terraform plan` shows drift as "changes to be made."
2. **Continuous:** Run `terraform plan` in CI every 6 hours. Use tools like Driftctl or Terraform Cloud.
3. **Alerting:** If drift detected, post to Slack with the exact resource and attribute.

**Remediation:**
1. **Automatic (GitOps):** Some tools auto-revert drift. I prefer manual review to avoid fighting intentional emergency changes.
2. **Manual:** Import the change (`terraform import`) or remove it (`terraform apply` to revert).
3. **Prevention:**
   - IAM policies restricting console access.
   - AWS Config rules detecting manual changes.
   - Tagging resources with `ManagedBy=Terraform`.

**Scenario:** A developer opened port 22 on a production SG for debugging and forgot to close it. Drift detection caught it in 4 hours. We revoked the rule and updated the runbook to use AWS Systems Manager Session Manager instead of SSH.

---

<a name="section-6"></a>
## SECTION 6: AWS, IaC & TERRAFORM — 8 QUESTIONS

### Q59. Design a highly available Kubernetes cluster on AWS. What components do you need?
**Answer:**
1. **VPC & Networking:**
   - Multi-AZ (3 AZs minimum).
   - Public subnets (ALB, NAT GW) + Private subnets (EKS nodes).
   - VPC CNI for pod networking (custom networking for IP conservation).
2. **EKS Control Plane:**
   - Managed by AWS, automatically multi-AZ.
   - Enable control plane logging (API, audit, authenticator).
   - Private endpoint + restricted public endpoint.
3. **Worker Nodes:**
   - Managed Node Groups or Karpenter for autoscaling.
   - Multi-AZ distribution using `topologySpreadConstraints`.
   - Spot instances for non-critical workloads (cost savings).
4. **Load Balancing:**
   - AWS ALB (Layer 7) for HTTP traffic. Ingress Controller.
   - NLB (Layer 4) for TCP/SSL passthrough.
5. **Storage:**
   - EBS CSI Driver for block storage (StatefulSets).
   - EFS for shared storage (ReadWriteMany).
6. **DNS:**
   - Route53 private hosted zone for internal services.
   - ExternalDNS to sync K8s Ingress to Route53.
7. **Security:**
   - IRSA for pod-level IAM permissions.
   - AWS WAF on ALB for DDoS/SQL injection protection.
   - Security Groups for pod-to-pod traffic (VPC CNI SG support).
8. **Backup:**
   - Velero for cluster resource and PV backup to S3.
   - AWS Backup for managed services (RDS, etc.).

### Q60. What is IRSA (IAM Roles for Service Accounts) and why is it better than node IAM roles?
**Answer:**
**IRSA:** Allows a Kubernetes service account to assume an AWS IAM role via OIDC federation.

**How it works:**
1. EKS cluster has an OIDC provider URL.
2. Create IAM role with trust policy allowing the OIDC provider to assume it for a specific service account (e.g., `system:serviceaccount:prod:app`).
3. Annotate the K8s service account: `eks.amazonaws.com/role-arn: arn:aws:iam::123:role/AppRole`.
4. When pod starts, EKS injects AWS STS token via OIDC. Pod uses temporary credentials.

**vs Node IAM Role:**
- **Node Role:** Every pod on the node gets the node's IAM permissions. If one pod is compromised, it can access all AWS resources the node can. Violates least privilege.
- **IRSA:** Per-pod, per-service-account permissions. Compromised pod can only access its specific S3 bucket, not others.

**My Implementation:** I eliminated all node IAM policies except ECR pull and SSM access. Every app uses IRSA. I also enforce this with OPA Gatekeeper: pods without `eks.amazonaws.com/role-arn` annotation cannot access AWS APIs.

### Q61. How do you design a VPC for a multi-region SaaS application?
**Answer:**
1. **IP Planning:**
   - Use non-overlapping CIDRs per region: `10.0.0.0/16` (us-east-1), `10.1.0.0/16` (eu-west-1).
   - Reserve `/20` per AZ within the region.
2. **Subnets:**
   - Public: ALB, NAT GW, Bastion (if needed).
   - Private: Application tier (EKS nodes, Lambda).
   - Database: Isolated subnets. No internet access. AWS PrivateLink for S3/DynamoDB if needed.
3. **Connectivity:**
   - **Inter-Region:** AWS Transit Gateway or VPC Peering. TGW is preferred for >2 VPCs.
   - **On-Prem:** AWS Direct Connect or VPN over TGW.
4. **DNS:**
   - Route53 private hosted zones associated with all VPCs.
   - Resolver endpoints for hybrid DNS.
5. **Security:**
   - AWS Network Firewall or Palo Alto VM-Series for east-west traffic inspection.
   - VPC Flow Logs to S3 for audit.
   - Security Groups referencing other SGs (not IP addresses) for dynamic environments.
6. **EKS Specifics:**
   - Enable custom networking if pod density > ENI limits.
   - Prefix delegation for high-density clusters.

### Q62. What are the differences between EBS, EFS, and S3? When do you use each in Kubernetes?
**Answer:**
| Feature | EBS | EFS | S3 |
|---------|-----|-----|-----|
| **Type** | Block storage | NFS (file) | Object storage |
| **Access** | Single AZ, single attachment (ReadWriteOnce) | Multi-AZ, multi-attach (ReadWriteMany) | HTTP API |
| **Performance** | High IOPS (io2), low latency | Good throughput, higher latency | High throughput, high latency |
| **K8s Use** | Databases (PostgreSQL), stateful apps | Shared config, WordPress, ML models | Backups, static assets, logs |
| **CSI Driver** | EBS CSI Driver | EFS CSI Driver | Mountpoint S3 CSI Driver |

**My Usage:**
- **EBS:** PostgreSQL StatefulSet with `volumeClaimTemplates`. Snapshots via AWS Backup.
- **EFS:** Shared model weights for ML inference pods (multiple pods read same data).
- **S3:** Application logs archived after 30 days in Loki. Static website assets. Terraform state backend.

### Q63. How does AWS Lambda fit into a DevOps/SRE workflow?
**Answer:**
**Use Cases:**
1. **Event-Driven Automation:**
   - Trigger on CloudWatch Alarm → post to Slack/Teams.
   - Trigger on S3 upload → run virus scan with ClamAV.
   - Trigger on EC2 termination → deregister from monitoring tools.
2. **Infrastructure Cleanup:**
   - Nightly Lambda to find and delete untagged EBS volumes, old AMIs, unused IAM roles.
   - Saves ~30% on cloud costs.
3. **Security Response:**
   - GuardDuty finding → Lambda isolates the EC2 instance (removes security group rules).
   - Lambda rotates exposed IAM keys via Vault.
4. **GitOps Webhooks:**
   - API Gateway + Lambda to receive GitHub webhooks and trigger Tekton pipelines.

**My Implementation:** I built a "Cloud Janitor" Lambda in Python that runs nightly, identifies resources missing `Owner` and `Environment` tags, notifies teams via SNS, and deletes them after 7 days if unclaimed. Reduced our AWS bill by $4,000/month.

### Q64. Explain AWS Auto Scaling groups vs. Kubernetes Cluster Autoscaler vs. Karpenter.
**Answer:**
| Feature | ASG | Cluster Autoscaler | Karpenter |
|---------|-----|-------------------|-----------|
| **Level** | EC2 instances | Kubernetes nodes | Kubernetes nodes |
| **Trigger** | CloudWatch metrics (CPU, custom) | Pending pods (unschedulable) | Pending pods |
| **Speed** | Minutes | 2-3 minutes | 15-30 seconds |
| **Instance Types** | Fixed Launch Template | Mixed Instance Policy (limited) | Any instance type (auto-selection) |
| **Consolidation** | No | No | Yes (moves pods, terminates underutilized nodes) |
| **Spot** | Yes | Yes | Yes (native) |

**My Choice:**
- **Cluster Autoscaler:** Good for stable workloads, predictable instance types.
- **Karpenter:** Better for variable workloads, batch jobs, or when you want "just-in-time" right-sized nodes. I use Karpenter for dev clusters (cost optimization) and CA for prod (predictability).

**Karpenter NodePool:**
```yaml
requirements:
  - key: node.kubernetes.io/instance-type
    operator: In
    values: ["m5.large", "m5.xlarge", "m6i.large"]
  - key: karpenter.sh/capacity-type
    operator: In
    values: ["spot", "on-demand"]
```

### Q65. How do you secure data at rest and in transit on AWS?
**Answer:**
**At Rest:**
1. **EBS/S3:** Server-side encryption (SSE-S3, SSE-KMS, SSE-C). I prefer SSE-KMS with customer-managed keys for audit trails.
2. **RDS:** Enable encryption at creation. Use AWS KMS.
3. **Secrets:** AWS Secrets Manager or HashiCorp Vault. Rotate automatically.
4. **etcd:** EKS encrypts etcd by default. For self-managed, use `encryption-provider-config`.
5. **Backups:** Encrypt Velero backups in S3 with KMS.

**In Transit:**
1. **TLS 1.2+:** ALB with ACM certificates. cert-manager in EKS.
2. **mTLS:** Istio service mesh for pod-to-pod encryption.
3. **VPN/Direct Connect:** IPsec for on-prem to AWS traffic.
4. **PrivateLink:** Access AWS services (S3, DynamoDB) without traversing the public internet.

**My Implementation:** All S3 buckets have bucket policies enforcing encryption. I use AWS Config rule `s3-bucket-server-side-encryption-enabled` to detect non-compliant buckets.

### Q66. What is AWS Organizations and how does it help with multi-account strategy?
**Answer:**
**AWS Organizations:** Central management for multiple AWS accounts with consolidated billing and policy enforcement.

**Multi-Account Strategy:**
1. **Security:** Blast radius isolation. Compromised dev account cannot access production.
2. **Cost:** Track spending per team/environment. Use SCPs to restrict expensive services in dev.
3. **Governance:** Service Control Policies (SCPs) enforce rules (e.g., "no EC2 in us-west-1", "must use MFA for root").
4. **Networking:** Shared VPCs via RAM (Resource Access Manager). Centralized egress via Transit Gateway.

**My Setup:**
- **Master Account:** Billing only. No workloads.
- **Security Account:** GuardDuty findings, Security Hub, AWS Config aggregators.
- **Shared Services:** ECR, Route53, Directory Service.
- **Prod/Staging/Dev:** Isolated workloads. Cross-account IAM roles for controlled access.
- **Sandbox:** For R&D with spending limits via SCP.

---

<a name="section-7"></a>
## SECTION 7: OBSERVABILITY, MONITORING & INCIDENT RESPONSE — 8 QUESTIONS

### Q67. Design a monitoring stack for a microservices platform. What do you monitor at each layer?
**Answer:**
**Stack:** Prometheus (metrics) + Grafana (visualization) + Loki (logs) + Alertmanager (alerting) + Jaeger/Tempo (traces).

**Layer 1: Infrastructure (Node/Cluster)**
- CPU, memory, disk I/O, network throughput.
- Node temperature, hardware failures (if bare metal).
- Kubernetes: Node ready status, pod count, API server latency.

**Layer 2: Container/Pod**
- Container CPU/memory usage vs. requests/limits.
- Container restart count, OOMKills.
- Image age (stale images = security risk).

**Layer 3: Application (Golden Signals)**
- Request rate, error rate, latency (p50/p99), saturation.
- Business metrics: orders/minute, payment success rate, cart abandonment.
- JVM heap usage, GC pauses, thread count.

**Layer 4: Dependency**
- Database connection pool usage, query latency, replication lag.
- Cache hit/miss rate (Redis).
- External API latency and error rate.

**Layer 5: User Experience**
- Real User Monitoring (RUM): page load time, JS errors.
- Synthetic monitoring: Pingdom/Grafana Cloud synthetic checks.

**My Dashboards:**
- "Cluster Health": Node status, resource utilization, pending pods.
- "Service Overview": 4 Golden Signals per microservice.
- "Business KPIs": Orders, revenue, active users (from application metrics).

### Q68. What is the difference between logs, metrics, and traces? When do you use each?
**Answer:**
| Feature | Logs | Metrics | Traces |
|---------|------|---------|--------|
| **Data** | Discrete events (text) | Numeric aggregates over time | Request path through services |
| **Cardinality** | High (every request is unique) | Low (pre-defined labels) | Medium (trace ID + span IDs) |
| **Use Case** | "What happened to request ID abc?" | "Is error rate increasing?" | "Where is the bottleneck?" |
| **Storage** | Expensive (raw text) | Cheap (time-series) | Moderate (sampled) |
| **Example** | `ERROR: payment failed for user=123` | `http_requests_total{status="500"}` | Span: `checkout-service` → `payment-service` (45ms) |

**Correlation:**
- Metrics alert: "Error rate > 1%."
- Logs investigate: "Which specific errors?"
- Traces pinpoint: "Which service in the chain is failing?"

**My Implementation:** I use Prometheus exemplars to link a metric timestamp to a trace ID. When I see a latency spike in Grafana, I click the exemplar to jump to the exact trace in Jaeger.

### Q69. How do you reduce alert fatigue? What makes a good alert?
**Answer:**
**Good Alert Criteria (Google SRE):**
1. **Actionable:** The recipient can do something about it. "Disk 90% full" = actionable. "CPU spiked for 10s" = not actionable.
2. **Noisy < 1/year:** If it fires weekly and is always benign, tune or delete it.
3. **Severity Appropriate:** Page someone only if revenue is at risk or data loss is imminent.
4. **Context-Rich:** Include runbook link, recent deployments, related dashboards.

**Reducing Fatigue:**
1. **Alert Hierarchy:**
   - **Critical (Page):** SLO breach, data loss, security incident.
   - **Warning (Ticket):** Capacity planning, non-urgent degradation.
   - **Info (Log):** Interesting but not actionable.
2. **Aggregation:** Group related alerts. "Multiple services in namespace X are failing" instead of 10 separate alerts.
3. **Dynamic Thresholds:** Use anomaly detection (e.g., latency > 3 standard deviations from weekly baseline) instead of static thresholds.
4. **Shift-Left:** Catch issues in CI (Trivy scan failures) rather than production alerts.

**My Result:** I inherited a system with 200 alerts/week. After pruning and tuning, we had 8 meaningful alerts/week. On-call satisfaction improved dramatically.

### Q70. You get paged at 3 AM: "Database latency p99 > 2s." What's your runbook?
**Answer:**
1. **Acknowledge:** PageDuty acknowledged within 2 minutes.
2. **Assess Impact:**
   - Check Grafana: Is it affecting users? What's the error rate?
   - Check SLO dashboard: Are we burning error budget?
3. **Quick Mitigation (if needed):**
   - If write latency: Enable read replicas, redirect reads. If necessary, enable maintenance mode.
   - If connection pool exhausted: Temporarily increase `max_connections` (risky but buys time).
4. **Diagnose:**
   - **Database:** `SHOW PROCESSLIST` (MySQL) or `pg_stat_activity` (PostgreSQL). Long-running queries? Locks?
   - **Metrics:** Did QPS spike? Is it a specific query pattern?
   - **Recent Changes:** Did a deployment introduce an N+1 query?
   - **Infrastructure:** Is the EBS volume hitting IOPS limits? Check CloudWatch `VolumeQueueLength`.
5. **Fix:**
   - Kill blocking queries.
   - Add missing index (if safe to do online).
   - Scale RDS instance class (if IOPS bound).
6. **Verify:** Error rate drops, latency normalizes.
7. **Post-Incident:** Schedule blameless post-mortem within 24 hours.

**Prevention:** I set up slow query log aggregation in Loki and automated `pt-query-digest` reports weekly.

### Q71. What is distributed tracing and how do you implement it in a service mesh?
**Answer:**
**Distributed Tracing:** Follows a request across multiple microservices, showing latency at each hop.

**Components:**
- **Trace:** End-to-end request (e.g., "User checkout").
- **Span:** Single operation within a trace (e.g., "checkout-service: validate cart" = 12ms).
- **Context Propagation:** Trace ID and parent span ID passed via HTTP headers (`x-b3-traceid`, `x-b3-spanid` for Zipkin/B3).

**Implementation with Istio:**
1. **Automatic:** Istio Envoy sidecars automatically generate spans for all HTTP/gRPC traffic.
2. **Application Integration:** Apps need to propagate headers. No code changes needed for basic tracing, but custom spans require OpenTelemetry SDK.
3. **Backend:** Jaeger or Tempo collects spans. Grafana visualizes traces.
4. **Sampling:** Head-based (decide at request start) or tail-based (decide after completion, better for catching rare errors but higher storage).

**My Setup:**
- Istio sends traces to Tempo via OTLP.
- Sampling rate: 10% in production (100% would be too expensive).
- Custom spans in Python apps using OpenTelemetry for database queries.

### Q72. How do you use Loki effectively without breaking the bank on storage?
**Answer:**
1. **Label Cardinality:** Low-cardinality labels only (`namespace`, `pod`, `level`). Avoid `user_id` or `request_id` as labels — they create too many streams.
2. **Structured Logging:** JSON logs with `logfmt` or `json` parser in Loki. Query: `{app="api"} | json | level="error"`.
3. **Retention:**
   - Hot storage (SSD): 7 days for debugging.
   - Cold storage (S3): 30-90 days for compliance.
   - Delete after 90 days (or archive to Glacier).
4. **Compaction & Index:** Use BoltDB-Shipper or TSDB index for better query performance.
5. **Alerting:** Don't query Loki directly for alerts (slow). Use Promtail metrics pipeline to extract metrics from logs (e.g., count ERROR lines → Prometheus metric).
6. **Chunk Size:** Tune `chunk_target_size` and `max_chunk_age` to balance memory and query speed.

**My Cost Reduction:** I reduced Loki costs by 60% by: (1) dropping DEBUG logs in production; (2) using `pack` to combine small streams; (3) moving to S3 with Query Frontend caching.

### Q73. What is the difference between push and pull monitoring? What are the trade-offs?
**Answer:**
| Feature | Pull (Prometheus) | Push (Datadog, CloudWatch, Graphite) |
|---------|------------------|-------------------------------------|
| **Architecture** | Server scrapes targets | Targets send data to server |
| **Discovery** | Service discovery (K8s, Consul) | Static configuration or agent |
| **Firewall** | Server needs access to targets | Targets need outbound access |
| **Short-lived Jobs** | Hard to scrape (needs Pushgateway) | Easy (send at end of job) |
| **Scale** | Horizontal (federated Prometheus) | Vertical (centralized collector) |
| **Security** | Targets expose /metrics endpoint | API keys/secrets on every target |

**My Hybrid Approach:**
- **Pull:** Prometheus for Kubernetes (native support, service discovery).
- **Push:** Datadog agent for bare metal/legacy VMs. Custom batch jobs push to Prometheus Pushgateway.

### Q74. How do you perform a chaos engineering experiment safely?
**Answer:**
1. **Hypothesis:** "If the payment database fails, the checkout service will gracefully degrade to cached payment methods."
2. **Blast Radius:** Limit to 5% of traffic or staging environment first.
3. **Abort Conditions:** Automatic rollback if error rate > 0.5% or p99 latency > 1s.
4. **Run Experiment:**
   - Use LitmusChaos or Chaos Mesh.
   - Kill a PostgreSQL pod: `kubectl delete pod postgres-0`.
   - Network partition: Block traffic between checkout and payment.
5. **Observe:** Monitor golden signals. Did graceful degradation work? Did alerts fire correctly?
6. **Learn:** Document findings. Fix gaps (e.g., circuit breaker timeout was too long).
7. **Expand:** Gradually increase blast radius after success.

**My Experiment:** I ran a "Friday Failure Injection" where I randomly killed pods in staging. We discovered that our Redis client had no connection timeout, causing 30s hangs. Fixed to 2s timeout with fallback.

---

<a name="section-8"></a>
## SECTION 8: CODING, SCRIPTING & AUTOMATION — 5 QUESTIONS

### Q75. Write a Python script to find unused IAM roles in AWS.
**Answer:**
```python
import boto3
from datetime import datetime, timedelta

def find_unused_iam_roles(days_unused=90):
    iam = boto3.client('iam')
    cloudtrail = boto3.client('cloudtrail')
    cutoff = datetime.now() - timedelta(days=days_unused)

    unused_roles = []

    # Paginate through all roles
    paginator = iam.get_paginator('list_roles')
    for page in paginator.paginate():
        for role in page['Roles']:
            role_name = role['RoleName']

            # Skip AWS service-linked roles
            if role['Path'].startswith('/aws-service-role/'):
                continue

            # Check last activity via CloudTrail (simplified)
            # Better: Use IAM Access Analyzer or last_used from iam.get_role
            try:
                role_details = iam.get_role(RoleName=role_name)['Role']
                last_used = role_details.get('RoleLastUsed', {})

                if 'LastUsedDate' not in last_used:
                    # Never used - check creation date
                    if role['CreateDate'].replace(tzinfo=None) < cutoff:
                        unused_roles.append({
                            'name': role_name,
                            'reason': 'Never used',
                            'created': role['CreateDate'].isoformat()
                        })
                elif last_used['LastUsedDate'].replace(tzinfo=None) < cutoff:
                    unused_roles.append({
                        'name': role_name,
                        'reason': f"Last used {last_used['LastUsedDate'].isoformat()}",
                        'created': role['CreateDate'].isoformat()
                    })
            except Exception as e:
                print(f"Error checking {role_name}: {e}")

    return unused_roles

if __name__ == "__main__":
    roles = find_unused_iam_roles()
    for role in roles:
        print(f"DELETE_CANDIDATE: {role['name']} | {role['reason']} | Created: {role['created']}")
```

**Improvements:**
- Use IAM Access Analyzer for comprehensive last-accessed data.
- Add SNS notification instead of print.
- Tag roles with `DeletionCandidate=true` instead of deleting immediately.
- Run as Lambda with EventBridge schedule.

### Q76. Write a Bash script to diagnose why a Kubernetes pod is crashing.
**Answer:**
```bash
#!/bin/bash
set -euo pipefail

POD_NAME=${1:-}
NAMESPACE=${2:-default}

if [ -z "$POD_NAME" ]; then
    echo "Usage: $0 <pod-name> [namespace]"
    exit 1
fi

echo "=== Pod Status ==="
kubectl get pod "$POD_NAME" -n "$NAMESPACE" -o wide

echo -e "\n=== Events ==="
kubectl get events -n "$NAMESPACE" --field-selector involvedObject.name="$POD_NAME" --sort-by='.lastTimestamp'

echo -e "\n=== Pod Description ==="
kubectl describe pod "$POD_NAME" -n "$NAMESPACE"

echo -e "\n=== Container Logs ==="
# Try current container
if kubectl logs "$POD_NAME" -n "$NAMESPACE" --tail=50 2>/dev/null; then
    echo "(Current logs shown above)"
else
    echo "No current logs available."
fi

# Try previous container (if crashed)
echo -e "\n=== Previous Container Logs (if restarted) ==="
if kubectl logs "$POD_NAME" -n "$NAMESPACE" --previous --tail=50 2>/dev/null; then
    echo "(Previous logs shown above)"
else
    echo "No previous container logs."
fi

# Resource usage
echo -e "\n=== Resource Usage ==="
kubectl top pod "$POD_NAME" -n "$NAMESPACE" 2>/dev/null || echo "metrics-server not available"

# Check if OOMKilled
STATUS=$(kubectl get pod "$POD_NAME" -n "$NAMESPACE" -o jsonpath='{.status.containerStatuses[0].lastState.terminated.reason}' 2>/dev/null || echo "")
if [ "$STATUS" == "OOMKilled" ]; then
    echo -e "\n[ALERT] Pod was OOMKilled! Consider increasing memory limits."
    echo "Current limits:"
    kubectl get pod "$POD_NAME" -n "$NAMESPACE" -o jsonpath='{.spec.containers[0].resources.limits.memory}'
    echo ""
fi

# Check node conditions
echo -e "\n=== Node Status ==="
NODE=$(kubectl get pod "$POD_NAME" -n "$NAMESPACE" -o jsonpath='{.spec.nodeName}')
kubectl describe node "$NODE" | grep -A 5 "Conditions"
```

### Q77. Write a Terraform module for an S3 bucket with security best practices.
**Answer:**
```hcl
variable "bucket_name" {
  type = string
}

variable "environment" {
  type = string
}

resource "aws_s3_bucket" "this" {
  bucket = var.bucket_name

  tags = {
    Name        = var.bucket_name
    Environment = var.environment
    ManagedBy   = "Terraform"
  }
}

resource "aws_s3_bucket_versioning" "this" {
  bucket = aws_s3_bucket.this.id
  versioning_configuration {
    status = "Enabled"
  }
}

resource "aws_s3_bucket_server_side_encryption_configuration" "this" {
  bucket = aws_s3_bucket.this.id
  rule {
    apply_server_side_encryption_by_default {
      sse_algorithm     = "aws:kms"
      kms_master_key_id = aws_kms_key.this.arn
    }
    bucket_key_enabled = true
  }
}

resource "aws_s3_bucket_public_access_block" "this" {
  bucket = aws_s3_bucket.this.id
  block_public_acls       = true
  block_public_policy     = true
  ignore_public_acls      = true
  restrict_public_buckets = true
}

resource "aws_s3_bucket_policy" "this" {
  bucket = aws_s3_bucket.this.id
  policy = jsonencode({
    Version = "2012-10-17"
    Statement = [
      {
        Sid       = "EnforceTLS"
        Effect    = "Deny"
        Principal = "*"
        Action    = "s3:*"
        Resource  = [
          aws_s3_bucket.this.arn,
          "${aws_s3_bucket.this.arn}/*"
        ]
        Condition = {
          Bool = {
            "aws:SecureTransport" = "false"
          }
        }
      }
    ]
  })
  depends_on = [aws_s3_bucket_public_access_block.this]
}

resource "aws_kms_key" "this" {
  description             = "KMS key for ${var.bucket_name}"
  deletion_window_in_days = 30
  enable_key_rotation     = true

  tags = {
    Environment = var.environment
  }
}

output "bucket_arn" {
  value = aws_s3_bucket.this.arn
}

output "kms_key_id" {
  value = aws_kms_key.this.id
}
```

### Q78. Write a Python function to parse Kubernetes pod status and alert on issues.
**Answer:**
```python
from kubernetes import client, config
from typing import List, Dict
import datetime

def check_pod_health(namespace: str = "default") -> List[Dict]:
    config.load_kube_config()  # or config.load_incluster_config()
    v1 = client.CoreV1Api()

    alerts = []
    pods = v1.list_namespaced_pod(namespace=namespace)

    for pod in pods.items:
        pod_name = pod.metadata.name

        # Check phase
        if pod.status.phase not in ["Running", "Succeeded"]:
            alerts.append({
                "pod": pod_name,
                "issue": f"Pod in {pod.status.phase} state",
                "severity": "critical" if pod.status.phase == "Failed" else "warning",
                "timestamp": datetime.datetime.utcnow().isoformat()
            })
            continue

        # Check container statuses
        if pod.status.container_statuses:
            for container in pod.status.container_statuses:
                # Check restarts
                if container.restart_count > 5:
                    alerts.append({
                        "pod": pod_name,
                        "container": container.name,
                        "issue": f"High restart count: {container.restart_count}",
                        "severity": "warning",
                        "timestamp": datetime.datetime.utcnow().isoformat()
                    })

                # Check waiting state
                if container.state.waiting:
                    reason = container.state.waiting.reason
                    if reason in ["CrashLoopBackOff", "ImagePullBackOff", "ErrImagePull"]:
                        alerts.append({
                            "pod": pod_name,
                            "container": container.name,
                            "issue": f"Container waiting: {reason}",
                            "severity": "critical",
                            "timestamp": datetime.datetime.utcnow().isoformat()
                        })

                # Check terminated state
                if container.state.terminated and container.state.terminated.reason == "OOMKilled":
                    alerts.append({
                        "pod": pod_name,
                        "container": container.name,
                        "issue": "Container OOMKilled - increase memory limits",
                        "severity": "critical",
                        "timestamp": datetime.datetime.utcnow().isoformat()
                    })

        # Check age (if pod is very old, might need restart for updates)
        age = datetime.datetime.utcnow() - pod.status.start_time.replace(tzinfo=None)
        if age.days > 30:
            alerts.append({
                "pod": pod_name,
                "issue": f"Pod age {age.days} days - consider rolling restart",
                "severity": "info",
                "timestamp": datetime.datetime.utcnow().isoformat()
            })

    return alerts

# Usage
if __name__ == "__main__":
    issues = check_pod_health(namespace="prod")
    for alert in issues:
        print(f"[{alert['severity'].upper()}] {alert['pod']}: {alert['issue']}")
```

### Q79. Write a Makefile target for common Terraform workflows.
**Answer:**
```makefile
.PHONY: init plan apply destroy fmt validate docs

ENV ?= dev
TF_DIR ?= ./terraform/$(ENV)

init:
	cd $(TF_DIR) && terraform init -backend-config="backend-$(ENV).hcl"

plan:
	cd $(TF_DIR) && terraform plan -out=tfplan

apply:
	cd $(TF_DIR) && terraform apply tfplan

destroy:
	cd $(TF_DIR) && terraform destroy

fmt:
	terraform fmt -recursive

validate:
	cd $(TF_DIR) && terraform validate

docs:
	terraform-docs markdown table --output-file README.md --output-mode inject $(TF_DIR)

security:
	trivy config $(TF_DIR)
	checkov -d $(TF_DIR)

lint: fmt validate security
	@echo "All checks passed"

clean:
	find . -type d -name ".terraform" -exec rm -rf {} +
	find . -name "*.tfstate*" -delete
	find . -name "tfplan" -delete
```

---

<a name="section-9"></a>
## SECTION 9: GOOGLE INTERVIEW STRATEGY & TIPS

### 🎯 Understanding Google's Interview Bar
Google SRE interviews assess:
1. **Problem-Solving:** Can you break down ambiguous problems systematically?
2. **Technical Depth:** Do you understand *why* things work, not just *how*?
3. **Communication:** Can you explain trade-offs clearly?
4. **Googleyness:** Intellectual humility, collaboration, user focus.

### 📐 The "Google SRE" Answer Framework
For technical questions, use **THIS** structure:
1. **Clarify:** Ask questions. "What's the scale? What's the budget? What's the SLO?"
2. **Design:** High-level first. "I'd use a multi-region setup with..."
3. **Deep Dive:** Pick one component and go deep. "For the database layer, I'd choose... because..."
4. **Trade-offs:** "The alternative is X, but I chose Y because of Z."
5. **Operationalize:** "For monitoring, I'd set SLIs for... Alerts would fire when..."

### 🗣️ Behavioral Questions — The STAR Method
Google expects specific, recent examples:
- **Situation:** One sentence. Context.
- **Task:** Your specific responsibility.
- **Action:** What *YOU* did (not "we"). Use "I designed...", "I implemented..."
- **Result:** Quantify. "Reduced MTTR by 40%", "Saved $4K/month".

### ⚠️ Common Mistakes
1. **Being too vague:** "I fixed the issue" → Bad. "I found the OOMKill via dmesg and increased limits from 512Mi to 1Gi" → Good.
2. **Not discussing trade-offs:** Every decision has a cost. Mention it.
3. **Ignoring the "Why":** Google cares about reasoning. "I chose Terraform because..."
4. **Not asking clarifying questions:** If asked "Design a URL shortener," ask about scale, latency requirements, and lifespan.

### 🔁 The "What Would You Do Differently?" Question
Always have an answer ready. Google loves growth mindset:
- "Looking back, I should have implemented canary deployments from day one instead of rolling updates. We had one bad rollout that caused a 15-minute outage."
- "I should have invested more in automated testing. Our CI caught only 60% of issues; I later increased this to 90% with integration tests."

### 📚 Recommended Study Resources
- **Book:** *Site Reliability Engineering* (Google) — Read the first 10 chapters.
- **Book:** *The Site Reliability Workbook* — For practical implementation.
- **Book:** *Building Secure & Reliable Systems* (Google) — For security questions.
- **Practice:** LeetCode Easy/Medium for Python. Focus on string manipulation, parsing, and basic algorithms.
- **System Design:** "Designing Data-Intensive Applications" (Martin Kleppmann).

### 🎤 Mock Interview Checklist
Before the interview, ensure you can clearly explain:
- [ ] Your resume's top 3 projects with metrics (latency, cost, uptime improvements).
- [ ] One major failure and what you learned (blameless post-mortem style).
- [ ] One time you disagreed with someone and how you resolved it.
- [ ] How you'd design a system from your resume at 10x scale.
- [ ] Your monitoring/alerting philosophy (alert fatigue, SLOs).

---

## ✅ FINAL CHECKLIST: 79 Questions Covered
| Section | Topic | Count |
|---------|-------|-------|
| 1 | Google Behavioral (STAR) | 15 |
| 2 | SRE Fundamentals | 10 |
| 3 | Linux, Networking, Systems | 10 |
| 4 | Kubernetes, Containers, Service Mesh | 12 |
| 5 | CI/CD, GitOps, DevSecOps | 10 |
| 6 | AWS, IaC, Terraform | 8 |
| 7 | Observability, Monitoring, Incidents | 8 |
| 8 | Coding, Scripting, Automation | 6 |
| **Total** | | **79** |

---

*Good luck with your Google SRE interview, Abhijit! Your resume shows strong hands-on experience with zero-trust security, GitOps, and observability — exactly what Google SRE teams value. Focus on telling stories with metrics, and remember: Google interviews are conversations, not interrogations. Show your curiosity and collaborative spirit.*
