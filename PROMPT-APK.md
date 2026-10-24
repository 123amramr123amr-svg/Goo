# 📱 برومبت جاهز: بناء APK لتطبيق «Multi-Model AI Studio»

> **التشغيل:**
> ```bash
> mkdir -p /workspace/app && cd /workspace/app
> opencode run "$(cat /workspace/PROMPT-APK.md)"
> ```
> البرومبت بين `<<<BEGIN PROMPT` و `END PROMPT>>>` — انسخ الجزء ده بس لو حبيت.
> الموديل الافتراضي المضبوط: `opencode/nemotron-3.5-super-vl-free` (سياق 1M، مستقر).

---

<<<BEGIN PROMPT

# أنت: مهندس موبايل أول (Senior Mobile Engineer) متخصص Flutter + تكاملات LLM APIs

مهمتك: تبني **تطبيق أندرويد كامل** (Flutter) وتطلّع **APK يشتغل على الهاتف فعليًا**.
التطبيق اسمه **AI Studio** ووظيفته: المستخدم يضيف **أي مزوّد ذكاء اصطناعي من أي مكان**
عن طريق **Base URL + API Key + اسم الموديل**، ويستخدمه في **الشات، توليد الصور،
وتوليد الفيديو** — كله من نفس التطبيق، بدون سيرفر وسيط (التطبيق يتصل بالمزوّد مباشرة).

اشتغل بشكل مستقل تمامًا: **متسألنيش أي سؤال**، خد قرارات هندسية معقولة وسجّلها في
`DECISIONS.md`، وكمّل لحد ما الـ APK يتبني بنجاح.

---

## 1) المواصفات الوظيفية (Functional Spec)

### 1.1 إدارة المزوّدين (Providers) — القلب بتاع التطبيق
- إضافة / تعديل / حذف / تعطيل / ترتيب (drag & drop) للمزوّدين.
- حقول المزوّد:
  - `name` (اسم حرّ)
  - `baseUrl` (مثال: `https://api.openai.com/v1`، `https://openrouter.ai/api/v1`،
    `https://inference.dahl.global/v1`، أو أي endpoint خاص)
  - `apiKey` (يُخزَّن مشفّرًا — ممنوع نص عادي)
  - `authStyle`: `bearer` (افتراضي) | `x-api-key` | `custom header name`
  - `apiKind`: `openai-compatible` | `anthropic` | `gemini` | `custom`
  - `extraHeaders` (Key/Value اختياري — لـ OpenRouter مثلاً `HTTP-Referer`)
  - `models[]`: كل موديل له `id` + `capabilities` (نص / streaming / صورة / فيديو / vision)
  - `imageEndpoint` / `videoEndpoint` (قابلة للتعديل لو المزوّد مش OpenAI-compatible)
- زر **Test Connection**: يعمل `GET {baseUrl}/models` ويعرض: نجاح/فشل + زمن الاستجابة +
  كود الخطأ + عدد الموديلات. ولو فشل: رسالة واضحة (401 مفتاح غلط، 402 رصيد،
  429 rate limit، timeout).
- زر **Auto-discover models**: يجيب الموديلات من `/models` وتعلّم كل موديل بقدراته.
- Presets جاهزة بضغطة: OpenAI، OpenRouter، Groq، Together، DeepInfra، Fireworks،
  Replicate، Fal.ai، Stability، Google Gemini، Anthropic، Ollama (LAN)، Custom.
- استيراد/تصدير المزوّدين JSON **بدون المفاتيح** (المفاتيح تتصدّر مشفّرة بكلمة سر يحددها المستخدم).

### 1.2 الشات النصي
- اختيار المزوّد + الموديل، ودعم **streaming** (SSE) مع إيقاف/استكمال.
- System prompt لكل محادثة، temperature / top_p / max_tokens، عدّاد توكنز، نسخ الرد ومشاركته.
- حفظ المحادثات محليًا (قابلة للبحث)، إعادة تسمية، حذف، تصدير Markdown.

