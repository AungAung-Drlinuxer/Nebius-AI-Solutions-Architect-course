# Module 03 — Exercise: Design Efficient & Cost-Aware Architectures

ဤတွင် design scenario ၅ ခု ပါဝင်ပါတယ်။ တစ်ခုချင်းစီအတွက် architecture နှင့် ကုန်ကျစရိတ်
စဉ်းစားပုံကို တုံ့ပြန်ချက် ရေးပါ — ကိန်းဂဏန်း အတိအကျ ဈေးနှုန်းများ မလိုအပ်ပါ။
ဆုံးဖြတ်ချက်တိုင်းကို "ဘာကြောင့်" နှင့် "ဘယ်လို ကန့်သတ်ချက် ရှိသလဲ" ဖြင့်
ဖြေရှင်းချက် ပူးတွဲပါ။

---

## Exercise 1 — Budget နှင့် timeline ကနေ capacity plan ရေးဆွဲခြင်း

**အခြေအနေ** — သင့် team သည် LLM fine-tuning project တစ်ခု လက်ခံရရှိပါတယ်။

- requirement: fine-tuned model ကို **၃ ပတ်** အတွင်း ပို့အပ်ရမည် (deadline တင်းကျ)
- team: ML engineer ၂ ဦး၊ infra engineer ၁ ဦး
- budget: compute အတွက် လစဉ် သတ်မှတ် budget တစ်ခု ရှိပြီး ပထမလ တွင် အများဆုံး သုံးစွဲခွင့် ရှိသည်
  (ပမာဏ အတိအကျ ကို သင့်ကိုယ်ပိုင် အခြေအနေဖြင့် စဉ်းစားနိုင်ပါတယ်)
- dataset ကို အခြား cloud တစ်ခုတွင် သိမ်းဆည်းထားပြီး ပမာဏ ကြီးမားပါတယ်
- team တွင် Nebius AI Cloud အသုံးပြုမှု အတွေ့အကြုံ အနည်းငယ်သာ ရှိပါတယ်

**တောင်းဆိုချက်များ**

1. ပြီးဆုံးရမည့် deadline ကနေ ပြန်လည် တွက်ချက်ပြီး GPU demand ခန့်မှန်းချက် တစ်ခု ရေးပါ
   (ခန့်မှန်းချက် ဖြစ်ကြောင်း ရှင်းလင်းပါ)။ မည်သည့် အချက်အလက် များကို ကြိုတင်
   စုဆောင်းရန် လိုအပ်သလဲ စာရင်းပြုစုပါ။
2. quota ကို design အဆင့်မှာ မည်သို့ ထည့်သွင်းစဉ်းစားမလဲ — မည်သည့်အခါ quota request
   တင်ရမလဲ၊ မတင်ဘဲ စတင်ပါက မည်သည့် risk ရှိသလဲ?
3. onboarding cost အမျိုးအစား အားလုံးကို စာရင်းပြုစုပြီး မည်သည့် အမျိုးအစားက
   ဘတ်ဂျက်ကို အလွယ်တကူ ကျော်လွန်စေနိုင်သလဲ ရှင်းပြပါ။
4. အနည်းဆုံး အဆင့် ၃ ဆင့်ပါသော timeline တစ်ခု ရေးဆွဲပါ (data ingestion → training →
   validation/handover)။

---

## Exercise 2 — Multi-team FinOps governance ဒီဇိုင်း

**အခြေအနေ** — ကုမ္ပဏီတစ်ခုတွင် team ၃ ခု (research, product, platform) က
တစ်ခုတည်းသော Nebius AI Cloud tenant အောက်တွင် project များဖြင့် အလုပ်လုပ်ပါတယ်။
ယခုအချိန်အထိ လစဉ် ငွေကုန်ကြေးကြီးမှု ဖြစ်ပွားပြီး မည်သည့် team က ကျော်လွန်သလဲ
မသိရှိပါ။ research team ၏ experiment များကြောင့် လစဉ် ကုန်ကျစရိတ် ရုတ်တရက်
တက်လေ့ရှိပါတယ်။

**တောင်းဆိုချက်များ**

