# سرعة

MVP لتوصيل وتنقل داخل المدن، مصمم للسوق السوداني والاتصال الضعيف.

## ما يعمل الآن

- Customer / Driver / Admin flows منفصلة.
- Demo Mode كامل يعمل بدون Supabase: تسجيل دخول للأدوار الثلاثة، إنشاء طلب، تعيين سائق، قبول، انتقال حالات، إكمال، تقييم، سجل، إشعارات، إدارة السائقين والعملاء والطلبات والتسعير.
- Production Mode يستخدم Supabase Auth + PostgreSQL + RLS + Realtime.
- Leaflet + OpenStreetMap.
- GPS بمعدل محدود، مع حدود PWA للـBackground Location.
- Cash payments في النسخة الأولى، مع schema قابل لإضافة وسائل أخرى.
- PWA وCapacitor جاهزان.

## التشغيل السريع Demo

1. `npm install`
2. انسخ `.env.example` إلى `.env.local`
3. ضع `VITE_DEMO_MODE=true`
4. `npm run dev`
5. من شاشة الدخول اختر عميل أو سائق أو إدارة.
6. رمز OTP في Demo هو `123456`.

أرقام Demo:
- Customer: `0911000001`
- Drivers: `0911000101` إلى `0911000110`
- Admin: `0911000999`

## Production / Supabase

ضع: `VITE_SUPABASE_URL` و `VITE_SUPABASE_ANON_KEY` فقط في Frontend. لا تضع Service Role Key داخل أي `VITE_` variable.

شغّل migrations بالترتيب:

- `supabase/migrations/0001_initial.sql`
- `supabase/migrations/0002_security_hardening.sql`
- `supabase/migrations/0003_rating_security.sql`

بعدها فعّل Phone Auth وSMS Provider من إعدادات Supabase.

## Seed

`npm run seed` يحتاج متغيرات server-only، وليس VITE_: `SUPABASE_SEED_URL` و `SUPABASE_SERVICE_ROLE_KEY`.

## Build

`npm run build`

## PWA

VitePWA ينشئ service worker وmanifest أثناء build.

## Android

```bash
npm run build
npx cap add android
npx cap sync
npx cap open android
```

اسم التطبيق: سرعة

## الخدمات الخارجية

- SMS: مطلوب لـOTP الحقيقي.
- Routing: الخريطة تعمل، بينما المسار الافتراضي في MVP هو خط بين النقطتين؛ يمكن إضافة OSRM/Valhalla/GraphHopper دون تغيير Order model.
- Background GPS الحقيقي: يحتاج Capacitor Native Location عند تشغيل التطبيق في الخلفية.
- Push Notifications: In-App موجودة؛ FCM يمكن إضافته لاحقًا.
- Online payments: غير مفعلة؛ Cash فقط.

## Security

- RLS هو خط الحماية الأساسي.
- Role محمي من التغيير الذاتي.
- انتقالات الطلبات تتحقق داخل PostgreSQL RPC.
- تقييمات الإنتاج تمر عبر RPC بعد التحقق من ملكية الطلب وإكماله.
- لا توجد مفاتيح Service Role في Frontend.

## Known limitations before production

1. إعداد SMS فعلي.
2. Routing provider.
3. Background GPS على Android.
4. Push notifications.
5. Payment provider.
6. مراقبة وأخطاء وتحليلات وE2E tests على أجهزة فعلية.
7. مراجعة قانونية وتشغيلية قبل الإطلاق التجاري.

## Build APK with GitHub Actions

The project includes `.github/workflows/android-apk.yml`. Upload the project to a GitHub repository, open **Actions**, choose **Build سرعة APK**, then select **Run workflow**. The workflow installs Node.js, Java, the Android SDK, npm dependencies, creates the Capacitor Android project, synchronizes it, runs Gradle, verifies `app-debug.apk`, and uploads the APK as an Actions artifact. GitHub-hosted runners provide the Android build environment; Gradle's official GitHub Actions integration supports Gradle setup and dependency caching.

### Exact GitHub steps

1. Create a new GitHub repository.
2. Upload all files from this project to the repository root.
3. Commit to `main`.
4. Open **Actions → Build سرعة APK**.
5. Click **Run workflow**.
6. Wait for the green successful run.
7. Open the completed run and download the artifact named `sar3a-debug-apk`.
8. Extract it and install `app-debug.apk` on Android.

The workflow uses `VITE_DEMO_MODE=true`, so the first APK is a self-contained demo build and does not require Supabase credentials.

For production, replace Demo Mode with the real Supabase environment and later add release signing. Android documents `assembleDebug` as the standard task for generating a debug APK suitable for testing and direct installation.
