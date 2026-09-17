# Cardboard Management — Backend Workspace

هذا هو المستودع المركزي لتشغيل Backend مشروع **Cardboard Management**. لا يحتاج المطوّر إلى الدخول إلى مشاريع Frappe أو ERPNext الثلاثة يدويًا؛ كل ما يلزم موجود تحت مجلد `apps/` ومثبت كـ Git submodule للحفاظ على تاريخ كل مشروع مستقلًا.

## مكونات النظام

| المسار | الوظيفة | المصدر |
|---|---|---|
| `apps/frappe` | Frappe Framework v15 | `frappe/frappe` — branch `version-15` |
| `apps/erpnext` | ERPNext v15 | `frappe/erpnext` — branch `version-15` |
| `apps/cardboard_management` | تطبيق Cardboard Management | `muhhammedeid/cardboard-management` — branch `MVP` |
| `sites/` و`config/` و`Procfile` | إعداد Bench المحلي | خاص بكل جهاز |

المستودع لا يحتوي على كلمات مرور أو API keys أو قواعد بيانات. هذه القيم تُنشأ محليًا على كل جهاز.

---

## المتطلبات

على Windows 10/11 يلزم تثبيت:

- Windows Subsystem for Linux 2 (WSL2)
- Ubuntu 24.04 داخل WSL
- Git داخل Ubuntu
- Python من الإصدار `3.10` إلى أقل من `3.15`
- Node.js `18` أو أحدث
- MariaDB/MySQL
- Redis
- حساب GitHub لديه صلاحية قراءة مستودع `cardboard-management` الخاص إن ظل Private

### تثبيت WSL وUbuntu من PowerShell كمسؤول

```powershell
wsl --install -d Ubuntu-24.04
```

أعد تشغيل Windows إذا طلب النظام ذلك، ثم افتح Ubuntu لأول مرة وأنشئ اسم المستخدم وكلمة المرور الخاصة بـ Linux.

> لا تشغّل أوامر `sudo` التالية من PowerShell؛ شغّلها داخل Ubuntu.

---

## 1. تثبيت متطلبات Ubuntu

```bash
sudo apt update
sudo apt install -y \
  git curl build-essential pkg-config \
  python3 python3-dev python3-venv python3-pip \
  mariadb-server mariadb-client libmariadb-dev \
  redis-server libffi-dev libssl-dev libjpeg-dev zlib1g-dev \
  xvfb libfontconfig
```

ابدأ الخدمات:

```bash
sudo service mariadb start
sudo service redis-server start
```

اختبرها:

```bash
mysqladmin ping
redis-cli ping
```

يجب أن تكون الاستجابة:

```text
mysqld is alive
PONG
```

لضبط كلمة مرور مستخدم قاعدة البيانات عند الحاجة:

```bash
sudo mysql_secure_installation
```

---

## 2. تثبيت Node.js وBench

ثبت NVM ثم Node.js. يفضل استخدام Node 20 LTS لهذا المشروع:

```bash
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.3/install.sh | bash
source ~/.nvm/nvm.sh
nvm install 20
nvm alias default 20
nvm use 20
node --version
npm --version
```

ثبّت Bench CLI:

```bash
python3 -m pip install --user frappe-bench
export PATH="$HOME/.local/bin:$PATH"
bench --version
```

إذا أردت بقاء PATH بعد إغلاق Ubuntu:

```bash
echo 'export PATH="$HOME/.local/bin:$PATH"' >> ~/.bashrc
source ~/.bashrc
```

---

## 3. استنساخ الـ Backend كاملًا

```bash
mkdir -p ~/frappe
cd ~/frappe
git clone --recurse-submodules https://github.com/muhhammedeid/cardboard-backend.git cardboard-bench
cd ~/frappe/cardboard-bench
```

إذا كان المستودع مستنسخًا بدون submodules:

```bash
git submodule update --init --recursive
```

> لأن `apps/cardboard_management` مستودع خاص، يجب أن يكون GitHub CLI أو Git Credential Manager مسجلًا بحساب لديه صلاحية الوصول إليه.

تأكد من وجود التطبيقات:

```bash
bench version
cat sites/apps.txt
```

يجب أن يظهر في `sites/apps.txt`:

```text
frappe
cardboard_management
erpnext
```

---

## 4. تجهيز Python وJavaScript dependencies

من جذر المشروع:

```bash
cd ~/frappe/cardboard-bench
bench setup requirements --python
bench setup requirements --node
bench build
```

إذا لم يكن مجلد البيئة موجودًا، ينشئه Bench أثناء تجهيز المتطلبات. لا تنقل مجلد `env` أو `node_modules` إلى Git؛ هذه ملفات محلية ويعاد إنشاؤها على الجهاز الجديد.

---

## 5. إنشاء Site محلي جديد

أنشئ Site باسم المشروع:

```bash
cd ~/frappe/cardboard-bench
bench new-site cardboard.localhost
```

