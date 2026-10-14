# Practice Exam 01 — Nebius AI Solutions Architect

Design-scenario multiple-choice practice exam. Every question has exactly
**one** correct answer and three distractors. Choose the *most appropriate*
design choice for the stated requirement — not just a technically possible one.

**Time limit: 20 minutes.** Read each scenario's constraints (workload, team,
budget, deadline) carefully; the correct answer depends on them.

---

## Domain 1 — Design High-Performing Architectures

**1.** A research team of 12 engineers wants to fine-tune a 70B-parameter LLM
across 32 GPU nodes with a 6-week deadline. Training throughput is the top
priority and the budget allows a dedicated cluster. Which design choice is most
appropriate for the training fabric?

- A. Connect all nodes over the public internet with WireGuard tunnels
- B. Attach every node to shared network block storage so gradients pass through disk
- C. Run all 32 GPUs inside a single VM with software-emulated networking
- D. Use a high-bandwidth, low-latency interconnect (InfiniBand/RDMA) between GPU nodes

**2.** A team trains computer-vision models that read a 40 TB image dataset
repeatedly for weeks, and writes checkpoints every 30 minutes. Which storage
architecture best fits this workload?

- A. Keep the dataset only in object storage and stream every sample over HTTP for each epoch
- B. Put the hot dataset and checkpoints on fast shared file/block storage; keep a durable copy of checkpoints in object storage
- C. Store the entire dataset on each node's local boot disk, copied per node
- D. Store checkpoints only on the coordinator node's local disk

**3.** A university lab with mixed experience (students, researchers, one
platform engineer) wants a managed training environment where users launch
multi-GPU jobs without managing Kubernetes objects. Which platform choice best
matches the workload and the team?

- A. A Slurm-based managed cluster with job queues and scheduler policies
- B. A self-managed bare Kubernetes cluster where every user writes YAML manifests
- C. One long-lived SSH VM per user with GPUs assigned manually
- D. A shared single-GPU workstation with tmux sessions

**4.** An ML team serves an interactive coding assistant with a strict
latency SLO, but traffic peaks during work hours and drops overnight. Which
deployment shape is most appropriate?

- A. One oversized GPU VM running a fixed batch job nightly
- B. A training cluster repurposed to answer requests only when idle
- C. An inference service on autoscaling GPU node capacity behind a load balancer, sized to the latency SLO
- D. A single spot instance the team SSHes into and restarts manually

**5.** A team must train a model too large to fit in the memory of one GPU
node, with 16 nodes available and a fixed 3-week deadline. Which distributed
training design is most appropriate?

- A. Split the dataset across nodes and train 16 independent full copies of the model (pure data parallelism) and average weights once at the end
- B. Combine model/layer sharding across nodes with data parallelism, using the cluster's fast interconnect
- C. Offload all model weights to CPU RAM and run single-GPU training with swap
- D. Train on one node only and extend the deadline

## Domain 2 — Design Resilient Architectures

**6.** A team runs a 10-day pretraining job on 64 GPUs. Each restart without
checkpointing loses up to 12 hours of compute. Which checkpoint/restart
strategy is most appropriate?

- A. Periodic asynchronous checkpoints to durable shared storage, with restart automation that resumes from the last good checkpoint
- B. Checkpoint once at the very end of training only
- C. Write checkpoints to the local disk of each worker only
- D. Disable checkpointing to maximize step throughput

**7.** The platform team must design observability for a new GPU training
platform. Stakeholders need to debug slow training, detect node failures, and
trace a user request through the inference API. Which observability scope is
most appropriate?

- A. Application logs only, collected on demand via SSH
- B. Billing dashboards only
- C. A single health-check ping per node group
- D. Logs, metrics, and traces together: GPU/node metrics, job logs, and request traces with alerting

**8.** Before handing a new training platform to its users, the architect must
define an acceptance plan. Which plan is most appropriate?

- A. Acceptance based only on the sales demo environment working once
- B. A validation and benchmark plan: correctness checks, throughput benchmarks against a baseline, failure-injection drills, and documented acceptance criteria
- C. Acceptance after a single unit test suite passes on one node
- D. Acceptance once the invoices arrive

**9.** A customer runs 24/7 fine-tuning jobs and needs an operational support
model for failures at 3 a.m. Which design is most appropriate?

- A. Users discover failures the next morning and restart jobs manually
- B. Promise that failures never occur and plan no response process
- C. Automated failure detection with alerting, runbooks, escalation paths, and an on-call rotation sized to the business impact
- D. Rely on the GPU vendor's warranty as the failure-handling plan

**10.** A company must migrate a training pipeline from another cloud to
Nebius within 2 months and ensure the internal team can operate it afterward.
Which migration and handover design is most appropriate?

- A. Move everything in one weekend with no rehearsal, then have the vendor operate it forever
- B. Rebuild everything from scratch with no documentation and a single video call as handover
- C. Migrate only the code and leave data and secrets behind for later
- D. Phased migration with a pilot workload, parallel run, data validation, documented runbooks, and team training before handover

