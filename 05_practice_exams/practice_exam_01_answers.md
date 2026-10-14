# Practice Exam 01 — Answer Key & Explanations

## Answer Key

| Q | Answer | Correct choice (abbreviated) |
|---|--------|------------------------------|
| 1 | D | High-bandwidth, low-latency interconnect (InfiniBand/RDMA) between GPU nodes |
| 2 | B | Hot dataset + checkpoints on fast shared file/block storage; durable checkpoint copy in object storage |
| 3 | A | Slurm-based managed cluster with job queues and scheduler policies |
| 4 | C | Inference service on autoscaling GPU capacity behind a load balancer, sized to the latency SLO |
| 5 | B | Model/layer sharding combined with data parallelism over the fast interconnect |
| 6 | A | Periodic async checkpoints to durable shared storage + automated restart from last good checkpoint |
| 7 | D | Logs + metrics + traces with alerting |
| 8 | B | Validation & benchmark plan with acceptance criteria and failure-injection drills |
| 9 | C | Automated detection, alerting, runbooks, escalation, on-call rotation |
| 10 | D | Phased migration with pilot, parallel run, data validation, runbooks, team training |
| 11 | A | Assess capacity & quotas early, request quota increases, plan onboarding cost |
| 12 | B | Reusable IaC (Terraform) modules with versioning and review |
| 13 | C | Budgets, alerts, cost allocation, right-sizing reviews, decommissioning |
| 14 | D | Preemptible/spot capacity with checkpointable, retryable jobs |
| 15 | B | Planned ingestion path with incremental sync |
| 16 | A | Individual identities, RBAC least privilege, dedicated service accounts per project |
| 17 | C | Private connectivity (VPC peering / private endpoints / VPN) + security groups |
| 18 | C | Managed secret store / KMS-backed encryption, rotation, audit logs |
| 19 | D | Isolated tenancy boundaries + documented shared-responsibility matrix |
| 20 | A | Data classification, encryption with managed keys, access by classification, incident response plan |

---

## ရှင်းလင်းချက်များ (Explanations)

### Domain 1 — Design High-Performing Architectures

**1. — D (InfiniBand/RDMA interconnect)**

