# Module 04 — Solution: Design Secure Architectures

> ဤဖိုင်တွင် exercise.md မှ scenario ၅ ခုအတွက် worked answer များ ပါဝင်သည်။
> ဖြေဆိုပုံမှာ — ဆုံးဖြတ်ချက် → အကြောင်းပြချက် → ရွေးချယ်စရာ ငြင်းချက် →
> သတိပေးချက် ပုံစံဖြစ်သည်။ Exam တွင် ဤ reasoning ပုံစံကို လိုချင်သည်။

---

## Solution 1 — Identity and access model

### 1) Group နှင့် role ဖွဲ့စည်းပုံ

Default `editors` ကို အားလုံးအတွက် သုံးခြင်းကို ရပ်ပါ — general role များက service
အားလုံးတွင် ခွင့်ပြုချက် ကျယ်ပြန့်လွန်းသဖြင့် Nebius ကိုယ်တိုင် custom group များကို
အကြံပြုထားသည်။ Function အလိုက် အောက်ပါအတိုင်း ခွဲသင့်သည်။

| Group | Member | Permission scope | အကြောင်းပြချက် |
|---|---|---|---|
| `platform-admins` | platform/SRE ၂ ယောက်၊ CTO | `admin` (tenant-wide) | infra ပြောင်းလဲမှု အားလုံး — လူနည်းနည်းသာ |
| `ml-engineers` | ML engineer ၁၂ ယောက် | custom: GPU cluster/VM ဖန်တီးခွင့် + training data read | လေ့ကျင့်ရေး လုပ်ငန်းအတွက် လုံလောက် |
| `data-engineers` | data engineer ၅ ယောက် | custom: storage read/write + pipeline infra | data လုပ်ငန်း — user/KMS management မလို |
| `backend-engineers` | backend ၄ ယောက် | custom: inference service resource များသာ | production bucket အားလုံး မလို |
| `security-reviewers` | security officer | `security-auditor` + `auditlogs.audit-event-viewer` | view-only — change ခွင့် မပေးပါနှင့် |
| `ci-terraform` | service account | custom: သတ်မှတ်ထားသော project resource level | see (2) |

Role ကို user တစ်ဦးချင်း မဟုတ်ဘဲ group အပေါ်တွင်သာ ချပါ — access permit ဖြင့်
project/resource level အထိ ကျဉ်းအောင် သတ်မှတ်ပါ။ ဤနည်းက joiner/mover/leaver ကို
group membership တစ်ခုတည်းဖြင့် စီမံနိုင်စေသည်။

### 2) Terraform CI pipeline identity

- **Service account သီးခြား** ဖန်တီးပါ (`ci-terraform`) — လူ့ account နှင့် မရောပါနှင့်။
  Pipeline တစ်ခုချင်းစီအတွက် သီးခြား service account ဆိုပါက ပိုကောင်းသည်။
- Authentication method — headless CI environment အတွက် **authorized key pair** ကို
  Nebius CLI/Terraform provider authentication တွင် သုံးပါ။ Object Storage (AWS-compatible API)
  သုံးပါက **access key** လိုအပ်မည်။
- Authorized key ၏ private key ကို CI disk ပေါ်တွင် plain မထားပါနှင့် —
  **SecretStash** တွင် သိမ်းပြီး job run အခါ ရယူပါ။
- `ci-terraform` ကို provision လုပ်မည့် project resource များအထိသာ permission ပေးပြီး
  production data access မပေးပါနှင့် (least privilege)။

### 3) Entra ID SSO ချိတ်ဆက်ခြင်း

