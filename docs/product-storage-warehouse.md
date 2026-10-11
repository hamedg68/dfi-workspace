# تشخیص انبار نگهداری محصول از شماتیک تولید

این مستند منبع داده، قواعد استخراج و قرارداد سرویس «انبار نگهداری محصول» را توضیح می‌دهد. این
سرویس برای نمایش «خلاصه بر اساس انبار نگهداری» در جزئیات سفارش‌ها ساخته شده است و فقط داده
می‌خواند.

## مسئله و منبع حقیقت

جدول `LGS3.PartStore` برای این قابلیت منبع قابل اتکایی نیست؛ چون ممکن است یک محصول به بیش از یک
انبار راهکاران متصل باشد، در حالی که کارخانه برای نگهداری محصول یک انبار عملیاتی مشخص در نظر
گرفته است.

منبع حقیقت این قابلیت، شماتیک اصلی و فعال محصول در DFI است:

* محصول فعال از `LGS3.Part` خوانده می‌شود؛
* شرط محصول در این سرویس `PartNature = 2`، `State = 1` و شروع کد با `11` است؛
* شماتیک از `eng.Methods` با شرط‌های `Main = 1` و `Active = 1` خوانده می‌شود؛
* انبار نگهداری، گره خروجی نهایی از نوع `wareHouse` در فیلد JSON شماتیک است؛
* مشخصات انبار از `eng.WareHouses` خوانده می‌شود؛
* `WareHouses.StoreId` شناسه انبار متناظر در راهکاران است.

این سرویس هیچ شناسه انباری را hard-code نمی‌کند. در داده فعلی، انبارهای تجاری مورد انتظار به
`StoreId`های `3` و `100000019` متصل‌اند، اما اضافه‌شدن یا تغییر انبار باید از خود شماتیک و
`eng.WareHouses` منعکس شود، نه با تغییر شرط ثابت در کد.

## الگوریتم استخراج خروجی شماتیک

قواعد استخراج با تعریف `AnalyzeMethodScheme` در `MethodsController` یکسان است:

1. اشیای دارای `vertex = 1` به‌عنوان گره خوانده می‌شوند؛
2. اشیای دارای `edge = 1` به‌عنوان یال خوانده می‌شوند؛
3. شناسه گره‌های خروجی از تفاضل `TargetIds - SourceIds` به دست می‌آید؛
4. فقط گره خروجی با `itemType = "wareHouse"` پذیرفته می‌شود؛
5. هر محصول فقط وقتی نگاشت معتبر دارد که دقیقاً یک متد اصلی و فعال و دقیقاً یک شناسه انبار خروجی
   متمایز داشته باشد؛
6. شناسه خروجی با `eng.WareHouses.Id` تطبیق داده می‌شود.

منطق مشترک خواندن خروجی شماتیک در فایل زیر قرار دارد:

```text
dfi-api/Services/Engineering/WarehouseOutputExtractor.cs
```

جریان قدیمی تعیین انبار در `SalesRequestBalanceController` نیز از همین استخراج‌کننده استفاده
می‌کند تا دو پیاده‌سازی مستقل از یک قاعده وجود نداشته باشد.

## قرارداد API

درخواست:

```http
GET /api/ProductStorageWarehouses
Authorization: Bearer <token>
```

endpoint فقط‌خواندنی است و برای همه کاربران احراز هویت‌شده قابل استفاده است. این سطح دسترسی لازم
است، چون `DetailOrderModal` علاوه بر صفحات لجستیک در صفحه حمل‌ونقل نیز استفاده می‌شود.

نمونه پاسخ هر محصول:

```json
{
  "partId": "123456789012345678",
  "partCode": "1101001",
  "partName": "نام محصول",
  "warehouseId": 57,
  "warehouseName": "انبار کالای تجاری",
  "rahkaranStoreId": "3",
  "mappingStatus": "mapped"
}
```

به‌علت مبدل JSON سراسری DFI، مقادیر `long` مانند `partId` و `rahkaranStoreId` به‌صورت رشته ارسال
می‌شوند. `warehouseId` از نوع `int` است و عدد باقی می‌ماند.

مقادیر ممکن `mappingStatus` عبارت‌اند از:

| مقدار | معنی |
| --- | --- |
| `mapped` | نگاشت یکتا و معتبر است |
| `missing-active-main-method` | متد اصلی و فعال وجود ندارد |
| `ambiguous-active-main-method` | بیش از یک متد اصلی و فعال وجود دارد |
| `missing-or-ambiguous-warehouse-output` | خروجی انبار وجود ندارد یا بیش از یک خروجی متمایز وجود دارد |
| `warehouse-not-found` | شناسه خروجی در `eng.WareHouses` پیدا نشده است |

## رفتار رابط کاربری

`DetailOrderModal` هنگام بازشدن، نگاشت انبارها و سلسله‌مراتب محصولات را مستقل و هم‌زمان دریافت
می‌کند. محصولات با `PartCode` نرمال‌شده به پاسخ سرویس متصل می‌شوند.

`SalesOrderSummaryStep` در حالت مخصوص جزئیات سفارش، خلاصه سوم را با عنوان «خلاصه بر اساس انبار
نگهداری» نمایش می‌دهد. برای هر انبار این مقادیر تجمیع می‌شوند:

* تعداد کارتن؛
* وزن خالص؛
* وزن ناخالص؛
* حجم.

اگر یک محصول نگاشت معتبر نداشته باشد، حذف یا به یکی از انبارها نسبت داده نمی‌شود و در گروه زیر
نمایش داده می‌شود:

```text
انبار نامشخص — فاقد شماتیک اصلی و فعال
```

نمایش این خلاصه opt-in است و فقط از `DetailOrderModal` فعال می‌شود؛ بنابراین استفاده‌های دیگر
`SalesOrderSummaryStep` در فرم ثبت سفارش تغییر نمی‌کنند. اگر کل درخواست API ناموفق باشد، بخش
خلاصه انبار نمایش داده نمی‌شود و خطای HTTP توسط interceptor مشترک فرانت‌اند اعلام می‌شود.

## فایل‌های مرتبط

### Backend

```text
dfi-api/Controllers/Engineering/ProductStorageWarehousesController.cs
dfi-api/Dtos/Engineering/ProductStorageWarehouseResponseDto.cs
dfi-api/Services/Engineering/ProductStorageWarehouseReadService.cs
dfi-api/Services/Engineering/WarehouseOutputExtractor.cs
dfi-api/Controllers/Logistic/SalesRequestBalanceController.cs
```

### Frontend

```text
dfi-web/src/services/logistic/salesRequestBalance.js
dfi-web/src/components/logistic/marketOrders/detailOrderModal.jsx
dfi-web/src/components/logistic/marketOrders/salesOrderSummaryStep.jsx
```

## قواعدی که باید حفظ شوند

هنگام تغییر این قابلیت:

* از `LGS3.PartStore` به‌عنوان fallback استفاده نکنید؛
* شناسه‌های انبار راهکاران را در کد ثابت نکنید؛
* در حالت چند متد یا چند خروجی، اولین مورد را به‌صورت دلخواه انتخاب نکنید؛
* محصولات فاقد نگاشت را از خلاصه حذف نکنید؛
* تغییر قرارداد API را در هر دو مخزن backend و frontend بررسی کنید؛
* این read model نباید شماتیک، انبار، محصول یا اسناد راهکاران را تغییر دهد.
