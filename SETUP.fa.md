# راه‌اندازی پروژه DFI روی یک سیستم جدید

این سند مراحل راه‌اندازی workspace کامل DFI را پوشش می‌دهد:

```text
DFI/
├── dfi-api   بک‌اند ASP.NET Core
└── dfi-web   فرانت‌اند React
```

> خلاصه: فقط تعریف دو env مربوط به ابزار query کافی نیست. برای اجرای کامل پروژه باید SDKها،
> دسترسی شبکه، اتصال‌های runtime بک‌اند، مسیر فایل‌های PCH و آدرس API فرانت‌اند نیز آماده باشند.

## ۱. پیش‌نیازها

نسخه‌هایی که در محیط فعلی پروژه با آن‌ها build موفق انجام شده است:

```text
Git       2.55
.NET SDK  10.0.111
Node.js   22.14
npm       10.9
SQL Server
```

نسخه دقیق Git اهمیت زیادی ندارد، اما بک‌اند به `.NET SDK 10` نیاز دارد. برای فرانت‌اند ترجیحاً
نسخه Node را با محیط فعلی هماهنگ نگه دارید.

سیستم جدید باید از نظر شبکه به این دو مقصد دسترسی داشته باشد:

- دیتابیس DFI
- دیتابیس راهکاران

در صورت قرار داشتن سرورها داخل شبکه سازمان، اتصال VPN، DNS، route و پورت SQL Server را نیز بررسی کنید.

## ۲. دریافت source code

بک‌اند و فرانت‌اند دو مخزن Git مستقل هستند و باید جداگانه clone شوند:

```bash
mkdir -p DFI
cd DFI

git clone <DFI_API_REPOSITORY_URL> dfi-api
git clone <DFI_WEB_REPOSITORY_URL> dfi-web
```

فایل `AGENTS.md` ریشه workspace را نیز کنار این دو پوشه قرار دهید؛ این فایل دستورالعمل مشترک پروژه
برای Codex است.

## ۳. تفاوت اتصال‌های runtime و اتصال‌های read-only

چهار متغیر مستقل داریم:

| کاربرد | متغیر | سطح دسترسی پیشنهادی |
|---|---|---|
| اجرای API روی دیتابیس DFI | `connectionString` | مطابق نیاز واقعی برنامه |
| اجرای API روی راهکاران | `drajeConnectionString` | مطابق نیاز واقعی برنامه |
| query خواندنی Codex روی DFI | `DFI_DB_CONNECTION_STRING` | فقط `db_datareader` |
| query خواندنی Codex روی راهکاران | `RAHKARAN_DB_CONNECTION_STRING` | فقط `db_datareader` |

متغیرهای ابزار read-only جایگزین اتصال‌های runtime برنامه نیستند. بهتر است برای ابزار query دو login
مجزا بسازید که فقط عضو `db_datareader` باشند.

## ۴. تعریف متغیرهای محیطی

در zsh فایل زیر را ایجاد یا ویرایش کنید:

```bash
nano ~/.zshenv
```

نمونه تنظیمات:

```bash
# اتصال‌های runtime بک‌اند
export connectionString='Server=DFI_SERVER;Database=DFI_DATABASE;User Id=APP_USER;Password=PASSWORD;Encrypt=True;TrustServerCertificate=True;'
export drajeConnectionString='Server=RAHKARAN_SERVER;Database=RAHKARAN_DATABASE;User Id=APP_USER;Password=PASSWORD;Encrypt=True;TrustServerCertificate=True;'

# اتصال‌های صرفاً خواندنی برای Codex و ابزار dfi-db
export DFI_DB_CONNECTION_STRING='Server=DFI_SERVER;Database=DFI_DATABASE;User Id=READ_ONLY_USER;Password=PASSWORD;Encrypt=True;TrustServerCertificate=True;'
export RAHKARAN_DB_CONNECTION_STRING='Server=RAHKARAN_SERVER;Database=RAHKARAN_DATABASE;User Id=READ_ONLY_USER;Password=PASSWORD;Encrypt=True;TrustServerCertificate=True;'

# تنظیمات امنیتی API؛ مقدار واقعی و قوی انتخاب شود
export AppSettings__Secret='A_LONG_RANDOM_SECRET'
export AppSettings__Issuer='Test.com'

# مسیر ذخیره فایل‌های PCH
export PchFiles__ImagesPath='/absolute/path/to/dfi-data/pch/images'
export PchFiles__PdfsPath='/absolute/path/to/dfi-data/pch/pdfs'

# کاربر تست محدود برای تست endpointهای محافظت‌شده توسط Codex
export DFI_TEST_USERNAME='codex-test'
export DFI_TEST_PASSWORD='A_STRONG_TEST_PASSWORD'
```

