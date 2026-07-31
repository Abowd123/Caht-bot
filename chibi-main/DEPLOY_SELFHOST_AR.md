# تنصيب بوت Chibi على سيرفرك الخاص (VPS)

يدعم هذا الدليل أي سيرفر لينكس (Ubuntu/Debian/CentOS...) سواء كان VPS من Hetzner, DigitalOcean, Contabo, AWS EC2... إلخ.

**ملاحظة مهمة:** بما إنك عدّلت البوت ليدعم الأزرار العربية، لازم تبني الصورة (Docker Image) من الكود المصدري بنفسك، وليس من الصورة الجاهزة `pysergio/chibi` على Docker Hub (لأنها بدون التعديلات).

---

## 1) المتطلبات على السيرفر
- سيرفر لينكس بذاكرة 1GB على الأقل (يفضل 2GB لأن فيه تحميل نموذج embeddings)
- اتصال SSH بالسيرفر

## 2) تثبيت Docker و Docker Compose
```bash
curl -fsSL https://get.docker.com | sh
sudo systemctl enable --now docker
```
تحقق من التثبيت:
```bash
docker --version
docker compose version
```

## 3) رفع المشروع إلى السيرفر
من جهازك (بعد فك ضغط الملف اللي أرسلته لك):
```bash
scp -r chibi-main root@YOUR_SERVER_IP:/root/chibi-bot
```
أو ارفعه عبر Git إذا كان موجود على GitHub:
```bash
ssh root@YOUR_SERVER_IP
git clone https://github.com/USERNAME/REPO_NAME.git /root/chibi-bot
```

## 4) إعداد المتغيرات
```bash
ssh root@YOUR_SERVER_IP
cd /root/chibi-bot
cp .env.example .env
nano .env
```
عبّي على الأقل:
```
TELEGRAM_BOT_TOKEN=توكنك_من_BotFather
```
واترك مفاتيح الذكاء الاصطناعي فاضية إذا رح تخلي كل مستخدم يدخل مفتاحه بنفسه عبر زر "🔑 تعيين مفتاح API" (لازم تفعّل `PUBLIC_MODE=true` بهذي الحالة).

## 5) بناء وتشغيل البوت
```bash
cd /root/chibi-bot
docker compose -f docker-compose.selfhost.yml up -d --build
```
هذا يبني الصورة من الكود المعدّل (يأخذ عدة دقائق أول مرة) ويشغّل البوت في الخلفية باستمرار.

## 6) التحقق من التشغيل
```bash
docker compose -f docker-compose.selfhost.yml logs -f
```
افتح تلغرام وأرسل `/start` للبوت — لازم تظهر لك لوحة الأزرار العربية.

## 7) أوامر إدارة مفيدة
| الأمر | الوظيفة |
|---|---|
| `docker compose -f docker-compose.selfhost.yml ps` | حالة الحاوية |
| `docker compose -f docker-compose.selfhost.yml stop` | إيقاف البوت |
| `docker compose -f docker-compose.selfhost.yml start` | تشغيله مجددًا |
| `docker compose -f docker-compose.selfhost.yml down` | إيقاف وحذف الحاوية (البيانات تبقى بالـ volume) |
| `docker compose -f docker-compose.selfhost.yml up -d --build` | إعادة البناء بعد أي تعديل بالكود |

## 8) تحديث البوت لاحقًا
```bash
cd /root/chibi-bot
git pull   # إذا كنت تستخدم Git
docker compose -f docker-compose.selfhost.yml up -d --build
```

## 9) تشغيله تلقائيًا بعد إعادة تشغيل السيرفر
`restart: unless-stopped` في ملف الـ compose يكفي لإعادة تشغيل الحاوية تلقائيًا بعد أي reboot طالما خدمة Docker نفسها مفعّلة (`systemctl enable docker` سويناها بالخطوة 2).

## 10) بيانات المستخدمين ومفاتيحهم
البيانات (المحادثات، مفاتيح API لكل مستخدم إن فعّلت الوضع العام) تُخزّن داخل الـ volume `chibi_data` المربوط بـ `/app/data`. لا تحذف هذا الـ volume إلا إذا كنت تريد مسح كل شيء.

---

## طريقة بديلة بدون Docker (تثبيت مباشر عبر pip)
إذا ما تحب تستخدم Docker:
```bash
sudo apt update && sudo apt install -y python3-pip python3-venv
python3 -m venv chibi-env
source chibi-env/bin/activate
pip install chibi-bot
chibi config
chibi start
```
لكن بهذي الطريقة تعديلات الأزرار العربية اللي سويناها بالكود ما راح تنعكس، لأن `pip install chibi-bot` يجيب النسخة الرسمية من PyPI. لتطبيق تعديلاتك، لازم تشغّل من الكود المصدري مباشرة:
```bash
cd chibi-main
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
cp .env.example .env && nano .env
python3 main.py
```
لتشغيله بشكل دائم في الخلفية (يبقى شغال بعد إغلاق SSH)، استخدم `systemd` أو `screen`/`tmux`، أو الأفضل استخدم طريقة Docker بالأعلى لأنها أكثر ثباتًا.