### 1.3 توليد الصور
- شاشة: prompt + negative prompt (لو مدعوم) + size + n + quality/style + seed.
- المسار الافتراضي `POST {baseUrl}/images/generations`، ودعم شكلين للرد:
  `{"data":[{"b64_json":"..."}]}` أو `{"data":[{"url":"..."}]}`.
- لو المزوّد مختلف (Gemini / Stability / Replicate) → adapter مخصص لكل واحد.
- النتيجة: معرض شبكي، عرض ملء الشاشة + zoom، حفظ في الاستوديو، مشاركة، حذف.
- كل صورة تتحفظ مع metadata (المزوّد / الموديل / prompt / seed / التاريخ) وتظهر في Gallery داخل التطبيق.

### 1.4 توليد الفيديو (أهم جزء — لازم يشتغل صح)
المزوّدون بيشتغلوا بطريقتين، والتطبيق لازم يدعم الاتنين:
1. **Async job** (الأشهر): `POST {videoEndpoint}` → يرجّع `job_id`/`id` → polling على
   `GET {videoEndpoint}/{id}` كل N ثانية (قابل للتعديل، افتراضي 5s) → لما الحالة تبقى
   `succeeded/completed/done` → نزّل الفيديو من `url`/`output`/`video_url`. ولو `failed` →
   اعرض رسالة الخطأ من الـ payload.
2. **Sync**: الرد نفسه فيه `url` أو `b64` → نزّله على طول.
- الطلب: prompt + duration + aspect_ratio + image-to-video (اختيار صورة من المعرض كـ first frame) + seed.
- **الاستمرارية**: الـ polling في **WorkManager foreground job** ويكمّل لو التطبيق اتقفل أو
  الشاشة اتغيرت، مع **إشعار محلي** عند الانتهاء أو الفشل.
- شاشة **Jobs**: كل الطلبات (queued/running/succeeded/failed) + progress + retry + cancel + حذف.
- الفيديو يتحفظ في مجلد التطبيق + يُضاف لمعرض الهاتف (MediaStore) + يشتغل بمشغّل داخلي
  (`video_player`) + مشاركة.

### 1.5 إدارة الطلبات والأخطاء
- كل طلب (نص/صورة/فيديو) يتسجل في جدول `requests`: المزوّد، الموديل، الكلفة لو متاحة،
  الحالة، الكمون، كود الخطأ.
- Retry تلقائي مع **backoff** للأكواد 429/500/502/503 (+ احترام `Retry-After`).
- Fallback: لو المزوّد فشل → جرّب المزوّد التالي في الترتيب (اختياري من الإعدادات).
- رسائل خطأ مفهومة لكل كود، وزر «انسخ تفاصيل الخطأ».

### 1.6 الإعدادات والأمان
- تخزين المفاتيح في **flutter_secure_storage** (Android Keystore / EncryptedSharedPreferences)،
  ومنع أي طباعة للمفاتيح في logs أو رسائل الخطأ (mask: `dahl_****...5`).
- قفل بيومتري اختياري لفتح التطبيق.
- Theme: light/dark/system، Material 3، ألوان قابلة للتخصيص.
- اللغة: **عربي (RTL) + إنجليزي** عبر `intl`/`flutter_localizations`، والتبديل فوري بدون restart.
- إعدادات الشبكة: timeout، عدد المحاولات، حجم الصفحة، proxy اختياري.
- شاشة «حول» فيها نسخة التطبيق + تنبيه إن المفاتيح على مسؤولية المستخدم + شروط المزوّدين.

## 2) تقنيات إجبارية (Tech Stack)

- **Flutter 3.x / Dart 3** (Material 3، null-safety كامل).
- الحزم: `flutter_riverpod` (state)، `dio` (HTTP + interceptors + SSE عبر `ResponseType.stream`)،
  `go_router`، `drift` أو `sqflite`، `flutter_secure_storage`، `workmanager`،
  `flutter_local_notifications`، `video_player`، `cached_network_image`،
  `gal`/`photo_manager` (حفظ في المعرض)، `share_plus`، `image_picker`، `path_provider`،
  `intl` + `flutter_localizations`، `freezed`/`json_serializable`، `mocktail`، `file_picker`.