Nebius **federation** ကို SAML SSO ဖြင့် Microsoft Entra ID နှင့် ချိတ်ဆက်ပါ
(documentation တွင် Entra ID configuration guide ရှိသည်)။ User များကို
federation မှတစ်ဆင့် tenant အတွင်း ဝင်စေပြီး IdP ဘက် group mapping ဖြင့်
Nebius group membership ကို စီမံပါ — IdP မှ user ထွက်သွားသည်နှင့် access ပြတ်သွားစေရန်
ဖြစ်သည်။ Service account များကိုမူ federation တွင် မထည့်ပါနှင့် — ၎င်းတို့သည်
machine identity ဖြစ်သည်။

### 4) Migration plan

1. Custom group များ ဖန်တီးပြီး role/access permit များ ကြိုတင် ချထားပါ။
2. ဝန်ထမ်းတစ်ဦးချင်းစီကို function အလိုက် custom group ထဲ ပြောင်းထည့်ပါ။
3. CI ကို service account သို့ ပြောင်းပြီး key ကို SecretStash တွင် ထားပါ။
4. `editors`/`admins` default group ထဲမှ မလိုအပ်သော member များ ဖယ်ရှားပါ
   (break-glass admin အနည်းငယ်သာ ချန်ပါ)။
5. Access review ကို runbook တွင် ထည့်ပါ — access creep ကို ကာကွယ်ရန်။

**ငြင်းချက်** — "အားလုံး `editors`" ဆက်ထားခြင်းသည် audit တွင် accountability ပျောက်စေပြီး
တစ်ဦးပေါက်ကြားပါက အားလုံးပါသွားသည်။ User တစ်ဦးချင်းကို resource တွင် တိုက်ရိုက်
permission ပေးခြင်းက ပြုပြင်ထိန်းသိမ်းမှုကို မဖြစ်နိုင်အောင် ရှုပ်စေသည်။

---

## Solution 2 — Private connectivity (health-tech)

### 1) Network layout

- **Project ခွဲ** — `prod-research` (patient data ပါ) နှင့် `shared-services`
  (monitoring/jump server) ခွဲထားပါ — access မှားမှု impact ကို ကန့်သတ်ရန်။
- **Private training subnet** — public IP မခွဲပေးနိုင်သော subnet
  (`--ipv4-public-pools-use-network-pools=False` ဖြင့် ဖန်တီး)။ Requirement (a) ကို
  subnet level ကတည်းက enforce ခြင်းက security group rule ထက် ခိုင်မာသည် —
  ဘယ်သူမှ မှားယွင်း public IP မချိတ်နိုင်ပါ။
- **Management subnet** — jump server/VPN Gateway VM များအတွက် (network တူခွင့်ပြုပြီး
  subnet ခွဲ)။
- သတိ — resource ကို ဖန်တီးပြီးမှ network/subnet ပြောင်း၍ မရပါ — layout ကို
  အစောပိုင်းကတည်းက ဆုံးဖြတ်ပါ။

### 2) Access method ရွေးချယ်မှု

| အသုံးပြုမည့်နေရာ | ရွေးချယ်မှု | အကြောင်းပြချက် |
|---|---|---|
| ရုံး ↔ cloud အမြဲချိတ်ဆက်မှု | **Nebius VPN Gateway** (site-to-site IPsec, IKEv2, BGP, active/passive HA) | on-prem data center နှင့် permanent link လိုသဖြင့် HA ပါသော site-to-site solution သင့်သည် |
| Operator တစ်ဦးချင်း ဝင်ကြည့်ခြင်း | **Jump server (WireGuard)** မှတစ်ဆင့် | client allowlist ဖြင့် ကန့်သတ်နိုင်ပြီး public IP တစ်ခုတည်းထားနိုင်သည် |
| Internal demo tool ပြသခြင်း | **Nebius Tunnels** (အကယ်၍ လိုအပ်ပါက) | inbound port မလို — သို့သော် app-level auth ကို app ဘက်တွင် ကိုယ်တိုင် ထည့်ရမည် |

Requirement (b) အရ operator ဝင်ခွင့်ကို ရုံး internal network (VPN tunnel ကျော်) မှသာ
ခွင့်ပြုပြီး jump server security rule ကို ရုံး CIDR မှသာ SSH ခွင့်ပြုစေပါ။