Domain 1 (distributed-training architecture tradeoffs)။ Multi-node LLM training
မှာ gradient sync က throughput ကို အဓိက ကန့်သတ်တယ်။ ဒါကြောင့် GPU node တွေကြား
high-bandwidth, low-latency interconnect (InfiniBand, RDMA) က မှန်ကန်တဲ့ design
choice ဖြစ်တယ် — Nebius ရဲ့ GPU cluster တွေလည်း ဒီအတွက် ဒီဇိုင်းထုတ်ထားတယ်
(ကိုးကား: https://docs.nebius.com/kubernetes/gpu/clusters)။ A က public internet
tunnel ဖြစ်လို့ latency နဲ့ bandwidth မလုံလောက်ဘူး။ B က gradient ကို disk
ကတစ်ဆင့် ပို့တာဖြစ်လို့ အဆမတန် နှေးတယ်။ C က single VM ထဲ 32 GPU ထည့်တာက
software emulation နဲ့ memory ကန့်သတ်ချက်ကြောင့် မဖြစ်နိုင်ဘူး။

**2. — B (fast shared storage + object storage backup)**

Domain 1 (storage architecture for AI)။ Dataset ကို epoch တိုင်း ထပ်ဖတ်ရမှာမို့
hot path မှာ fast file/block storage လိုတယ် (အမျိုးအစားများကို
https://docs.nebius.com/compute/storage/types မှာ နှိုင်းယှဉ်ထားတယ်)။
Checkpoint တွေက node failure မှာ ရှင်သန်ရမှာမို့ durable copy ကို object storage
မှာ သိမ်းတာက fault tolerance နဲ့ cost နှစ်ခုလုံးအတွက် မှန်ကန်တယ်။ A က
HTTP streaming က epoch တိုင်း 40 TB ဖတ်တာ ကုန်ကျစရိတ်ကြီးပြီး နှေးတယ်။
C က dataset ကို node တိုင်းပေါ် ကူးတာ storage ဆုံးရှုံးမှု အကြီးကြီး။ D က
coordinator node တစ်ခုတည်းပေါ် checkpoint ထားတာ single point of failure ဖြစ်တယ်။

**3. — A (managed Slurm cluster)**

Domain 1 (workload outcomes & deployment shape / orchestration overlays)။
User experience က job တင်ပြီး GPU ရဖို့ပဲ လိုတာ — Kubernetes object တွေ မလိုဘူး။
Slurm-based managed cluster က job queue, fair-share, scheduler policy ပါပြီးသား
ဖြစ်လို့ research/academic team အတွက် သင့်တော်တယ် (ကိုးကား:
https://docs.nebius.com/slurm-soperator)။ B က self-managed Kubernetes က
platform engineer တစ်ယောက်တည်းအတွက် ဝန်ထုပ်ဝန်ပိုး ကြီးလွန်းတယ်။ C က
manual GPU assignment က scale မလုပ်နိုင်ဘူး။ D က single workstation က
multi-GPU job အတွက် မလုံလောက်ဘူး။

**4. — C (autoscaling inference behind load balancer)**

Domain 1 (workload outcomes & deployment shape)။ Interactive service မှာ
latency SLO ရှိပြီး traffic pattern က variable ဖြစ်တယ် — ဒါကြောင့် load balancer
နောက်ကွယ်က autoscaling GPU capacity က အကောင်းဆုံးပဲ။ Latency SLO အတိုင်း sizing
လုပ်ထားရင် peak မှာလည်း ခံနိုင်ပြီး off-peak မှာ scale down လို့ ရတယ်။
A က batch job က interactive request အတွက် မဟုတ်ဘူး။ B က training cluster ကို
repurpose လုပ်တာ latency guarantee မပေးနိုင်ဘူး။ D က single spot instance က
manual restart နဲ့ SLO မဖြည့်ဆည်းနိုင်ဘူး။

**5. — B (model sharding + data parallelism)**

Domain 1 (distributed-training architecture tradeoffs)။ Model တစ်ခုလုံးက
node တစ်ခု memory ထဲ မဝင်ရင် model/layer sharding (pipeline သို့မဟုတ် tensor
parallelism) ကို node တွေကြား ခွဲပြီး data parallelism နဲ့ ပေါင်းရတယ်။
Fast interconnect ပေါ်မှာ ဒီ design က အလုပ်လုပ်တယ်။ A က pure data parallelism က
model တစ်ခုလုံး copy လုပ်ဖို့ node memory မလုံလောက်ဘူး။ C က CPU offload က
throughput အတွက် လက်မခံနိုင်လောက်အောင် နှေးတယ် (deadline 3 ပတ်နဲ့ မကိုက်)။
D က requirement ကို ဖြေရှင်းတာ မဟုတ်ဘူး — deadline ကို ရွှေ့တာပဲ။

### Domain 2 — Design Resilient Architectures

**6. — A (periodic async checkpoints + automated restart)**

Domain 2 (checkpoint/restart strategies)။ 10 ရက်ကြာ job မှာ failure က
မရှောင်မလွဲပဲ — ဒါကြောင့် periodic checkpoint ကို durable shared storage မှာ
asynchronous သိမ်းပြီး last good checkpoint ကနေ automated restart လုပ်တာက
lost compute ကို အနည်းဆုံး ဖြစ်စေတယ်။ B က end-only checkpoint က failure
တစ်ခါဖြစ်ရင် 12 နာရီ ဆုံးရှုံးနိုင်တယ်။ C က local disk-only checkpoint က
node failure မှာ checkpoint ပါ ပျက်သွားနိုင်တယ်။ D က checkpointing ပိတ်တာက
ယာယီ throughput အတွက် reliability ကို စွန့်လွှတ်တာ — 10 ရက် job အတွက်
မတန်တဆ စွန့်စားမှု ဖြစ်တယ်။

**7. — D (logs + metrics + traces with alerting)**

Domain 2 (observability scope)။ ပြဿနာ ၃ မျိုးလုံး — slow training (metrics),
node failure (metrics + logs), request debugging (traces) — ကို ဖြေရှင်းဖို့
logs, metrics, traces သုံးမျိုးလုံး လိုတယ် (ကိုးကား: https://docs.nebius.com/observability)။
Alerting မပါရင် failure ကို အချိန်မီ မသိနိုင်ဘူး။ A က SSH နဲ့ log သွားယူတာ
reactive ဖြစ်လွန်းတယ်။ B က billing dashboard က operational debugging မလုပ်ပေးနိုင်ဘူး။
C က health-check ping တစ်ခုတည်းက ဘာမှ မဖော်ပြနိုင်ဘူး — GPU utilization,
job progress, request latency ဘာမှ မမြင်ရဘူး။

**8. — B (validation & benchmark plan with acceptance criteria)**

Domain 2 (validation/benchmark/acceptance plan)။ Acceptance ဆိုတာ တစ်ခါ
အလုပ်လုပ်တာ မဟုတ်ဘူး — correctness check, baseline နဲ့ benchmark နှိုင်းယှဉ်မှု,
failure-injection drill (ဥပမာ node တစ်ခု ဖျက်ကြည့်ခြင်း), စိစစ်ရမည့် criteria
တွေ စာရွက်စာတမ်း တစ်ရပ်လုံး ပါမှ ဖြစ်မယ်။ A က demo တစ်ခါ အလုပ်လုပ်တာ
production readiness ကို မသက်သေပြနိုင်ဘူး။ C က single-node unit test က
distributed system တစ်ခုလုံးကို မစစ်ဆေးနိုင်ဘူး။ D က invoice က acceptance criteria
မဟုတ်ဘူး။

**9. — C (detection + alerting + runbooks + on-call)**

Domain 2 (failure handling & operational support model)။ 24/7 workload အတွက်
failure detection, alerting, runbook, escalation path, on-call rotation ကို
business impact ပမာဏအတိုင်း design လုပ်ရမယ် (runbook/operational practice ကို
observability အစီအစဉ်နဲ့ ချိတ်ဆက်ထားသင့်တယ် — ကိုးကား:
https://docs.nebius.com/observability)။ A က failure ကို နောက်နေ့မှ သိတာ
24/7 requirement နဲ့ ဆန့်ကျင်ဘက်။ B က "failure မဖြစ်ဘူး" ဆိုတာ realistic
မဟုတ်ဘူး — resilient design ရဲ့ အခြေခံက assumption မှားတယ်။ D က
hardware warranty က operational response plan မဟုတ်ဘူး။

**10. — D (phased migration + parallel run + handover)**

Domain 2 (migration & handover)။ 2 လ deadline အတွက် phased approach —
pilot workload နဲ့ စမ်းသပ်ခြင်း, parallel run, data validation, runbook
ရေးခြင်း, team training — က risk ကို လျှော့ချပြီး handover ကို အောင်မြင်စေတယ်။
Terraform ကဲ့သို့ IaC က migration မှာ environment ကို ပြန်ဖန်တီးလို့ ရစေတယ်
(ကိုးကား: https://docs.nebius.com/terraform-provider)။ A က rehearsal မပါတဲ့
big-bang cutover က rollback မရှိဘူး။ B က documentation မပါတဲ့ handover က
team ကို operate လုပ်နိုင်စေမှာ မဟုတ်ဘူး။ C က data နဲ့ secrets ကို ချန်ထားခဲ့ရင်
pipeline က အလုပ်မလုပ်နိုင်ဘူး — secrets migration က security review ပါ လိုတယ်။

### Domain 3 — Design Efficient & Cost-Aware Architectures

**11. — A (capacity/quota assessment + onboarding cost planning)**

Domain 3 (capacity, quotas & onboarding cost)။ GPU capacity က ကန့်သတ်ထားတဲ့
resource ဖြစ်လို့ quota ကို ကြိုတင် စစ်ဆေးပြီး လိုအပ်ရင် quota increase
တောင်းထားမှ launch date က ယုံကြည်စိတ်ချရတယ် (ကိုးကား:
https://docs.nebius.com/overview/quotas)။ Onboarding cost (GPU instance အမျိုးအစား၊
storage၊ network) ကို budget နဲ့ timeline အတွင်း plan လုပ်ရမယ်။ B က unlimited
capacity assumption က စီမံကိန်း risk အကြီးဆုံးပဲ။ C က workload estimate မပါဘဲ
oversize ဝယ်တာ cost waste ဖြစ်တယ်။ D က launch မှာ individual request တွေ
quota bottleneck ဖြစ်စေမယ်။

**12. — B (reusable Terraform modules)**

Domain 3 (reuse & automation)။ အလုပ်ထပ်နေတဲ့ cluster setup ၃ ခုကို handwritten
script တွေနဲ့ လုပ်တာ ထက် shared Terraform modules (versioning, code review)
က operational effort ကို အများကြီး လျှော့ချပေးပြီး consistency ကို ရစေတယ်
(ကိုးကား: https://docs.nebius.com/terraform-provider)။ A က independence
ဆိုပေမယ့် ထပ်နေတဲ့ အလုပ်အတွက် ကုန်ကျချိန် ဆက်ပြီး ကြီးနေမယ်။ C က
spreadsheet က automation မပေးဘူး။ D က အလုပ်ကို ရပ်လိုက်တာ ဖြေရှင်းမှု
မဟုတ်ဘူး။

**13. — C (budgets + allocation + right-sizing + decommissioning)**

Domain 3 (FinOps governance)။ FinOps က lifecycle တစ်ခုလုံး — စောင့်ကြည့်ခြင်း၊
တာဝန်ခွဲဝေခြင်း၊ right-sizing၊ decommissioning — ကို ဖုံးအုပ်ရတယ်။
Budgets နဲ့ alert တွေ (ကိုးကား: https://docs.nebius.com/signup-billing/budgets)
က overspend ကို အချိန်မီ ဖမ်းပေးတယ်။ A က တစ်နှစ်တစ်ခါ စစ်တာ အလွန်နောက်ကျတယ်။
B က unlimited quota က governance ရဲ့ ဆန့်ကျင်ဘက်။ D က cost visibility ပိတ်တာ
accountability ကို ဖျက်စီးတယ်။

**14. — D (preemptible/spot + checkpointable retryable jobs)**

Domain 3 (cost-vs-performance tradeoffs)။ Fault-tolerant batch job, hard
deadline မရှိ — ဒါက preemptible/spot capacity အတွက် အကောင်းဆုံး အခြေအနေပဲ။
Job ကို checkpoint/ retry လုပ်နိုင်အောင် design ထားရင် interruption ကို
ခံနိုင်ပြီး cost ကို သိသိသာသာ လျှော့ချနိုင်တယ် (cost က workload ရဲ့
အရေးပါမှုပေါ် မူတည်ကြောင်း design tradeoff — ကိုးကား:
https://docs.nebius.com/signup-billing/budgets)။ A က 24/7 premium capacity က
ဒီ workload အတွက် မလိုအပ်ဘူး — cost က performance ထက် ကြီးနေမယ်။ B က
3 ရက်ကြာတဲ့ CPU run က night window ကို ချိုးဖောက်မယ်။ C က requirement
ကို မဖြည့်ဆည်းဘူး။

**15. — B (planned ingestion + incremental sync)**

Domain 3 (multi-cloud connectivity & data-ingestion)။ လစဉ် 20 TB ဆိုတာ
အဆက်မပြတ် ingestion plan လိုတယ် — dedicated link သို့မဟုတ် staged bulk
transfer နဲ့ စပြီး နောက်ပိုင်း incremental sync (အသစ်ဝင်လာတဲ့ data ပဲ
ရွှေ့တာ) က network cost နဲ့ time နှစ်ခုလုံး သက်သာစေတယ်။ A က full archive
ပြန်တင်တာ လစဉ် အလုပ်ထပ်ပြီး ကုန်ကျစရိတ် ကြီးတယ်။ C က data အများစုကို
စွန့်ပစ်သလို ဖြစ်သွားမယ်။ D က WAN ကနေ epoch တိုင်း stream လုပ်တာ
training throughput ကို သေစေတယ် (hot data က လိုအပ်ရင် cloud storage ဘက်မှာ
ထားသင့်တယ် — ကိုးကား: https://docs.nebius.com/compute/storage/types)။

### Domain 4 — Design Secure Architectures

**16. — A (individual identities + least privilege + service accounts)**

Domain 4 (identity & access model)။ Researchers တွေ job တင်ဖို့ လိုပေမယ့်
billing/IAM/တစ်ခြား team data ကို မထိရ — ဒါကြောင့် individual identity +
role-based least privilege + dedicated service account ကို project scope အတွင်း
ထားတာ မှန်ကန်တယ် (ကိုးကား: https://docs.nebius.com/iam/overview)။
B က shared root credential က audit ကို ဖျက်စီးပြီး အန္တရာယ် အရမ်းကြီးတယ် —
credential တစ်ခု ပေါက်ကြားရင် အားလုံး ပျက်မယ်။ C က project-owner အားလုံးပေးတာ
least privilege ကို ချိုးဖောက်တယ် (accidental billing/IAM change ကို
မကာကွယ်နိုင်ဘူး)။ D က network access တစ်ခုတည်းက identity မပါဘဲ
authorization မလုပ်နိုင်ဘူး။

**17. — C (private connectivity + security groups)**

Domain 4 (private connectivity for control-plane/operator access)။
Control plane/API ကို public internet မထိတွေ့ဘဲ VPC peering, private endpoint,
သို့မဟုတ် VPN ကနေ private ချိတ်ဆက်ပြီး security group တွေနဲ့ source
ကန့်သတ်တာ မှန်ကန်တယ် (ကိုးကား:
https://docs.nebius.com/vpc/security-groups/overview)။ A က "security through
obscurity" က စစ်မှန်တဲ့ control မဟုတ်ဘူး။ B က rate limiting က exposure
ကို မလျှော့ချနိုင်ဘူး — attack surface က public ဖြစ်နေဆဲ။ D က shared admin
laptop က single point of compromise ဖြစ်တယ်။

**18. — C (managed secret store / KMS + rotation + audit logs)**

Domain 4 (secrets, audit, compliance)။ Secret တွေကို managed secret store
(KMS-backed encryption) မှာ သိမ်းပြီး rotate လုပ်ခြင်း၊ access audit log
ထားခြင်း (ကိုးကား: https://docs.nebius.com/kms) က compliance အတွက်
လိုအပ်သလို credential leak ရဲ့ သက်ရောက်မှုကိုလည်း ကန့်သတ်ပေးတယ်။
A က repo မှာ commit လုပ်တာ git history ထဲ ထာဝရ ကျန်ခဲ့မယ်။ B က plaintext
env var က ဘယ်သူမဆို host ဝင်ဖတ်လို့ ရတယ်။ D က chat မှာ ပို့တာ ကိုယ်ထိန်းချုပ်မှု
လုံးဝ မရှိဘူး — log ထဲ ထာဝရ ကျန်မယ်။

**19. — D (tenancy boundaries + shared-responsibility matrix)**

Domain 4 (shared responsibility, tenant isolation, least privilege)။
External client တွေ ပါတဲ့ multi-tenant platform မှာ separate project/
namespace, network policy, storage separation ကနေ tenant boundary တည်ဆောက်ပြီး
shared-responsibility matrix (ဘယ်အပိုင်းက provider၊ ဘယ်အပိုင်းက customer)
ကို စာရွက်စာတမ်း တစ်ရပ်အဖြစ် ရှင်းရှင်းလင်းလင်း ရေးထားရမယ်။ A က
flat network က lateral movement ကို ဖိတ်ခေါ်တယ်။ B က "မကြည့်နဲ့" ဆိုတာ
technical control မဟုတ်ဘူး။ C က shared service account က tenant ကြား
ခွဲခြားမှု မရှိစေဘူး — audit လည်း မလုပ်နိုင်ဘူး။

**20. — A (classification + encryption + IR plan)**

Domain 4 (data classification, encryption, incident response)။ Healthcare data
အတွက် design တစ်ခုလုံး — data classification, in-transit/at-rest encryption
(managed key နဲ့), classification အလိုက် access control, incident-response plan
နဲ့ audit trail — ကို ပေါင်းစပ်ရမယ်။ B က encryption မပါရင် regulated data
အတွက် လုံးဝ မလုံခြုံဘူး (private subnet က network isolation ပဲ ပေးတယ် —
encryption မဟုတ်ဘူး)။ C က key ကို source code မှာ ထားတာ code leak
တစ်ခုတည်းနဲ့ အားလုံး ပျက်စေတယ် — key management ကို managed KMS နဲ့
ခွဲထားရမယ် (ကိုးကား: https://docs.nebius.com/kms)။ D က log ဖျက်တာ
incident ရဲ့ root cause စစ်ဆေးမှုကို ဖျက်စီးပြီး audit trail ထိန်းသိမ်းမှု
(compliance requirement) ကို ချိုးဖောက်တယ် — ပြီးတော့ တရားမဝင်
သဲလွန်စ ဖျက်စီးမှု ဖြစ်နိုင်တယ်။

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
