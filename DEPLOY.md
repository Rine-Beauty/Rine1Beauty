# دليل نشر المشروع (استضافة مجانية بالكامل، بدون Replit)

المشروع صار مبني من 3 أجزاء منفصلة، كل وحدة على خدمة مجانية:

| الجزء | الخدمة | ليش |
|---|---|---|
| قاعدة البيانات | [Neon](https://neon.tech) | Postgres مجاني |
| تخزين الصور | [Cloudflare R2](https://dash.cloudflare.com) | تخزين مجاني (10GB) |
| الباك اند (API) | [Render](https://render.com) | استضافة Node.js مجانية |
| الفرونت اند (المتجر) | [Cloudflare Pages](https://pages.cloudflare.com) | استضافة مواقع ثابتة مجانية |

> ملاحظة: باقات Render المجانية "تنام" بعد فترة عدم استخدام وبتاخد كام ثانية
> لتصحى بأول طلب. إذا هاد مزعج ممكن تدفع خطة صغيرة ($7/شهر) بس مش إجباري.

---

## 0. قبل ما ترفع عالـ GitHub — خطوة إجبارية

المشروع ما فيه `pnpm-lock.yaml` (لازم يتولّد عندك بعد التعديلات). لازم تعمل
هاد قبل أول push:

```bash
pnpm install
```

هاد بيولّد ملف `pnpm-lock.yaml` — **لازم ترفعه مع باقي الكود على GitHub**
(مش موجود بـ `.gitignore`، فراح ينرفع تلقائياً لو استخدمت `git add .`).
بدون هاد الملف، بعض منصات الاستضافة (Cloudflare مثلاً) ممكن تستخدم أداة
تنصيب غلط (زي `bun`) وتتجاهل بنية الـ workspace بالكامل.

---

## 1. قاعدة البيانات (Neon)

1. أنشئ حساب على neon.tech وأنشئ مشروع جديد.
2. انسخ الـ **Connection string** (شكله `postgresql://user:pass@host/db?sslmode=require`).
3. من جهازك (بعد ما تعمل `pnpm install` بجذر المشروع):
   ```bash
   cd lib/db
   DATABASE_URL="الرابط يلي نسخته" pnpm push
   ```
   هاد بينشئ الجداول كلها بقاعدة البيانات.

---

## 2. تخزين الصور (Cloudflare R2)

1. من لوحة Cloudflare -> **R2** -> Create bucket -> سمّيه مثلاً `rine-beauty`.
2. من **R2 -> Manage API Tokens** -> Create API Token -> صلاحية Object Read & Write على الـ bucket.
3. احتفظ بـ:
   - `Account ID` (موجود بالصفحة الرئيسية لـ R2)
   - `Access Key ID`
   - `Secret Access Key`

---

## 3. تسجيل دخول الأدمن (Google OAuth)

تسجيل الدخول للوحة التحكم صار حصراً عبر حساب Google.

1. روح لـ [Google Cloud Console -> Credentials](https://console.cloud.google.com/apis/credentials).
2. أنشئ مشروع (لو ما عندك واحد)، وأنشئ **OAuth client ID** من نوع **Web application**.
3. لو رح تستخدم proxy الـ Cloudflare Pages (الموصى فيه، خطوة 5) — الموقع والـ API
   بيصيروا بنفس الدومين، فحط بـ **Authorized redirect URIs**:
   ```
   https://اسم-موقعك.pages.dev/api/callback
   ```
   ولو رح تستخدم رابط Render مباشرة، ضيف رابط Render كمان.
4. خذ الـ `Client ID` و `Client secret` وحطهم بمتغيرات البيئة على الباك اند:
   `GOOGLE_CLIENT_ID` و `GOOGLE_CLIENT_SECRET`.

**المشرف الأساسي** هو `eng.salah.a.barham@gmail.com` — ثابت بالكود
(`artifacts/api-server/src/lib/adminAuth.ts`) وما ينحذف. هو الوحيد يلي فيه
تبويب "المشرفون" بلوحة التحكم، وفيه يقدر يضيف أو يحذف أي حساب Google تاني
كمشرف (بيتخزنوا بقاعدة البيانات).

---

## 4. نشر الباك اند (Render)

1. ارفع المشروع كامل على GitHub (repo واحد فيه كل شي كما هو).
2. Render -> New -> Web Service -> اختار الـ repo.
3. الإعدادات:
   - **Root Directory**: (اتركه فاضي — جذر الـ repo، لأنه monorepo)
   - **Build Command**:
     ```
     pnpm install && pnpm run build:backend
     ```
   - **Start Command**:
     ```
     pnpm --filter @workspace/api-server run start
     ```
   - **Environment Variables** (من Render Dashboard -> Environment):
     - `DATABASE_URL` = رابط Neon
     - `R2_ACCOUNT_ID`, `R2_ACCESS_KEY_ID`, `R2_SECRET_ACCESS_KEY`, `R2_BUCKET_NAME`
     - `GOOGLE_CLIENT_ID`, `GOOGLE_CLIENT_SECRET`
     - `COOKIE_CROSS_SITE` = `false` (لأننا رح نستخدم proxy، شوف الخطوة 5)
4. بعد ما ينشر، انسخ الرابط يلي بيديك ياه Render (مثلاً `https://rine-beauty-api.onrender.com`).

راجع `artifacts/api-server/.env.example` لقائمة كاملة بالمتغيرات.

---

## 5. نشر الفرونت اند (Cloudflare Pages)

1. عدّل ملف `artifacts/rine-beauty/public/_redirects` وحط رابط Render الحقيقي بدل
   `REPLACE-WITH-YOUR-API-DOMAIN.onrender.com`.
2. Cloudflare Pages -> Create a project -> اربطه بنفس الـ GitHub repo.
3. الإعدادات:
   - **Framework preset**: Vite
   - **Build command**:
     ```
     pnpm install && pnpm run build:frontend
     ```
     > ⚠️ **مهم**: Cloudflare أحياناً بيعبّي هاد الحقل تلقائياً بأمر افتراضي (زي
     > `pnpm run build`) لما يكتشف إنه مشروع Vite — وهاد الأمر الافتراضي غلط
     > لمشروعنا (بيحاول يعمل typecheck للمشروع كامل ومش بس الفرونت اند). لازم
     > تتأكد إنك **كتبت الأمر يدوياً** بالحقل وضغطت حفظ، مش بس اعتمدت على القيمة
     > المقترحة.
   - **Build output directory**: `artifacts/rine-beauty/dist/public`
4. بعد النشر، الموقع رح يشتغل عادي، وأي طلب لـ `/api/*` رح ينعمله proxy تلقائياً
   على الباك اند بفضل ملف `_redirects` — يعني المتصفح بيشوف كل شي كأنه نفس
   الموقع، وما رح تصير مشاكل كوكيز أو صلاحيات.

---

## 6. التحقق

- افتح الموقع، تأكد إن المتجر بيظهر.
- روح على `/admin` وسجّل دخول بحساب Google (`eng.salah.a.barham@gmail.com` كبداية).
- من تبويب "المشرفون" جرّب تضيف حساب Google تاني وتأكد إنه صار يقدر يسجّل دخول.
- جرّب ترفع صورة منتج تأكد إنها بتنزل على R2 وبتظهر.

---

## ملاحظات إضافية

- إذا حبيت لاحقاً تستخدم دومين خاص فيك تربطه من Cloudflare Pages مباشرة (مجاني).
- إذا قررت لاحقاً ما تستخدم الـ proxy وتخلي الفرونت يتواصل مباشرة مع رابط Render،
  فعّل `VITE_API_URL` (بملف `artifacts/rine-beauty/.env.example`) وحط
  `COOKIE_CROSS_SITE=true` بالباك اند.
- ما قدرنا نختبر `pnpm install` أو build كامل بهاي البيئة (ما في صلاحية إنترنت
  هون)، فلو طلعت أي مشكلة بسيطة بالبناء (اسم باكج، نسخة...) رجعلي بالخطأ وبنصلحه.