### 3) Security group design (default deny)

| Rule | Direction | Source/Destination | Port | Access |
|---|---|---|---|---|
| mgmt-in | ingress | jump server SG သို့မဟုတ် ရုံး CIDR | 22 | ALLOW |
| inter-node | ingress | node SG အချင်းချင်း | GPU cluster internal ports | ALLOW |
| egress-essential | egress | package repo / model registry | 443 | ALLOW |
| default | ingress/egress | အားလုံး | အားလုံး | DENY (priority အနိမ့်ဆုံး) |

Priority 1 (အမြင့်) မှ 1000 (အနိမ့်) ဖြစ်သဖြင့် explicit ALLOW rule များကို အမြင့် priority
ဖြင့် ထားပါ။ Default policy က deny ဖြစ်စေရန် rule structure ကို ရည်ရွယ်ပါ —
security group သည် packet-level firewall ဖြစ်သဖြင့် မလိုအပ်သော port အားလုံး ပိတ်ထားနိုင်သည်။
Requirement (c) အတွက် security team ကို `security-auditor` role (view) ပေးပြီး
audit event (data plane event ပါ လိုပါက) မှတစ်ဆင့် စောင့်ကြည့်စေပါ —
workload ထဲ ဝင်စရာ မလိုပါ။

### 4) Single point of failure

- **Jump server တစ်ခုတည်း** — active/passive pair ထားပါ သို့မဟုတ် VPN Gateway ၏
  active/passive HA ကို အားကိုးပါ။
- **VPN Gateway VM-based** ဖြစ်သဖြင့် ၎င်းကိုယ်တိုင် monitor လုပ်ရမည် (HA pair မဖြစ်မနေ)။
- **Single region dependency** — region-level incident အတွက် break-glass procedure
  (မည်သို့ ပြန်ဝင်မလဲ) ကို runbook တွင် ကြိုတင် ရေးထားပါ။

---

## Solution 3 — Secrets, audit, compliance (fintech)

### 1) Secret management design

- DB password ကို **SecretStash** secret အဖြစ် သိမ်းပါ (Terraform manifest သို့မဟုတ်
  app config တွင် hardcode မလုပ်ပါနှင့်)။
- ၉၀ ရက် rotation အတွက် — secret **version အသစ်** ဖန်တီးပြီး **primary** အဖြစ်
  သတ်မှတ်ပါ။ Reference (config/script) က secret ID ကိုသာ ညွှန်းထားသဖြင့်
  redeploy မလိုဘဲ version ပြောင်းသွားသည် — ဤအချက်က requirement (a) ၏ အဓိက အဖြေ ဖြစ်သည်။
- Rotation runbook တွင် ဤအဆင့်များ ထည့်ပါ — (i) version အသစ် ဖန်တီး၊
  (ii) primary အဖြစ် သတ်မှတ် (CLI မှ ဖန်တီးပါက အလိုအလျောက် primary မဖြစ်ပါ —
  မေ့လွယ်သော အဆင့်)၊ (iii) app က ယူသုံးသည်ကို verify၊ (iv) version အဟောင်းကို
  retention ကာလပြီးမှ ဖျက်။
- CI အတွက် secret update ခွင့်ကို `editor` role ဖြင့် လုံလောက်အောင် ထားပါ။

### 2) Audit scope နှင့် retention

- **Control plane event** အားလုံးကို scope အဖြစ် သတ်မှတ်ပါ — "ဘယ် user က ဘယ် resource
  ကို ပြောင်းခဲ့သလဲ" ဆိုသော requirement ကို တိုက်ရိုက် ဖြည့်ဆည်းသည်။
- Data plane event (object read/write) မှာ volume များသဖြင့် regulated data access
  ပါ ပြချင်ပါကသာ ထည့်ပြီး ကုန်ကျစရိတ်/volume ကို ချိန်ပါ။
