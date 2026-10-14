# Module 02 — Solution: Design Resilient Architectures

> `exercise.md` ရှိ design scenario ငါးခုအတွက် အဖြေ အပြည့်အစုံ ဖြစ်ပါသည်။
> ဖြေရှင်းပုံတွင် ဆုံးဖြတ်ချက်နှင့် အကြောင်းပြချက်ကို အတွဲဖြင့် ဖော်ပြထားပြီး
> Nebius ထုတ်ကုန် အချက်အလက်များကို docs.nebius.com အတိုင်း ကိုးကားထားပါသည်။

---

## Solution 1 — Checkpoint နှင့် restart strategy ဒီဇိုင်း

### 1) Checkpoint frequency ရွေးချယ်မှု

ဂဏန်း သီးသန့် မသတ်မှတ်ဘဲ အောက်ပါ အချက်သုံးချက်ကို အခြေခံ၍ ဆုံးဖြတ်သည် —

- **Overhead budget** — checkpoint write အတွက် training time ရဲ့ ရာခိုင်နှုန်း
  ဘယ်လောက် ခွင့်ပြုမလဲ (run အတွက် အနှောင့်အယှက် အနည်းဆုံး ဖြစ်စေမည့်
  ပမာဏ)
- **Expected lost work** — node failure ကြားချိန်ခန့်မှန်းချက် (MTBF) နှင့်
  checkpoint ကြာချိန် — failure က ကြိုမသိနိုင်သဖြင့် ပျမ်းမျှ ဆုံးရှုံးမှုက
  checkpoint ကြာချိန်၏ တစ်ဝက်ခန့် ဖြစ်သည်
- **Checkpoint အရွယ်နှင့် restore ချိန်** — optimizer state အပါအဝင်
  checkpoint ကြီးမားပါက write နှင့် load ချိန် နှစ်ခုလုံး ထည့်တွက်ရသည်

ဒီဇိုင်း ဆုံးဖြတ်ချက် — checkpoint overhead budget (ဥပမာ training time ၏
အနည်းငယ်) အတွင်း ကျရုံ အတိုဆုံး ကြာချိန် သတ်မှတ်ပြီး MTBF နှင့်
နှိုင်းယှဉ်၍ ချိန်ညှိသည်။ 64 GPU ကဲ့သို့ cluster ကြီးတွင် node failure
ဖြစ်နိုင်ခြေ မြင့်သဖြင့် run ရှည် (၇ ရက်) အတွက် ကြာချိန်ကို အလွန်ရှည်ခွင့်
မပေးပါ — ဥပမာအားဖြင့် "overhead 5% အောက် ကျနိုင်သမျှ တိုအောင်" ဆိုသော
စည်းမျဉ်းဖြင့် စမ်းသပ် တိုင်းတာ၍ သတ်မှတ်သည် (ဂဏန်း သီးသန့် မဟုတ်ပါ —
workload ပေါ် မူတည်သည်)။

### 2) Synchronous vs asynchronous

**Asynchronous checkpointing ကို ရွေးသည်** — အကြောင်းမှာ run ရှည်ပြီး
checkpoint အရွယ် ကြီးမားသဖြင့် synchronous ရပ်နားချိန်က စုစုပေါင်း
training အချိန်ကို သိသိသာသာ ထိခိုက်စေနိုင်သည်။ သို့သော် —

- သိမ်းဆည်းထားသော state သည် မည်သည့် step အတွက်ဖြစ်ကြောင်း metadata တွင်
  ရှင်းလင်းစွာ သတ်မှတ်ထားရမည်
- write မပြီးမီ failure ဖြစ်ပါက checkpoint ဆုံးရှုံးနိုင်သဖြင့် ရှေ့နောက်
  checkpoint နှစ်ခု အနည်းဆုံး ထားရှိရမည်
- in-flight state copy အတွက် memory သုံးစွဲမှု တိုးလာနိုင်သဖြင့် HBM/system
  memory budget တွင် ထည့်တွက်ရမည်

Checkpoint အရွယ် သေးငယ်ပြီး overhead ခံနိုင်ပါက synchronous က
ရိုးရှင်း၍ ပိုစိတ်ချရသည် — ဤနေရာတွင် "run အရှည် × checkpoint အရွယ်"
က ဆုံးဖြတ်ချက် ဖြစ်သည်။

