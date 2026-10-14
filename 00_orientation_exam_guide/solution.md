# Module 00 — Solution: Orientation & Exam Guide

## Exercise 1 — Role ခွဲခြားခြင်း (အဖြေ)

1. **AI Solutions Architect** — capacity နှင့် ကုန်ကျစရိတ် ဆိုင်ရာ ဆုံးဖြတ်ချက်ချပြီး cost estimate ထုတ်ပေးခြင်းသည် architect ၏ တာဝန်ဖြစ်သည် (workload, budget အကဲဖြတ်ခြင်း)။
2. **AI CloudOps Engineer** — runbook အတိုင်း incident တုံ့ပြန်ခြင်း (drain, restart) သည် day-to-day operations ဖြစ်ပြီး architecture ဆုံးဖြတ်ချက် မပါဝင်ပါ။
3. **AI Infrastructure Engineer** — architect ၏ design ကို implement လုပ်ခြင်းဖြစ်သည်။
4. **AI Solutions Architect** — customer requirements ကို အကဲဖြတ်ပြီး GPU type, storage tier, orchestrator ရွေးချယ်ကာ proposal ရေးခြင်းသည် architect အလုပ်၏ အနှစ်သာရဖြစ်သည်။
5. **AI CloudOps Engineer** — metrics/alerting စောင့်ကြည့်ခြင်းသည် operational support model ဖြစ်သည်။

**အဓိက သော့ချက် —** "ဆုံးဖြတ်ချက်ချပြီး တာဝန်ယူသူ" = architect၊ "runbook အတိုင်း လုပ်ဆောင်သူ" = CloudOps၊ "design အတိုင်း တည်ဆောက်သူ" = Infrastructure Engineer။

## Exercise 2 — Domain map သို့ ချိတ်ဆက်ခြင်း (အဖြေ)

1. **Domain 1 (high-performing)** — GPU architecture နှင့် interconnect (InfiniBand) ရွေးချယ်မှုသည် "Selecting GPU architecture and platform-level building blocks" နှင့် "Recommending distributed-training architecture" အောက် ကျသည်။
2. **Domain 2 (resilient)** — checkpoint/restart strategy သည် "Checkpoint and restart strategies" နှင့် "failure handling" အောက် ကျသည်။
3. **Domain 3 (cost-aware)** — storage class lifecycle ဖြင့် ကုန်ကျစရိတ် လျှော့ချခြင်းသည် "FinOps: cost monitoring, decommissioning" အောက် ကျသည်။
4. **Domain 4 (secure)** — service account နှင့် least privilege သည် "Identity and access model design" အောက် ကျသည်။

မှတ်ချက် — design ဆုံးဖြတ်ချက် အများစုက domain တစ်ခုထက် ပိုထိနိုင်ပေမယ့် စာမေးပွဲမှာ မေးခွန်းက ဘယ် domain ရဲ့ objective ကို ဦးတည်လဲဆိုတာ ကြည့်ပြီး အဓိက ချိတ်ဆက်သင့်ပါတယ်။

## Exercise 3 — Ops မေးခွန်းကို design မေးခွန်းအဖြစ် ပြောင်းခြင်း (အဖြေ)

1. ဒီမေးခွန်းက procedure/command ကို မေးနေလို့ ဖြစ်ပါတယ် — အဖြေက တစ်ခုတည်းသော မှန်ကန်တဲ့ operation ဖြစ်ပြီး requirements ချိန်ဆရတာ မပါဝင်ပါဘူး။ ဒါက runbook-driven (CloudOps) မေးခွန်း ပုံစံပါ။
2. **Design မေးခွန်း ဥပမာ (သင်ကြားရေး ဖန်တီးမှု) —**

   > "ML team တစ်ခုက ရက်သတ္တပတ် ၂ ပတ်ကြာမယ့် distributed training job တွေကို တစ်ပတ်လျှင် ၃ ကြိမ် တင်ပါမယ်။ ကြားကာလမှာ cluster က အခြား preprocess jobs တွေအတွက်လည်း အသုံးပြုချင်ပါတယ်။ ကုန်ကျစရိတ် သက်သာချင်ပြီး queue ကို team ကိုယ်တိုင် စီမံချင်ပါတယ်။ ဒီ workload အတွက် ဘယ် job scheduling approach က အကောင်းဆုံးလဲ — (A) လက်ပိုင်း VM တွေပေါ်မှာ job တွေ တစ်ခုချင်း run ခြင်း၊ (B) batch scheduler ပါတဲ့ Slurm cluster ဖြင့် queue စီမံခြင်း၊ (C) request/response service အဖြစ် run ခြင်း၊ (D) တစ်နေ့တစ်ခါ manual start ခြင်း။"

