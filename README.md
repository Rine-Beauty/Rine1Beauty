# Rine Beauty

متجر إلكتروني (React + Express + Postgres) بدون أي اعتمادية على Replit —
جاهز للاستضافة المجانية على Cloudflare Pages + Render + Neon + Cloudflare R2.

للنشر خطوة بخطوة، شوف [`DEPLOY.md`](./DEPLOY.md).

## البنية

هاد monorepo مبني بـ pnpm workspaces:

```
artifacts/
  rine-beauty/     الفرونت اند (React + Vite + Tailwind) — المتجر ولوحة التحكم
  api-server/      الباك اند (Express + Drizzle ORM)
lib/
  db/                    schema وconnection لقاعدة البيانات (Postgres)
  api-zod/               أنواع/schemas مولّدة من OpenAPI spec
  api-client-react/      عميل الـ API المولّد + React Query hooks
  replit-auth-web/       hook تسجيل الدخول (useAuth) — بالرغم من الاسم، صار
                         عام ويشتغل بأي مكان (اسمه القديم من قبل الترحيل)
  object-storage-web/    hook رفع الملفات (useUpload)
```

## التشغيل محلياً

```bash
pnpm install
cp artifacts/api-server/.env.example artifacts/api-server/.env   # وعبّي القيم
pnpm --filter @workspace/db run push   # ينشئ الجداول على قاعدة البيانات

# تشغيل الباك اند
pnpm --filter @workspace/api-server run dev

# بنافذة ترمنال تانية — تشغيل الفرونت اند
pnpm --filter @workspace/rine-beauty exec vite
```

## أهم القرارات التقنية بعد الترحيل عن Replit

- **تخزين الصور**: Cloudflare R2 بدل Replit Object Storage (نفس الواجهة
  الداخلية، طلب واحد فقط للتحميل بدل عدة طلبات HEAD/GET متكررة).
- **تسجيل الدخول**: Google OAuth بدل Replit Auth. المشرف الأساسي ثابت
  بالكود (`artifacts/api-server/src/lib/adminAuth.ts`) وهو الوحيد يلي يقدر
  يضيف/يحذف مشرفين آخرين (يتخزنوا بجدول `admins`).
- **الفرونت اند**: لوحة التحكم محمّلة بشكل lazy (code-split) — الزوار
  العاديين ما بيحمّلوا كود الإدارة إطلاقاً.