### 3) Storage ရွေးချယ်မှု — ပေါင်းစပ် အသုံးပြုပုံ

| Storage | အသုံးပြုပုံ |
|---|---|
| Local disk | မသုံးပါ (node failure တွင် checkpoint ပါ ဆုံးရှုံးမည်) — အလွန်ဆို staging အတွက်သာ |
| Shared filesystem | active run ၏ နောက်ဆုံး checkpoint N ခု — restart မြန်စေရန် |
| Object Storage | durable copy နှင့် ရေရှည် retention — shared filesystem ပျက်မှု ကာကွယ်ရန် async replicate |

Nebius AI Cloud တွင် Compute shared filesystem ကို VM များနှင့် attach/mount
လုပ် အသုံးပြုနိုင်ပြီး (docs: compute/storage/manage — VM နှင့် volume project
တူရမည်၊ READ_WRITE attach mode နှင့် mount tag သတ်မှတ်နိုင်) Object Storage ကို
မူ log export ကဲ့သို့ durable object သိမ်းဆည်းမှုများတွင် အသုံးပြုပြီးသား ဖြစ်သည်။
Checkpoint ကို shared filesystem ပေါ်တွင် အရင်ရေးပြီး Object Storage သို့
async replicate လုပ်သော pattern က recovery နှစ်ဆင့် ပေးသဖြင့် ရွေးချယ်သည်။

### 4) Restart granularity

**Job-level restart ကို အခြေခံ သတ်မှတ်သည်** — checkpoint ကနေ run တစ်ခုလုံး
ပြန်စခြင်း။ အဘယ့်ကြောင့်ဆိုသော် ဒီဇိုင်း ရိုးရှင်းပြီး framework အမျိုးမျိုးနှင့်
တည်ငြိမ်စွာ အလုပ်လုပ်နိုင်သည်။ Node-level/elastic restart (ပျက်သွားသော node
ကို ဖယ်၍ ကျန် node များဖြင့် ဆက်လုပ်ခြင်း) ကိုမူ framework က global batch size
ပြောင်းလဲမှုကို ကိုင်တွယ်နိုင်ပါက ရွေးချယ်နိုင်ပြီး မလိုအပ်ပါက မသုံးပါ —
များသောအားဖြင့် job-level restart နှင့် cluster ပြန်လည် တည်ဆောက်မှု
(node အစားထိုးပြီးမှ ပြန်စ) က ပို၍ ခန့်မှန်းရ လွယ်သည်။

### 5) Restore drill — "ရှိခြင်း" နှင့် "အလုပ်လုပ်ခြင်း"

Production run မစမီ အောက်ပါ စမ်းသပ်ချက် လုပ်ဆောင်ရမည် —

1. လက်ရှိ checkpoint တစ်ခုကို ရွေးပြီး သီးသန့် environment တစ်ခုတွင်
   restore လုပ်ကြည့်သည်
2. Restore ပြီးနောက် training ကို အနည်းငယ် ဆက်လုပ်ပြီး step counter,
   loss curve ဆက်လက် မှန်ကန်မှု စစ်ဆေးသည်
3. ဖြစ်နိုင်ပါက node failure တစ်ခု တမင် ဖန်တီးပြီး restart အချိန် တိုင်းတာသည်
   (RTO အတွက်)
4. ရလဒ်ကို acceptance checklist တွင် မှတ်တမ်းတင်သည်

ဒီ drill မပါဘဲ run စခြင်းက corruption/missing file/ version mismatch များကို
failure ဖြစ်မှသာ ဖော်ထုတ်မိတတ်သည်။

---

## Solution 2 — Observability scope ဒီဇိုင်း

### 1) Golden signals ပြန်ဖွဲ့စည်းမှု

| Signal | Training platform အတွက် တိုင်းတာမည့် အရာ |
|---|---|
| Latency | step time, checkpoint save time, job queue wait time |
| Traffic | samples/sec သို့ tokens/sec, data loader throughput |
| Errors | job failure count, restart count, NCCL timeout, Xid/ECC error count |
| Saturation | GPU utilization, HBM ပြည့်မှု, storage I/O, InfiniBand utilization |