سيطلب الأمر كلمة مرور مستخدم قاعدة البيانات ثم كلمة مرور Administrator الخاصة بالـ Site.

ثبت التطبيقات بالترتيب:

```bash
bench --site cardboard.localhost install-app erpnext
bench --site cardboard.localhost install-app cardboard_management
bench use cardboard.localhost
```

تحقق من التطبيقات المثبتة:

```bash
bench --site cardboard.localhost list-apps
```

يجب أن يظهر `frappe` و`erpnext` و`cardboard_management`.

### إعدادات محلية اختيارية

لا تضع الأسرار داخل Git. إذا احتجت إعدادات خاصة بالـ Site، عدّل الملف محليًا فقط:

```bash
nano sites/cardboard.localhost/site_config.json
```

أي قيم مثل `db_password` و`encryption_key` و`openrouter_api_key` و`hermes_api_key` تخص الجهاز وصاحب الحساب، وليست جزءًا من المستودع.

---

## 6. تشغيل الـ Backend

من جذر المشروع:

```bash
cd ~/frappe/cardboard-bench
bench start
```

الخدمات الأساسية:

- Backend / API: `http://cardboard.localhost:8000`
- Socket.IO: المنفذ `9000`
- Redis cache: المنفذ `13000`
- Redis queue: المنفذ `11000`

أوقف التشغيل من نفس الطرفية باستخدام `Ctrl+C`.

---

## 7. تشغيل الواجهة Nova مع الـ Backend — اختياري

الواجهة مشروع منفصل حتى يمكن تطويرها ونشرها بشكل مستقل:

```bash
cd ~
git clone https://github.com/muhhammedeid/cardboard-frontend-nova.git
cd cardboard-frontend-nova
npm install
cp .env.example .env
```

ثم شغّلها في طرفية ثانية:

```bash
npm run dev
```

افتح الواجهة باستخدام اسم الـ Site وليس `localhost`:

```text
http://cardboard.localhost:5173
```

هذا مهم لأن Frappe يحدد الـ Site والـ session من `Host` header.

---

## تشغيل النظام كله بأمر واحد

بعد استنساخ الواجهة بجانب الـ Backend في المسار الافتراضي `~/cardboard-frontend-nova`، شغّل من جذر الـ Backend:

```bash
cd ~/frappe/cardboard-bench
bash scripts/cardboard-start
```

السكريبت:

1. يشغل Nova على المنفذ `5173` إذا لم تكن تعمل.
2. يشغل Bench Backend في نفس الطرفية.
3. يترك Backend في المقدمة حتى يظهر أي خطأ مباشرة.
4. يوقف Backend عند `Ctrl+C`.

إذا كانت الواجهة في مسار مختلف:

```bash
CARDBOARD_FRONTEND_DIR=/path/to/cardboard-frontend-nova bash scripts/cardboard-start
```

العناوين بعد التشغيل:

```text
Frontend: http://cardboard.localhost:5173
Backend:  http://cardboard.localhost:8000
```

---

## التحديث لاحقًا

لتحديث المستودع المركزي والتطبيقات المضمنة:

```bash
cd ~/frappe/cardboard-bench
git pull --ff-only
git submodule update --init --recursive
bench setup requirements --python
bench setup requirements --node
bench migrate
bench build
```

لتحديث مشروع فرعي بشكل مستقل:

```bash
cd apps/cardboard_management
git status
git pull --ff-only
```

ثم عد إلى جذر Bench وشغّل:

```bash
cd ~/frappe/cardboard-bench
bench migrate
bench build
```

---

## استكشاف الأخطاء

### `bench: command not found`

```bash
export PATH="$HOME/.local/bin:$PATH"
source ~/.bashrc
```

### `Could not connect to Redis`

```bash
sudo service redis-server start
redis-cli ping
```

### `Can't connect to local MySQL server`

```bash
sudo service mariadb start
mysqladmin ping
```

### الواجهة تظهر لكن API يرجع 403 أو Site غير معروف

افتح الواجهة بهذا العنوان:

```text
http://cardboard.localhost:5173
```

ولا تستخدم:

```text
http://localhost:5173
```

### الـ submodule الخاص لا يُستنسخ

تحقق من تسجيل GitHub ومن صلاحية الحساب:

```bash
git ls-remote https://github.com/muhhammedeid/cardboard-management.git
```

---

## هيكل التشغيل المختصر

```text
Windows
└── WSL2 / Ubuntu 24.04
    └── ~/frappe/cardboard-bench
        ├── apps/frappe
        ├── apps/erpnext
        ├── apps/cardboard_management
        ├── sites
        ├── config
        ├── Procfile
        └── scripts/cardboard-start
```

المستودع المركزي هو نقطة الدخول الوحيدة المطلوبة لتجهيز الـ Backend؛ المشاريع الثلاثة داخل `apps/` يتم تنزيلها تلقائيًا كـ submodules مع الحفاظ على تاريخها المستقل.