سطح دسترسی فایل را محدود و تنظیمات را بارگذاری کنید:

```bash
chmod 600 ~/.zshenv
source ~/.zshenv
```

هیچ connection string یا رمز واقعی را داخل Git، فایل `AGENTS.md`، پیام چت یا command history قرار ندهید.

کاربر `DFI_TEST_USERNAME` باید یک حساب اختصاصی و محدود باشد. فقط roleهای لازم برای تست‌های موردنظر
را به آن بدهید؛ برای محاسبات بهای تمام‌شده معمولاً `Computing` و در صورت نیاز
`PreviousComputationalPeriod` کافی است. از حساب شخصی یا حساب دارای دسترسی گسترده `Main` استفاده نکنید.

## ۵. آماده‌سازی مسیر فایل‌های PCH

مسیرهای تعریف‌شده در env باید موجود و برای کاربر اجراکننده API قابل نوشتن باشند:

```bash
mkdir -p /absolute/path/to/dfi-data/pch/images
mkdir -p /absolute/path/to/dfi-data/pch/pdfs
```

در محیط production ویندوز می‌توان مسیرهای مناسب ویندوزی تعریف کرد. فایل‌های موجود PCH به‌صورت خودکار
از سیستم قبلی منتقل نمی‌شوند؛ در صورت نیاز باید محتوای image و PDF جداگانه migration یا copy شود.

## ۶. آماده‌سازی و اجرای بک‌اند

```bash
cd DFI/dfi-api

dotnet restore Draje.csproj
dotnet build Draje.csproj --no-restore

ASPNETCORE_ENVIRONMENT=Development \
dotnet run --project Draje.csproj --urls 'http://0.0.0.0:5000'
```

تعریف env فقط اتصال را فراهم می‌کند؛ دیتابیس مقصد باید schema و داده‌های لازم را نیز داشته باشد.
Migrationها را بدون بررسی و backup روی دیتابیس production اجرا نکنید.

## ۷. آماده‌سازی و اجرای فرانت‌اند

فرانت‌اند آدرس‌های runtime را از متغیرهای React می‌خواند. فایل `.env.development` موجود برای اجرای
local به‌صورت پیش‌فرض از این آدرس‌ها استفاده می‌کند:

```text
REACT_APP_API_URL=http://localhost:5000/api/
REACT_APP_APP_URL=http://localhost:3000
```

برای نصب و اجرا:

```bash
cd DFI/dfi-web

npm ci
npm start
```

برای مقصد متفاوت، `REACT_APP_API_URL` و `REACT_APP_APP_URL` را متناسب با محیط تنظیم کنید. مقدار
`REACT_APP_API_URL` باید به `/api/` ختم شود.

## ۸. تست API محافظت‌شده توسط Codex

Codex می‌تواند با متغیرهای `DFI_TEST_USERNAME` و `DFI_TEST_PASSWORD` از endpoint زیر JWT دریافت کند:

```text
POST http://localhost:5000/api/Login/Login/authenticate
```

توکن باید فقط موقتاً در `/tmp` نگهداری و با header زیر ارسال شود:

```text
Authorization: Bearer <token>
```

نام کاربری، رمز و JWT نباید در خروجی ترمینال، log، چت یا Git نمایش داده شوند. پس از تغییر متغیرهای
محیطی، Codex و terminal را restart کنید تا sessionهای جدید آن‌ها را دریافت کنند.

برای بازکردن یا دسترسی به پورت محلی، sandbox ممکن است approval نمایش دهد. `AGENTS.md` به Codex اجازه
می‌دهد اجرای سرویس مرتبط با کار را امتحان کند، اما نمی‌تواند سازوکار امنیتی approval را دور بزند.
در اولین درخواست اجرای backend یا frontend، approval محدود زیر را در صورت تمایل تأیید و persist کنید:

