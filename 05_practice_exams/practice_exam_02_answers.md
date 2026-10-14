# Practice Exam 02 — Answer Key & Explanations

## Answer Key

| Q | Answer | Correct choice (abbreviated) |
|---|--------|------------------------------|
| 1 | A | Right-size GPU instance type to the model's memory and compute needs |
| 2 | C | Managed notebook environment with prebuilt versioned container images and shared dataset mounts |
| 3 | D | Extend existing Kubernetes with GPU node groups and workload-specific scheduling |
| 4 | A | Many independent data-parallel single-node runs managed by a job queue |
| 5 | C | Centralized object storage + local caching at replicas |
| 6 | D | Checkpoint frequency scaled per job type |
| 7 | A | GPU utilization / data-pipeline / job-progress metrics + logs + alerting |
| 8 | C | SLO-based canary/parallel run + benchmark vs baseline + documented sign-off |
| 9 | D | Cross-zone redundancy, health checks, automated traffic-shift runbook |
| 10 | A | Runbooks, diagrams, known-issues, training, shadow/assist ownership period |
| 11 | D | Time-bounded quota increases for the event, scale back afterward |
| 12 | B | Version-controlled Terraform with PR review and policy checks |
| 13 | C | Consistent tagging/labeling + per-team cost dashboards + budget alerts |
| 14 | A | Committed baseline capacity + flexible bursts within budget |
| 15 | D | Right-size + suspend/scale down outside working hours |
| 16 | B | Attributable break-glass accounts, time-boxed elevation, audit, post-incident review |
| 17 | C | Private connectivity for CI/CD runners + isolated segment + short-lived credentials |
| 18 | A | Managed secret store + dual-credential rotation + gradual automated rollout |
| 19 | D | Namespaces + network policies + RBAC + quotas + storage separation |
| 20 | B | Documented IR plan, roles, alerting, log retention, post-incident reports |

---

## ရှင်းလင်းချက်များ (Explanations)

### Domain 1 — Design High-Performing Architectures

**1. — A (right-size the GPU instance type)**

