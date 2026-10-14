# Module 02 — Exercise: Design Resilient Architectures

> ဤလေ့ကျင့်ခန်းများသည် exam ရှုထောင့်မှ design scenario များ ဖြစ်ပါသည်။
> command များ ရိုက်ထည့်ရန် မဟုတ်ဘဲ design decision များကို အကြောင်းပြချက်နှင့်
> ချမှတ်နိုင်စွမ်း လေ့ကျင့်ရန် ဖြစ်ပါသည်။ ဖြေရှင်းနည်း အပြည့်အစုံကို
> `solution.md` တွင် ကြည့်ပါ။

---

## Exercise 1 — Checkpoint နှင့် restart strategy ဒီဇိုင်း

သင်သည် LLM training run တစ်ခုအတွက် architecture ကို ဒီဇိုင်းဆွဲရမည် —
ခန့်မှန်းခြေ ၇ ရက် ကြာမည့် run ကို 64 GPU (8 node × 8 GPU) ဖြင့် လုပ်ဆောင်မည်။
node တစ်ခုချင်းစီ၏ failure rate က ကြိုတင် မသိနိုင်သော်လည်း cluster အရွယ်
ကြီးမားသဖြင့် run အတွင်း node failure ဖြစ်နိုင်ခြေ မြင့်မားကြောင်း သိရှိထားသည်။
Optimizer state အပါအဝင် checkpoint တစ်ခု၏ အရွယ်အစားက model အရွယ်ပေါ်
မူတည်၍ ကြီးမားနိုင်သည်။

ဒီဇိုင်း ဆုံးဖြတ်ချက်များ ချမှတ်ပါ —

1. Checkpoint ကြာချိန် (frequency) ကို မည်သို့ ရွေးချယ်မည်နည်း? ဘယ်အချက်
   သုံးချက်ကို အခြေခံ၍ ဆုံးဖြတ်မည်နည်း (ဂဏန်း သီးသန့် မလိုပါ)?
2. Synchronous နှင့် asynchronous checkpointing မှ မည်သည်ကို ရွေးမည်နည်း?
   အကြောင်းပြချက် ပေးပါ။
3. Checkpoint ကို မည်သည့် storage ပေါ်တွင် သိမ်းမည်နည်း (local disk,
   shared filesystem, Object Storage)? ပေါင်းစပ် အသုံးပြုမည်ဆိုပါက
   မည်သို့ ခွဲဝေမည်နည်း?
4. Restart granularity ကို မည်သို့ သတ်မှတ်မည်နည်း?
5. "Checkpoint ရှိတယ်" ဆိုတာနှင့် "restore လုပ်လို့ ရတယ်" ဆိုတာ မတူကြောင်း
   ထည့်သွင်း စဉ်းစားမည့် စမ်းသပ်ချက်တစ်ခု ထည့်ပါ။

---

## Exercise 2 — Observability scope ဒီဇိုင်း

Training platform တစ်ခုအတွက် observability scope ကို ဒီဇိုင်းဆွဲရမည်။
platform တွင် Managed Service for Kubernetes ပေါ်၌ training jobs များ တက်နေပြီး
GPU node များ InfiniBand ဖြင့် ချိတ်ဆက်ထားသည်။ လက်ရှိတွင် problem ဖြစ်မှ
log များ ပြန်ရှာတတ်သော်လည်း ကြိုတင် သတိပေးချက် စနစ် မရှိပေ။

1. Golden signals ကို training platform အတွက် မည်သို့ ပြန်ဖွဲ့စည်းမည်နည်း?
   metric အမည် မဟုတ်ဘဲ တိုင်းတာမည့် signal အမျိုးအစားများကို ဖော်ပြပါ။
2. Logs, metrics, traces တစ်ခုချင်းစီအတွက် ဘာ data ကောက်ယူမည်နည်း?
   ဘယ် signal က ဘယ် မေးခွန်းကို ဖြေပေးနိုင်သနည်း?
3. Nebius AI Cloud Observability ပေါ်တွင် metrics နှင့် logs ကို မည်သို့
   ဆက်တင်ထားမည်နည်း? (GPU metrics scrape ခွဲခြမ်း/shard လုပ်မည့် အကြံဉာဏ်
   တစ်ခု ပါ ပေးပါ)
4. Alert design — ဘယ်အခြေအနေများအတွက် alert ထားမည်နည်း? severity နှင့်
   routing ကို မည်သို့ ခွဲမည်နည်း? alert fatigue ကို မည်သို့ ရှောင်ရှားမည်နည်း?
5. Retention ကို hot/cold ခွဲ၍ မည်သို့ စီမံမည်နည်း?

---

## Exercise 3 — Validation, benchmark နှင့် acceptance plan

32 node (8 GPU/node) GPU cluster အသစ် တစ်ခု ရောက်ရှိလာမည်။ ML team က
cluster ပေါ်တွင် training run ကြီး စတင်တော့မည်။ သင်က acceptance plan
တစ်ခု ရေးဆွဲရမည် —

