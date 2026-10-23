# كوتش مين ⚽🧢 — تطبيق خطط تدريب كرة القدم بالذكاء الاصطناعي

المستخدم يختار عمره ومركزه ومستواه وهدفه، فيولّد التطبيق **خطة اليوم** أو **خطة أسبوعين** من مهام يضع عليها علامة عند إنجازها، مع نقاط ومستويات وسلسلة أيام متتالية.

## كيف يعمل

```
تطبيق Flutter  ──►  خادم Cloudflare Worker  ──►  Gemini (مجاني)
  (على الجوال)        (يخفي مفتاح الـAPI،          (يختار التمارين
                       ويتحقق من الخطة)              من مكتبة جاهزة)
```

- الذكاء الاصطناعي يختار فقط من **مكتبة تمارين جاهزة** (`shared/exercises.json`)، فلا يخترع تمارين.
- الخادم يتحقق من كل خطة: عدد الأيام، أيام الراحة، مدة الجلسة، ومناسبة التمرين للعمر والمعدات. إن كانت الخطة رديئة يعيد المحاولة حتى 3 مرات.
- المستخدم لا يكتب نصاً حراً؛ يختار من قوائم فقط، فيبقى المدرب في كرة القدم.
- لو تعطّل الخادم أو انتهت الحصة المجانية، يبني التطبيق خطة احتياطية من القوالب على الجوال بدون إنترنت.

## الخطوات

### 1) مفتاح Gemini المجاني
من https://aistudio.google.com/apikey أنشئ مفتاحاً.

### 2) نشر الخادم (مجاني على Cloudflare)
```bash
cd worker
npm install
npx wrangler login
npx wrangler secret put GEMINI_API_KEY    # الصق مفتاح Gemini
npx wrangler secret put APP_KEY           # اخترع كلمة سر طويلة (يرسلها التطبيق مع كل طلب)
npx wrangler deploy
```
سيظهر لك رابط مثل `https://kora-coach-api.اسمك.workers.dev`، احتفظ به.

لتغيير النموذج دون تعديل الكود: أضف في `wrangler.toml` السطر `[vars]` ثم `GEMINI_MODEL = "..."` بالاسم الظاهر في Google AI Studio.

### 3) بناء ملف APK بدون تثبيت أي شيء على جهازك
1. أنشئ مستودعاً على GitHub وارفع محتويات هذا المجلد كما هي.
2. في المستودع: Settings ← Secrets and variables ← Actions
   - تبويب **Variables**: أضف `API_BASE` = رابط الخادم من الخطوة 2.
   - تبويب **Secrets**: أضف `APP_KEY` = نفس كلمة السر من الخطوة 2.
3. تبويب **Actions** ← **Build APK** ← **Run workflow**.
4. بعد 5 إلى 10 دقائق افتح التشغيل وحمّل `kora-coach-apk` من قسم Artifacts، وفيه `app-release.apk`.
5. انقله إلى الجوال وثبّته (فعّل "التثبيت من مصادر غير معروفة").

### بناء محلي (اختياري)
```bash
cd app
cp ../shared/exercises.json assets/exercises.json
flutter create . --platforms=android --org com.koracoach --project-name kora_coach
# أضف <uses-permission android:name="android.permission.INTERNET"/> إلى android/app/src/main/AndroidManifest.xml
flutter pub get
flutter build apk --release --dart-define=API_BASE=https://رابط-الخادم --dart-define=APP_KEY=كلمة-السر
```

### اختبار الخادم
```bash
cd worker && npm test
```

## ملاحظات مهمة

- **APP_KEY** داخل التطبيق يمكن استخراجه من ملف APK بجهد بسيط، فهو يمنع الإساءة العابرة فقط. للحماية الأقوى أضف قاعدة Rate Limiting من لوحة Cloudflare، ثم لاحقاً تسجيل دخول للمستخدمين.
- الطبقة المجانية من Gemini لها حد يومي وحد بالدقيقة، وشروطها تتغير؛ راجعها في Google AI Studio. وقد تُستخدم بيانات الطبقة المجانية لتحسين منتجات Google، فلا ترسل بيانات شخصية حساسة (التطبيق يرسل العمر والمركز فقط).
- ملف APK هنا موقّع بمفتاح التجربة. للنشر في Google Play تحتاج مفتاح توقيع خاصاً بك وحساب مطوّر.
- **الربح لاحقاً:** أسهل مسار هو أسبوع مجاني ثم اشتراك (Google Play Billing عبر حزمة `in_app_purchase` أو RevenueCat)، مع إبقاء خطة اليوم مجانية وقفل خطة الأسبوعين.
- الطابع الكوري: شخصية "كوتش مين" مبتكرة وليست مقلِّدة لأي شخص حقيقي، وكلمات مثل 화이팅 (فايتينغ) وألوان باستيل ومستويات بأسماء كورية.

## هيكل المشروع
```
shared/exercises.json     مكتبة التمارين (مصدر واحد للخادم والتطبيق)
worker/                   الخادم + الاختبارات
app/lib/                  كود Flutter (الشاشات، الخدمات، الخطة الاحتياطية)
.github/workflows/        بناء APK تلقائياً
```