- ၂ နှစ် retention ကို platform default ပေါ်တွင် မမူတည်ပါနှင့် — **audit event export ကို
  Object Storage bucket သို့ ပုံမှန် (ဥပမာ လစဉ်) ထုတ်ပြီး** bucket ထဲတွင်
  retention policy အရ ၂ နှစ် သိမ်းပါ။ Export သည် event region နှင့် တူသော bucket ကိုသာ
  သုံးနိုင်သည် — multi-region ဆိုပါက bucket per region လိုအပ်မည်။
- Export သည် one-time asynchronous operation ဖြစ်ပြီး state (`RUNNING`/`DONE`/`FAILED`)
  ကို monitoring/alerting ထဲ ထည့်ပါ — export ကျနေသည်ကို မသိပါက retention ကွာသွားမည်။

### 3) Role design

| လုပ်ဆောင်ချက် | ပေးမည့် role | ဘယ်သူ |
|---|---|---|
| Secret/version update | `editor` (သို့မဟုတ် custom, ကျဉ်းသော scope) | `ci-deploy` service account |
| Secret payload ကြည့်ခွင့် | `mysterybox.payload-viewer` | သီးခြား duty ရှိသော security engineer အနည်းငယ်သာ |
| Audit event ကြည့်ခွင့် | `auditlogs.audit-event-viewer` | security team |
| Audit export ထုတ်ခွင့် | `auditlogs.audit-event-exporter` + bucket အတွက် `storage.uploader` | security team (requirement (c)) |

### 4) Separation of duties

- Secret ဖန်တီး/ပြေင်းသူ (CI) နှင့် secret ဖတ်သူ (security engineer) ကို ခွဲထားသည် —
  `editor` က payload မကြည့်နိုင်ခြင်းက Nebius role model တွင် ပါပြီးသား ဖြစ်သည်။
- Audit ကြည့်သူ (view) နှင့် export ထုတ်သူ (export) ကို role ခွဲထားသည်။
- Platform admin သည် incident အချိန်တွင် containment လုပ်ရန် ဖြစ်ပြီး နေ့စဉ် audit
  စောင့်ကြည့်မှု မဟုတ်ပါ — ဤခွဲခြားမှုက compromised admin credential အန္တရာယ်ကို လျှော့ချသည်။

---

## Solution 4 — Data classification နှင့် encryption mapping

### 1) Classification

| Dataset | Classification |
|---|---|
| Public training corpus | Public |
| Licensed artwork | Confidential |
| Customer photo (PII) | Regulated |
| Model weights (IP) | Confidential (ကုမ္ပဏီ၏ အဖိုးတန်ဆုံး asset — အနိမ့်ဆုံး Confidential) |

### 2) Mapping table

| Dataset | Storage | Encryption / key policy | Network exposure |
|---|---|---|---|
| Public corpus | Object Storage bucket | platform default (AES-256, KMS-managed) လုံလောက် | read access ကို လိုသလို ခွင့်ပြုနိုင် |
| Licensed artwork | disk သို့မဟုတ် private bucket | ကိုယ်ပိုင် KMS symmetric key + rotation policy | private subnet |
| Customer photo (PII) | isolated project + encrypted storage | ကိုယ်ပိုင် KMS key + strict rotation + audit data-plane event ပါ export | private subnet၊ access အနည်းဆုံး group သို့သာ |
| Model weights | private bucket + versioned | ကိုယ်ပိုင် KMS key | private subnet၊ training job service account သို့သာ |

### 3) WEKA encryption decision