### 2) Logs / metrics / traces ဘာကောက်မလဲ

- **Metrics** — အထက်ပါ signal များကို node, job, GPU အဆင့်ခွဲ၍ ကောက်သည်။
  GPU utilization မြင့်နေသော်လည်း throughput ကျနေပါက saturation မဟုတ်ဘဲ
  communication/storage bottleneck ရှိကြောင်း ညွှန်းသဖြင့် metrics အမျိုးစုံ
  လိုအပ်သည်
- **Logs** — job lifecycle events (start/stop/restart), checkpoint save/load
  အောင်မြင်/မအောင်မြင်, NCCL warnings/timeouts, kernel/Xid errors,
  Kubernetes node events — "ဘာဖြစ်ခဲ့လဲ" ကို ဖြေပေးသည်
- **Traces** — step တစ်ခုအတွင်း data loading / forward / backward /
  allreduce span များ — "ဘယ်အဆင့်မှာ ကြာနေလဲ" ကို ဖြေပေးသည်။ framework
  profiler များ (ဥပမာ PyTorch profiler, Nsight) မှ ထုတ်ယူပြီး သီးသန့် pipeline
  ဖြင့် စုဆောင်းသည် (Nebius built-in trace ingestion အတည်မပြုနိုင်သေးပါ —
  needs verification)

### 3) Nebius AI Cloud Observability ပေါ်တွင် ဆက်တင်ထားပုံ

- Metrics အတွက် service account တစ်ခုကို `viewer` အနည်းဆုံး ပါဝင်သော group
  ထဲ ထည့်ပြီး OBSERVABILITY static key ထုတ်ကာ Prometheus federate endpoint
  (`read.monitoring.<region>.nebius.cloud`) ကို ချိတ်သည် — GPU metrics ကို
  `gpu` bucket မှ scrape သည်
- **Shard အကြံဉာဏ်** — GPU အရေအတွက် များပါက `match[]` selector ဖြင့်
  `uuid` label အလိုက် scrape job များ ခွဲသည် (ဥပမာ `GPU-[0-7]`, `GPU-[8-9a-f]`
  — docs မှ ဥပမာ ပုံစံ)။ selector များ ထပ်နေမှု မရှိစေရ၊ scrape interval ကို
  15 စက္ကန့် အနည်းဆုံး ထားသည်
- Logs အတွက် Observability → Logs တွင် user logs (Managed Service for
  Kubernetes မှ ingest) နှင့် service logs ကို ကြည့်ပြီး LogQL ဖြင့် filter
  လုပ်သည်။ ကိုယ်ပိုင် application logs ကိုလည်း ingest ထည့်နိုင်သည်
- Dashboards ကို role အလိုက် ခွဲပေးသည် — operator view (alerts + golden
  signals) နှင့် deep-dive view (per-GPU/per-job)

### 4) Alert design

| Severity | ဥပမာ အခြေအနေ | Routing |
|---|---|---|
| SEV1 | training run ရပ် / job failure rate မြင့် | on-call ကို ချက်ချင်း page + architect ကို အသိပေး |
| SEV2 | throughput သိသိသာသာ ကျ / checkpoint save မအောင်မြင် ထပ်ဖြစ် | ops ticket queue |
| SEV3 | node တစ်ခု ပြန်ကောင်း / single GPU degradation | မှတ်တမ်း + အလုပ်ချိန်အတွင်း ကိုင်တွယ် |

- Alert ကို symptom အပေါ် ထားသည် — "GPU utilization ကျ" ထက် "training
  throughput ကျ" က user impact ကို တိုက်ရိုက် ဖော်ပြသည်
- Alert တစ်ခုချင်းစီတွင် runbook link ထည့်သည် — alert ဖြစ်ပါက ဘာလုပ်ရမည်
  ကြိုတင် ရှင်းလင်းစေရန်
- Alert fatigue ကာကွယ်ရန် — ထပ်ဖြစ်နေသော်လည်း ဘယ်သူမှ မလုပ်သော alert
  ကို threshold ပြန်ညှိခြင်း/ပိတ်ခြင်း လုပ်ပြီး alert အသစ်ထည့်ခြင်းထက်
  အရည်အသွေးကို ဦးစားပေးသည်