- معمارية **Clean + Feature-first**:
  ```
  lib/
    core/            (http client, result type, error mapping, i18n, theme, di)
    features/
      providers/     data/ domain/ presentation/
      chat/          data/ domain/ presentation/
      image_gen/     data/ domain/ presentation/
      video_gen/     data/ domain/ presentation/
      jobs/          data/ domain/ presentation/
      settings/      ...
    shared/          (widgets, extensions)
  ```
- **Adapters**: واجهة موحّدة
  ```dart
  abstract class AiProviderAdapter {
    Future<List<RemoteModel>> listModels();
    Stream<String> chatStream(ChatRequest req);
    Future<ImageResult> generateImage(ImageRequest req);
    Future<VideoJob> startVideo(VideoRequest req);
    Future<VideoJob> pollVideo(String jobId);
  }
  ```
  وتنفيذات: `OpenAiCompatAdapter`, `AnthropicAdapter`, `GeminiAdapter`, `CustomAdapter`
  (الأخير يقرأ مسارات وحقول JSON من إعدادات المستخدم).
  أي مزوّد متوافق مع OpenAI **لازم يشتغل تلقائيًا** بالـ adapter الأول.

## 3) قواعد إلزامية (Hard Rules)

1. **ممنوع** أي API key داخل الكود أو الـ repo أو ملفات الـ tests — كل المفاتيح من المستخدم.
2. **ممنوع** TODO أو أزرار فاضية: كل زر وكل شاشة لازم تشتغل فعليًا.
3. **ممنوع** خطأ في `flutter analyze` (لازم 0 issues) وكل `flutter test` ينجح.
4. كل طلب شبكة يمرّ عبر `ApiClient` واحد فيه: timeout، retry، logging آمن (مفاتيح masked)،
   وتحويل الأخطاء لـ `AppFailure` بنوع واضح (auth/quota/rateLimit/network/server/parse).
5. البيانات الكبيرة (فيديو/صور) streaming/تنزيل على أجزاء — ممنوع تحميل فيديو كامل في الـ memory.
6. الكود يتحمّل: rotation، إغلاق التطبيق أثناء job، إنترنت مقطوع، ردود JSON ناقصة/غريبة.

## 4) خطة التنفيذ بالترتيب (نفّذها كلها)

1. **Init**: `flutter create --org com.aistudio --project-name ai_studio .`، اضبط
   `pubspec.yaml` و`analysis_options.yaml` صارم وهيكل المجلدات.
2. **Core**: `ApiClient` (dio + interceptors + retry + masked logging)، `AppFailure`، `Result`،
   DI بـ riverpod، الثيم، i18n (ar/en + RTL)، `SecureStore`.
3. **Domain + DB**: entities + جداول providers / models / conversations / messages /
   images / video_jobs / requests.
4. **Adapters**: نفّذ الأربعة + اختبارات وحدة بـ fake http client
   (نجاح / 401 / 402 / 429 مع Retry-After / 500 / JSON غلط / timeout).
5. **الشاشات** بالترتيب: Providers (+Test +Discover) → Chat (streaming) → Image Gen + Gallery
   → Video Gen + Player → Jobs → Settings → About.
6. **الخلفية**: WorkManager لـ polling الفيديو + إشعارات + استكمال بعد إعادة تشغيل الجهاز.
7. **الجودة**: `dart format .` ، `flutter analyze` = 0، `flutter test` كله ينجح،
   واختبار ويدجت واحد على الأقل لشاشة Providers وشاشة Video.
8. **الـ APK**:
   - لو الـ toolchain متاح (`flutter doctor` نظيف): `flutter build apk --release --split-per-abi`
     وتأكد إن الملف في `build/app/outputs/flutter-apk/`.
   - **لو Flutter/Java/Android SDK مش متاحين** (زي بيئة سيرفر فاضية): ثبّتهم بالترتيب
     (OpenJDK 17 → Android cmdline-tools + platform 35 + build-tools → Flutter stable)
     وبعدها ابنِ الـ APK. **ولو الموارد مش كافية** (رام/مساحة): سلّم بدلًا منها
     `.github/workflows/build-apk.yml` يبني APK على GitHub Actions ويرفع الـ artifact،
     **وكمان** `scripts/build_apk.sh` يبني محليًا. واكتب في `BUILD_NOTES.md`
     إيه اللي اشتغل فعلًا وإيه اللي متبقّي.
