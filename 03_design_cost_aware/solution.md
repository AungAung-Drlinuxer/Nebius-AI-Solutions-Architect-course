# Module 03 — Solution: Design Efficient & Cost-Aware Architectures

ဤတွင် exercise ၅ ခုအတွက် ဆုံးဖြတ်ချက် စဉ်းစားပုံ နမူနာ ဖြေရှင်းချက်များ ဖြစ်ပါတယ်။
စာမေးပွဲတွင် အဖြေတစ်ခုတည်း မဟုတ်ဘဲ **assumption ဖော်ပြခြင်း၊ tradeoff ရှင်းပြခြင်း၊
ကန့်သတ်ချက် ဝန်ခံခြင်း** က အဓိက ဖြစ်ပါတယ်။

---

## Solution 1 — Budget နှင့် timeline ကနေ capacity plan

### 1) GPU demand ခန့်မှန်းချက်

ပထမ ကြိုတင် စုဆောင်းရမည့် အချက်အလက်များ —

- model size နှင့် fine-tuning method (full fine-tune / LoRA စသည် — memory နှင့်
  compute လိုအပ်ချက် ကွာခြားစေ)
- dataset size, epoch အရေအတွက်
- per-GPU throughput — ကိုယ်ပိုင် အသေးစား benchmark run (ဥပမာ ၁ ရက်) သို့မဟုတ်
  ယခင် run မှတ်တမ်းမှ
- ရရှိနိုင်သော wall-clock: ၂၁ ရက် (၃ ပတ်) ထဲမှ data ingestion နှင့် validation
  အတွက် ချန်လှပ်ပြီး ကျန်သည်

ခန့်မှန်းပုံ (illustrative) —

```
GPU-hours = စုစုပေါင်း အလုပ်ပမာဏ ÷ (per-GPU throughput × utilization)
လိုအပ် GPU အရေအတွက် N = GPU-hours ÷ (training အတွက် ခွဲဝေ ရက် × ၂၄)
```

ဥပမာ — benchmark က GPU-hours ~X လိုအပ်ကြောင်း ပြပါက (X ကို ကိုယ်ပိုင် ဒေတာဖြင့်
အစားထိုးပါ) training အတွက် ရက် ၁၄ ရက် ရှိပါက N = X ÷ (14 × 24) ဖြစ်ပါတယ်။
**ခန့်မှန်းချက် ဖြစ်ကြောင်း အမြဲ ဖော်ပြပါ** — run အစောပိုင်း ဒေတာဖြင့် ပြန်ညှိပါ။
ထို့နောက် scaling efficiency (N တိုးလျှင် throughput လိုက်လာမှု) ကို ထည့်သွင်းပြီး
N ကို ပြန်ညှိပါ — deadline တင်းကျသောကြောင့် N ကို buffer (ဥပမာ ၁၀–၂၀%) နှင့်
ခန့်မှန်းပါ။

### 2) quota ကို design အဆင့်မှာ ထည့်သွင်းခြင်း

- လိုအပ်မည့် GPU အရေအတွက် N ကို **Administration → Limits → Quotas** တွင်
  လက်ရှိ quota နှင့် နှိုင်းယှဉ်ပါ — region တစ်ခုချင်းစီတွင် quota ကွဲသောကြောင့်
  run မည့် region အတွက် သီးခြား စစ်ပါ။
- မလုံလောက်ပါက **run စတင်မည့် ရက်များစွာ ကြိုတင်** quota increase request တင်ပါ —
  request ကို support center မှ လိုက်နာနိုင်ပြီး ကြာချိန် ရှိနိုင်သောကြောင့် timeline တွင်
  ကြိုတင် ထည့်သွင်းပါ။
- quota မတင်ဘဲ စတင်ပါက — run စချိန်တွင် resource မရရှိခြင်း၊ deadline လွဲခြင်း၊
  သို့မဟုတ် အခြား region သို့ ရုတ်တရက် ပြောင်းရှောင်ရခြင်း (dataset transfer ထပ်ကုန်)
  risk ရှိပါတယ်။