WEKA filesystem ကို **ဖန်တီးစဉ်ကတည်းက** encryption ဖွင့်ရမည် —
encrypted filesystem အဖြစ် နောက်မှ ပြောင်း၍ မရပါ (documentation အရ)။
ထို့ကြောင့် scratch space အတွက် ဆိုသော်လည်း နောက်ပိုင်းတွင် PII ရောက်လာနိုင်ခြေ
ရှိ/မရှိ ကြိုတွေးပြီး **အစကတည်းက encrypted ဖြင့် ဖန်တီးခြင်း** က safe default ဖြစ်သည်
(XTS-AES-256၊ AES-256 key နှစ်ခု)။ တစ်ဖက်တွင် ကုန်ကျစရိတ်/စွမ်းဆောင်ရည် သက်ရောက်မှုကို
platform team နှင့် အတည်ပြုပါ (needs verification)။

### 4) KMS key design

- **Default platform encryption** (service + infrastructure double encryption) က
  public corpus အတွက် လုံလောက်သည်။
- Confidential နှင့် Regulated dataset များအတွက် **ကိုယ်ပိုင် KMS symmetric key**
  သီးခြား ဖန်တီးပါ — envelope encryption (generate data key) ဖြင့် large object များ
  encrypt ပါ။
- Regulated dataset key အတွက် **automatic rotation period** သတ်မှတ်ပါ (rotation သည်
  version အသစ်ကို default ဖြစ်စေသဖြင့် app ပြင်စရာ မလို)။
- Key တစ်ခုတည်းကို dataset အားလုံးအတွက် မသုံးပါနှင့် — key တစ်ခု compromised ဖြစ်ပါက
  impact ကို classification အလိုက် ကန့်သတ်ထားနိုင်ရန် ဖြစ်သည်။
- WEKA အတွက် DEK ကို WEKA service က စီမံပြီး KEK ကို KMS က စီမံသည် —
  key policy ကို filesystem creation decision နှင့် အတူ ချပါ။

---

## Solution 5 — Incident-response posture

### 1) ပထမ ၂၄ နာရီ

| အဆင့် | လုပ်ဆောင်ချက် | Nebius capability |
|---|---|---|
| Detect | ထို access key ဖြင့် မည်သည့် operation များ ဖြစ်ခဲ့သလဲ စစ်ထုတ်ပါ | Audit Logs — event filter ဖြင့် service account ၏ control plane event များ၊ data plane event (bucket read/write) များ ရှာပါ |
| Contain | access ချက်ချင်း ဖြတ်ပါ | service account ကို group မှ ဖယ်ရှားခြင်း (access ပြတ်သွားသည်)၊ လိုအပ်ပါက security rule ဖြင့် network ကန့်သတ်ခြင်း |
| Eradicate | credential အဟောင်း အသုံးမဝင်အောင် လုပ်ပါ | access key အသစ် ထုတ်ပြီး အဟောင်း ပြန်ဖျက်ခြင်း (method ဆိုင်ရာ အသေးစိတ် needs verification)၊ အကယ်၍ secret မှတစ်ဆင့် ပေါက်ကြားပါက SecretStash တွင် version အသစ် ဖန်တီးပြီး primary လုပ်ခြင်း၊ KMS key rotation လိုအပ်ပါက rotate ခြင်း |
| Recover | service ပြန်လည် လည်ပတ်စေပါ | key အသစ်ဖြင့် pipeline ပြန် run ပြီး bucket access ပုံမှန် ဖြစ်ကြောင်း verify |
| Report | အချိန်ဇယားအတိုင်း အစီရင်ခံပါ | audit event ကို Object Storage သို့ export ပြီး evidence သိမ်းပါ |

### 2) Evidence preservation

- Incident ဖြစ်ပြီး **မကြာမီ** သံသယရှိသော အချိန်အပိုင်းအခြားအတွက် audit event များကို
  bucket သို့ export ပါ (export သည် one-time asynchronous ဖြစ်ပြီး `DONE` state
  ရောက်မှ ပြီးသည်)။
- Export ခွင့် role များ (`auditlogs.audit-event-exporter` + `storage.uploader`) ကို
  incident မဖြစ်မီ ကတည်းက security team အတွက် ကြိုတင် ချထားပါ —
  incident အချိန်တွင် role လျှောက်ခြင်းက အချိန်ကုန်သည်။
