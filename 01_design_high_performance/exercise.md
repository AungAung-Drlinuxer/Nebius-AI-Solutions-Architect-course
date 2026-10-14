# Module 01 — Exercise: Design High-Performing Architectures

အောက်ပါ design scenarios များကို ဖြေရာတွင် Module 00 မှ **scenario → requirements → design choice** နည်းလမ်းကို အသုံးပြုပါ။ တစ်ခုချင်းစီအတွက် —

1. Scenario ထဲက constraints များကို စာရင်းပြုပါ။
2. Architecture တစ်ခု (GPU/platform, storage, deployment shape, orchestrator) ကို ရွေးပါ။
3. ရွေးချယ်မှုကို အကြောင်းပြပြီး အခြား ရွေးချယ်စရာများ ဘာကြောင့် မကိုက်တာလဲ ရှင်းပြပါ။

အဖြေများကို `solution.md` မှာ ကြည့်နိုင်ပါတယ်။

## Exercise 1 — LLM fine-tuning ငယ် တစ်ခု

Startup တစ်ခုက ကိုယ်ပိုင် customer support data နှင့် chatbot LLM တစ်ခုကို fine-tune ချင်ပါတယ် —

- Pre-trained model မှာ parameters ဘီလျံ ၇ ခန့် ရှိပြီး dataset က GB ရာဂဏန်း အတွင်းသာ ရှိပါတယ်။
- Fine-tune ချိန် ရက်ပိုင်း ကြာရုံ မျှော်လင့်ပါတယ်၊ GPU ၁ လုံး သို့မဟုတ် ၂ လုံး အထိသာ သုံးချင်ပါတယ် (ကုန်ကျစရိတ် သက်သာစေရန်)။
- ML engineer ၂ ဦးသာ ရှိပြီး cluster စီမံချင်ခြင်း မရှိပါ။
- Checkpoint များကို ပုံမှန် သိမ်းပြီး ပြီးသွားသော model ကို အခြား service များဆီ ဖြန့်ချိချင်ပါတယ်။

**မေးခွန်းများ —**
1. Deployment shape နှင့် GPU/platform ကို ဘယ်လို ရွေးမလဲ (InfiniBand လို/မလို အပါအဝင်)?
2. Dataset, checkpoints, final model အတွက် storage ဘယ်လို ခွဲမလဲ?
3. Job တွေကို ဘယ်လို run မလဲ — raw VM, Slurm/Soperator, Kubernetes, သို့မဟုတ် Serverless AI?

## Exercise 2 — Multi-node pre-training

Research lab တစ်ခုက parameters ဘီလျံ ရာဂဏန်း ရှိသော model အသစ် တစ်ခုကို pre-training လုပ်ချင်ပါတယ် —

- Training က ရက်သတ္တပတ် များစွာ ကြာမည်၊ GPU ၃၂ လုံး (node ၄ လုံး x 8-GPU) အထိ သုံးမည်။
- Dataset က ရာဂဏန်း TB ရှိပြီး ပုံမှန် update ဖြစ်ပါမယ်။
- Checkpoint ကို နာရီတိုင်း သိမ်းချင်ပြီး node တစ်ခု ကျရင် နောက်ဆုံး checkpoint မှ ပြန်စချင်ပါတယ်။
- Researchers ၁၀ ဦးခန့်က job queue ကို ကိုယ်တိုင် စီမံချင်ပါတယ်။

**မေးခွန်းများ —**
1. GPU platform အမျိုးအစားနှင့် cluster setting ကို ဘယ်လို ရွေးမလဲ? PCIe GPU များနှင့် ရ/မရ ရှင်းပြပါ။
2. Training data နှင့် hourly checkpoints အတွက် storage architecture ဘယ်လို ချမလဲ?
3. Distributed training မှာ GPU ၃၂ လုံး အတွက် parallelism ဘယ်လို စဉ်းစားမလဲ (data parallel အနေနှင့် စပြီး ဘယ်အခါ model/pipeline parallel လိုမလဲ)?
4. Job scheduling အတွက် overlay ဘာကို ရွေးမလဲ?

## Exercise 3 — Inference serving အတွက် architecture

Fintech company တစ်ခုက fraud-detection model ကို production မှာ serve ချင်ပါတယ် —

