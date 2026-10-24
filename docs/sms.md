# ماژول ارسال پیامک کاوه‌نگار

این سند مرجع فنی و عملیاتی قابلیت «ارسال پیامک» در DFI است. هر تغییری در قرارداد API،
تنظیمات، دسترسی، محدودیت‌ها، رابط کاربری یا نحوهٔ ارتباط با کاوه‌نگار باید هم‌زمان در این فایل
ثبت شود.

آخرین به‌روزرسانی: ۱۴۰۵/۰۸/۰۲ (۲۰۲۶-۱۰-۲۴)

## دامنهٔ فعلی

ماژول فعلی برای ارسال یک متن یکسان به یک یا چند شمارهٔ موبایل ایرانی طراحی شده است. ارسال فقط
از بک‌اند DFI انجام می‌شود و مرورگر هیچ‌وقت API Key کاوه‌نگار را دریافت نمی‌کند.

قابلیت‌های فعلی:

* ارسال یک پیام به حداکثر ۲۰۰ گیرنده در هر درخواست؛
* پذیرش شماره‌های فارسی، عربی و لاتین در قالب‌های متداول ایران؛
* حذف شماره‌های تکراری پیش از ارسال؛
* استفاده از خط پیش‌فرض حساب کاوه‌نگار یا فرستندهٔ تنظیم‌شده روی سرور؛
* نمایش شناسه، وضعیت اولیه، فرستنده، گیرنده، زمان و هزینهٔ اعلام‌شده برای هر پیام؛
* کنترل دسترسی مستقل با نقش `SmsSend`؛
* نمایش زمان ثبت ارسال در منطقهٔ زمانی ایران.

این نسخه موارد زیر را انجام **نمی‌دهد**:

* تاریخچه یا متن پیامک‌ها را در دیتابیس DFI ذخیره نمی‌کند؛
* وضعیت نهایی تحویل را از کاوه‌نگار استعلام نمی‌کند؛
* webhook دریافت وضعیت یا پیام ورودی ندارد؛
* دفترچه تلفن، زمان‌بندی ارسال و قالب‌های Lookup کاوه‌نگار ندارد؛
* پس از خطای شبکه، ارسال را خودکار تکرار نمی‌کند.

نتیجهٔ نمایش‌داده‌شده در صفحه، پاسخ اولیهٔ کاوه‌نگار است و با refresh صفحه از بین می‌رود. شناسه
و وضعیت اولیهٔ ارسال الزاماً به معنی تحویل نهایی پیامک به گوشی نیست. برای بررسی نهایی باید از پنل
کاوه‌نگار استفاده شود یا قابلیت استعلام وضعیت در توسعهٔ بعدی اضافه شود.

## منابع رسمی

پیاده‌سازی بر اساس منابع رسمی زیر است:

* [مستندات REST کاوه‌نگار](https://kavenegar.com/rest.html)
* [SDK رسمی .NET کاوه‌نگار](https://github.com/kavenegar/kavenegar-dotnet)

قرارداد خارجی مورد استفاده:

```text
POST https://api.kavenegar.com/v1/{API_KEY}/sms/send.json
Content-Type: application/x-www-form-urlencoded
```

فیلدهای ارسالی `receptor`، `message` و در صورت تنظیم، `sender` هستند. جدول کامل کدهای پاسخ و
وضعیت‌ها باید از مستندات رسمی خوانده شود؛ کدهای سرویس‌دهنده را داخل برنامه به متن ثابت و قابل
منسوخ‌شدن تبدیل نکنید.

## معماری و جریان داده

```text
کاربر دارای نقش SmsSend
        ↓
صفحه /Sms در dfi-web
        ↓
POST /api/sms/send با JWT
        ↓
SmsController: اعتبارسنجی و نرمال‌سازی
        ↓
ISmsSender / KavenegarSmsSender
        ↓
REST API کاوه‌نگار
        ↓
پاسخ اولیه و هزینه ← API ← جدول نتیجه در رابط کاربری
```

مسیرهای اصلی کد:

| بخش | فایل |
| --- | --- |
| endpoint و مجوز بک‌اند | [`dfi-api/Controllers/General/SmsController.cs`](../dfi-api/Controllers/General/SmsController.cs) |
| نام نقش | [`dfi-api/Controllers/General/SmsRoles.cs`](../dfi-api/Controllers/General/SmsRoles.cs) |
| قرارداد سرویس | [`dfi-api/Services/Sms/ISmsSender.cs`](../dfi-api/Services/Sms/ISmsSender.cs) |
| اتصال به کاوه‌نگار | [`dfi-api/Services/Sms/KavenegarSmsSender.cs`](../dfi-api/Services/Sms/KavenegarSmsSender.cs) |
| تنظیمات سرویس | [`dfi-api/Services/Sms/KavenegarOptions.cs`](../dfi-api/Services/Sms/KavenegarOptions.cs) |
| نرمال‌سازی شماره‌ها | [`dfi-api/Services/Sms/SmsRecipientNormalizer.cs`](../dfi-api/Services/Sms/SmsRecipientNormalizer.cs) |
| DTOهای API | [`dfi-api/Dtos/Sms/`](../dfi-api/Dtos/Sms/) |
| ثبت DI و HttpClient | [`dfi-api/Startup.cs`](../dfi-api/Startup.cs) |
| صفحهٔ ارسال | [`dfi-web/src/components/sms/SmsSendPage.jsx`](../dfi-web/src/components/sms/SmsSendPage.jsx) |
| سرویس HTTP فرانت‌اند | [`dfi-web/src/services/sms/smsService.js`](../dfi-web/src/services/sms/smsService.js) |
| نرمال‌سازی سمت رابط کاربری | [`dfi-web/src/components/sms/smsRecipients.js`](../dfi-web/src/components/sms/smsRecipients.js) |
| منو و route | [`dfi-web/src/components/general/menuConfig.js`](../dfi-web/src/components/general/menuConfig.js) و [`dfi-web/src/App.jsx`](../dfi-web/src/App.jsx) |

## تنظیمات سرور

بخش تنظیمات برنامه `Kavenegar` نام دارد و از قرارداد استاندارد ASP.NET Core برای environment
variable استفاده می‌کند:

| کلید | الزامی | توضیح |
| --- | --- | --- |
| `Kavenegar__ApiKey` | بله | API Key حساب کاوه‌نگار؛ فقط روی سرور یا secret store نگهداری شود. |
| `Kavenegar__DefaultSender` | خیر | شمارهٔ خط ارسال. خالی باشد، خط پیش‌فرض حساب کاوه‌نگار استفاده می‌شود. در حساب فعلی DFI باید خط فعال صریحاً تنظیم شود. |

### اجرای Development از ابتدا

این دستورها را در یک ترمینال و به همین ترتیب اجرا کنید:

```bash
cd /home/hamedg/projects/Draje/DFI/dfi-api

# دریافت امن API Key بدون نمایش روی صفحه یا ذخیره در command history
read -rs "Kavenegar__ApiKey?API Key کاوه‌نگار: "
echo
export Kavenegar__ApiKey

# شمارهٔ واقعی یکی از خطوط ارسال فعال در پنل کاوه‌نگار
export Kavenegar__DefaultSender='<ACTIVE_KAVENEGAR_SENDER>'

# عبور مستقیم از proxy فقط برای دامنه‌های کاوه‌نگار
export NO_PROXY="${NO_PROXY:+${NO_PROXY},}api.kavenegar.com,.kavenegar.com"
export no_proxy="${no_proxy:+${no_proxy},}api.kavenegar.com,.kavenegar.com"

ASPNETCORE_ENVIRONMENT=Development \
dotnet run --project Draje.csproj --urls "http://0.0.0.0:5000"
```

`<ACTIVE_KAVENEGAR_SENDER>` باید با شمارهٔ واقعی خط فعال پنل جایگزین شود؛ عبارت نمونه یا شمارهٔ
موبایل شخصی معتبر نیست. همهٔ `export`ها باید پیش از `dotnet run` و در همان ترمینال انجام شوند.
بسته‌شدن ترمینال یا reboot این تنظیمات موقت را حذف می‌کند.

بعد از بالا آمدن API، کاربر دارای نقش `SmsSend` وارد DFI می‌شود، صفحهٔ `/Sms` را باز می‌کند،
گیرنده و متن را می‌نویسد و ارسال هزینه‌دار را در پنجرهٔ تأیید نهایی می‌پذیرد.

مقادیر نمونه را با دادهٔ واقعی جایگزین کنید، اما آن‌ها را در source code، `appsettings*.json`،
فایل‌های مستندات، command history قابل اشتراک، خروجی CI یا Git ثبت نکنید. در production از secret
store یا تنظیمات امن سرویس میزبان استفاده شود و پس از تغییر تنظیمات، پردازش API restart شود.

برای بررسی وجود API Key بدون نمایش مقدار آن:

```bash
if [[ -n ${Kavenegar__ApiKey:-} ]]; then
  echo 'Kavenegar API key is configured'
else
  echo 'Kavenegar API key is missing'
fi
```

Timeout ارتباط خروجی ۳۰ ثانیه است. آدرس پایه فقط باید HTTPS و دامنهٔ رسمی
`api.kavenegar.com` باشد.

### ارسال مستقیم اختیاری برای smoke test

روش عادی ارسال، صفحهٔ `/Sms` است. برای تست مستقیم endpoint، حساب تست باید نقش `SmsSend` داشته
باشد و `DFI_TEST_USERNAME` و `DFI_TEST_PASSWORD` از قبل در محیط تنظیم شده باشند. دستورهای زیر JWT
را فقط در `/tmp` نگه می‌دارند:

```bash
cd /home/hamedg/projects/Draje/DFI

jq -n \
  --arg username "$DFI_TEST_USERNAME" \
  --arg password "$DFI_TEST_PASSWORD" \
  '{username:$username,password:$password}' \
| curl --noproxy 192.168.3.54 \
    --fail-with-body --silent --show-error \
    -H 'Content-Type: application/json' \
    --data-binary @- \
    -o /tmp/dfi-sms-login.json \
    http://192.168.3.54:5000/api/Login/Login/authenticate

jq -r '.token' /tmp/dfi-sms-login.json > /tmp/dfi-sms-jwt
chmod 600 /tmp/dfi-sms-jwt

read -r "sms_receptor?شماره گیرنده: "
read -r "sms_message?متن پیامک: "
read -r sms_jwt < /tmp/dfi-sms-jwt

jq -n \
  --arg receptor "$sms_receptor" \
  --arg message "$sms_message" \
  '{receptors:[$receptor],message:$message}' \
| curl --noproxy 192.168.3.54 \
    --fail-with-body --silent --show-error \
    -H "Authorization: Bearer $sms_jwt" \
    -H 'Content-Type: application/json' \
    --data-binary @- \
    http://192.168.3.54:5000/api/sms/send

unset sms_jwt sms_receptor sms_message
```

آخرین `curl` یک ارسال واقعی و هزینه‌دار انجام می‌دهد و نباید خودکار تکرار شود. IP و port در محیطی
که API روی آدرس دیگری اجرا می‌شود باید متناسب با همان محیط تغییر کند.

## قرارداد داخلی API

### درخواست

```http
POST /api/sms/send
Authorization: Bearer <JWT>
Content-Type: application/json
```

```json
{
  "receptors": ["09120000000", "+989130000000"],
  "message": "متن آزمایشی پیامک"
}
```

اجرای این endpoint یک عملیات واقعی و هزینه‌دار است. در تست دستی فقط از شمارهٔ کنترل‌شده و متن
واضح آزمایشی استفاده کنید.

### پاسخ موفق

```json
{
  "recipientCount": 2,
  "totalCost": 2400,
  "entries": [
    {
      "messageId": 123456789,
      "status": 1,
      "statusText": "در صف ارسال",
      "sender": "100000000",
      "receptor": "09120000000",
      "sentAt": "2026-10-24T05:30:00+00:00",
      "cost": 1200
    }
  ]
}
```

مقادیر این نمونه ساختگی‌اند. `sentAt` یک instant با offset صریح UTC است و فرانت‌اند آن را با
`JalaliDateField` در زمان ایران نمایش می‌دهد. `totalCost` جمع `cost` تمام entryهای پاسخ است.

### پاسخ خطا

```json
{
  "error": "متن قابل نمایش برای کاربر",
  "providerCode": 400
}
```

`providerCode` فقط وقتی وجود دارد که کدی از کاوه‌نگار در دسترس باشد.

| HTTP status | معنی |
| --- | --- |
| `400` | payload، شماره یا متن نامعتبر است. |
| `401` | کاربر وارد سیستم نشده یا JWT معتبر نیست. |
| `403` | نقش `SmsSend` در JWT کاربر وجود ندارد. |
| `502` | کاوه‌نگار درخواست را رد کرده یا پاسخ قابل پردازش نبوده است. |
| `503` | تنظیمات وجود ندارد، ارتباط/timeout رخ داده یا خطای سرویس‌دهنده موقتی تشخیص داده شده است. |

کدهای `409`، `429`، `451` و کدهای ۵۰۰ به بالا در adapter فعلی موقتی در نظر گرفته می‌شوند. با
وجود این دسته‌بندی، retry خودکار عمداً فعال نیست؛ چون در timeout ممکن است کاوه‌نگار پیام را پذیرفته
باشد اما پاسخ به DFI نرسیده باشد و retry باعث ارسال تکراری شود.

## قواعد اعتبارسنجی

* حداقل یک گیرنده لازم است.
* پس از حذف موارد تکراری، حداکثر ۲۰۰ گیرنده مجاز است.
* متن بعد از trim نباید خالی یا بیشتر از ۱۸۰۰ کاراکتر باشد.
* ارقام فارسی `۰۱۲...` و عربی `٠١٢...` به لاتین تبدیل می‌شوند.
* فاصله، خط تیره و پرانتز شماره حذف می‌شود.
* قالب‌های `0912...`، `912...`، `98912...`، `+98912...` و `0098912...` پذیرفته می‌شوند.
* همهٔ شماره‌ها پیش از ارسال به قالب محلی `09xxxxxxxxx` تبدیل می‌شوند.
* شماره‌های تکراری پس از نرمال‌سازی فقط یک‌بار ارسال می‌شوند.
* متن برای همهٔ گیرندگان یکسان است.

اعتبارسنجی در فرانت‌اند برای بازخورد سریع کاربر انجام می‌شود، اما مرجع امنیتی و نهایی همیشه
بک‌اند است.

## دسترسی و منو

نام فنی نقش `SmsSend` و عنوان آن «ارسال پیامک» است. migration زیر نقش را به‌صورت top-level، فعال
و idempotent ایجاد یا اصلاح می‌کند:

```text
dfi-api/Migrations/User/20261024055216_AddSmsSendAccessRole.cs
```

اسکریپت تحویل production:

```text
dfi-api/Database/Scripts/20261024055216_AddSmsSendAccessRole.sql
```

برای استقرار production، فایل SQL را کامل و با **Execute SQL Script** در SSMS یا DBeaver اجرا
کنید؛ اجرای statement محل cursor کافی نیست. اسکریپت `GO` یا دستور sqlcmd ندارد و migration را فقط
در صورت نبودن در تاریخچه اعمال می‌کند.

بعد از استقرار:

1. نقش «ارسال پیامک» را از مدیریت کاربران به افراد مجاز اختصاص دهید.
2. کاربر باید logout و دوباره login کند تا JWT جدید شامل نقش شود.
3. وجود منوی «ارسال پیامک» و دسترسی به route `/Sms` را بررسی کنید.

مخفی‌بودن منو فقط برای تجربهٔ کاربری است؛ کنترل اصلی دسترسی با
`[Authorize(Roles = "SmsSend")]` در بک‌اند انجام می‌شود.

## رفتار رابط کاربری

صفحهٔ `/Sms` امکانات زیر را دارد:

* ورود شماره‌ها با سطر جدید، فاصله، ویرگول یا ویرگول فارسی؛
* نمایش تعداد گیرندگان یکتا و تعداد کاراکترهای متن؛
* هشدار اینکه خط فرستنده از تنظیمات سرور تعیین می‌شود؛
* تأیید صریح پیش از درخواست هزینه‌دار؛
* نمایش نتیجه در جدول صفحه‌بندی‌شده؛
* نمایش زمان با تقویم جلالی و منطقهٔ زمانی ایران؛
* نمایش مجموع هزینهٔ اعلام‌شده در پاسخ.

خطاهای HTTP توسط interceptor مرکزی
[`dfi-web/src/services/httpService.js`](../dfi-web/src/services/httpService.js) نمایش داده می‌شوند؛
صفحه نباید همان خطا را دوباره با `message.error` نمایش دهد.

## امنیت و حریم خصوصی

* API Key نباید به فرانت‌اند، payload داخلی، log، exception، Git یا مستندات وارد شود.
* چون API Key در path درخواست کاوه‌نگار قرار دارد، logging پیش‌فرض HttpClient این client حذف شده
  است. هنگام تغییر DI این حفاظت را حذف نکنید.
* شمارهٔ موبایل و متن پیام دادهٔ شخصی/سازمانی‌اند؛ body درخواست و پاسخ را log نکنید.
* ارسال باید فقط از بک‌اند انجام شود. فراخوانی مستقیم کاوه‌نگار از React ممنوع است.
* API Key را دوره‌ای rotate کنید و پس از افشای احتمالی فوراً جایگزین کنید.
* نقش `SmsSend` فقط به کاربران واقعاً مجاز داده شود.
* هیچ credential، JWT یا شمارهٔ واقعی مشتری نباید در تست خودکار قرار گیرد.

## استقرار و smoke test

### استقرار Production روی سرور اصلی Linux

در production از `dotnet run` و `export` موقت در یک ترمینال استفاده نکنید. نسخهٔ Release باید
publish شود و `Draje.dll` زیر نظر service manager موجود سرور اجرا شود.

#### ۱. آماده‌سازی artifactها

در ماشین build یا مسیر source مورد اعتماد:

```bash
cd /path/to/DFI/dfi-api
dotnet restore Draje.csproj
dotnet publish -c Release --no-restore -o ./publish Draje.csproj

cd /path/to/DFI/dfi-web
npm ci
npm run build
```

خروجی backend در `dfi-api/publish/` و خروجی frontend در `dfi-web/build/` است. این دو خروجی را با
فرایند release موجود سازمان روی سرور اصلی جایگزین کنید. connection stringها، secret مربوط به JWT
و سایر تنظیمات production موجود را حفظ کنید.

#### ۲. اعمال دسترسی دیتابیس

بعد از backup و بازبینی، فایل زیر را روی دیتابیس production با **Execute SQL Script** اجرا کنید:

```text
dfi-api/Database/Scripts/20261024055216_AddSmsSendAccessRole.sql
```

روی production فرمان `dotnet ef database update` اجرا نکنید. فایل تحویل idempotent است و برای
اجرای کامل در SSMS و DBeaver آماده شده است.

#### ۳. نگهداری امن تنظیمات SMS

برای نمونه، یک EnvironmentFile فقط‌خواندنی برای root بسازید:

```bash
sudo install -d -o root -g root -m 700 /etc/dfi
sudoedit /etc/dfi/dfi-api-sms.env
sudo chown root:root /etc/dfi/dfi-api-sms.env
sudo chmod 600 /etc/dfi/dfi-api-sms.env
```

محتوای فایل، بدون کلمهٔ `export`:

```dotenv
Kavenegar__ApiKey='<PRODUCTION_KAVENEGAR_API_KEY>'
Kavenegar__DefaultSender='<ACTIVE_PRODUCTION_SENDER>'
NO_PROXY='localhost,127.0.0.1,api.kavenegar.com,.kavenegar.com'
no_proxy='localhost,127.0.0.1,api.kavenegar.com,.kavenegar.com'
```

اگر `NO_PROXY` سرور مقصدهای دیگری دارد، آن‌ها را حذف نکنید و دامنه‌های کاوه‌نگار را به فهرست
موجود اضافه کنید. API Key داخل unit file، repository، فایل publish یا command line قرار نگیرد.

#### ۴. اتصال تنظیمات به systemd

اگر سرویس موجود مثلاً `dfi-api.service` نام دارد:

```bash
sudo systemctl edit dfi-api.service
```

این drop-in را اضافه کنید:

```ini
[Service]
EnvironmentFile=/etc/dfi/dfi-api-sms.env
```

سپس:

```bash
sudo systemctl daemon-reload
sudo systemctl restart dfi-api.service
sudo systemctl status dfi-api.service --no-pager
```

اگر نام سرویس متفاوت است، هر سه فرمان را با نام واقعی همان سرویس اجرا کنید. سرویس موجود باید
نسخهٔ publish‌شده را با الگویی شبیه دستور زیر اجرا کند:

```bash
ASPNETCORE_ENVIRONMENT=Production \
dotnet Draje.dll --urls "http://0.0.0.0:5000"
```

`ASPNETCORE_ENVIRONMENT`، binding، کاربر سرویس، `WorkingDirectory` و مسیر `Draje.dll` را مطابق
تنظیمات فعلی production نگه دارید؛ برای افزودن SMS نیازی به تغییر آن‌ها نیست.

#### ۵. فعال‌سازی کاربر و تست نهایی

1. در مدیریت کاربران نقش «ارسال پیامک» (`SmsSend`) را به کاربران مجاز بدهید.
2. کاربران را وادار کنید logout/login کنند تا JWT جدید صادر شود.
3. از خود سرور اتصال مستقیم را کنترل کنید:

   ```bash
   curl --noproxy '*' -I https://api.kavenegar.com
   ```

   پاسخ HTTP مانند `405 Method Not Allowed` برای `HEAD` نیز اثبات می‌کند DNS و TLS برقرار است.
4. از صفحهٔ production یک پیام کوتاه به شمارهٔ کنترل‌شده بفرستید.
5. Message ID، وضعیت و هزینه را در UI و پنل کاوه‌نگار تطبیق دهید.
6. رسیدن پیام به گوشی کنترل‌شده را تأیید کنید.

اگر backend به‌جای systemd با Docker، IIS یا ابزار دیگری اجرا می‌شود، همین چهار متغیر
`Kavenegar__ApiKey`، `Kavenegar__DefaultSender`، `NO_PROXY` و `no_proxy` باید در تنظیمات دائمی همان
runtime قرار گیرند؛ آن‌ها را فقط در shell کاربر export نکنید.

### چک‌لیست عمومی استقرار

چک‌لیست استقرار:

1. backend و frontend شامل فایل‌های این ماژول deploy شده باشند.
2. `Kavenegar__ApiKey` و در صورت نیاز `Kavenegar__DefaultSender` در محیط سرور تنظیم شده باشند.
3. سرویس backend پس از تنظیم secret restart شده باشد.
4. سرور به `https://api.kavenegar.com` دسترسی خروجی HTTPS داشته باشد.
5. اسکریپت نقش در دیتابیس production اجرا شده باشد.
6. نقش به کاربر آزمایش اختصاص یافته و کاربر دوباره login کرده باشد.
7. یک پیام کوتاه فقط به شمارهٔ کنترل‌شده ارسال شود.
8. Message ID، وضعیت و هزینه در UI و همان رکورد در پنل کاوه‌نگار کنترل شود.

هر smoke test واقعی هزینه دارد و نباید در CI، تست خودکار یا بدون تأیید اپراتور اجرا شود.

## تست‌های خودکار

تست‌های backend، نرمال‌سازی، payload خروجی، پاسخ موفق و خطاهای سرویس‌دهنده را پوشش می‌دهند:

```bash
cd dfi-api
dotnet test tests/Draje.UnitTests/Draje.UnitTests.csproj \
  --no-restore \
  --filter 'FullyQualifiedName~Sms'
```

تست‌های frontend، ورودی شماره‌ها و ثبت درست منو/نقش را پوشش می‌دهند:

```bash
cd dfi-web
npm test -- --watchAll=false --runInBand \
  src/components/sms/smsRecipients.test.js \
  src/components/general/menuConfig.test.js
```

این تست‌ها هیچ پیامک واقعی ارسال نمی‌کنند.

## عیب‌یابی

### منوی ارسال پیامک دیده نمی‌شود

* نقش `SmsSend` را برای کاربر بررسی کنید.
* کاربر بعد از تخصیص نقش باید logout/login کند.
* از اعمال‌شدن migration نقش در دیتابیس همان محیط مطمئن شوید.

### خطای «سرویس پیامک هنوز پیکربندی نشده است»

* وجود `Kavenegar__ApiKey` را بدون چاپ مقدار بررسی کنید.
* نام متغیر باید دقیقاً دارای دو underscore باشد.
* پس از تغییر environment، سرویس backend را restart کنید.

### خطای 502

* `providerCode` و متن خطای نمایش‌داده‌شده را با جدول رسمی کدهای کاوه‌نگار تطبیق دهید.
* اعتبار API Key، خط فرستنده، فعال‌بودن حساب و قواعد حساب کاوه‌نگار را بررسی کنید.
* `providerCode: 412` یعنی شمارهٔ فرستنده معتبر نیست. اگر `Kavenegar__DefaultSender` تنظیم نشده
  باشد، کاوه‌نگار فرستندهٔ پیش‌فرض حساب را انتخاب می‌کند؛ در این حالت وجود و فعال‌بودن خط پیش‌فرض
  را در پنل بررسی کنید. در غیر این صورت `Kavenegar__DefaultSender` را دقیقاً برابر یکی از خطوط
  ارسال فعال حساب قرار دهید و backend را restart کنید. شمارهٔ موبایل شخصی یا عبارت نمونه، خط
  فرستنده محسوب نمی‌شود.
* پیام یا اطلاعات حساس را در ticket و log عمومی کپی نکنید.

### خطای 503 یا timeout

* DNS و دسترسی HTTPS خروجی سرور به `api.kavenegar.com` را بررسی کنید.
* وضعیت سرویس و اعتبار حساب کاوه‌نگار را از پنل کنترل کنید.
* اگر روی میزبان `HTTP_PROXY` یا `HTTPS_PROXY` فعال است، اتصال مستقیم را نیز آزمایش کنید. ممکن
  است proxy تونل HTTPS را ایجاد کند اما handshake TLS را با خطای `unexpected EOF` ببندد:

  ```bash
  curl --noproxy '*' -I https://api.kavenegar.com
  ```

  دریافت پاسخ HTTP، حتی `405 Method Not Allowed` برای متد `HEAD`، نشان می‌دهد DNS و TLS مستقیم
  برقرارند. در این حالت دامنهٔ رسمی کاوه‌نگار را بدون حذف مقصدهای قبلی به bypass list اضافه و
  پردازش backend را restart کنید:

  ```bash
  export NO_PROXY="${NO_PROXY:+${NO_PROXY},}api.kavenegar.com,.kavenegar.com"
  export no_proxy="${no_proxy:+${no_proxy},}api.kavenegar.com,.kavenegar.com"
  ```

  خاموش‌کردن سراسری proxy توصیه نمی‌شود؛ bypass باید فقط برای دامنهٔ رسمی کاوه‌نگار باشد.
* پیش از ارسال دوباره، پنل کاوه‌نگار را با گیرنده و بازهٔ زمانی بررسی کنید؛ ممکن است درخواست اول
  پذیرفته شده ولی پاسخ آن به DFI نرسیده باشد.

### فرستنده مطابق انتظار نیست

* مقدار `Kavenegar__DefaultSender` را بررسی کنید.
* اگر مقدار خالی است، خط پیش‌فرض حساب کاوه‌نگار استفاده می‌شود.
* مجازبودن خط برای حساب باید در پنل کاوه‌نگار بررسی شود.

### نتیجه بعد از refresh صفحه از بین رفته است

این رفتار نسخهٔ فعلی است؛ DFI تاریخچهٔ ارسال را نگهداری نمی‌کند. مرجع عملیاتی فعلی پنل کاوه‌نگار
است. افزودن تاریخچه نیازمند مدل دیتابیس، migration، سیاست نگهداری داده، دسترسی خواندن و صفحهٔ
گزارش مستقل است.

## قواعد توسعهٔ بعدی

* interface `ISmsSender` مرز DFI و سرویس‌دهنده است؛ controller نباید مستقیماً HttpClient یا API
  کاوه‌نگار را صدا بزند.
* controller و DTO داخلی نباید API Key یا جزئیات احراز هویت سرویس‌دهنده را بپذیرند.
* افزودن Lookup، استعلام تحویل، زمان‌بندی یا دریافت پیام باید با endpoint، DTO، دسترسی و تست مستقل
  انجام شود.
* برای عملیات هزینه‌دار جدید، تأیید صریح کاربر حفظ شود.
* اگر تاریخچه افزوده شد، زمان‌ها به UTC ذخیره و در API با offset صریح برگردانده شوند.
* اگر retry یا صف ارسال افزوده شد، idempotency و جلوگیری از ارسال تکراری باید قبل از فعال‌سازی
  طراحی و آزمایش شود.
* هر تغییر در محدودیت گیرنده، طول پیام، mapping خطا، تنظیمات یا پاسخ API باید هم در backend، هم
  frontend، هم تست‌ها و هم این سند اعمال شود.
