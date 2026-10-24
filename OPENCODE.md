# opencode — التثبيت والموديلات المجانية

## التثبيت (اللي اتعمل فعلاً في الساندبوكس)

```bash
curl -fsSL https://opencode.ai/install | bash -s -- --no-modify-path
export PATH="$HOME/.opencode/bin:$PATH"   # اتضاف لـ ~/.bashrc و ~/.profile
opencode --version                       # 1.19.4
```

- المسار: `~/.opencode/bin/opencode` (ملف واحد، مفيش npm global).
- الإعدادات: `~/.config/opencode/opencode.jsonc`
- الجلسات/الكاش: `~/.local/share/opencode/`
- التثبيت مجاني ومش محتاج أي تسجيل أو مفتاح API للموديلات المجانية.

## الموديلات المجانية (بدون تسجيل — 0 credentials)

دي القائمة اللي `opencode models` بيرجّعها على تثبيت جديد (أكتوبر 2026)،
وكلها `cost: input 0 / output 0` من مزوّد `opencode`:

| الموديل | Context | أقصى Output | ملاحظات |
|---|---|---|---|
| `opencode/beam-free` | 1,048,576 | 131,072 | أكبر سياق، من غير صور |
| `opencode/deepseek-v4.1-flash-free` | 200,000 | 32,000 | يدعم الصور/المرفقات |
| `opencode/flint-40b-free` | 262,144 | 65,536 | |
| `opencode/glm-5.3-flash-free` | 200,000 | 32,000 | يدعم المرفقات |
| `opencode/mimo-v2.6-flash-free` | 200,000 | 32,000 | يدعم المرفقات |
| `opencode/nemotron-3-ultra-free` | 1,000,000 | 128,000 | من غير صور |
| `opencode/nemotron-3.5-lightning-free` | 262,144 | 262,144 | سريع جدًا |
| `opencode/nemotron-3.5-super-vl-free` | 1,048,576 | 131,072 | يدعم الصور/PDF |
| `opencode/big-pickle` | 200,000 | 32,000 | الموديل الافتراضي للنسخة المجانية |

كلهم `tool_call: true` و `reasoning: true`، يعني بيشتغلوا مع أدوات opencode
(قراءة/كتابة ملفات، أوامر شل، البحث...) — اتجرّب فعليًا باستخدام
`big-pickle` و `glm-5.3-flash-free` و `beam-free` و `flint-40b-free`
و `nemotron-3.5-lightning-free`.

## طريقة الاستخدام

```bash
cd /path/to/project
opencode                          # واجهة TUI (اختر الموديل بـ /models)
opencode run "اشرح الملف ده"       # أمر واحد غير تفاعلي
opencode run -m opencode/beam-free "لخّص الريبو"
opencode models                   # قائمة الموديلات المتاحة
opencode models --verbose         # نفس القائمة + التكلفة والسياق (JSON)
opencode stats                    # استهلاك التوكنز والتكلفة
```

## اللي مش شغال بدون حساب

موديلات كتير على models.dev متسجلة `Free` لكن مش متاحة من غير تسجيل
(مثلاً `opencode/grok-code`, `opencode/qwen3.6-plus-free`,
`opencode/minimax-m3.1-flash-free`) وبترجع
`UnknownError: Unexpected server error`.

عشان تفتح الباقي (بما فيه المدفوع أو الـ free المرتبط بحساب):

```bash
opencode providers login          # أو: opencode auth login
opencode providers list           # عرض الـ credentials
```

أو حط مفتاح أي مزوّد في `~/.config/opencode/opencode.jsonc` أو env
(`ANTHROPIC_API_KEY`, `OPENAI_API_KEY`, ...) وبعدها `opencode models` هيكبر.

---

# 🔑 استخدام مفاتيحك الخاصة (اتجرّب فعليًا على 1.19.4)

أيوه، ينفع تشغّل مفاتيحك جوه opencode. فيه 3 طرق — كلها متأكد منها بالتشغيل:

## 1) متغيرات بيئة (الأسهل والأأمن للأسرار)

```bash
export ANTHROPIC_API_KEY=sk-ant-...
opencode models anthropic      # بيطلع موديلات المزوّد
opencode providers list        # بيعرضها تحت Environment
opencode run -m anthropic/claude-sonnet-4-5 "مرحبا"
```