1. စစ်ဆေးမည့် အဆင့်များကို အနည်းဆုံး သုံးဆင့် စီစဉ်ပြီး တစ်ဆင့်ချင်းအတွက်
   အသုံးပြုမည့် tool/command အမျိုးအစားနှင့် အောင်မြင်မှု သတ်မှတ်ချက်
   (pass criteria) ရေးပါ။
2. Acceptance threshold များကို မည်သည့်အချိန်တွင် သတ်မှတ်ရမည်နည်း?
   ဘာကြောင့်နည်း?
3. ရလဒ်များကို မည်သို့ စီမံမည်နည်း (baseline သိမ်းဆည်းမှု အပါအဝင်)?
4. Resilience စမ်းသပ်ချက် တစ်ခု ထည့်ပါ — checkpoint restore drill သို့မဟုတ်
   node failover test အနေဖြင့် စဉ်းစားပါ။
5. Sign-off checklist အတွက် အနည်းဆုံး လေးချက် ဖော်ပြပါ။

---

## Exercise 4 — Failure handling နှင့် operational support model

Training platform အတွက် support model တစ်ခု ဒီဇိုင်းဆွဲရမည်။ အဖွဲ့တွင်
CloudOps operator ၃ ဦး နှင့် platform engineer ၂ ဦး ရှိပြီး hardware
ပြဿနာများကို Nebius support ကိုသာ ticket ဖွင့် ဖြေရှင်းရမည်။

1. Runbook tiers သုံးဆင့် (L1/L2/L3) ကို သတ်မှတ်ပြီး တစ်ဆင့်ချင်းစီ၏
   တာဝန်နှင့် လုပ်ပိုင်ခွင့် (ဥပမာ node ဖျက်ခွင့် ရှိ/မရှိ) ကို ရေးပါ။
2. Escalation matrix တစ်ခု ရေးဆွဲပါ — severity အဆင့် အနည်းဆုံး နှစ်ဆင့်ခွဲပြီး
   မည်သူက မည်သည့်အချိန်အတွင်း တုံ့ပြန်မည်ကို ဖော်ပြပါ (အချိန် ဂဏန်းများကို
   ကိုယ်ပိုင် SLA placeholder အဖြစ် သတ်မှတ်နိုင်ပြီး တရားဝင် ဂဏန်း မဟုတ်ကြောင်း
   မှတ်သားပါ)။
3. On-call design ကို ရေးပါ — rotation, handoff, escalation timeout။
4. Hardware fault တစ်ခု အတည်ပြုပြီးနောက် RMA flow ကို အဆင့်ဆင့်
   ဖော်ပြပါ။ ဤအခြေအနေတွင် architect က ချမှတ်ရမည့် design-level
   ဆုံးဖြတ်ချက် ဘာဖြစ်မည်နည်း?
5. Incident တစ်ခု ပြီးဆုံးပြီးနောက် ဘာလုပ်ရမည်နည်း?

---

## Exercise 5 — Migration နှင့် handover path

Project team က ၃ လ ကြာ တည်ဆောက်ခဲ့သော training platform ကို လုပ်ငန်းခွင်
ဝင်တော့မည်။ platform ကို ops team (CloudOps operator ၃ ဦး) က
လွှဲယူ လည်ပတ်စေမည်။ သင်က handover plan ရေးဆွဲရမည် —

1. Handover deliverables အနည်းဆုံး ငါးခု စာရင်းပြီး တစ်ခုချင်းစီတွင်
   ဘာပါဝင်ရမည်ကို ရေးပါ။
2. Handover လမ်းကြောင်းကို အဆင့်ဆင့် ဖော်ပြပါ (documentation, shadowing,
   hypercare, sign-off အပါအဝင်)။
3. RACI အလိုက် architect နှင့် operator တို့၏ တာဝန် ကွာခြားမှုကို
   ရှင်းလင်းပါ။
4. Handover မအောင်မြင်လေ့ ရှိသော အချက် သုံးချက်ကို ဖော်ပြပြီး
   ကာကွယ်နည်း ပေးပါ။

---

## လုပ်ဆောင်ရန် အကြံပြုချက်

- တစ်ခုချင်းစီအတွက် အဖြေကို အမှတ်အသား ပြုလုပ်ထားသော စာရွက်ဖြင့်
  ၁၀–၁၅ မိနစ်အတွင်း အကြမ်းဖျဉ်း ရေးကြည့်ပါ
- ပြီးမှ `solution.md` နှင့် နှိုင်းယှဉ်ပြီး design reasoning ကွာခြားမှုကို
  ပြန်စစ်ပါ — exam တွင် အဖြေ မှန်/မှား ထက် design decision ရဲ့
  အကြောင်းပြချက် ပြည့်စုံမှုက ပိုအရေးကြီးပါတယ်
