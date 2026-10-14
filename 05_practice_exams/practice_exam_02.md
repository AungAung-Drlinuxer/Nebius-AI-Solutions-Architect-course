# Practice Exam 02 — Nebius AI Solutions Architect

Design-scenario multiple-choice practice exam. Every question has exactly
**one** correct answer and three distractors. Choose the *most appropriate*
design choice for the stated requirement.

**Time limit: 20 minutes.**

---

## Domain 1 — Design High-Performing Architectures

**1.** A startup serves a 3B-parameter embedding model with steady traffic and
a strict monthly budget. Profiling shows the model fits comfortably on a
single mid-range GPU. Which GPU platform design is most appropriate?

- A. Right-size the GPU instance type to the model's memory and compute needs instead of choosing the largest available GPUs
- B. Deploy the model on the largest GPU tier available so future models can also fit
- C. Run the model on CPU-only instances and accept multi-second latency
- D. Run eight GPUs behind every replica to maximize theoretical throughput

**2.** A data-science team of 20 runs interactive experiments: notebooks,
small-scale fine-tuning, and dataset exploration. They need shared datasets,
reproducible environments, and quick onboarding for new students each
semester. Which user environment design is most appropriate?

- A. Each user installs their own CUDA drivers and Python packages directly on a shared host
- B. A shared Windows workstation with one giant conda environment everyone edits
- C. A managed notebook-based environment with prebuilt, versioned container images and shared dataset mounts
- D. USB drives distributed with the datasets and no central environment

**3.** A company's platform team already operates Kubernetes microservices and
wants to add batch training jobs and GPU inference services using the same
tooling, CI/CD, and monitoring. Which orchestration design is most
appropriate?

- A. Replace Kubernetes with a second, separate orchestration system just for AI workloads
- B. Run GPU workloads on standalone VMs with cron jobs, outside the cluster
- C. Give each data scientist their own single-node Kubernetes cluster
- D. Extend the existing Kubernetes platform with GPU node groups and workload-specific scheduling for training and inference

**4.** An ML team must run 200 short hyperparameter-sweep jobs (each under 2
hours) on 64 GPUs this week. Each job fits on one node. Which distributed
training design is most appropriate?

- A. Schedule the jobs as many independent data-parallel single-node runs across the GPUs, with a job queue managing placement
- B. Combine all 200 jobs into one giant model-parallel run across all 64 GPUs
- C. Run the 200 jobs strictly one at a time on the full cluster to keep things simple
- D. Run each sweep job on a single CPU core to avoid touching the GPU quota

**5.** A recommendation platform loads 300 GB of model artifacts into memory at
startup and scales to 50 inference replicas during peak. Which storage design
for the model artifacts is most appropriate?

- A. Bake 300 GB of artifacts into every replica's OS image and rebuild images for each model update
- B. Store artifacts in each replica's ephemeral scratch space, retraining if a replica restarts
- C. Keep artifacts in centralized object storage, with replicas pulling on startup and caching locally to speed cold starts
- D. Serve artifacts from one replica's local disk and have all other replicas fetch from it over the internal network

## Domain 2 — Design Resilient Architectures

**6.** A team runs many short fine-tuning jobs (30–90 minutes each) alongside a
few week-long runs on the same cluster. Which checkpoint policy best fits?

- A. One global checkpoint interval (every 12 hours) for every job, long or short
- B. No checkpointing for any job; both job types are short enough to restart
- C. Checkpoint every training step for every job regardless of cost
- D. Checkpoint frequency scaled per job type — frequent lightweight checkpoints for long runs, lighter or end-of-job checkpoints for short runs

**7.** After a migration, training throughput dropped 40% with no error logs.
The architect realizes nobody tracks GPU utilization, data-loader wait time,
or job progress. Which observability design addresses this gap?

- A. Metrics for GPU utilization, data pipeline wait, and job progress, plus logs and alerting on training slowdowns
- B. More disk space for storing application stdout
- C. A daily manual screenshot of the training dashboard
- D. Nothing — throughput regressions are always caused by the model code

**8.** A business is migrating a customer-facing inference API to Nebius and
must decide when the new platform is "done". Which validation and acceptance
plan is most appropriate?

- A. Cut over on day one and validate by watching the customer support inbox
- B. Acceptance when the Terraform apply finishes without errors
- C. Define SLOs (latency, error rate, throughput), run a canary/parallel run against the old system, benchmark against baseline, then sign off against documented criteria
- D. Acceptance when a single load test runs for five minutes on one replica

**9.** An e-commerce team runs GPU inference that must survive a full zone
degradation during a sales event. Which failure-handling design is most
appropriate?

- A. Depend on a single node group in one zone and hope the event passes
- B. Suspend the service and notify customers to come back tomorrow
- C. Keep a fully idle second cluster with no data replication for emergencies
- D. Redundant node capacity across zones (or a tested failover path), health checks, and a runbook that triggers traffic shift automatically

**10.** After a consulting engagement, the customer's 4-person platform team
must own and operate the delivered system. Which handover design is most
appropriate?