أسماء المتغيرات المتوقعة لكل مزوّد (من models.dev):

| المزوّد | ID | متغيرات البيئة |
|---|---|---|
| Anthropic | `anthropic` | `ANTHROPIC_API_KEY` |
| OpenAI | `openai` | `OPENAI_API_KEY` |
| Google Gemini | `google` | `GOOGLE_API_KEY` / `GOOGLE_GENERATIVE_AI_API_KEY` / `GEMINI_API_KEY` |
| OpenRouter | `openrouter` | `OPENROUTER_API_KEY` |
| Groq | `groq` | `GROQ_API_KEY` |
| DeepSeek | `deepseek` | `DEEPSEEK_API_KEY` |
| xAI | `xai` | `XAI_API_KEY` |
| Mistral | `mistral` | `MISTRAL_API_KEY` |
| Cerebras | `cerebras` | `CEREBRAS_API_KEY` |

عدد الموديلات المتاحة لكل مزوّد لما المفتاح يكون موجود:
`openrouter 391` · `openai 51` · `google 38` · `mistral 34` · `anthropic 17`
· `groq 14` · `xai 13` · `deepseek 4` · `cerebras 2`.
(مفاتيح OpenRouter مفيدة جدًا لأن مفتاح واحد بيفتح مئات الموديلات.)

## 2) `opencode providers login` (تسجيل تفاعلي)

```bash
opencode providers login                 # يختار المزوّد والطريقة
opencode providers login -p anthropic    # يروح على طول لخانة "Enter your API key"
opencode providers logout anthropic      # حذف المفتاح
```

- بيكتب المفتاح في `~/.local/share/opencode/auth.json` (نص عادي، chmod 600).
- الشكل: `{"anthropic":{"type":"api","key":"sk-ant-..."}}`
- ⚠️ **محتاج تيرمنال تفاعلي حقيقي**؛ الـ pipe (`printf 'key' | opencode providers login`)
  بيعلّق، فشغّله بنفسك مش من سكربت.
- بيدعم كمان OAuth لمزوّدين زي GitHub Copilot / Claude / ChatGPT (بيحفظ `type: "oauth"`).

## 3) ملف الـ config (لمزوّد مخصّص أو Base URL مختلف)

`~/.config/opencode/opencode.jsonc` (أو `opencode.json` في جذر المشروع):

```jsonc
{
  "$schema": "https://opencode.ai/config.json",
  "provider": {
    "myprovider": {
      "npm": "@ai-sdk/openai-compatible",
      "name": "My Provider",
      "options": {
        "baseURL": "https://api.example.com/v1",
        "apiKey": "{env:MY_PROVIDER_KEY}"   // أو المفتاح مباشرة (مش مستحب)
      },
      "models": { "my-model": { "name": "My Model" } }
    }
  }
}
```

بعد كده: `opencode models myprovider` → `myprovider/my-model` ✅

ملاحظات من التجربة:
- الـ key القديم `"provider"` (مفرد) هو اللي شغال في 1.19.4 رغم إن الـ schema بيقول
  `deprecated`. النسخة الجديدة `"providers"` (جمع) بترجع
  `Error: Provider not found` في النسخة دي.
- نفس الفكرة تنفع لأي API متوافق مع OpenAI (Ollama, LM Studio, vLLM, OpenRouter...).

## أمان
- أي مفتاح بيدخل الساندبوكس بيتخزن على الجهاز (`auth.json` أو env) — والمفروض
  ميتعملش commit ولا يتحفظ في الريبو.
- الساندبوكس ممكن يتمسح في أي وقت، فالمفاتيح اللي هتتحط هنا مؤقتة؛ لو عايزها
  دائمة حطها في `~/.bashrc` أو في config على جهازك.
- الأحسن: متبعتش المفتاح في الشات — حطّه بنفسك بـ `export` أو `opencode providers login`.
  لو بعتّه، غيّره/اعمله revoke بعد التجربة.

---

# 🧪 تجربة مزوّد cleanapis + OmniRoute (٢٤ أكتوبر ٢٠٢٦)

## 1) مزوّد cleanapis — فشل

```bash
curl https://cleanapis.com/v1/models -H "Authorization: Bearer <KEY>"   # 200 ✅
curl https://cleanapis.com/v1/chat/completions -d '{"model":"kimi-k3",...}'  # 402 ❌
```