1. budget များကို မည်သို့ ချမှတ်မလဲ — ဘယ်နှစ်ခု၊ scope ဘယ်လို (All usage / Project /
   Product condition)၊ period ဘယ်လို၊ alert threshold များ ဘယ်လို ထားမလဲ?
   alert recipient ကိုလည်း စဉ်းစားပါ။
2. budget သည် usage ကို မရပ်တန့်ကြောင်း သိပါက ကုန်ကျစရိတ် အမှန်တကယ် မကျော်လွန်စေရန်
   မည်သည့် အပို guardrail များ ထားရှိမလဲ (quota, policy, operational process)?
3. cost monitoring routine တစ်ခု ရေးဆွဲပါ — ဘယ် tool များ (Usage page, CSV export,
   cost breakdown PDF, alert) ကို ဘယ်အချိန်မှာ သုံးမလဲ?
4. project တစ်ခု ပြီးဆုံးသွားပါက decommissioning ကို မည်သို့ စီမံမလဲ — ဘယ် resource ကို
   ဖျက်မလဲ၊ ဘာကို သိမ်းဆည်းမလဲ၊ ဘယ်သူ တာဝန်ယူမလဲ?

---

## Exercise 3 — Cost vs performance tradeoff: GPU အရေအတွကး ရွေးချယ်ခြင်း

**အခြေအနေ** — image-generation model training run တစ်ခုအတွက် အောက်ပါ ရွေးချယ်စရာ
၃ ခု ရှိပါတယ် (ယေဘုယျ အခြေအနေ ဖြစ်ပြီး စျေးနှုန်း ကိန်းဂဏန်း မပါပါ) —

- **Option A** — GPU အနည်းငယ် (ဥပမာ ၄ လုံး) ဖြင့် ရက်ပေါင်းများစွာ run ခြင်း
- **Option B** — GPU အများ (ဥပမာ ၁၆ လုံး) ဖြင့် ရက်အနည်းငယ် run ခြင်း (scaling efficiency
  ခန့်မှန်းချက် ပေးထားရန် လိုအပ်)
- **Option C** — GPU အများဖြင့် spot/preemptible capacity သုံးပြီး checkpoint/resume
  mechanism ထားရှိ run ခြင်း

deadline က ၂ ပတ် အတွင်း ဖြစ်ပြီး run တစ်ခုချင်းစီတွင် အလယ်အလတ် checkpoint
များ ရှိပါတယ်။ model quality ကို precision လျှော့ချခြင်းဖြင့် မြှင့်တင်နိုင်/မနိုင်လည်း
စဉ်းစားပါ။

**တောင်းဆိုချက်များ**

1. စုစုပေါင်း ကုန်ကျကို GPU-hours ဖြင့် မည်သို့ နှိုင်းယှဉ်မလဲ — တစ်ခုချင်းစီအတွက်
   ပုံသေနည်းဖြင့် ချပြပါ။ GPU ပိုသုံးခြင်းက စုစုပေါင်း ကုန်ကျ ကျစေနိုင်/မကျစေနိုင်သည့်
   အခြေအနေကို ရှင်းပြပါ။
2. Option C ကို ရွေးချယ်ပါက မည်သည့် design လိုအပ်ချက်များ (checkpoint frequency,
   resume, job retry) လိုအပ်သလဲ — interruption ဖြစ်ပါက ကုန်ကျစရိတ် မည်သို့
   သက်ရောက်သလဲ?
3. မည်သည့် option ကို ရွေးမလဲ — deadline လွဲခြင်း၏ ကုန်ကျကို ထည့်သွင်း ရှင်းပြပါ။
4. precision vs throughput tradeoff ကို ဤ run တွင် မည်သို့ ကိုင်တွယ်မလဲ?

---

## Exercise 4 — Multi-cloud data-ingestion pipeline ဒီဇိုင်း

**အခြေအနေ** — သင့် team သည် အခြား cloud provider တစ်ခုတွင် သိမ်းဆည်းထားသော
dataset ကြီး (ရုပ်ပုံ + annotation) ကို Nebius AI Cloud တွင် training ပြုလုပ်ရန်
ပြောင်းရွှေ့ရပါမည်။