- Nebius console ရှိ Observability → Alerts ဖြင့် VM နှင့် Object Storage
  bucket အတွက် threshold+duration အခြေခံ alert နှင့် email notification
  ထားနိုင်ပြီး (`Firing`/`Pending`/`Normal` အခြေအနေများ) paging လိုအပ်ပါက
  external notification pipeline ကို ထပ်ချိတ်ရသည်

### 5) Retention

- **Hot** — active debugging အတွက် လတ်ဆတ်သော metrics/dashboards နှင့်
  logs (console/Grafana မှ တိုက်ရိုက် ကြည့်နိုင်သည့် ကာလ)
- **Cold** — log export ကို Object Storage bucket ထဲ `.parquet` သို့
  `.json.gzip` အဖြစ် သိမ်းဆည်းခြင်း (docs: observability/logs/export —
  `nebius logging export create` ဖြင့်လည်း လုပ်နိုင်သည်)
- ကာလ ဂဏန်းများကို ကိုယ့် compliance လိုအပ်ချက်အလိုက် သတ်မှတ်ပြီး platform
  limit ဂဏန်းများကို တရားဝင် အတည်မပြုနိုင်သေးပါ (needs verification)

---

## Solution 3 — Validation, benchmark နှင့် acceptance plan

### 1) စစ်ဆေးမှု အဆင့်များ

| အဆင့် | Tool/command အမျိုးအစား | Pass criteria |
|---|---|---|
| Physical/link | `ibstatus` | node တိုင်းရှိ InfiniBand device အားလုံး `state: ACTIVE` နှင့် `phys state: LinkUp` |
| Bandwidth | `perftest` ၏ `ib_send_bw --report_gbits` (VM နှစ်ခုကြား) | bytes sent, average bandwidth, message rate အားလုံး non-zero ဖြစ်ပြီး design threshold ထက် မနိမ့် (peak က theoretical max ကို မမီနိုင်ကြောင်း docs တွင် ဖော်ပြထား) |
| Collective | NCCL tests (`all_reduce_perf`) | average bus bandwidth သည် design threshold ထက် မနိမ့် — docs (mpirun) အရ 300 GB/s အထက် ဆိုပါက connection တည်ငြိမ်ကြောင်း ဆိုထားသည် |
| Application | workload benchmark (model အတိုင်းအတာ ငယ် version) | baseline နှင့် ကွာခြားမှု သတ်မှတ်ကန့်သတ်ချက် အတွင်း |

### 2) Threshold သတ်မှတ်ချိန်

**Test မစမီ** သတ်မှတ်ရမည် — ရလဒ် ထွက်ပြီးမှ threshold ကို ရလဒ်နှင့် ညှိပါက
acceptance မဟုတ်တော့ပါ။ Threshold များကို design requirement (training
run ပြီးရန် လိုအပ်သော စွမ်းဆောင်ရည်) မှ ဆင်းသက် ချမှတ်ရပါမည်။

### 3) ရလဒ် စီမံခန့်ခွဲမှု

- Test ရလဒ် အားလုံးကို ရက်စွဲနှင့်တကွ မှတ်တမ်းတင်ပြီး baseline repository
  (version control) တွင် သိမ်းဆည်းသည်
- နောက်ပိုင်း "နှေးလာသည်" ဆိုသော တိုင်ကြားမှုများကို baseline နှင့်
  နှိုင်းယှဉ်၍ regression ဖြစ်/မဖြစ် ဆုံးဖြတ်နိုင်စေသည်
- Node အသစ် ထပ်ထည့်ခြင်း/node အစားထိုးပြီးခြင်းတွင် အဆင့် 1–3 ကို
  ပြန်လည် run ပြီး ရလဒ် အသစ် ထပ်မှတ်သည်

### 4) Resilience စမ်းသပ်ချက်

**Checkpoint restore drill + node failover test** — training run အတု တစ်ခုကို
စတင်ပြီး ကြားချိန်တွင် node တစ်ခုကို ရပ်လိုက်သည် (သို့မဟုတ် Managed
Kubernetes ၏ automatic recovery အခြေအနေ တစ်ခုကို ဖန်တီးသည်)။ ထို့နောက် —