Domain 1 (GPU/platform building blocks per workload)။ Model က mid-range GPU
တစ်ခုပေါ်မှာ အဆင်ပြေပြီး traffic က steady ဖြစ်တယ် — ဒါကြောင့် GPU instance
type ကို workload ရဲ့ memory/compute လိုအပ်ချက်အတိုင်း right-size လုပ်တာက
budget ကို ထိန်းပြီး performance requirement ကိုလည်း ဖြည့်ဆည်းပေးတယ်
(VM/GPU type အမျိုးအစားများကို ကိုးကား:
https://docs.nebius.com/compute/virtual-machines/types)။ B က "future model"
အတွက် ဆိုပြီး အကြီးဆုံး GPU ယူတာ premature optimization —
လစဉ် budget ကို အလဟဿ ဖြစ်စေတယ်။ C က CPU-only က latency SLO ကို
ချိုးဖောက်မယ်။ D က GPU ရှစ်ခု per replica က မလိုအပ်တဲ့ throughput အတွက်
ကုန်ကျစရိတ် ရှစ်ဆ တက်စေမယ် — right-sizing ရဲ့ ဆန့်ကျင်ဘက်။

**2. — C (managed notebook environment + shared datasets + versioned images)**

Domain 1 (training-platform support stack & user environment)။ သုံးစွဲသူတွေ
က ကျွမ်းကျင်မှု အမျိုးမျိုးရှိပြီး စမ်းသပ်မှု အမျိုးမျိုး လုပ်ကြတာမို့
managed notebook environment, prebuilt versioned container image (reproducibility),
shared dataset mount (onboarding မြန်) တွေ ပေါင်းစပ်ထားတာ အသင့်တော်ဆုံးပဲ။
A က shared host ပေါ်မှာ user တစ်ဦးချင်း driver/package ထည့်တာ
environment drift ကို ဖိတ်ခေါ်တယ်။ B က လူတိုင်း ပြင်လို့ရတဲ့ shared
environment က "works on my machine" ပြဿနာ ဖြစ်စေမယ်။ D က shared dataset
management မရှိဘူး — version ထိန်းချုပ်မှုလည်း မရှိဘူး။

**3. — D (extend existing Kubernetes with GPU node groups)**

Domain 1 (orchestration & workload-framework overlays)။ Team က
Kubernetes tooling, CI/CD, monitoring ရှိပြီးသား — ဒါကြောင့် ရှိပြီးသား
platform ကို GPU node group တွေနဲ့ workload-specific scheduling (training
job နဲ့ inference service အတွက် ကွဲပြားတဲ့ requirement) ထပ်ပေါင်းတာက
tooling တစ်မျိုးတည်းနဲ့ operate လုပ်လို့ ရစေတယ် (ကိုးကား:
https://docs.nebius.com/kubernetes/gpu/clusters)။ A က orchestration system
နှစ်ခု ထားတာ operational overhead ကို နှစ်ဆ လုပ်တယ်။ B က cron + VM က
existing monitoring/CI နဲ့ ကင်းကွာသွားမယ်။ C က per-user cluster က
governance, quota, cost control ကို ဖျက်စီးတယ်။

**4. — A (independent single-node runs with a job queue)**

Domain 1 (distributed-training architecture tradeoffs)။ Job တိုင်းက
node တစ်ခုတည်းမှာ fit ဖြစ်ပြီး job အရေအတွက် များတယ် — ဒါကြောင့်
GPU တွေကို independent job တွေအဖြစ် ခွဲပြီး job queue ကနေ manage လုပ်တာက
wall-clock time ကို အနည်းဆုံး ဖြစ်စေပြီး GPU utilization အမြင့်ဆုံး ရစေတယ်။
Model-parallel (B) က လိုအပ်ချက် မရှိဘဲ coordination overhead ဖြစ်စေမယ်။
C က sequential run က 200 × 1–2 နာရီ = deadline တစ်ပတ်ကို ကျော်သွားနိုင်တယ်။
D က CPU က GPU sweep အတွက် အလွန် နှေးမယ် — quota ကို စီမံရမှာ
မေ့ထားလို့ မရဘူး။

**5. — C (object storage + startup pull + local cache)**

Domain 1 (storage architecture for AI)။ Artifact ဗဟိုပြု object storage
မှာ သိမ်းပြီး replica တွေက startup မှာ ဆွဲယူကာ local cache ထားတာက
scale-out (replica 50 ခု) ကို လွယ်စေပြီး artifact update ကိုလည်း
image rebuild မလိုဘဲ ဖြန့်ဝေလို့ ရတယ် (ကိုးကား:
https://docs.nebius.com/compute/storage/types)။ A က image rebuild တိုင်း
300 GB ပါသွားတာ deployment ကို အလွန် နှေးစေတယ်။ B က ephemeral scratch
မှာပဲ သိမ်းတာ restart မှာ artifact ဆုံးရှုံးမယ် — artifact က retrain
လုပ်ရမဲ့ အရာ မဟုတ်ဘူး၊ versioned release ဖြစ်တယ်။ D က replica
တစ်ခုကို dependency လုပ်တာ single point of failure ဖြစ်ပြီး startup storm
မှာ ပြိုကျနိုင်တယ်။

### Domain 2 — Design Resilient Architectures

**6. — D (checkpoint frequency scaled per job type)**

Domain 2 (checkpoint/restart strategies)။ Job အမျိုးအစား နှစ်မျိုး ရောနှော
နေတဲ့ cluster မှာ policy တစ်ခုတည်းက မကိုက်ဘူး — long run တွေအတွက်
ပိုကြာချိန် မဖြစ်ရဘူး (restart cost ကြီးတယ်)၊ short run တွေအတွက်
checkpoint overhead က job runtime ရဲ့ အများကြီး မဖြစ်ရဘူး။ ဒါကြောင့်
job type အလိုက် checkpoint frequency ချိန်ညှိတာ မှန်ကန်တယ်။ A က
12 နာရီ တစ်ကြောင်းတည်း သတ်မှတ်တာ long run မှာ loss ကြီးပြီး
short run မှာ မလိုဘဲ overhead ဖြစ်တယ်။ B က week-long run မှာ
restart တစ်ခါချင်းစီ ဆုံးရှုံးမှု ကြီးလွန်းတယ်။ C က per-step checkpoint က
I/O overhead ကြောင့် throughput ကို သိသိသာသာ ကျစေမယ်။

**7. — A (GPU/pipeline/job-progress metrics + alerting)**

Domain 2 (observability scope)။ Throughput ကျပေမယ့် error မရှိတဲ့
ပြဿနာမျိုးက စောင့်ကြည့်မှု မရှိလို့ ဖြစ်တယ် — GPU utilization,
data-loader wait, job progress ကို metrics အဖြစ် စုပြီး log နဲ့ alert
ပါ ချိတ်မှ နောက်တစ်ခါ ဖြစ်ရင် အချိန်မီ သိနိုင်မယ် (ကိုးကား:
https://docs.nebius.com/observability)။ B က disk space က ပြဿနာ မဟုတ်ဘူး။
C က manual screenshot က human error ရှိပြီး alerting မပေးဘူး။ D က
assumption မှားတယ် — bottleneck က data pipeline, network, သို့မဟုတ်
GPU placement မှာလည်း ဖြစ်နိုင်တယ် (ဒါကြောင့် metrics လိုတာ ဖြစ်တယ်)။

**8. — C (SLO-based canary + benchmark + documented criteria)**

Domain 2 (validation/benchmark/acceptance plan)။ Customer-facing API
migration မှာ acceptance ကို business metric — latency, error rate,
throughput SLO — အပေါ် တည်ဆောက်ရမယ်။ Canary သို့မဟုတ် parallel run
(ဟောင်းနဲ့အသစ် နှိုင်းယှဉ်ခြင်း) က risk ကို လျှော့ချပြီး baseline နဲ့
benchmark နှိုင်းယှဉ်မှ ရလဒ် ယုံကြည်ရတယ်။ A က customer တွေကို
test subject လုပ်တာ ဖြစ်တယ်။ B က infra apply ပြီးတာ နဲ့ system
အလုပ်လုပ်တာ မတူဘူး။ D က ငါးမိနစ် single-replica test က
peak load နဲ့ scale behavior ကို မစစ်ဆေးနိုင်ဘူး။

**9. — D (cross-zone redundancy + health checks + automated failover runbook)**

Domain 2 (failure handling & operational support model)။ Zone degradation
ကို survive လုပ်ဖို့ capacity ကို zone ကျော်လွန် ထားခြင်း (သို့မဟုတ်
စမ်းသပ်ပြီးသား failover path)၊ health check နဲ့ traffic shift runbook
တို့ ပေါင်းစပ်ရမယ် — ဒါက redundancy နဲ့ operability နှစ်ခုလုံး ပါတဲ့
design ဖြစ်တယ်။ A က single zone က failure တစ်ခုတည်းနဲ့ ပြိုကျမယ်။
B က business requirement ကို စွန့်လွှတ်တာ ဖြစ်တယ်။ C က data replication
မပါတဲ့ idle cluster က failover မှာ အလုပ်မလုပ်နိုင်ဘူး — test မလုပ်ထားရင်
failover က အလုပ်လုပ်မှာ မဟုတ်ဘူး။

**10. — A (runbooks + training + shadow/assist period)**

Domain 2 (migration & handover)။ Handover ဆိုတာ credential လွှဲတာ မဟုတ်ဘူး —
runbook, architecture diagram, known-issues list, operator training ပါပြီး
shadow/assist period (လက်တွဲလုပ်ခြင်း) နဲ့ ownership ကို အတည်ပြုမှ
customer team က တကယ် operate လုပ်နိုင်မယ်။ B က စာရွက်စာတမ်း မပါတဲ့
credential လွှဲမှုက knowledge transfer မဖြစ်ဘူး (security risk ပါ ကြီးတယ်)။
C က vendor lock-in ကို ဆက်ထားတာ ဖြစ်တယ်။ D က ကျင့်ဝတ်နဲ့ မညီဘူး —
handover requirement ကို ဖြေရှင်းတာ မဟုတ်ဘူး။

### Domain 3 — Design Efficient & Cost-Aware Architectures

**11. — D (time-bounded quota increase, scale back after)**

Domain 3 (capacity, quotas & onboarding cost)။ သီတင်းနှစ်ပတ် event
အတွက် time-bounded quota increase တောင်းပြီး event ပြီးရင် ပြန်လျှော့တာက
spike ကို ဖြည့်ဆည်းပေးပြီး idle cost ကို ရှောင်ရှားပေးတယ် (quota က
ကန့်သတ်ထားတဲ့ resource ဖြစ်လို့ ကြိုတင်တောင်းခြင်း လိုတယ် —
ကိုးကား: https://docs.nebius.com/overview/quotas)။ A က three-year
reserved capacity က ၂ ပတ် event အတွက် အလွန် လွဲမှားတယ်။ B က
zero quota ထားရင် launch မှာ capacity မရဘဲ event ပျက်နိုင်တယ် (quota
က အချိန်ယူပြီး တောင်းရတယ်)။ C က year-round idle cost က
budget ကို ဖြုန်းတာ ဖြစ်တယ်။

**12. — B (version-controlled Terraform + PR review + policy checks)**

Domain 3 (reuse & automation)။ Unreviewed manual edit ကနေ ပြဿနာ ဖြစ်တာမို့
change တိုင်းကို version-controlled IaC (Terraform) ထဲ ရေးပြီး pull-request
review နဲ့ policy check ဖြတ်မှ apply လုပ်တဲ့ workflow က audit trail ရော
approval gate ရော ပေးတယ် (ကိုးကား: https://docs.nebius.com/terraform-provider)။
A က chat က searchable, reviewable change history မပေးဘူး — root cause က
process ဖြစ်လို့ စာပိုရှည်ဖို့ မဟုတ်ဘူး။ C က shared credential က
attributable change မဖြစ်စေဘူး — problem ကို ပိုဆိုးစေတယ်။ D က
change freeze က business ကို ရပ်စေတယ် (မဖြစ်နိုင်ပါ)။

**13. — C (tagging + per-team allocation + budget alerts)**

Domain 3 (FinOps governance/monitoring)။ Cost attribution ရှိမှ spike ရဲ့
ဇာစ်မြစ်ကို ရှာလို့ ရမယ် — resource tagging/labeling (team, project,
environment) ကို convention အဖြစ် သတ်မှတ်ပြီး per-team dashboard နဲ့
budget alert ချိတ်မှ (ကိုးကား: https://docs.nebius.com/signup-billing/budgets)
team တွေကိုယ်တိုင် ကုန်ကျစရိတ် မြင်ပြီး ပြင်နိုင်မယ်။ A က
attribution မပါတဲ့ discussion က ဘာမှ မဖြေရှင်းနိုင်ဘူး။ B က
visibility ပိတ်တာ accountability ကို ဖျက်စီးတယ်။ D က ညီမျှခြင်း
အတုက usage signal ကို ဖျက်စီးပြီး ကျိုးကြောင်းဆီလျော်မှု မရှိဘူး။

**14. — A (committed baseline + flexible bursts within budget)**

Domain 3 (cost-vs-performance tradeoffs)။ Steady workload အတွက်
committed/reserved baseline က unit cost ကို လျှော့ချပြီး burst အတွက်
flexible capacity ကို budget အတွင်းကနေ ချိန်ညှိတာ က hard deadline နဲ့
fixed budget နှစ်ခုလုံးနဲ့ ကိုက်တယ်။ B က premium capacity တစ်ခုတည်းက
budget ကို အလျင်စလို ကုန်စေမယ် (six-month run ကို ခံနိုင်ရည် မရှိဘူး)။
C က last-week burst က deadline ပြီးမှ ရောက်မယ် — deadline ကို ပျက်စေတယ်။
D က requirement ပြောင်းတာ ဖြစ်ပြီး design ဖြေရှင်းမှု မဟုတ်ဘူး။

**15. — D (right-size + scheduled suspend/scale-down)**

Domain 3 (cost dimensions & right-sizing)။ Office hour ပြင်ပမှာ
dev/test ကို suspend/scale down လုပ်ပြီး လိုအပ်ချိန်မှ shared ephemeral
capacity ပေးတာက idle cost ကို လျှော့ချပြီး developer experience ကို
ထိန်းသိမ်းပေးတယ် (dev/test sizing က production နဲ့ ကွဲနိုင်တယ် —
VM type ရွေးချယ်မှု ကိုးကား:
https://docs.nebius.com/compute/virtual-machines/types)။ A က
production-sized dev/test က cost ကို တက်စေမယ်။ B က dev/test ဖျက်တာ
development workflow ကို ဖျက်စီးတယ် (production မှာ develop တာ
security/availability risk ကြီးတယ်)။ C က permanent multi-GPU per
developer က ဒီပြဿနာရဲ့ အဆိုးဆုံး ဗားရှင်း ဖြစ်တယ်။

### Domain 4 — Design Secure Architectures

**16. — B (attributable time-boxed break-glass with audit)**

Domain 4 (identity & access model)။ Emergency access ဆိုတာ လိုအပ်တယ် —
ဒါပေမယ့် ပုံမှန် မဟုတ်တဲ့ အခြေအနေအတွက် ဖြစ်လို့ သီးခြား ထိန်းချုပ်မှု
(individual account, time-boxed elevation, strong audit logging,
post-incident review) နဲ့ ဒီဇိုင်း လုပ်ရမယ် (IAM model ကို ကိုးကား:
https://docs.nebius.com/iam/overview)။ A က permanent admin ပေးတာ
least privilege ကို ဖျက်စီးပြီး everyday risk ကို တက်စေတယ်။ C က
shared password က attribution မရှိဘူး — wiki မှာ ထားတာ leak တစ်ခုတည်းနဲ့
production ပျက်မယ်။ D က emergency access မရှိရင် outage က ရှည်ကြာပြီး
business impact ကြီးမယ် — break-glass ရဲ့ ရည်ရွယ်ချက်ကို နားမလည်တာ ဖြစ်တယ်။

**17. — C (private CI/CD connectivity + isolated runners + short-lived credentials)**

Domain 4 (private connectivity for control-plane access)။ CI/CD runner တွေကို
private endpoint/peering ကနေ cluster နဲ့ ချိတ်ဆက်ပြီး runner တွေကို
သီးခြား isolated network segment မှာ ထားခြင်း၊ short-lived credential
သုံးခြင်း (secret ထားသိမ်းမှု — ကိုးကား: https://docs.nebius.com/kms)
က exposure ကို အနည်းဆုံး ဖြစ်စေတယ် (network control အတွက် security
group — ကိုးကား:
https://docs.nebius.com/vpc/security-groups/overview)။ A က provider
IP range အကုန် allowlist တာ attack surface ကို ကျယ်စေတယ်။ B က
SSH push က credential နဲ့ audit ထိန်းချုပ်မှု အားနည်းပြီး node တွေကို
ထိခိုက်လွယ်စေတယ်။ D က public reverse proxy + basic auth က
API ကို public ဖြစ်နေစေဆဲ — requirement ကို မဖြည့်ဆည်းဘူး။

**18. — A (managed secret store + dual-credential rotation + gradual rollout)**

Domain 4 (secrets, audit, compliance)။ Overlapping validity window
(credential အဟောင်းနဲ့အသစ် ခဏတာ တွဲသုံးခြင်း) နဲ့ consumer တွေကို
automation နဲ့ တဖြည်းဖြည်း rollout လုပ်တာက rotation ကို zero-downtime
ဖြစ်စေတယ် (managed secret store နဲ့ KMS-backed encryption —
ကိုးကား: https://docs.nebius.com/kms)။ B က တစ်နှစ်တစ်ခါ rotation က
credential leak ရဲ့ exposure window ကို ကြီးစေတယ်။ C က all-at-once
redeploy က single point of failure ဖြစ်ပြီး image ထဲ secret ထည့်တာ
secret management ကို ဖျက်စီးတယ်။ D က rotation ရပ်တာ compliance ကို
ချိုးဖောက်ပြီး risk ကို စုပုံစေတယ်။

**19. — D (namespaces + network policies + RBAC + quotas + storage separation)**

Domain 4 (shared responsibility, tenant isolation, least privilege)။
Data sensitivity မတူတဲ့ unit နှစ်ခု အတူတူူ ရှိတဲ့ cluster မှာ
namespace separation, network policy, RBAC scoping, resource quota,
storage separation ကို ပေါင်းစပ်မှ boundary တစ်ခု ဖြစ်လာတယ် —
shared responsibility matrix အတိုင်း ဘယ်အပိုင်းက ဘယ်သူ တာဝန်ရှိတယ်ဆိုတာ
ရှင်းရမယ်။ A က shared namespace က isolation လုံးဝ မပေးဘူး။
B က naming convention က technical control မဟုတ်ဘူး။ C က folder
separation က filesystem permission တစ်ခု အမှားနဲ့ ပေါက်ကြားနိုင်တယ်။

**20. — B (documented IR plan + log retention + post-incident reports)**

Domain 4 (data classification, encryption, incident response)။ Audit
requirement ဆိုတာ ပြန်ပြစရာ အထောက်အထား လိုတယ် — documented incident-
response plan (role, alerting trigger, escalation), forensic အတွက်
လုံလောက်တဲ့ log retention, post-incident report တွေ က audit evidence
အဖြစ် ထိန်းသိမ်းမှ လိုအပ်ချက် ဖြည့်ဆည်းနိုင်တယ် (audit trail နဲ့
observability ချိတ်ဆက်မှု — ကိုးကား:
https://docs.nebius.com/observability)။ A က verbal report က
auditor အတွက် အထောက်အထား မဖြစ်ဘူး — inconsistent ဖြစ်တယ်။ C က
24-နာရီ retention က forensic window ကို ဖျက်စီးပြီး audit မှာ
အထောက်အထား မရှိစေဘူး။ D က ad-hoc response က reproducible,
reviewable process မဖြစ်စေဘူး — incident response က documented
procedure ဖြစ်ရမယ်။

---

## ကိုးကားထားသော စာရွက်စာတမ်းများ

- GPU clusters: https://docs.nebius.com/kubernetes/gpu/clusters
- VM types: https://docs.nebius.com/compute/virtual-machines/types
- Storage types: https://docs.nebius.com/compute/storage/types
- Slurm operator: https://docs.nebius.com/slurm-soperator
- Observability: https://docs.nebius.com/observability
- IAM: https://docs.nebius.com/iam/overview
- KMS: https://docs.nebius.com/kms
- Security groups: https://docs.nebius.com/vpc/security-groups/overview
- Quotas: https://docs.nebius.com/overview/quotas
- Budgets: https://docs.nebius.com/signup-billing/budgets
- Terraform provider: https://docs.nebius.com/terraform-provider

> မှတ်ချက်: မေးခွန်းများသည် design concept များကို စစ်ဆေးရန်ဖြစ်သောကြောင့်
> Nebius-specific CLI command၊ ဈေးနှုန်း၊ quota limit များကို အသုံးပြုထားခြင်း
> မရှိပါ။ အထက်ပါ docs page များတွင် နောက်ဆုံးပေါ် product behavior များကို
> အမြဲ စစ်ဆေ်ပါ။