- dataset က ပမာဏ ကြီးမားပြီး တစ်စိတ်တစ်ပိုင်းကို လစဉ် update လုပ်ပါတယ်
- company network က အခြား production workload များလည်း သုံးနေပါတယ်
- training မစမီ dataset ပြည့်စုံ/မပျက်ကို အာမခံချက် လိုအပ်ပါတယ်
- project ပြီးဆုံးပြီး ၆ လ အကြာတွင် dataset ကို ပြန်လည် အသုံးပြုရန် အလားအလာ ရှိပါတယ်

**တောင်းဆိုချက်များ**

1. data-ingestion pipeline architecture တစ်ခု ရေးဆွဲပါ (ingest → staging → validation →
   training → artifact)။ staging bucket ထားရှိရသည့် အကြောင်းရင်း ရှင်းပြပါ။
2. ingest bandwidth နှင့် transfer window ကို မည်သို့ စဉ်းစားမလဲ — ခန့်မှန်းချက်
   ပုံသေနည်းဖြင့် ချပြပြီး မည်သည့် အချက်အလက် လိုအပ်သလဲ ဖော်ပြပါ။
3. ရေရှည် ကုန်ကျစရိတ် လျှော့ချရန် storage class နှင့် lifecycle ကို မည်သို့ အသုံးပြုမလဲ —
   ၆ လ အကြာ ပြန်လည် အသုံးပြုမှုကိုပါ ထည့်သွင်းစဉ်းစားပါ။
4. multi-cloud connectivity ကို network design ရှုထောင့်မှ မည်သိုး စဉ်းစားမလဲ
   (Virtual Networks, external connectivity, IP addressing)?

---

## Exercise 5 — Idle resource နှင့် end-of-project teardown policy

**အခြေအနေ** — သင့် team သည် တစ်နှစ်အတွင်း project ၅ ခု လက်ခံ ဆောင်ရွက်ပြီးစီးပါမည်။
ယခင် project များတွင် —

- training ပြီးသွားသော်လည်း GPU node များ ရက်ပေါင်းများစွာ ဆက်လက် run နေခဲ့ဖူးပါတယ်
- VM များ stop ထားသော်လည်း volume များ ဖျက်ရန် မေ့နေခဲ့ပါတယ်
- snapshot နှင့် checkpoint ဟောင်းများ bucket များတွင် စုပုံနေခဲ့ပါတယ်
- project ပြီးဆုံးပြီးနောက်လည်း အချို့ resource များ ဆက်လက် ကုန်ကျနေခဲ့ပါတယ်

**တောင်းဆိုချက်များ**

1. idle resource policy တစ်ခု ရေးဆွဲပါ — မည်သည့် resource ကို မည်သည့် အခြေအနေတွင်
   ပြန်လည် သုံးသပ်/ဖျက်သိမ်းမလဲ (VM, volume, snapshot, bucket, cluster)?
2. end-of-project teardown checklist တစ်ခု ရေးပါ — ဘာကို ဖျက်မလဲ၊ ဘာကို သိမ်းဆည်းမလဲ
   (model, checkpoint, dataset)၊ ဘယ်လို အတည်ပြုမလဲ (ကုန်ကျမှု ရပ်ပြီ/မရပ်)?
3. ဤ policy ကို လက်တွေ့ အကောင်အထည်ဖော်ရာတွင် Terraform က မည်သို့ ကူညီနိုင်သလဲ —
   မည်သည့် ကန့်သတ်ချက် ရှိသလဲ?
4. အပေါ်ပါ လက်ရှိ ပြဿနာများအားလုံးအတွက် ကြိုတင် ကာကွယ်ရန် monitoring/alerting
   အဆင့် အနည်းဆုံး ၂ ခု အကြံပြုပါ။

---

> ဖြေရှင်းချက် နမူနာများကို `solution.md` တွင် ကြည့်ရှုနိုင်ပါတယ်။ ဖြေရှင်းချက်များသည်
> တစ်ခုတည်းသော အဖြေ မဟုတ်ဘဲ ဆုံးဖြတ်ချက် စဉ်းစားပုံ နမူနာများ ဖြစ်ပါတယ်။