1. Node problem ကို platform က ကိုယ်တိုင် ကိုင်တွယ်ပုံ (NPD condition များ
   တက်လာခြင်း၊ node recovery) ကို စောင့်ကြည့်သည်
2. Job က နောက်ဆုံး checkpoint မှ ပြန်စခြင်း အလုပ်လုပ်/မလုပ် စစ်ဆေးသည်
3. Restart ကြာချိန် (RTO) နှင့် ဆုံးရှုံးသော အလုပ်ပမာဏ (RPO သဘော) ကို
   တိုင်းတာပြီး runbook တွင် မှတ်တမ်းတင်သည်

### 5) Sign-off checklist

- [ ] Link/bandwidth/collective/application test အားလုံး pass criteria ပြည့်မီ
- [ ] Checkpoint restore drill အောင်မြင်ပြီး restart ကြာချိန် မှတ်တမ်းတင်ပြီး
- [ ] Dashboards နှင့် alerts အသုံးပြုနိုင်ပြီး test alert များ တုံ့ပြန်မှု ရှိ
- [ ] Runbooks နှင့် documentation ပြီးစီးပြီး ops team လက်ခံ
- [ ] Access model (IAM) လွှဲပြောင်းပြီး

---

## Solution 4 — Failure handling နှင့် operational support model

### 1) Runbook tiers

| Tier | တာဝန် | လုပ်ပိုင်ခွင့် |
|---|---|---|
| L1 — CloudOps on-call (၃ ဦး) | alert verify, runbook အတိုင်း ပထမဆုံး တုံ့ပြန်, job restart | job restart ခွင့်ပြု; node ဖျက်ခြင်း မပြုရ (L2 သို့ escalate) |
| L2 — platform engineer (၂ ဦး) | runbook မဖြေရှင်းနိုင်သော ပြဿနာ, node replacement/reimage, driver/infra change | node ဖျက်/အသစ်ဖန်တီးခွင့် ပြု; hardware fault အတည်ပြုပြီးမှ L3 သို့ |
| L3 — Nebius support | hardware/vendor အဆင့် | RMA ဖြင့် hardware အစားထိုးတောင်းဆို |
| Architect | design-level ဆုံးဖြတ်ချက် | capacity/design change, degraded mode ဆုံးဖြတ်ခြင်း |

### 2) Escalation matrix (SLA ဂဏန်းများ = အဖွဲ့ placeholder)

| Severity | သတ်မှတ်ချက် | ပထမ တုံ့ပြန်မှု | Escalation |
|---|---|---|---|
| SEV1 | training run ရပ် / ထုတ်လုပ်မှု ရပ်တန့် | on-call ကို ချက်ချင်း page — 15 မိနစ်အတွင်း အတည်ပြု (placeholder) | 30 မိနစ် မတုံ့ပြန်ပါက L2 → 1 နာရီအတွင်း architect အသိပေး (placeholder) |
| SEV2 | performance သိသိသာသာ ကျ / redundancy ဆုံးရှုံး | ops ticket — အလုပ်ချိန်အတွင်း အတည်ပြု | 24 နာရီအတွင်း L2 (placeholder) |
| SEV3 | အနည်းငယ် ထိခိုက် / သတိပေးချက် | အလုပ်ချိန်အတွင်း ကိုင်တွယ် | လိုအပ်ပါက L2 |

(ဤအချိန် ဂဏန်းများသည် ဥပမာ placeholder သာ ဖြစ်ပြီး Nebius တရားဝင် SLA
မဟုတ်ပါ — ကိုယ့်အဖွဲ့ သဘောတူညီချက်ဖြင့် သတ်မှတ်ပါ)

### 3) On-call design

- Rotation ကို L1 (operator ၃ ဦး) အလှည့်ကျ သတ်မှတ်ပြီး တစ်ပတ်စီ လှည့်သည်
- Handoff မှတ်တမ်း — shift ပြောင်းချိန်တွင် ဖြစ်ပွားနေသော incident နှင့်
  စောင့်ကြည့်နေသော ကိစ္စများကို လက်ဆင့်ကမ်းသည်