- GPU အမျိုးအစား ပြောင်းလျှင် quota လည်း ပြန်စစ်ပါ — quota သည် service/resource
  အမျိုးအစားအလိုက် ခွဲခြား သတ်မှတ်ထားနိုင်ပါတယ်။

### 3) onboarding cost အမျိုးအစားများ

| အမျိုးအစား | ပါဝင်သော အရာ | ကျော်လွန်လွယ်မှု |
|---|---|---|
| Compute | GPU node များ run အချိန် (N × T) | **အမြင့်ဆုံး** — run ရှည်လျှင် တိုက်ရိုက် တိုး |
| Storage | dataset, checkpoint, snapshot များ | အလယ်အလတ် — ပြီးမှ ဖျက်မေ့ပါက စုပုံ |
| Network | dataset ingest/egress ပမာဏ | အလယ်အလတ် — ပြောင်းရွှေ့မှု ကြီးပါက သတိပြု |
| Managed services | PostgreSQL, MLflow စသည် (လိုမှ သုံး) | နိမ့် — ရှင်းလင်းစွာ စီမံပါက |
| Operational effort | setup, monitoring, teardown အချိန် | အလယ်အလတ် — team အတွေ့အကြုံ နည်းသောကြောင့် automation ဖြင့် လျှော့ချ |

အလွယ်တကူ ကျော်လွန်စေနိုင်သည်က **compute** (deadline ကြောင့် GPU ကြာချိန် တိုးလာခြင်း)
နှင့် **storage** (checkpoint/snapshot ဟောင်းများ စုပုံခြင်း) ဖြစ်ပါတယ်။
budget + quota decrease ကို ပူးတွဲ အသုံးပြုပါ။

### 4) timeline နမူနာ

1. **ရက် ၁–၃** — Terraform environment တည်ဆောက်ခြင်း၊ quota အတည်ပြု/request တင်ခြင်း၊
   benchmark run အသေးဖြင့် GPU demand ခန့်မှန်းချက် ပြန်ညှိခြင်း
2. **ရက် ၂–၅ (parallel)** — data ingestion (transfer window စီစဉ်) → staging →
   validation
3. **ရက် ၅–၁၈** — training run (checkpoint ပုံမှန် သိမ်းခြင်း၊ mid-run ခန့်မှန်းချက် ပြန်ညှိခြင်း)
4. **ရက် ၁၈–၂၁** — validation, artifact (model registry) သို့ ပို့ခြင်း၊ handover —
   ပြီးဆုံးပါက idle resource များ decommission ခြင်း

---

## Solution 2 — Multi-team FinOps governance

### 1) budget ချမှတ်ပုံ

| Budget | Scope | Period | Threshold နမူနာ | Recipient |
|---|---|---|---|---|
| Account-wide | All usage | Monthly | 50%, 80%, 100% | tenant contact (shared mailbox) |
| Research team | Project condition (research projects) | Monthly | 60%, 100% | research lead + tenant contact |
| Product team | Project condition (product projects) | Monthly | 60%, 100% | product lead |
| Platform team | Project condition (platform projects) | Monthly | 60%, 100% | platform lead |
| GPU စောင့်ကြည့် | Product condition (GPU) + Project (research) | Monthly | 50%, 100% | research lead |

- threshold ကို **percentage နှင့် absolute value ရော** ထားနိုင်ပြီး forecast မဟုတ်ဘဲ
  **actual spending** ကိုသာ အခြေခံသောကြောင့် ကြိုတင် သတိပေးချင်ပါက
  threshold ကို နိမ့်နိမ့်ထားပါ (data update က နာရီအနည်းငယ် ကြာနိုင်)။
- recipient အတွက် **tenant contact** (shared mailbox) ကို billing အတွက် အကြံပြုပြီး
  team ခေါင်းဆောင်များကို project-scoped budget တွင် ထည့်ပါ — ဘယ် team က ကျော်သလဲ
  ချက်ချင်း သိရှိစေပါတယ်။
