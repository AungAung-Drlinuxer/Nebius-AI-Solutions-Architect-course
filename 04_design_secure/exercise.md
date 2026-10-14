# Module 04 — Exercise: Design Secure Architectures

> အောက်ပါ design scenario ၅ ခုသည် exam ၏ Domain 4 ပုံစံအတိုင်း ရေးထားခြင်း ဖြစ်သည်။
> တစ်ခုချင်းစီအတွက် ဒီဇိုင်းဆုံးဖြတ်ချက်၊ အကြောင်းပြချက် (why) နှင့်
> မရွေးချယ်မိသော ရွေးချယ်စရာများကို ဘာကြောင့် ငြင်းသလဲ ဆိုတာ ပါဝင်ပါစေ။
> တိကျသော Nebius command/flag များ မသိပါက ရှင်းလင်းချက်ကို concept level တွင်
> ရေးပြီး "needs verification" ဟု မှတ်သားပါ — မဖန်တီးပါနှင့်။

---

## Exercise 1 — Identity and access model for a growing AI startup

**Scenario:** AI startup တစ်ခုတွင် ဝန်ထမ်း ၂၅ ယောက် ရှိသည် — ML engineer ၁၂ ယောက်၊
data engineer ၅ ယောက်၊ backend engineer ၄ ယောက်၊ platform/SRE ၂ ယောက်၊
CTO နှင့် security/compliance officer တစ်ယောက်စီ။ Nebius AI Cloud ပေါ်တွင်
training platform တစ်ခု တည်ဆောက်နေပြီး CI/CD pipeline များက Terraform ဖြင့်
infrastructure ကို provision လုပ်မည်။ လက်ရှိတွင် ဝန်ထမ်းအားလုံးကို default `editors`
group ထဲ ထည့်ထားသည်။

**တောင်းဆိုချက်များ:**
1. Group နှင့် role ဖွဲ့စည်းပုံကို ဒီဇိုင်းရေးဆွဲပါ (group အမည်များ၊ ပေးမည့် role/permission
   အကျယ်အဝန်း၊ ဘာကြောင့် ထိုအတိုင်း ခွဲသလဲ ပါဝင်ပါစေ)။
2. Terraform CI pipeline အတွက် identity ကို ဘယ်လို ဖြေရှင်းမလဲ (authentication method
   ရွေးချယ်မှု အပါအဝင်)။
3. ကုမ္ပဏီတွင် Microsoft Entra ID ကို corporate SSO အဖြစ် အသုံးပြုနေသည် —
   cloud access နှင့် ဘယ်လို ချိတ်ဆက်မလဲ။
4. လက်ရှိ "အားလုံး editor" အခြေအနေမှ ကူးပြောင်းမှု plan တိုတောင်းစွာ ရေးပါ။

---

## Exercise 2 — Private connectivity for a regulated health-tech platform

**Scenario:** Health-tech company တစ်ခုက patient data ပါဝင်သော ML training platform ကို
Nebius AI Cloud ပေါ်တွင် တည်ဆောက်မည်။ Compliance requirement များ —
(a) training subnet အတွင်းရှိ VM များတွင် public IP လုံးဝ မရှိရ၊
(b) operator များသည် ရုံး internal network မှသာ ဝင်ခွင့်ရှိရမည်၊
(c) security team က workload ကို မထိခိုက်ဘဲ network traffic ကို စောင့်ကြည့်နိုင်ရမည်။
ကုမ္ပဏီတွင် on-prem data center တစ်ခု ရှိပြီး ရုံးနှင့် အမြဲတမ်း ချိတ်ဆက်ထားလိုသည်။

**တောင်းဆိုချက်များ:**
1. Network layout ကို ဒီဇိုင်းရေးဆွဲပါ (subnet ခွဲခြားမှု၊ public/private boundary)။
2. Operator access အတွက် jump server (WireGuard)၊ Nebius VPN Gateway နှင့်
   Nebius Tunnels တို့အနက် မည်သည့်အရာကို မည်သည့်နေရာတွင် သုံးမလဲ —
   ရွေးချယ်မှုအတွက် အကြောင်းပြချက်ပေးပါ။
3. Security group design ကို rule အဆင့်အထိ ရေးဆွဲပါ (default policy၊
   management traffic ကန့်သတ်ချက်)။
4. ဤဒီဇိုင်းတွင် single point of failure ဖြစ်နိုင်သော အချက်များကို ဖော်ပြပြီး
   ဖြေရှင်းနည်း အကြံပြုပါ။

---

## Exercise 3 — Secrets, audit နှင့် compliance posture for a fintech workload