9. **التوثيق**: `README.md` (الشاشات + إزاي تضيف مزوّد خطوة بخطوة + أمثلة curl لكل نوع)،
   `DECISIONS.md`، `BUILD_NOTES.md`، و`PROVIDERS.md` (جدول المزوّدين المدعومين وحدودهم).

## 5) معايير القبول (لازم تتحقق كلها)

- [ ] مزوّد **OpenAI-compatible** بـ baseURL + key فقط → «Test» أخضر → الموديلات ظهرت →
      شات streamed شغال → صورة اتولدت واتحفظت في المعرض.
- [ ] مزوّد فيديو async → Job ظهر بـ progress → اكتمل → الفيديو اتشغّل داخليًا واتحفظ واتشارك.
- [ ] مفتاح فاسد → رسالة «401 مفتاح غير صالح» بدون crash وبدون طباعة المفتاح.
- [ ] 429 → retry مع backoff واحترام `Retry-After` ورسالة واضحة للمستخدم.
- [ ] إغلاق التطبيق أثناء job فيديو → يرجع يكمل لوحده ويطلع إشعار.
- [ ] العربي RTL مظبوط، مفيش نص مقطوع، والثيم dark/light شغال.
- [ ] `flutter analyze` = 0 issues و`flutter test` كله ناجح.
- [ ] APK موجود فعليًا (أو CI workflow جاهز + `BUILD_NOTES.md` يوضح الحالة).

## 6) المخرجات النهائية (سلّمها في آخر الرد)

1. شجرة الملفات الكاملة.
2. أوامر التشغيل/البناء + مسار الـ APK وحجمه.
3. جدول المزوّدين المدعومين وأنواع الـ API لكل واحد.
4. ملخص الاختبارات (عدد/ناجح/فاشل) ونتيجة `flutter analyze`.
5. حدود معروفة + الخطوة التالية المقترحة.

## 7) ابدأ الآن

ابدأ فورًا بإنشاء المشروع، واشتغل خطوة خطوة، وشغّل الأوامر بنفسك وأصلح أي خطأ
لحد ما توصل لـ APK (أو الـ CI fallback) — **متوقفش تسأل، كمّل**.

END PROMPT>>>

---

## 🧪 إزاي تستخدمه صح

```bash
# 1) مجلد فاضي للمشروع (متخليش الموديل يلخبط ملفات تانية)
mkdir -p /workspace/app && cd /workspace/app

# 2) شغّل البرومبت
opencode run "$(cat /workspace/PROMPT-APK.md)"

# 3) لو عايز تكمّل نفس الجلسة بعدين
opencode --continue "كمّل من حيث ما وقفت"

# 4) لو عايز موديل تاني
opencode run -m dahl/MiniMaxAI/MiniMax-M2.7 "$(cat /workspace/PROMPT-APK.md)"
opencode run -m opencode/beam-free "$(cat /workspace/PROMPT-APK.md)"
```

### نصائح مهمة
- **شغّله في مجلد فاضي** — وإلا الموديل ممكن يعدّل ملفات مشروعك.
- البرومبت بيجبر الموديل يبني APK؛ في بيئة من غير Flutter/Java هيثبّت toolchain (تحميل ~2GB)
  أو يسلّم GitHub Actions workflow يبني الـ APK في الـ CI — الاتنين مذكورين صريح.
- لو عايز نسخة أصغر/أسرع: احذف الأقسام 1.4 (فيديو) و1.6 (بيومتري) واطلب MVP في شات + صور الأول.
- لو الموديل وقف في نص الطريق: `opencode --continue "كمّل، لسه فاضل ..."` بدل ما تبدأ من الأول.