- budget ပမာဏတွင် VAT မပါဝင်ကြောင်း သိထားပါ။

### 2) အပို guardrail များ

- **quota** — research team ၏ project အတွက် GPU quota ကို ခွဲဝေ သတ်မှတ်ခြင်း (project
  တစ်ခုချင်းစီအတွက် quota ကွဲသတ်မှတ်နိုင်) — ရုတ်တရက် ငွေကုန်ကြေးကြီးမှုကို
  hard limit ဖြင့် ကာကွယ်ပါ (budget က alert သက်သက် ဖြစ်သောကြောင့်)။
  လိုအပ်ပါက quota decrease request ကို လည်း သုံးနိုင်ပါတယ်။
- **policy/process** — experiment အသစ် မစမီ ခန့်မှန်း GPU-hours တင်ပြရမည့်
  လုပ်ငန်းစဉ်၊ ကြီးမားသော run များအတွက် review အဆင့်။
- **self-service with guardrail** — Terraform module ဖြင့် team များကိုယ်တိုင် environment
  ထုတ်ယူခွင့် ပေးသော်လည်း module တွင် quota/tag/policy ကို built-in ထားပါ။

### 3) cost monitoring routine

- **နေ့စဉ်/အပတ်စဉ်** — **Billing → Usage** တွင် team scope အလိုက် ကြည့်ရှုခြင်း
- **လစဉ်** — **cost breakdown PDF** (Billing → Payments → Transactions →
  Download cost breakdown) ဖြင့် product အလိုက် အနှစ်ချုပ် review ခြင်း
- **လိုအပ်သလို** — နာရီအလိုက် **usage CSV export** ဖြင့် run တစ်ခုချင်းစီ၏ ကုန်ကျ
  ခွဲခြမ်းခြင်း (ဘယ် run က ကုန်ကျ မြင့်သလဲ ရှာဖွေရန်)
- **alert ရလျှင်** — budget scope အတိုင်း Usage တွင် filter လုပ်ပြီး အကြောင်းရင်း
  ရှာဖွေခြင်း — မျှော်လင့်ထားသော သုံးစွဲမှု ဖြစ်ပါက balance ဖြည့်ခြင်း၊ မဟုတ်ပါက
  resource ဖျက်ခြင်း

### 4) decommissioning စီမံခန့်ခွဲမှု

- ဖျက်သိမ်းမည် — GPU node/node group များ၊ run အတွက်သာ လိုသော VM နှင့် volume များ၊
  staging bucket အတွင်း မလိုတော့သော raw data၊ snapshot/checkpoint ဟောင်းများ၊
  managed cluster များ (PostgreSQL/MLflow — project အတွက်သာ လိုပါက)
- သိမ်းဆည်းမည် — final model (model registry)၊ လက်ခံနိုင်သော checkpoint နောက်ဆုံး
  version၊ ပြန်လည် အသုံးပြုရန် ရှိသော dataset (storage class ပြောင်း သို့မဟုတ် lifecycle rule)၊
  run မှတ်တမ်းနှင့် cost record များ
- တာဝန် — project owner က teardown checklist ကို လက်မှတ်ထိုးပြီးမှ handover
  ပြီးဆုံးကြောင်း သတ်မှတ်ပါ။ **VM stop ထားရုံနှင့် volume ကုန်ကျ ဆက်နေ**သောကြောင့်
  "stop ပြီး" ကို "ပြီးဆုံး" ဟု မမှတ်ယူပါနှင့်။

---

## Solution 3 — Cost vs performance: GPU အရေအတွက် ရွေးချယ်ခြင်း

### 1) GPU-hours ဖြင့် နှိုင်းယှဉ်ခြင်း

```
စုစုပေါင်း ကုန်ကျ ∝ GPU-hours = N × T = အလုပ်ပမာဏ ÷ scaling efficiency (η)
```