- Escalation timeout — သတ်မှတ်ချိန်အတွင်း မတုံ့ပြန်ပါက အောက်အဆင့်မှ
  အထက်အဆင့်သို့ အလိုအလျောက် တက်သည်
- On-call ကို page လုပ်ရမည့် incident အမျိုးအစားကို ကြိုတင် သတ်မှတ်ထားပြီး
  SEV3 များ page မလုပ်ပါ — alert fatigue ကာကွယ်ရန်

### 4) RMA flow နှင့် architect ၏ ဆုံးဖြတ်ချက်

1. L2 က `nvidia-smi`/`dcgmi`/`dmesg` (Xid/ECC) စသည့် tool များဖြင့် hardware
   fault ကို အတည်ပြုပြီး node ကို cordon/isolate လုပ်သည်
2. Nebius support သို့ ticket ဖွင့်ပြီး RMA/အစားထိုး တောင်းဆိုသည်
   (အစားထိုး node ရရှိမှု ကာလကို support က အတည်ပြုမည် — ဂဏန်း မဖော်ပြပါ,
   needs verification)
3. **Architect ၏ design-level ဆုံးဖြတ်ချက်** — အစားထိုး node မရမီ ကာလအတွင်း
   (က) run ကို node အနည်းငယ် လျှော့ပြီး ဆက်မည်လား (global batch size
   သို့ data parallel degree ပြောင်းလဲမှု ကိုင်တွယ်နိုင်ပါက)၊ (ခ) ရပ်ထားပြီး
   checkpoint ကို ကာကွယ်မည်လား၊ (ဂ) အခြား node များဖြင့် ပြန်ဖွဲ့မည်လား —
   ဤဆုံးဖြတ်ချက်သည် architect တာဝန်ဖြစ်ပြီး runbook က ဆုံးဖြတ်ပေးမည် မဟုတ်ပါ
4. Hardware အစားထိုးပြီးပါက အဆင့် 1–3 validation ပြန် run ပြီး cluster ထဲ
   ပြန်ထည့်သည်

### 5) Incident ပြီးဆုံးပြီးနောက်

- **Blameless post-incident review** — ဖြစ်ပွားပုံ၊ တုံ့ပြန်ပုံ၊ ပြန်လည် ကာကွယ်
  နိုင်မည့် အစီအမံ ရေးမှတ်သည်
- Runbook ကို ပြန် update လုပ်သည် — runbook မပါသော အဆင့် တစ်ဆင့်က
  နောက် incident တွင် အချိန်ထပ်ကုန်စေသည်
- Alert က အလုပ်မလုပ်/မလုပ် (အချိန်မီ မသိရှိရခြင်း) ကို ပြန်စစ်ပြီး
  observability scope ကို ဖြည့်စွက်သည်
- လိုအပ်ပါက training/knowledge sharing အတွက် ops team အား ပြန်လည်
  လေ့ကျင့်ပေးသည်

---

## Solution 5 — Migration နှင့် handover path

### 1) Handover deliverables

1. **Architecture documentation** — component များ၊ data flow၊ design decision
   တစ်ခုချင်းစီ၏ အကြောင်းပြချက် (ဘာကြောင့် ဤ storage/ဤ topology ကို
   ရွေးခဲ့သနည်း)
2. **Runbooks** — incident အမျိုးအစားအလိုက် အဆင့်ဆင့် လုပ်ဆောင်ပုံနှင့်
   escalation လမ်းကြောင်း (Exercise 4 ရှိ support model နှင့် ချိတ်ဆက်)
3. **Infrastructure as Code** — Terraform configuration နှင့် state လက်ခံ
   လွှဲပြောင်းမှု — platform တစ်ခုလုံးကို reproduce လုပ်နိုင်ရန်
4. **Observability assets** — dashboards, alert rules, alert routing
   (observability as code ဖြစ်နိုင်ပါက ပိုကောင်းသည်)
5. **Access model နှင့် training** — IAM group/service account/role များ
   လွှဲပြောင်းမှု၊ ops team walkthrough နှင့် tabletop incident drill
   မှတ်တမ်း၊ acceptance နှင့် sign-off checklist

