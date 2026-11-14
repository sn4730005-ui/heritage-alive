# 🚀 نشر «تراثنا الحي» على رابط دائم

المشروع جاهز للنشر بالكامل (بدون أي حزم خارجية). اختر الطريق الأنسب لك:

| الطريق | الرابط الناتج | التكلفة | مناسب إذا |
|---|---|---|---|
| **أ) Render** | `https://heritage-alive.onrender.com` | مجاني | تريد رابطًا دائمًا بسرعة ومن غير بطاقة بنكية |
| **ب) Railway** | `https://heritage-alive.up.railway.app` | مجاني بحدود / 5$ رصيد | تريد نشرًا أسرع مع سطر أوامر واحد |
| **ج) VPS (موصى به للاستمرارية)** | `https://your-domain.com` | ~4–6$ شهريًا | تريد تحكمًا كاملًا وقرصًا دائمًا وإمكانية ربط دومين |
| **د) Fly.io + Docker** | `https://heritage-alive.fly.dev` | مجاني بحدود | تريد قرصًا دائمًا (Volume) مدمجًا |

> ⚠️ مهم: على الخطط المجانية في (Render / Railway) نظام الملفات **مؤقت**؛ أي إعادة تشغيل قد تُرجع المحتوى إلى نسخة المشروع الأصلية. لذلك أضفت لك زر **«تصدير نسخة / استيراد نسخة»** في لوحة المالك ← الإعدادات، لحفظ كل تعديلاتك واستعادتها في ثانية. أمّا مع VPS أو Fly.io (Volume) أو قرص مدفوع على Render فالتعديلات **دائمة تلقائيًا**.

---

## أ) Render — أسرع رابط دائم مجاني

1. أنشئ مستودعًا على GitHub وارفع المشروع:
   ```bash
   cd heritage
   git init && git add . && git commit -m "تراثنا الحي"
   git branch -M main
   git remote add origin https://github.com/USERNAME/heritage-alive.git
   git push -u origin main
   ```
2. افتح [render.com](https://render.com) → **New +** → **Web Service** → اربط مستودع GitHub.
3. الإعدادات:
   - **Runtime:** Node
   - **Build Command:** `npm install --omit=dev`
   - **Start Command:** `node server.js`
   - **Health Check Path:** `/api/health`
   - **Environment:** أضف `ADMIN_PASSWORD` = كلمة مرور قوية للمالك
4. اضغط **Create Web Service** — ستحصل على رابط دائم مثل:
   `https://heritage-alive.onrender.com`
5. (اختياري) لجعل التعديلات دائمة: **Disks** → أضف قرصًا بحجم 1GB على المسار `/var/data`، ثم أضف متغيرات البيئة:
   `DATA_DIR=/var/data` و `UPLOAD_DIR=/var/data/uploads`
6. (اختياري) دومين خاص: **Settings → Custom Domain** → أضف نطاقك واتبع تعليمات DNS (CNAME).

أو استخدم ملف `render.yaml` الموجود بالمشروع عبر **Blueprint** لنشر تلقائي.

---

## ب) Railway — نشر بسطر أوامر

```bash
npm i -g @railway/cli
railway login
cd heritage
railway init
railway up
```
ثم من لوحة Railway: **Variables** → `ADMIN_PASSWORD=كلمة-مرور-قوية`، ومن **Settings → Networking** اضغط **Generate Domain**.

للحصول على تخزين دائم: **Volumes** → أضف Volume على `/var/data` وأضف المتغيرين `DATA_DIR=/var/data` و `UPLOAD_DIR=/var/data/uploads`.

---

## ج) VPS (Ubuntu) — رابط دائم + دومين + شهادة SSL مجانية

على الخادم (بعد توجيه الدومين إلى IP الخادم):

```bash
# 1) تثبيت Node
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -
sudo apt-get install -y nodejs nginx

# 2) رفع المشروع
sudo mkdir -p /var/www/heritage && sudo chown -R $USER:$USER /var/www/heritage
# انسخ ملفات المشروع إلى /var/www/heritage (scp أو git clone)

# 3) تشغيل الخدمة
cd /var/www/heritage
sudo tee /etc/systemd/system/heritage.service > /dev/null <<'SERVICE'
[Unit]
Description=Heritage Alive
After=network.target

[Service]
WorkingDirectory=/var/www/heritage
ExecStart=/usr/bin/node server.js
Environment=PORT=3000
Environment=ADMIN_PASSWORD=ضع-كلمة-مرور-قوية
Restart=always
User=www-data

[Install]
WantedBy=multi-user.target
SERVICE
sudo systemctl enable --now heritage
```

```bash
# 4) Nginx كوسيط عكسي
sudo tee /etc/nginx/sites-available/heritage > /dev/null <<'NGINX'
server {
  listen 80;
  server_name your-domain.com www.your-domain.com;
  client_max_body_size 100M;

  location / {
    proxy_pass http://127.0.0.1:3000;
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection 'upgrade';
    proxy_set_header Host $host;
    proxy_set_header X-Forwarded-Proto $scheme;
    proxy_cache_bypass $http_upgrade;
  }
}
NGINX
sudo ln -s /etc/nginx/sites-available/heritage /etc/nginx/sites-enabled/
sudo nginx -t && sudo systemctl reload nginx

# 5) شهادة SSL مجانية (Let's Encrypt)
sudo apt-get install -y certbot python3-certbot-nginx
sudo certbot --nginx -d your-domain.com -d www.your-domain.com
```

النتيجة: `https://your-domain.com` — والتعديلات تُحفظ دائمًا في `/var/www/heritage/data/db.json` والملفات في `public/uploads/`.

---

## د) Fly.io (Docker + قرص دائم)

```bash
curl -L https://fly.io/install.sh | sh
fly auth signup          # أو fly auth login
cd heritage
fly launch --copy-config --name heritage-alive
fly volumes create heritage_data --size 1
fly secrets set ADMIN_PASSWORD=كلمة-مرور-قوية
fly deploy
```
ملف `fly.toml` الجاهز يضبط القرص على `/var/data` والمتغيرين `DATA_DIR` و `UPLOAD_DIR` تلقائيًا.

---

## النسخ الاحتياطي والاستعادة

- **تصدير:** لوحة المالك ← الإعدادات ← «⬇ تصدير نسخة» → ينزّل ملف `heritage-backup-YYYY-MM-DD.json`.
- **استعادة:** «⬆ استيراد نسخة» → اختر الملف → تُحدَّث الشخصيات والرئاسة والتعليم والفيديوهات فورًا.
- نسخة يدوية للخادم: `cp data/db.json backup.json && tar czf uploads.tgz public/uploads`

## قائمة تحقق قبل النشر

- [ ] تغيير كلمة مرور المالك (`ADMIN_PASSWORD` أو من لوحة المالك).
- [ ] التأكد من أن `/api/health` يرجع `{"ok":true}`.
- [ ] رفع حجم الطلبات في Nginx إلى 100M إذا كان المالك سيرفع فيديوهات.
- [ ] ضبط `DATA_DIR` و`UPLOAD_DIR` على قرص دائم (إن أردت تعديلات دائمة).
- [ ] أخذ نسخة احتياطية بعد إدخال المحتوى النهائي.
