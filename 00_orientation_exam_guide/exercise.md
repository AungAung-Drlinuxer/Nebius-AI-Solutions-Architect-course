# Module 00 — Exercise: Orientation & Exam Guide

ဒီလေ့ကျင့်ခန်းတွေက Module 00 မှာ လေ့လာခဲ့တဲ့ role ခွဲခြားမှု၊ domain map၊ design-case study method တွေကို လက်တွေ့ အသုံးချနိုင်/မနိုင် စစ်ဆေးဖို့ ဖြစ်ပါတယ်။ အဖြေများကို `solution.md` မှာ ကြည့်နိုင်ပါတယ်။

## Exercise 1 — Role ခွဲခြားခြင်း

အောက်ပါ လုပ်ဆောင်ချက် ၅ ခုကို ကြည့်ပြီး တစ်ခုချင်းစီအတွက် တာဝန်ရှိသော role ကို ရွေးပါ — **AI Solutions Architect**, **AI CloudOps Engineer**, သို့မဟုတ် **AI Infrastructure Engineer**။

1. လစဉ် GPU capacity ကုန်ကျစရိတ်ကို လျှော့ချရန် ဘယ် workload ကို preemptible ပြောင်းမလဲ ဆုံးဖြတ်ပြီး cost estimate အသစ် ထုတ်ပေးခြင်း။
2. Training job တစ်ခု fail သွားသဖြင့် runbook အတိုင်း node ကို drain လုပ်ပြီး job ကို restart လုပ်ခြင်း။
3. Architect ပေးထားသော design အတိုင်း GPU cluster + shared filesystem + object storage ကို Terraform ဖြင့် တည်ဆောက်ပေးခြင်း။
4. Customer ၏ LLM fine-tuning workload အတွက် GPU type, storage tier, orchestrator ကို ရွေးချယ်ပြီး architecture proposal ရေးခြင်း။
5. စာမေးပွဲအတွက် မဟုတ်ဘဲ နေ့စဉ် platform လည်ပတ်မှုအတွက် GPU node များ error log ကို စောင့်ကြည့်ပြီး alert များ တုံ့ပြန်ခြင်း။

## Exercise 2 — Domain map သို့ ချိတ်ဆက်ခြင်း

အောက်ပါ design ဆုံးဖြတ်ချက် ၄ ခုကို exam domain ၄ ခု (1. high-performing, 2. resilient, 3. cost-aware, 4. secure) ထဲက အသင့်တော်ဆုံးဟာနှင့် တွဲပါ။ တစ်ခုချင်းစီအတွက် ဘာကြောင့် အဲဒီ domain ဖြစ်ရလဲ စာကြောင်းတိုတစ်ကြောင်းနှင့် ရှင်းပြပါ။

1. ရက်သတ္တပတ်များစွာ ကြာမည့် distributed training အတွက် 8-GPU SXM nodes များကို InfiniBand GPU cluster အဖြစ် ချိတ်ဆက်ရွေးခြင်း။
2. Node ပျက်ပါက နောက်ဆုံး checkpoint မှ ပြန်စရန် checkpoint interval နှင့် restart strategy ကို design ထဲ ထည့်ခြင်း။
3. Training ပြီးသွားသော model weights များကို storage class lifecycle rule ဖြင့် ဈေးသက်သာသော class သို့ အလိုအလျောက် ပြောင်းပေးခြင်း။
4. ML team များအတွက် service account နှင့် least-privilege access model ကို ချမှတ်ခြင်း။

## Exercise 3 — Ops မေးခွန်းကို design မေးခွန်းအဖြစ် ပြောင်းခြင်း

အောက်ပါ ops မေးခွန်းကို ဖတ်ပါ —

> "Slurm cluster ပေါ်က job တစ်ခု pending ဖြစ်နေရင် ဘယ် command နဲ့ အခြေအနေ ကြည့်မလဲ?"

1. ဒီမေးခွန်းက ဘာကြောင့် *ops* မေးခွန်းဖြစ်ရတာလဲ ရှင်းပြပါ။
2. အလားတူ အကြောင်းအရာ (job scheduling) ကို အခြေခံပြီး *architect-level* design မေးခွန်းတစ်ခု ပြန်ရေးပါ — scenario (workload, team, budget/timeline) ပါရမည်၊ မေးခွန်းက "ဘယ်ဟာ ရွေးမလဲ" ဖြစ်ရမည်။
3. သင်ရေးလိုက်တဲ့ design မေးခွန်းအတွက် scenario → requirements → design choice အဆင့် ၃ ဆင့်နှင့် အဖြေကြမ်း တစ်ခု ရေးပါ။

## Exercise 4 — Study roadmap တည်ဆောက်ခြင်း

အောက်ပါ profile ရှိ လေ့လာသူတစ်ဦးအတွက် Module 00 မှာ ပြထားတဲ့ study path ကို အခြေခံပြီး ၆ ပတ်စာ လေ့လာရေး plan တစ်ခု ရေးပါ —

- Backend/cloud engineer အဖြစ် အတွေ့အကြုံ ၄ နှစ် (Kubernetes နှင့် Terraform ကျွမ်း)။
- GPU training အတွေ့အကြုံ နည်းပါး (demo လောက်သာ လုပ်ဖူး)။
- ရက်စဉ် လေ့လာချိန် ၁ နာရီ ရနိုင်သည်။

Plan တွင် အောက်ပါတို့ ပါဝင်ရမည် — ဘယ် module ကို ဘယ်အပတ်မှာ လုပ်မလဲ၊ အားနည်းတဲ့ နယ်ပယ် (GPU/distributed training) အတွက် အပိုအချိန် ဘယ်လို ထည့်မလဲ၊ practice exams ကို ဘယ်အချိန်မှာ ဖြေမလဲ။

## Exercise 5 — အချက်အလက် မှန်ကန်မှု စစ်ဆေးခြင်း

အောက်ပါ အဆိုပြုချက်များကို ဖတ်ပြီး တစ်ခုချင်းစီ **မှန်/မှား/needs verification** ခွဲခြားပါ။ မှားပါက ဘာကြောင့်လဲ ရှင်းပြပါ။

1. "AI Solutions Architect စာမေးပွဲကို ဒီနေ့ စာရင်းပေးပြီး ဖြေလို့ ရပြီ။"
2. "စာမေးပွဲမှာ မေးခွန်း ၂၀ ပါပြီး ၂ နာရီ ပေးထားတယ်။"
3. "Architect စာမေးပွဲမှာ domain ၄ ခု ရှိပြီး တစ်ခုချင်းစီ ခန့်မှန်းခြေ ၂၅% ပတ်ဝန်းကျင် ပါဝင်တယ်။"
4. "Nebius Certified AI CloudOps Engineer က architecture decisions တွေကို ချပေးတဲ့ role ဖြစ်တယ်။"