- A. Deliver runbooks, architecture diagrams, known-issues list, and operator training sessions, then verify ownership with a shadow/assist period
- B. Hand over root credentials and a one-paragraph summary email
- C. Leave the system in the consultant's cloud account indefinitely
- D. Refuse to document anything so the customer must keep the consulting contract

## Domain 3 — Design Efficient & Cost-Aware Architectures

**11.** A game studio needs GPU capacity for a two-week live-ops AI feature
launch, then usage drops to near zero. Which capacity and quota strategy is
most appropriate?

- A. Buy three-year reserved capacity for the full spike
- B. Keep zero quota and expect on-demand capacity to appear instantly at launch
- C. Rent the maximum quota permanently and accept idle cost year-round
- D. Request time-bounded quota increases matched to the event, then scale back and release capacity after the launch window

**12.** Infrastructure changes are reviewed in chat messages and applied
manually; two outages last quarter came from unreviewed edits. Which
automation approach most effectively prevents this?

- A. Longer chat threads with more approvers
- B. Version-controlled IaC (Terraform) with pull-request review and policy checks before apply
- C. A shared password to the console so everyone can fix things fast
- D. Freeze all infrastructure changes forever

**13.** Finance cannot tell which of six ML teams caused last month's GPU cost
spike. Which FinOps design fixes this?

- A. Send one invoice to the whole company and discuss it at the all-hands
- B. Disable cost reporting so teams are not blamed
- C. Consistent resource tagging/labeling (team, project, environment) feeding per-team cost allocation dashboards and budget alerts
- D. Divide the bill by six regardless of usage

**14.** A pharma company has a fixed 6-month budget and a hard deadline for a
drug-discovery training run that needs a stable GPU baseline for the whole
period, plus bursts of extra jobs. Which cost-vs-performance design is most
appropriate?

- A. Secure a committed baseline of capacity for the steady workload and handle bursts with flexible capacity that scales with the budget
- B. Run everything on the most expensive capacity type available to minimize risk
- C. Skip the baseline and buy all capacity as one big burst on the last week
- D. Extend the deadline by six months to run everything on leftover capacity

**15.** A company's dev/test GPU environments run 24/7 but are actively used
only during office hours, burning half the GPU budget. Which right-sizing
choice is most appropriate?

- A. Move dev/test to production-sized GPU fleets for consistency
- B. Delete dev/test entirely and develop in production
- C. Give every developer their own permanent multi-GPU environment
- D. Right-size dev/test instances and schedule them to suspend or scale down outside working hours, keeping shared ephemeral capacity for the rest

## Domain 4 — Design Secure Architectures

**16.** A security policy requires emergency "break-glass" access to production
clusters when the normal IAM path fails. Which design is most appropriate?

- A. Give every on-call engineer permanent admin rights so the emergency never needs special handling
- B. A sealed break-glass procedure with individually attributable accounts, time-boxed elevation, strong audit logging, and post-incident review
- C. A shared break-glass password in an unencrypted wiki page
- D. No emergency access at all — if IAM fails, the system stays down

**17.** A CI/CD system must deploy to the cluster and read secrets, but the
security team forbids exposing the cluster API or secrets endpoints to the
public internet. Which connectivity design is most appropriate?

- A. Allowlist the CI/CD provider's entire public IP range on the public API forever
- B. Let the CI/CD system push artifacts by SSH into production nodes directly
- C. Private connectivity between the CI/CD runners and the cluster (private endpoints/peering), with the runners themselves in an isolated network segment and short-lived credentials
- D. Put the cluster API behind a public reverse proxy with basic auth

**18.** A platform team needs to rotate database credentials used by production
training pipelines without causing outages. Which secrets design is most
appropriate?

- A. Store credentials in a managed secret store, use overlapping validity windows (dual-credential rotation), and roll consumers gradually with automation
- B. Rotate manually once a year and announce it in a team meeting
- C. Hardcode the new credential in the application image and redeploy everything at once
- D. Stop rotating credentials so nothing can break

**19.** Two business units share one Kubernetes GPU cluster for cost reasons
but handle data of different sensitivity levels. Which isolation design is
most appropriate?

- A. Same namespace for both units so scheduling is simpler
- B. Isolation by node hostname convention only
- C. One shared volume for both units with folder-name separation
- D. Separate namespaces per unit with network policies, RBAC scoping, resource quotas, and storage separation matched to data sensitivity

**20.** A financial-services customer must prove to auditors how a suspected
security incident on their AI platform was detected and handled. Which design
is most appropriate?

- A. Rely on individual memory and verbal reports during the audit
- B. Documented incident-response plan with defined roles, alerting triggers, log retention sufficient for forensics, and post-incident reports kept as audit evidence
- C. Delete all logs older than 24 hours to reduce storage cost
- D. Handle incidents differently each time so attackers cannot predict the response

---

## Self-scoring

- Time limit: **20 minutes**. Stop the clock when you finish question 20.
- Check your answers in `practice_exam_02_answers.md`.
- Scoring bands (out of 20):
  - **17–20** — exam-ready; review only the questions you missed.
  - **13–16** — solid base; re-study the domains where you lost points.
  - **9–12** — revisit the course modules for the four domains, then retry.
  - **0–8** — reread the course material before attempting another practice exam.