- Bucket ကို incident အတွင်း မဖျက်/မပြင်ပါနှင့် — ဒေတာကို ထိန်းသိမ်းထားခြင်းက
  နောက်ပိုင်း investigation နှင့် regulatory အတွက် အရေးကြီးသည်။

### 3) ကြိုတင် ရှိထားသင့်သော design decision များ

- **Identity** — CI အတွက် service account သီးခြား ဖြစ်ခြင်းက human account များကို
  impact မပါစေပါ။ Production bucket scope ကို pipeline အလိုက် ကျဉ်းအောင် ချထားပါက
  blast radius ထပ်ငယ်သည်။
- **Secret management** — key/secret ကို SecretStash တွင် version ဖြင့် သိမ်းထားပါက
  eradicate အဆင့်တွင် version အသစ် ဖန်တီးရုံဖြင့် ပြီးသည် — hardcode ထားပါက
  code/config အားလုံး ပြန်ရှာပြင်ရမည်။
- **Network** — private subnet + jump server ဖြင့် data သို့ ရောက်ရှိမှု လမ်းကြောင်း
  ကျဉ်းအောင် ထားပါက key ပေါက်ကြားသော်လည်း data သို့ တိုက်ရိုက် မရောက်ပါ။
- **Audit** — logging scope နှင့် export ကြိုတင် စီစဉ်ထားပါက detection နှင့် evidence
  နှစ်ခုစလုံး အချိန်မီ ရသည်။

### 4) Post-incident review အစီရင်ခံခြင်း

Architect သည် technical narrative ကို stakeholder ဘာသာစကားဖြင့် ပြန်ဆိုရမည်။
အစီရင်ခံစာတွင် ပါဝင်သင့်သည်များ — timeline (audit event မှ တည်ဆောက်ထားသော)၊
impact scope (မည်သည့် bucket/data ကို ထိခိုက်ခဲ့သလဲ)၊ တုံ့ပြန်မှု အဆင့်များနှင့်
အချိန်ကုန်သုံးစွဲမှု၊ design gap (မည်သည့် ကြိုတင်ဒီဇိုင်းက impact ကို လျှော့ချပေးခဲ့သလဲ)
နှင့် ပြင်ဆင်ရန် action item များ (owner နှင့် ကာလ သတ်မှတ်ပြီး)။
Compliance officer အတွက် evidence export ရှိ/မရှိ ကို ပြသပါ။

---

## Technical review notes

- Solution 3 နှင့် 5 တွင် ဖော်ပြထားသော audit export command/role များကို
  documentation မှ အတည်ပြုထားသည်။ သို့သော် access key revocation
  (လုပ်ဆောင်ပုံ အသေးစိတ်) နှင့် Audit Logs ၏ retention ကာလတို့ကို documentation တွင်
  မတွေ့ရပါ — needs verification။
- Solution 2 တွင် security rule port များကို ဥပမာအဖြစ် ဖော်ပြထားခြင်း ဖြစ်ပြီး
  GPU cluster အမျိုးအစားပေါ် မူတည်၍ လိုအပ်သော port များ ကွဲပြားနိုင်သည်
  (workload framework documentation ဖြင့် အတည်ပြုပါ)။
- Solution 4 တွင် WEKA encryption ဖွင့်ခြင်း၏ စွမ်းဆောင်ရည်/ကုန်ကျစရိတ် သက်ရောက်မှု
  ကို documentation တွင် မတွေ့ရပါ — needs verification။
- "Storage class / tier" ဆိုင်ရာ class ခွဲခြားမှု အသေးစိတ် (lifecycle policy စသည်) ကို
  ဤ module တွင် မဖော်ပြပါ — Object Storage documentation တွင် သီးခြား စစ်ဆေးပါ
  (needs verification)။