- **Option A (N=4, T=days)** — η မြင့် (overhead နည်း) သော်လည်း T ရှည် →
  GPU-hours = အလုပ်ပမာဏ ÷ η_A
- **Option B (N=16, T=days/≈4 အောက်)** — η_B < η_A ဖြစ်လေ့ရှိ (communication overhead)။
  η_B သည် η_A ၏ ၇၅% ထက် ပိုမြင့်ပါက GPU-hours ကျနိုင်ပြီး **စုစုပေါင်း ကုန်ကျ ကျ**နိုင်ပါတယ် —
  ဤသည်က "GPU ပိုသုံးခြင်းက ကုန်ကျ ကျစေနိုင်" သော အခြေအနေ ဖြစ်ပါတယ် (batch ကြီးပြီး
  per-GPU throughput မြင့်ခြင်း၊ parallelism ထိရောက်ခြင်း စသည်)။
  η_B နိမ့်လွန်းပါက GPU-hours တိုးပြီး ကုန်ကျ မြင့်နိုင်ပါတယ် — benchmark ဖြင့် အတည်ပြုပါ။
- **Option C (N=16, spot)** — GPU-hours ဗဟိုပြု ချဉ်းကပ်မှု Option B နှင့် တူသော်လည်း
  interruption ကြောင့် ပြန်လည် run ရမှု (rework) အတွက် GPU-hours အပို ထည့်ခန့်မှန်းပါ —
  ဥပမာ checkpoint ကြောင့် ပြန်စရသော အလုပ် နည်းပါက ထိခိုက်မှု သေးပါတယ်။

### 2) Option C design လိုအပ်ချက်များ

- **checkpoint frequency** — အလယ်အလတ် checkpoint ကို ပုံမှန် သိမ်းခြင်း (checkpoint ကြား
  အလုပ်ပမာဏသာ ပြန်လုပ်ရ) — checkpoint bucket ကို သီးခြား bucket တွင် သိမ်းခြင်း
- **resume/retry** — job queue က interruption ပြီး အလိုအလျောက် ပြန်စနိုင်ရမည်
- **interruption ကုန်ကျ** — spot စျေး ပိုနည်းသော်လည်း ပြန်လည် run GPU-hours နှင့်
  deadline ကြန့်ကြာမှု ပေါင်းပြီး စုစုပေါင်း ကြည့်ပါ — spot price သည် အချိန်နှင့်အမျှ
  ပြောင်းလဲနိုင်ကြောင်း ထည့်သွင်းစဉ်းစားပါ။

### 3) ရွေးချယ်မှု နမူနာ

deadline ၂ ပတ် တင်းကျပြီး run တစ်ခုလုံး ရက်ပေါင်းများစွာ ကြာမည် ဆိုပါက —
**mixed ပုံစံ** ကို ရွေးပါမည်: critical training run ကို Option B (on-demand) ဖြင့်
deadline အာမခံပြီး၊ ပြန်လည် လုပ်ဆောင်နိုင်သော အလုပ်များ (preprocessing, ablation,
evaluation) ကို Option C (spot) ဖြင့် လုပ်ဆောင်ခြင်းက စုစုပေါင်း ကုန်ကျ ကို လျှော့ချပေးပါတယ်။
Option A က GPU အနည်းဆုံး သုံးသော်လည်း deadline လွဲပါက စီမံကိန်း ကြန့်ကြာမှု ကုန်ကျက
GPU စျေးနှုန်း ကွာခြားချက်ထက် ကြီးမားနိုင်သောကြောင့် မရွေးပါ။

### 4) precision vs throughput

- အရင်ဆုံး quality requirement ကို သတ်မှတ်ပါ — မည်သည့် metric အောက် ကျပါက
  လက်ခံနိုင်သလဲ
- ထို့နောက် mixed precision (fp16/bf16) ကို benchmark run ဖြင့် စမ်းသပ်ပြီး
  quality မကျပါက သုံးပါ — throughput မြင့်ခြင်းက GPU-hours ကို တိုက်ရိုက် လျှော့ချပေးနိုင်