3. **အဖြေကြမ်း —**
   - *Scenario:* ရှည်လျားသော batch training jobs + shared cluster အသုံးပြုမှု + ကုန်ကျစရိတ် သက်သာမှု + team ကိုယ်တိုင် စီမံချင်မှု။
   - *Requirements:* multi-day job များအတွက် queue/scheduling၊ resource sharing၊ cost control။
   - *Design choice:* (B) Slurm-based batch scheduling — queue ဖြင့် multi-day jobs ကို စီမံနိုင်ခြင်း၊ resource မျှဝေသုံးနိုင်ခြင်း၊ team ကိုယ်တိုင် စီမံနိုင်ခြင်းကြောင့် ဖြစ်သည်။ (A) က scheduling မပါ၊ (C) က long-running batch နှင့် မကိုက်၊ (D) က လူအားဖြင့် လုပ်ရ၍ မတည်ငြိမ်။

## Exercise 4 — Study roadmap တည်ဆောက်ခြင်း (အဖြေ ဥပမာ)

| အပတ် | လုပ်ဆောင်မှု |
|---|---|
| ၁ | Module 00 + Exam Guide PDF ဖတ်ခြင်း၊ role/domain map ကျွမ်းကျင်အောင် လုပ်ခြင်း |
| ၂–၃ | Module 01 (high-performance) — GPU/distributed training အားနည်းသဖြင့် ရက်စဉ် ၁ နာရီထဲက ၄၅ မိနစ်ကို ဒီ module အတွက် အသုံးပြုပြီး docs.nebius.com မှ GPU/storage pages များ ဖတ်ခြင်း |
| ၄ | Module 02 (resilient) — Kubernetes အတွေ့အကြုံဖြင့် အနည်းငယ် မြန်နိုင်ခြင်း |
| ၅ | Module 03 (cost-aware) + Module 04 (secure) — Terraform/ IAM အတွေ့အကြုံဖြင့် မြန်နိုင်ခြင်း |
| ၆ | Module 05 practice exams ၂ စုံ အချိန်တိုင်းပြီး ဖြေခြင်း → အားနည်း domain ပြန်လည် လေ့လာခြင်း၊ waitlist အခြေအနေ ပြန်စစ်ခြင်း |

အားနည်းနယ်ပယ် (GPU/distributed training) အတွက် — Module 01 ကို ၂ ပတ် ပေးထားခြင်းဖြစ်ပြီး practice exam မှာ domain 1 ရမှတ် နည်းပါက week 6 မှာ ထပ်မံ ပြန်လည် လေ့လာရန် ချန်ထားပါသည်။

## Exercise 5 — အချက်အလက် မှန်ကန်မှု စစ်ဆေးခြင်း (အဖြေ)

1. **မှားသည်** — စာမေးပွဲက waitlist အဆင့်သာ ရှိသေးသဖြင့် ယခု ဖြေနိုင်ခြင်း မရှိပါ။ Waitlist စာရင်းသွင်းထားနိုင်ပြီး ဖွင့်လှစ်ချိန်ကို စောင့်ရပါမည်။
2. **မှားသည်** — တရားဝင် Exam Guide အရ ပုံစံက multiple choice (အဖြေမှန် ၁ ခု + distractors ၃ ခု) ဖြစ်ပြီး အချိန်က ၁ နာရီ ဖြစ်သည်။ "မေးခွန်း ၂၀ / ၂ နာရီ" ဆိုသည်မှာ မှားယွင်းသည် (မေးခွန်းအရေအတွက်ကို Exam Guide က သတ်မှတ်မထားပါ — အတည်ပြုရန် လိုအပ်ပါသည်)။
3. **မှန်သည်** — domain ၄ ခု ရှိပြီး တစ်ခုချင်းစီ ခန့်မှန်းခြေ ~25% ဖြစ်သည် (Exam Guide အတိုင်း)။
4. **မှားသည်** — AI CloudOps Engineer (NCO-201) က platform ကို operate လုပ်သူဖြစ်ပြီး architecture decisions များကို မချပါ။ Architecture decisions ချခြင်းက AI Solutions Architect ၏ တာဝန် ဖြစ်သည်။

## Technical review notes

- Exercise 5၊ အချက် ၂ တွင် ဖော်ပြထားသကဲ့သို့ **စာမေးပွဲ မေးခွန်းအရေအတွက်ကို တရားဝင် Exam Guide က သတ်မှတ်မထားပါ** — "20 MCQ" ကို course ရဲ့ practice exam ပုံစံအဖြစ်သာ အသုံးပြုထားခြင်း ဖြစ်သည် (needs verification against Exam Guide at exam launch)။
- Exercise 3၊ အချက် ၂ ၏ မေးခွန်းနှင့် အဖြေရွေးချယ်စရာများသည် သင်ကြားရေးအတွက် ဖန်တီးထားခြင်းဖြစ်ပြီး စာမေးပွဲ မေးခွန်း အစစ်များ မဟုတ်ပါ။
- Exercise 4 ၏ အပတ်စဉ် ခွဲဝေမှုသည် ဥပမာ plan ဖြစ်ပြီး လေ့လာသူတစ်ဦးချင်းစီ၏ အချိန်နှင့် အတွေ့အကြုံပေါ် မူတည်၍ ပြောင်းလဲနိုင်ပါသည်။