- الـ endpoint والمفتاح بيعدّوا، والكتالوج فيه **33 موديل** منهم `kimi-k3` و `gpt-5.6-luna`
  (`gpt-5.6-sol/terra`, `claude-opus-5.5`, `grok-4.6`, `deepseek-v4-pro-max`, `qwen3.8-max` …).
- أي محاولة توليد بترجع:
  `402 {"code":"insufficient_quota","message":"Billing verification failed."}`
  على كل الموديلات (جرّبت كمان `gemma-2-2b` و `glm-5.3`) → **المشكلة في الحساب/الدفع مش في الموديل**.
- يعني المفتاح شكله valid بس مفيش عليه رصيد/الفاتورة مش متأكدة.

## 2) OmniRoute — اشتغل، بس الـ keyless محدود

```bash
npm install -g --prefix /workspace/.tools omniroute      # 3.8.52 (~2.8GB بعد التثبيت)
export PATH=/workspace/.tools/bin:$PATH
OMNIROUTE_MEMORY_MB=1536 omniroute serve --headless --no-open --port 20128
# API: http://localhost:20128/v1   (REQUIRE_API_KEY=false افتراضيًا محليًا)
```

- السيرفر شغال، استهلاك الذاكرة ~130MB في وضع headless، و`/v1/chat/completions` بيرد.
- **بدون أي حساب** المزوّد الوحيد المتاح هو **AI Horde** (`noauth`):

```bash
omniroute providers add aihorde --no-credential --json
curl -s http://127.0.0.1:20128/v1/chat/completions \
  -H 'Content-Type: application/json' \
  -d '{"model":"aihorde/google/gemma-4-31b","messages":[{"role":"user","content":"Reply PONG"}]}'
# → 200 {...,"model":"google/gemma-4-31b","content":"PONG"} ✅
```

- باقي المزوّدين (pollinations / logfare / nara / llm7 / opencode-zen / gemini / groq …)
  محتاجين credential: `providers add pollinations --allow-no-credential` بيرجع `✖ Invalid request`.
- ⚠️ **AI Horde مش بينفع كـ agent**: موديلاته مش بتدعم tool calling، فأي تشغيل من opencode
  بيرجع: `Model "google/gemma-4-31b" does not support tool calling` (حتى بعد `"tool_call": false`).
  ينفع شات عادي بس.
- المسار المجاني اللي فيه `kimi-k3` و `gpt-5.6-luna` هو مزوّد **logfare** جوه OmniRoute —
  بيدي مفتاح API مجاني فورًا من logfare.ai/register (كل request بيتسجل للأبحاث)،
  يعني محتاج تسجيل حساب بإيميلك.

## 3) أقوى موديل مجاني شغال فعلاً (بمقارنة عملية)

كل الموديلات دي اشتغلت **بدون أي مفتاح** في opencode، وكلها عدّت مهمتين برمجيتين
(سكربت + تشغيل + unittest):

| الموديل | Context | Output | مدة المهمة | نتيجة الاختبارات |
|---|---|---|---|---|
| `opencode/beam-free` | 1,048,576 | 131,072 | **5–12s** ⚡ | ✅ 4/4 |
| `opencode/nemotron-3-ultra-free` | 1,000,000 | 128,000 | 16–19s | ✅ 4/4 |
| `opencode/nemotron-3.5-super-vl-free` | 1,048,576 | 131,072 | 25s | ✅ 4/4 (ويدعم صور/PDF) |
| `opencode/deepseek-v4.1-flash-free` | 200,000 | 32,000 | 6s | ✅ |
| `opencode/glm-5.3-flash-free` | 200,000 | 32,000 | 6s | ✅ |

**الافتراضي المضبوط حاليًا:** `opencode/nemotron-3.5-super-vl-free`
(أحدث جيل + سياق 1M + multimodal). للسرعة استخدم `opencode/beam-free`.

## 4) OmniRoute كـ provider جوه opencode

`~/.config/opencode/opencode.jsonc`:

```jsonc
{
  "$schema": "https://opencode.ai/config.json",
  "model": "opencode/nemotron-3.5-super-vl-free",
  "provider": {
    "omniroute": {
      "npm": "@ai-sdk/openai-compatible",
      "name": "OmniRoute (local gateway)",
      "options": { "baseURL": "http://127.0.0.1:20128/v1", "apiKey": "local-no-auth" },
      "models": {
        "aihorde/google/gemma-4-31b": { "name": "Gemma 4 31B (AI Horde, keyless)", "tool_call": false }
      }
    }
  }
}
```

`opencode models omniroute` بيعرض الموديلات ✅ — بس unfit للـ agent بسبب غياب tool calling.

## 5) إعادة التثبيت لو الساندبوكس اتمسح

```bash
npm install -g --prefix /workspace/.tools --allow-scripts=omniroute,keytar,onnxruntime-node,@parcel/watcher,@swc/core,protobufjs,esbuild omniroute
export PATH=/workspace/.tools/bin:$PATH
OMNIROUTE_MEMORY_MB=1536 omniroute serve --headless --no-open --port 20128
# أو بدون --headless لو عايز الداشبورد على http://localhost:20128
```

> 🔐 المفتاح اللي اتبعت في الشات **مش متخزن** في أي ملف هنا؛ الأفضل تعمله revoke/rotate
> لأنه اتكتب في محادثة.

---

# 🔬 سويب كامل على مزوّد cleanapis (33 موديل × مفتاح واحد)

الطريقة: `POST /v1/chat/completions` لكل موديل بـ `max_tokens=16`
(5 طلبات متوازية، وبعدها إعادة محاولة هادئة للموديلات اللي رجّعت 429).

**النتيجة: 0 موديل شغال من 33.**

| الحالة | العدد | الموديلات |
|---|---|---|
| `402 insufficient_quota` (Billing verification failed) | 28 | claude-fable-5, claude-fable-5.1, claude-mythos-preview, claude-opus-4.6, claude-opus-4.7, claude-opus-4.8, claude-opus-5, claude-opus-5.5, claude-sonnet-5, deepseek-v4-flash-0731, deepseek-v4-pro-0813, deepseek-v4-pro-max, gemini-3.1-pro, gemini-3.6-flash, gemini-3.7-flash, gemma-2-2b, glm-5.2, glm-5.3, gpt-5.5, gpt-5.5-pro, gpt-5.6-luna, gpt-5.6-sol, gpt-5.6-terra, grok-4.5, grok-4.6, kimi-k2.6, kimi-k3, muse-spark-1.1 |
| `429` Cloudflare **error 1015** في السويب المتوازي → رجعت **402** بعد إعادة المحاولة | 5 | qwen3.7-max, qwen3.8-27b, qwen3.8-max, seed-2.1-pro, seed-2.1-turbo |
| `200 OK` | **0** | — |

### إثبات إن المفتاح صحيح والمشكلة في الفاتورة

```bash
# مفتاح غلط (control) → 401
authorization: Bearer cc_THIS_IS_NOT_A_REAL_KEY_0000
{"error":{"code":"invalid_api_key","type":"authentication_error"}}   # http=401

# مفتاحك → 402
{"error":{"code":"insufficient_quota","type":"insufficient_funds"}}   # http=402
```

يعني: المصادقة بتنجح (المفتاح متسجل ومش ملغي) لكن **الحساب ما عداش التحقق من الدفع**،
وكل الموديلات (بما فيها الأرخص `gemma-2-2b`) محجوبة. الـ `429/1015` كان rate-limit
من Cloudflare بسبب الطلبات المتوازية، مش مشكلة موديل.

### كمان اتأكدت إنه:
- `stream: true` → نفس 402.
- `/v1/completions` و `/v1/responses` → 404 (الـ API بيوفّر `/v1/chat/completions` و `/v1/models` بس).
- `/v1/credits`, `/v1/dashboard/billing/*`, `/v1/me`, `/v1/key` → 404 (مفيش نقطة استعلام عن الرصيد).

### الخلاصة العملية
عبّي/أكّد طريقة الدفع في حساب cleanapis (أو خُد مفتاح جديد من حساب متحقق)، وبعدها
المزوّد هيتوصل في opencode بالطريقة الموثقة فوق (npm `@ai-sdk/openai-compatible` +
`baseURL: https://cleanapis.com/v1`). لحد ما ده يحصل، الشغال فعلاً هو free tier بتاع opencode.