- quality မကိုက်ပါက precision ကို မလျှော့ဘဲ GPU အမျိုးအစား/အရေအတွက် ဘက်မှ
  ဖြေရှင်းပါ — "ဈေးအသက်သာဆုံး" ထက် "လိုအပ်ချက် ဖြည့်ဆည်းနိုင်သော အနိမ့်ဆုံး
  ကုန်ကျ" ကို ရွေးပါ

---

## Solution 4 — Multi-cloud data-ingestion pipeline

### 1) pipeline architecture

`source (other cloud) → transfer window အတွင်း ingest → staging bucket → validation/checksum
→ training data bucket → GPU cluster/Slurm → checkpoint bucket → model registry bucket`

staging bucket ထားရှိရသည့် အကြောင်းရင်း —

- ပြင်ပမှ ပို့လိုက်သော data ကို တိုက်ရိုက် training data သို့ မဝင်စေဘဲ checksum/validation
  ဖြတ်သန်းစေပြီး ပျက်နေ/မပြည့်စုံသော data ဖြင့် training မစခြင်းကို ကာကွယ်ပေး
- partial transfer ပျက်သွားပါက staging မှ ပြန်လည် စစ်ဆေး/ပြန်ပို့ရ လွယ်ကူ
- raw/staging data ကို lifecycle rule ဖြင့် သီးခြား ကုန်ကျစရိတ် စီမံခန့်ခွဲနိုင်

### 2) ingest bandwidth နှင့် transfer window

```
ပြောင်းရွှေ့ချိန် = dataset size ÷ effective transfer throughput
```

လိုအပ်သော အချက်အလက် — dataset အရွယ်အစား၊ ရရှိနိုင်သော link bandwidth နှင့် လက်တွေ့
effective throughput (overhead ထက် နိမ့်တတ်)၊ incremental update ပမာဏ။

- ပြောင်းရွှေ့ချိန်က training deadline မတိုင်မီ ပြီးရမည် — မပြီးနိုင်ပါက bandwidth
  တိုးခြင်း/data အစိတ်အပိုင်းခွဲပို့ခြင်း စဉ်းစားပါ
- transfer window — company network ကို အခြား workload များနှင့် ဝေမျှသုံးရသောကြောင့်
  သတ်မှတ် အချိန်ပိုင်း (ဥပမာ ညဘက်/weekend) တွင် ပို့ခြင်းက production traffic ကို
  မထိခိုက်စေပါ — လစဉ် incremental update ကိုလည်း ပုံမှန် window တစ်ခု သတ်မှတ်ပါ
- staging တွင် data ရောက်မှ training စခြင်းကို milestone အဖြစ် သတ်မှတ်ပါ

### 3) storage class နှင့် lifecycle

- actively သုံးနေသော training dataset ကို **Enhanced Throughput** တွင် ထားပါ —
  bandwidth ကန့်သတ်ချက် မပါဘဲ training စားသုံးမှု အမြန်နှုန်း ကူညီပြီး operations နှင့်
  data egress အခမဲ့ ဖြစ်သောကြောင့် data-movement များသော workload အတွက် သင့်လျေား
- experiment ပြီးဆုံးပြီး ၆ လ အကြာ ပြန်သုံးရန် ရှိသော dataset ကို **Intelligent** class
  တွင် ထားပါ — ဝင်ကြည့်မှု မရှိပါက ရက် ၃၀ ကျော်လျှင် Cold tier သို့ အလိုအလျောက်
  ပြောင်းပြီး ကုန်ကျ လျော့ကျပါတယ် (performance မပြောင်း) — ပြန်သုံးချိန်တွင်
  ပြန်လည် ဝင်ရောက် အသုံးပြုနိုင်ဆဲ ဖြစ်ပါတယ်