**Scenario:** Fintech company တစ်ခုက Nebius ပေါ်တွင် fraud-detection model ကို run နေသည်။
Requirement များ —
(a) application က database password ကို သုံးနေပြီး password ကို ၉၀ ရက်တစ်ခါ
ပြောင်းရမည် (config redeploy မပါဘဲ)၊
(b) regulator က "ဘယ် user က ဘယ်အချိန်တွင် ဘယ် resource ကို ပြောင်းခဲ့သလဲ" ကို
အနည်းဆုံး ၂ နှစ် ကြာ ပြန်ပြနိုင်ရမည်၊
(c) audit evidence ကို security team ကသာ export ထုတ်ခွင့် ရှိရမည်။

**တောင်းဆိုချက်များ:**
1. Secret management design ရေးဆွဲပါ (SecretStash အသုံးပြုပုံ၊ version/primary version
   စီမံခန့်ခွဲမှု၊ rotation procedure)။
2. Audit scope ကို သတ်မှတ်ပါ (control plane / data plane event ရွေးချယ်မှု အပါအဝင်)
   နှင့် ၂ နှစ် retention ကို ဘယ်လို အကောင်အထည်ဖော်မလဲ။
3. Role design ရေးဆွဲပါ — secret payload ကြည့်ခွင့်၊ secret update ခွင့်နှင့်
   audit export ခွင့်တို့ကို ဘယ်သူတွေကို ပေးမလဲ။
4. ဤဒီဇိုင်းတွင် separation of duties ကို ဘယ်လို ထိန်းသလဲ ရှင်းပြပါ။

---

## Exercise 4 — Data classification နှင့် encryption mapping

**Scenario:** Media company တစ်ခုက Nebius ပေါ်တွင် generative AI platform တည်ဆောက်နေသည်။
Dataset လေးမျိုး ရှိသည် —
(1) web မှ စုဆောင်းထားသော public training corpus (TB ဆယ်ဂဏန်းအဆင့်)၊
(2) licensed artwork များ (contract အရ confidential)၊
(3) customer များ upload လုပ်ထားသော personal photo များ (PII ပါဝင်)၊
(4) trained model weights များ (ကုမ္ပဏီ ၏ intellectual property)။
WEKA filesystem ကို training scratch space အဖြစ် အသုံးပြုမည်။

**တောင်းဆိုချက်များ:**
1. Dataset တစ်ခုချင်းစီအတွက် classification level သတ်မှတ်ပါ။
2. တစ်ခုချင်းစီအတွက် storage choice၊ encryption/key policy နှင့် network exposure
   ကို ချိတ်ဆက်ပြီး mapping table ရေးပါ။
3. WEKA filesystem အတွက် encryption decision ကို ဘယ်အချိန်တွင် ချရမလဲ —
   ဘာကြောင့်လဲ ရှင်းပြပါ။
4. KMS key design — ဘယ် dataset အတွက် default platform encryption နှင့်
   ဘယ် dataset အတွက် ကိုယ်ပိုင် KMS key + rotation policy လိုမလဲ ဆုံးဖြတ်ပါ။

---

## Exercise 5 — Incident-response posture ဒီဇိုင်း

**Scenario:** E-commerce company တစ်ခု၏ Nebius environment တွင် CI pipeline အတွက်
သုံးထားသော service account access key ကို public GitHub repository တစ်ခုတွင်
တွေ့ရှိခဲ့ကြောင်း security report ဝင်လာသည်။ ထို service account သည်
production bucket များကို access လုပ်ခွင့် ရှိသည်။ Company တွင် incident response plan
မရှိသေးပါ။

**တောင်းဆိုချက်များ:**
1. ပထမ ၂၄ နာရီအတွင်း လုပ်ဆောင်ရမည့် အဆင့်များကို detect→contain→eradicate→recover
   ပုံစံဖြင့် ရေးပါ။ တစ်ဆင့်ချင်းအတွက် Nebius ပေါ်တွင် အသုံးပြုမည့် capability
   (role၊ audit event type၊ secret/KMS operation) ကို ချိတ်ဆက်ပြပါ။
2. Evidence preservation အတွက် ဘာကြိုတင် ပြင်ဆင်ထားရမလဲ (audit export အပါအဝင်)။
3. ဤ incident မဖြစ်မီ ကြိုတင် ရှိထားသင့်သော design decision များကို စာရင်းပြုပါ
   (identity ၊ secret management ၊ network — ဘယ်ဒီဇိုင်းက incident ၏ impact ကို
   လျှော့ချပေးနိုင်သလဲ)။
4. Post-incident review အတွက် stakeholder များကို မည်သို့ အစီရင်ခံမလဲ
   (architect ၏ အခန်းကဏ္ဍ အပါအဝင်) အကြမ်းဖျဉ်း ရေးပါ။

---

> အဖြေများကို `solution.md` တွင် ကြည့်ပါ။ ဖြေဆိုရာတွင် "ဘာကြောင့်" ကို
> ဦးစားပေးပါ — exam သည် tool command ကို မမေးဘဲ design reasoning ကို မေးသည်။