---

# ✅ مزوّد Dahl (inference.dahl.global) — شغال

```jsonc
"dahl": {
  "npm": "@ai-sdk/openai-compatible",
  "name": "Dahl inference (gonka/vLLM)",
  "options": { "baseURL": "https://inference.dahl.global/v1", "apiKey": "dahl_..." },
  "models": {
    "zai-org/GLM-5.3-Flash": { "name": "GLM-5.3-Flash (Dahl)" },
    "MiniMaxAI/MiniMax-M2.7": { "name": "MiniMax-M2.7 (Dahl)" },
    "deepseek-ai/DeepSeek-V4-Flash-0731": { "name": "DeepSeek-V4-Flash-0731 (Dahl)" }
  }
}
```

### الكتالوج (3 موديلات بس، سيرفر vLLM/Gonka)
`zai-org/GLM-5.3-Flash` · `MiniMaxAI/MiniMax-M2.7` · `deepseek-ai/DeepSeek-V4-Flash-0731`

### نتائج التشغيل الحقيقية (opencode agent: كتابة ملف + تنفيذه)

| الموديل | النتيجة | التفاصيل |
|---|---|---|
| `MiniMaxAI/MiniMax-M2.7` | ✅ **شغال ونضيف** | 25s، كتب `fib.py` و`sum.py` ونفّذهم صح (34 / 55) — **الافتراضي الحالي** |
| `zai-org/GLM-5.3-Flash` | ⚠️ شغال بس متقطّع | كتب `fib.py` ونفّذه صح، لكن مرة دخل **doom loop** (`true` متكرر → permission rejected)، ومرة طلّع كلام برّه الموضوع. وكمان بيرجع `429 model_concurrency` كتير |
| `deepseek-ai/DeepSeek-V4-Flash-0731` | ⚠️ بيعلّق | كتب الملف ونفّذه صح (0..34) وبعدها علّق لحد الـ timeout |

### Tool calling
مدعوم ✅ (`tool_calls` سليمة بـ args صح) لو بعتّ `tools` + system prompt + `temperature: 0`.

### حدود الخدمة (مهم)
- الخدمة **free tier بـ concurrency محدودة**: لما الموديل يكون مشغول:
  ```json
  429 {"error":{"code":"model_concurrency",
    "message":"This model is at concurrency capacity. Paid accounts are admitted first.
     Top up ... — $0.03 per 1M tokens (in + out + cached) on any model."}}
  ```
  وبيقولك الموديلات المتاحة بداله في نفس اللحظة.
- السامبلر على السيرفر بيرجّع أحيانًا تكرار مشوّش (`PONGPONGPONG`، `391What is 17*23?...`)
  وردود GLM بتبدأ reasoning في `content` — فالأفضل متعتمدش عليه في مهام طويلة.
- الردود بتخلط `<think>` جوه الـ content (MiniMax/DeepSeek)، فممكن تشوف تفكير الموديل في المخرجات.

### التبديل السريع
```bash
opencode run -m dahl/MiniMaxAI/MiniMax-M2.7 "..."        # الافتراضي (مستقر)
opencode run -m dahl/zai-org/GLM-5.3-Flash "..."         # لو فيه سعة
opencode run -m opencode/nemotron-3.5-super-vl-free "..."  # free tier بتاع opencode (مستقر دائمًا)
```

---

# 📱 برومبت بناء APK

البرومبت الجاهز في [`PROMPT-APK.md`](./PROMPT-APK.md) — تطبيق أندرويد (Flutter) اسمه
**AI Studio**: يضيف أي مزوّد AI بـ Base URL + API Key + موديل، ويولّد **شات + صور + فيديو**،
مع Jobs في الخلفية (WorkManager)، تخزين مشفّر للمفاتيح، عربي RTL/إنجليزي، وبناء APK
(أو GitHub Actions fallback لو الـ toolchain مش متاح).

```bash
mkdir -p /workspace/app && cd /workspace/app
opencode run "$(cat /workspace/PROMPT-APK.md)"
```

> ملاحظة: البيئة هنا فيها Node/Bun/Python بس — **مفيش Java ولا Flutter ولا Android SDK**،
> فالبرومبت بيسمح للموديل يثبّت toolchain أو يسلّم workflow يبني الـ APK على CI.