- **lifecycle rule** ဖြင့် staging ရှိ raw data ကို သတ်မှတ်ကာလအလိုက် Intelligent သို့
  ပြောင်းခြင်း/ဖျက်ခြင်း စီစဉ်ပါ — checkpoint ဟောင်းများကိုလည်း ထိုနည်းတူ စီမံပါ
- region ရွေးချယ်မှုတွင် storage class ရရှိနိုင်မှု ကွာခြားနိုင်သောကြောင့် documentation တွင်
  အတည်ပြုပါ

### 4) multi-cloud connectivity (network design)

- Nebius ဘက်တွင် **Virtual Networks** ဖြင့် network/subnet ကို customized ဖန်တီးပြီး
  external infrastructure နှင့် connectivity တည်ဆောက်ခြင်းကို documentation က
  customized networking infrastructure ၏ ဥပမာအဖြစ် ဖော်ပြထားပါတယ်
- IP addressing schema ကို ကြိုတင် စီစဉ်ပါ — CIDR block များ မထပ်စေရန် သတ်မှတ်ပြီး
  multi-region စီမံကိန်း ရှိပါက region အလိုက် ခွဲဝေပါ (လက်ရှိတွင် IPv4 သာ ပံ့ပိုး)
- data ပြောင်းရွှေ့မှုအတွက် source ဘက် ထွက်ပေါက် (egress) ကုန်ကျနှင့် Nebius ဘက်
  ingest ပမာဏကို နှစ်ဖက်စလုံးမှ ခန့်မှန်းပါ — egress ကုန်ကျသည် ပမာဏနှင့် ဦးတည်ရာပေါ်
  မူတည်သောကြောင့် (အသေးစိတ် နှုန်းထားကို pricing စာမျက်နှာတွင် အတည်ပြုပါ)
- security ရှုထောင့် — staging bucket ဝင်ရောက်ခွင့်ကို ကန့်သတ်ပြီး transfer
  credentials ကို secret စီမံခန့်ခွဲမှု ဖြင့် ကိုင်တွယ်ပါ

---

## Solution 5 — Idle resource နှင့် teardown policy

### 1) idle resource policy နမူနာ

| Resource | ပြန်လည် သုံးသပ်ရန် အခြေအနေ | လုပ်ဆောင်ချက် |
|---|---|---|
| GPU node / node group | GPU utilization နိမ့်ပြီး run ပြီးဆုံး; သို့မဟုတ် idle ရက် သတ်မှတ်အရေအတွက် ကျော် | run ပြီးပါက ပြန်လည် ဖျက်သိမ်း (scale down) |
| VM | run မလိုတော့ပါက | stop မဟုတ်ဘဲ **delete** — stop ထားပါက volume ကုန်ကျ ဆက်နေ |
| Volume / snapshot | snapshot အသက် သတ်မှတ်ကာလ ကျော် | ပြန်လည် သုံးသပ်ပြီး ဖျက် |
| Bucket object | checkpoint/raw data ဟောင်း | lifecycle rule ဖြင့် class ပြောင်း/ဖျက် |
| Managed cluster (PostgreSQL/MLflow) | project ပြီးဆုံး | cluster ဖျက် (ကုန်ကျ ရပ်စေသော လုပ်ဆောင်ချက်) |

### 2) end-of-project teardown checklist

1. training/inference run အားလုံး ပြီးဆုံးပြီး model ကို acceptance ပြုလုပ်ပြီးပြီ ဆိုခြင်း
   အတည်ပြု
2. သိမ်းဆည်းမည် — final model (model registry bucket)၊ နောက်ဆုံး checkpoint
   (လိုအပ်ပါက)၊ ပြန်အသုံးပြုရန် dataset (Intelligent class သို့ ပြောင်းပြီး)၊
   run မှတ်တမ်း/cost record
3. ဖျက်သိမ်းမည် — GPU node/node group အားလုံး၊ VM နှင့် **volume** များ (stop ထားရုံနှင့်
   မပြီး)၊ snapshot ဟောင်းများ၊ staging bucket အတွင်း raw data၊ run အတွက်သာ လိုသော
   managed cluster များ