- Traffic က နေ့ဘက် များပြီး ညဘက် နည်းပါတယ် (autoscaling လိုချင်ပါတယ်)။
- Model weights က GB ရာဂဏန်း ရှိပြီး အသစ် retrain လုပ်တိုင်း weights အသစ်ကို node အားလုံးဆီ ဖြန့်ချိရပါမယ်။
- Latency sensitive ဖြစ်ပြီး internal network ကနေသာ ဝင်ရောက် အသုံးပြုပါမယ်။
- Platform team က Kubernetes ကို ကျွမ်းကျင်ပြီးသား ဖြစ်ပါတယ်။

**မေးခွန်းများ —**
1. Deployment shape နှင့် GPU အမျိုးအစား (inference workload အတွက် SXM/InfiniBand လို/မလို) ကို ဘယ်လို ရွေးမလဲ?
2. Model weights ကို node အများကြား ဘယ်လို မျှဝေမလဲ?
3. Autoscaling အတွက် orchestrator ဘာကို ရွေးမလဲ — ဘာကြောင့်လဲ?

## Exercise 4 — Petabyte-scale data pipeline + training

Genomics company တစ်ခုက dataset ကြီး တစ်ခု (PB အဆင့်) ကို preprocess လုပ်ပြီး model train ချင်ပါတယ် —

- Raw data က object storage ထဲ ရှိပြီးသား ဖြစ်ပါတယ်။
- Preprocess က CPU-bound ဖြစ်ပြီး training က GPU-bound ဖြစ်ပါတယ်။
- Training node တွေက intermediate data ကို အလွန်မြန်စွာ ဖတ်ချင်ပါတယ် (GPU တွေ မဆာစေရန်)။
- ကုန်ကျစရိတ် ထိန်းချုပ်မှု အရေးကြီးပြီး intermediate data ကို ပြန်ဖန်တီးနိုင်ပါတယ်။

**မေးခွန်းများ —**
1. Data pipeline တစ်လျှောက်လုံးအတွက် storage tiers ကို ဘယ်လို ခွဲမလဲ (raw data → intermediate → training input → artifacts)?
2. Training node တွေရဲ့ "GPU မဆာစေရန်" ဆိုတဲ့ requirement ကို storage ဘက်ကနေ ဘယ်လို ဖြည့်ဆည်းမလဲ?
3. ပြန်ဖန်တီးနိုင်တဲ့ intermediate data အတွက် ဘယ် storage option ကို သုံးမလဲ — ဘာကြောင့်လဲ?

## Exercise 5 — ရွေးချယ်စရာ ချိန်ဆခြင်း (tradeoff analysis)

Team တစ်ခုက LLM တစ်ခုကို fine-tune ဖို့ plan ရေးနေပြီး အောက်ပါ ၃ ခုထဲက ရွေးနေပါတယ် —

- (A) `gpu-l40s-d` platform ပေါ်က 1-GPU VM (PCIe GPU, InfiniBand မပါ)
- (B) `gpu-h200-sxm` platform ပေါ်က 1-GPU VM (SXM, InfiniBand ရနိုင်သော်လည်း တစ်လုံးတည်း)
- (C) `gpu-h200-sxm` platform ပေါ်က 8-GPU VM (SXM, NVLink)

Model က parameters ဘီလျံ ၃၀ ခန့် ရှိပြီး full fine-tuning (optimizer state အပါ) လုပ်မည်။ ကုန်ကျစရိတ် သက်သာမှုက အရေးကြီးပြီး training က တစ်ပတ်အတွင်း ပြီးရမည်။

**မေးခွန်းများ —**
1. ပထမဆုံး စစ်ဆေးရမည့် hard constraint က ဘာလဲ (hint — memory fit)?
2. ၃ ခုထဲက ဘယ်ဟာက ဘယ်အခြေအနေမှာ ကိုက်သလဲ တစ်ခုချင်း ခွဲခြား ရှင်းပြပါ။
3. ရွေးချယ်မှု တစ်ခုချင်းစီအတွက် "ဘာကြောင့် မကိုက်တာလဲ" ကို သေချာ ရေးပြပါ (distractor များကို ဖယ်ရှားနိုင်မှုက စာမေးပွဲအတွက် အရေးကြီးပါတယ်)။