## Domain 3 — Design Efficient & Cost-Aware Architectures

**11.** A startup plans to onboard 40 engineers onto GPU training next
quarter. Before committing to a launch date, what is the most appropriate
capacity planning step?

- A. Assess required capacity and quotas early, request quota increases, and plan onboarding cost against budget and timeline
- B. Assume unlimited GPU capacity will be available on demand
- C. Buy the largest possible cluster now with no workload estimate
- D. Skip planning and let each engineer request GPUs individually at launch

**12.** Three separate teams in one organization keep building near-identical
GPU cluster setups by hand, each spending weeks on configuration. Which
approach reduces operational effort most effectively?

- A. Each team continues with its own handwritten scripts to stay independent
- B. Reusable infrastructure-as-code modules (e.g., Terraform) with shared patterns, versioning, and review
- C. A single shared spreadsheet listing manual console steps
- D. Freeze all new clusters until the teams merge

**13.** A FinOps-aware organization wants GPU spend under control across the
full lifecycle. Which governance design is most appropriate?

- A. Check total spend once per year at renewal
- B. Give every team unlimited quota and review only after incidents
- C. Budgets and alerts, per-team cost allocation, regular right-sizing reviews, and a decommissioning process for idle resources
- D. Hide cost data from engineers so they are not distracted

**14.** A team runs fault-tolerant nightly batch inference over 500 GB of
logs with no hard deadline inside the night window. Which cost-vs-performance
choice is most appropriate?

- A. Reserve the most expensive on-demand GPU capacity 24/7 for safety
- B. Run on CPU only for 3 days straight
- C. Keep the batch job stopped and process logs manually once a month
- D. Use lower-cost preemptible/spot capacity with checkpointable, retryable jobs

**15.** A media company ingests 20 TB of new training data per month from its
on-premises storage to Nebius, and trains weekly. Which design best balances
cost, speed, and reliability?

- A. Ship all data over a single consumer uplink every week and re-upload the full archive each time
- B. A planned ingestion path (dedicated link or staged bulk transfer) with incremental sync, so only new data moves each cycle
- C. Train only on whatever data fits on one laptop and upload later
- D. Keep data on-premises and stream every training sample over the WAN for each epoch

## Domain 4 — Design Secure Architectures

**16.** A 30-person ML organization needs an access model for shared GPU
clusters where researchers launch jobs but must not alter billing, IAM
policies, or other teams' data. Which identity and access design is most
appropriate?

- A. Individual identities with role-based least-privilege policies; jobs run under dedicated service accounts scoped to their project
- B. One shared root credential distributed to all researchers for convenience
- C. Give every researcher project-owner rights and rely on good behavior
- D. No accounts at all — network access is the only control

**17.** A security team requires that operators manage the Kubernetes control
plane and cluster APIs without exposing them to the public internet. Which
connectivity design is most appropriate?

- A. Expose the API server publicly with security through obscurity
- B. Public API server with rate limiting only
- C. Private connectivity (VPC peering / private endpoints / VPN) for control-plane and operator access, with security groups limiting sources
- D. Email-based access requests to a shared admin laptop

**18.** A platform stores model registry credentials and dataset signing keys.
Which secrets and audit design is most appropriate?

- A. Commit secrets to the infrastructure repository for reviewability
- B. Keep secrets in plaintext environment variables on a shared jump host
- C. Store secrets in a managed secret store / KMS-backed encryption, rotate them, and keep audit logs of access
- D. Send secrets to team chat so everyone can find them

**19.** Multiple external clients share one GPU platform as tenants. Which
design best addresses shared responsibility and tenant isolation?

- A. One flat network where every tenant can reach every other tenant's workloads
- B. Isolation by convention — tenants are told not to look at each other's data
- C. A single shared service account for all tenants
- D. Isolated tenancy boundaries (separate projects/namespaces, network policies, storage separation) with a documented shared-responsibility matrix

**20.** A healthcare customer plans to run inference on patient data and must
be able to respond to a suspected data exposure. Which security design is
most appropriate?

- A. Classify the data, encrypt it in transit and at rest with managed keys, restrict access by classification level, and define an incident-response plan with audit trails
- B. Store patient data unencrypted but in a private subnet
- C. Encrypt data but keep decryption keys in the application source code
- D. Handle incidents by deleting logs first to limit exposure

---

## Self-scoring

- Time limit: **20 minutes**. Stop the clock when you finish question 20.
- Check your answers in `practice_exam_01_answers.md`.
- Scoring bands (out of 20):
  - **17–20** — exam-ready; review only the questions you missed.
  - **13–16** — solid base; re-study the domains where you lost points.
  - **9–12** — revisit the course modules for the four domains, then retry.
  - **0–8** — reread the course material before attempting another practice exam.