4. အတည်ပြု — **Billing → Usage** တွင် project scope ဖြင့် ကုန်ကျမှု သုညနီးပါး
   ကျဆင်းသွားကြောင်း ရက်အနည်းငယ် စောင့်ကြည့် အတည်ပြု (VM ဖျက်ပြီး volume ကျန်ခဲ့သေးပါက
   ဤနေရာတွင် ဖမ်းမိပါမည်)
5. တာဝန်ခံ လက်မှတ်ဖြင့် checklist ပြီးဆုံးကြောင်း မှတ်တမ်းတင်

### 3) Terraform ၏ အခန်းကဏ္ဍ

- project တစ်ခုလုံးကို code ဖြင့် တည်ဆောက်ထားပါက teardown ကို environment တစ်ခုလုံး
  ဖြုတ်ချခြင်းဖြင့် လုပ်ဆောင်နိုင်ပြီး artifact များကို သီးခြား state/module တွင်
  ခွဲထားခြင်းက ဖျက်သိမ်းမှုကို ထိန်းချုပ်နိုင်စေ
- module တွင် idle/teardown အတွက် tag/label များ ထည့်သွင်းထားပါက ဘယ် resource က
  ဘယ် project အတွက်လဲ ခွဲခြား ရလွယ်
- **ကန့်သတ်ချက်** — Terraform က lifecycle policy (storage lifecycle rule) နှင့်
  utilization-based idle detection ကို ကိုယ်တိုင် မလုပ်ပေးပါ — policy/monitoring နှင့်
  ပေါင်းစပ် အသုံးပြုရပါမယ်။ ထို့အပြင် လက်ဖြင့် ဖန်တီးထားသော resource များကို
  state မပါဘဲ ဖျက်ရန် ခက်ခဲနိုင်ပါတယ် — "အားလုံးကို code ဖြင့်" စည်းမျဉ်း ထားပါ။

### 4) monitoring/alerting အကြံပြုချက်များ

1. **budget alert (50% / 100%)** ကို project တစ်ခုချင်းစီအတွက် ထားပြီး threshold ကျော်ပါက
   Usage တွင် အကြောင်းရင်း ရှာဖွေသော runbook ရေးပါ
2. **run ပြီးဆုံးပြီးနောက် N ရက် ကျော်လျှင်** GPU node များ ဆက်ရှိနေပါက သတိပေးသော
   စောင့်ကြည့်မှု (usage/asset စာရင်းအပေါ် အခြေခံ) — idle resource ကို လူက မမေ့စေရန်
3. (နောက်ထပ်) လစဉ် cost breakdown PDF review ကို ပုံမှန် အစီအစဉ်အဖြစ် ထားပါ

---

## Technical review notes

- ဖြေရှင်းချက်များတွင် ပါဝင်သော Nebius လုပ်ဆောင်ချက် အချက်အလက်များ (quota ပြောင်းလဲ
  လုပ်ငန်းစဉ်၊ budget scope/threshold/recipient၊ Usage/CSV/cost breakdown tool များ၊
  storage class နှင့် lifecycle၊ PAYG "stop being charged" လုပ်ဆောင်ချက်များနှင့်
  VM volume ကုန်ကျမှု) ကို docs.nebius.com တွင် အတည်ပြုပြီး ဖြစ်ပါတယ်။
- **Needs verification** — spot/preemptible ၏ interruption အသေးစိတ် အပြုအမူနှင့် pricing policy
  အသေးစိတ်၊ scaling efficiency ကိန်းဂဏန်း ဥပမာများ (illustrative သက်သက် — ကိုယ်ပိုင်
  benchmark ဖြင့် အစားထိုးရန်)၊ egress နှုန်းထား အတိအကျများ။ စျေးနှုန်း ကိန်းဂဏန်း
  တစ်ခုမျှ မဖော်ပြထားပါ။
