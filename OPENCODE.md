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
