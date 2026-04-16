# نشر الموقع الثاني على نفس الـ VPS

هذا الدليل يشغّل نسخة `Two-main` كموقع مستقل على نفس الـ VPS باستخدام إعداد `site-3`.

الملف الجاهز للبيئة موجود هنا:

- `deploy/vps/sites/site-3/site.env`

اسمَي العمليات المتوقعين في `pm2`:

- `habib-site3-app`
- `habib-site3-worker`

## 1. الدخول إلى الـ VPS

من جهازك المحلي:

```bash
ssh root@YOUR_VPS_IP
```

## 2. الذهاب إلى مجلد المشروع على الخادم

إذا كان المشروع الثاني مرفوعًا في هذا المسار مثلًا:

```bash
cd /var/www/habib/two-main
```

إذا لم تكن رفعته بعد، ارفعه أولًا أو اسحبه من GitHub:

```bash
mkdir -p /var/www/habib
cd /var/www/habib
git clone <REPO_URL> two-main
cd /var/www/habib/two-main
```

## 3. تثبيت الحزم وبناء المشروع

```bash
pnpm install
pnpm build
chmod +x ./scripts/vps/run-site.sh
```

## 4. التأكد من ملف البيئة

لازم يكون هذا الملف موجودًا على الخادم:

```bash
/var/www/habib/two-main/deploy/vps/sites/site-3/site.env
```

وتقدر تتأكد بسرعة:

```bash
cat /var/www/habib/two-main/deploy/vps/sites/site-3/site.env
```

## 5. تشغيل التطبيق والواتساب على نفس الـ VPS

إذا تريد تشغيل التطبيق نفسه + عامل الواتساب:

```bash
cd /var/www/habib/two-main
pm2 start ./scripts/vps/run-site.sh --name habib-site3-app --interpreter /bin/bash -- app ./deploy/vps/sites/site-3/site.env
pm2 start ./scripts/vps/run-site.sh --name habib-site3-worker --interpreter /bin/bash -- worker ./deploy/vps/sites/site-3/site.env
pm2 save
pm2 startup
```

إذا كان التطبيق على Vercel وتريد فقط عامل الواتساب على الـ VPS:

```bash
cd /var/www/habib/two-main
pm2 start ./scripts/vps/run-site.sh --name habib-site3-worker --interpreter /bin/bash -- worker ./deploy/vps/sites/site-3/site.env
pm2 save
pm2 startup
```

## 6. التحقق من التشغيل

```bash
pm2 status
pm2 logs habib-site3-app --lines 100
pm2 logs habib-site3-worker --lines 100
```

إذا كان التطبيق شغالًا على نفس الـ VPS، افحص الحالة محليًا:

```bash
curl http://127.0.0.1:3003/api/whatsapp/status
```

## 7. ربط جلسة واتساب الجديدة

بعد تشغيل `worker` افتح QR من المتصفح على الموقع الثاني، ثم امسحه من الجوال الخاص بالحساب الجديد.

إذا كان التطبيق على نفس الـ VPS عبر nginx أو IP مباشر، سيكون المسار عادة:

```text
http://YOUR_DOMAIN_OR_IP/api/whatsapp/qr
```

## 8. أوامر الصيانة

إعادة تشغيل:

```bash
pm2 restart habib-site3-app
pm2 restart habib-site3-worker
```

إيقاف:

```bash
pm2 stop habib-site3-app
pm2 stop habib-site3-worker
```

حذف وإعادة إنشاء العملية إذا احتجت:

```bash
pm2 delete habib-site3-app
pm2 delete habib-site3-worker
```

## 9. ملاحظات مهمة

- هذا الإعداد مصمم ليعمل على نفس الـ VPS الأول بدون شراء VPS جديد.
- هذه النسخة معزولة عن الأولى لأن لها `Supabase` مختلف و `WHATSAPP_INSTANCE_SLUG=two-main` ومسارات واتساب مستقلة داخل `/var/www/habib/shared/two-main/`.
- لو كان عندك مشروع أول يعمل على نفس الـ VPS، لا تستخدم نفس أسماء عمليات `pm2` ولا نفس ملفات جلسة الواتساب.