### 2) Handover လမ်းကြောင်း

1. **Documentation ပြီးစီး** — project အတွင်း အဆက်မပြတ် update လုပ်ထားသော
   docs ကို review လုပ်ပြီး ကွာဟချက် ဖြည့်သည်
2. **Shadowing** — ops team က incident drill နှင့် runbook exercise များတွင်
   project team နှင့်အတူ ပါဝင် လေ့ကျင့်သည်
3. **Hypercare period** — လွှဲပြောင်းပြီး ကာလတစ်ခုအထိ project team က
   backup support ပေးပြီး ops team က ဦးဆောင် လုပ်ဆောင်သည်
4. **Sign-off** — acceptance checklist ပြည့်မီပြီး documentation review
   ပြီးဆုံးမှ တာဝန် လွှဲပြောင်းမှု ပြီးဆုံးကြောင်း လက်မှတ်ရေးထိုးသည်

### 3) RACI

| အလုပ် | Architect | CloudOps operator | ML team |
|---|---|---|---|
| Design decision (capacity, topology) | Accountable | Consulted | Consulted |
| Runbook execution | Informed | Responsible | — |
| Training job လည်ပတ်မှု | Consulted | Consulted | Responsible |
| Incident အပြီး review | Accountable | Responsible | Informed |

Architect က ဒီဇိုင်း ဆုံးဖြတ်ချက်များအတွက် accountable ဖြစ်ပြီး operator က
procedure လိုက်နာ ဆောင်ရွက်မှုအတွက် responsible ဖြစ်သည် — runbook တွင်
ဆုံးဖြတ်ချက် မပါစေရန် ဒီဇိုင်းဆွဲထားပြီး design-level ဆုံးဖြတ်ချက်များကို
architect ဆီ တက်ရမည့် လမ်းကြောင်း သတ်မှတ်ထားသည်။

### 4) Handover မအောင်မြင်တတ်သော အချက်များနှင့် ကာကွယ်နည်း

| အချက် | ကာကွယ်နည်း |
|---|---|
| Documentation ဟောင်း/မပြည့်စုံ — incident မှာ runbook အလုပ်မလုပ် | Handover မတိုင်မီ drill ဖြင့် runbook တစ်ခုချင်း စမ်းသပ် |
| Knowledge က လူ တစ်ယောက်တည်းတွင် စုပုံနေခြင်း | Shadowing နှင့် pair-on-call ဖြင့် အနည်းဆုံး လူနှစ်ယောက် ကျွမ်းကျင်စေခြင်း |
| Access လွှဲပြောင်းမှု နောက်ကျခြင်း | IAM လွှဲပြောင်းမှုကို handover အစီအစဉ်တွင် ကြိုတင် အဆင့် သတ်မှတ် |

---

## Technical review notes

- Nebius ထုတ်ကုန် အချက်အလက်များ (Observability metrics/Prometheus/logs/
  alerts, NPD health checks, `ibstatus`/`ib_send_bw`, NCCL tests, storage
  manage) ကို docs.nebius.com တွင် အတည်ပြုပြီး ဖြစ်ပါသည်။
- NCCL average bus bandwidth 300 GB/s ကို docs.nebius.com/3p-integrations/mpirun
  တွင် connection တည်ငြိမ်မှု လက္ခဏာအဖြစ် ဖော်ပြထားခြင်း ဖြစ်ပြီး
  acceptance threshold အဖြစ် မသတ်မှတ်ထားပါ — ကိုယ့် design threshold ကို
  ကိုယ်ပိုင် သတ်မှတ်ပါ။
- Support SLA အချိန်များ (15/30 မိနစ်၊ 24 နာရီ စသည်) နှင့် RMA/အစားထိုး
  node ရရှိမှု ကာလများသည် ဥပမာ placeholder များသာ ဖြစ်ပြီး တရားဝင် ဂဏန်း
  မဟုတ်ပါ (needs verification)။
- လွှဲပြောင်းမှု hypercare ကာလ သတ်မှတ်ချက်သည် အဖွဲ့အလိုက် သဘောတူညီချက်
  ဖြစ်ပြီး တရားဝင် စံ မဟုတ်ပါ။