```text
dotnet run --no-build --project Draje.csproj --launch-profile Draje
npm start
```

بعد از هر تغییر در کد بک‌اند، پیش از اجرای فرمان دارای `--no-build` باید
`dotnet build Draje.csproj --no-restore` با موفقیت اجرا شده باشد؛ در غیر این صورت ممکن است نسخه قدیمی
API اجرا شود. اگر وابستگی‌های فرانت‌اند نصب نیستند، قبل از `npm start` فرمان `npm ci` را اجرا کنید.

قبل از اجرای سرویس جدید باید پورت‌های `5000`، `5001` و `3000` بررسی شوند؛ اگر سرویس مناسب در حال اجراست
از همان استفاده می‌شود.

## ۹. آماده‌سازی ابزار query خواندنی Codex

ابزار داخل مخزن بک‌اند قرار دارد و به نصب `sqlcmd` نیاز ندارد:

```bash
cd DFI/dfi-api

dotnet restore Tools/DfiDbQuery/DfiDbQuery.csproj
dotnet build Tools/DfiDbQuery/DfiDbQuery.csproj --no-restore
```

بررسی اتصال هر دو دیتابیس:

```bash
./Tools/DfiDbQuery/dfi-db --database dfi --check
./Tools/DfiDbQuery/dfi-db --database rahkaran --check
```

اجرای query:

```bash
./Tools/DfiDbQuery/dfi-db --database dfi --file query.sql
./Tools/DfiDbQuery/dfi-db --database rahkaran --file query.sql
```

ابزار فقط `SELECT` و CTE را می‌پذیرد، اما تضمین اصلی امنیت همان login دیتابیس با دسترسی صرفاً
خواندنی است.

## ۱۰. چک‌لیست نهایی

- [ ] هر دو مخزن در مسیر درست clone شده‌اند.
- [ ] فایل `AGENTS.md` در ریشه workspace قرار دارد.
- [ ] `.NET SDK 10` نصب است.
- [ ] Node.js و npm نصب هستند.
- [ ] دسترسی شبکه به SQL Serverهای DFI و راهکاران برقرار است.
- [ ] اتصال‌های runtime بک‌اند تنظیم شده‌اند.
- [ ] اتصال‌های read-only ابزار query تنظیم شده‌اند.
- [ ] secret مربوط به JWT/API مقدار امن دارد.
- [ ] کاربر تست محدود Codex و متغیرهای `DFI_TEST_USERNAME` و `DFI_TEST_PASSWORD` تنظیم شده‌اند.
- [ ] مسیرهای image و PDF مربوط به PCH موجود و قابل نوشتن هستند.
- [ ] `dotnet build Draje.csproj` موفق است.
- [ ] اتصال هر دو profile ابزار `dfi-db` موفق است.
- [ ] `npm ci` و `npm start` موفق هستند.
- [ ] فرانت‌اند می‌تواند API را از آدرس تنظیم‌شده فراخوانی کند.

## ۱۱. عیب‌یابی سریع

### متغیر محیطی پیدا نمی‌شود

پس از ویرایش `~/.zshenv` اجرا کنید:

```bash
source ~/.zshenv
```

سپس terminal، IDE و Codex را کاملاً بسته و دوباره باز کنید.

### خطای network یا SQL Server error 10013

موارد زیر را بررسی کنید:

- VPN یا شبکه داخلی سازمان
- IP یا hostname سرور
- پورت SQL Server
- firewall سیستم و سرور
- فعال بودن remote connection
- دسترسی sandbox؛ اجرای query توسط Codex ممکن است به تأیید network نیاز داشته باشد

### API اجرا می‌شود ولی فرانت‌اند وصل نیست

- مقدار `REACT_APP_API_URL` را بررسی کنید.
- وجود `/api/` در انتهای URL را بررسی کنید.
- CORS، firewall و پورت API را کنترل کنید.
- پس از تغییر env فرانت‌اند، dev server را restart کنید.

### تصویر یا PDF ذخیره نمی‌شود

- مقدار `PchFiles__ImagesPath` و `PchFiles__PdfsPath` را بررسی کنید.
- وجود directoryها و write permission کاربر API را کنترل کنید.
- فضای آزاد دیسک را بررسی کنید.
