# Workflow V4 — منشی هوشمند کلینیک

این بسته نسخهٔ تمیزشدهٔ Workflowهای n8n است. نسخه‌های قدیمی و تکراری از مسیر Import حذف شده‌اند و برای هر قابلیت فقط نسخهٔ منتخب `Corrected`، `Guarded` یا `Final` نگه داشته شده است.

> همهٔ فایل‌ها را یک‌جا Active نکنید. ابتدا دیتابیس را بسازید، Credentialها را متصل کنید و سپس فقط Workflowهای لازم برای حالت Mock یا Production را فعال کنید.

## ساختار پوشه‌ها

| پوشه | کاربرد |
|---|---|
| `00_SETUP` | تست اتصال، ساخت schema پایه و دادهٔ دموی اختیاری |
| `01_BOOTSTRAP` | Migrationهای تکمیلی دیتابیس؛ یک بار و به ترتیب عددی اجرا شوند |
| `02_ENTRYPOINTS` | Gateway ورودی، Normalize کانال و Health Check |
| `03_SCHEDULERS` | زمان‌بندی کارهای عملیاتی و نگهداری |
| `04_WORKERS` | صف پیام، Retry، Confirmation، Follow-up و Cleanup |
| `05_CONVERSATION` | Orchestrator، Safety، Tool Router و Response Composer |
| `05_DOMAIN_APPOINTMENT` | یافتن وقت، ثبت، لغو و جابه‌جایی نوبت |
| `05_CLINICAL` | Intake و RAG بالینی |
| `05_COMMUNICATION` | Consent، صف و وضعیت پیام |
| `05_DOMAIN_HANDOFF` | ارجاع به اپراتور انسانی |
| `05_DOMAIN_SUPPORT` | دارو، بیمه، بازخورد و اطلاعات کلینیک |
| `05_PATIENT` | ثبت بیمار، پروفایل، مدارک و راهنمای مراجعه |
| `06_ADMIN_PLATFORM` | مدیریت کلینیک، پزشک، خدمات و امنیت Gateway |
| `06_ADMIN_OBSERVABILITY` | Health، Audit، Dashboard و Dead Letter |
| `07_TESTS` | Seed تست، Mockها، اجرای E2E و گزارش |

جزئیات وضعیت هر پوشه در `IMPORT_MANIFEST.md` آمده است.

## پیش‌نیازها

- n8n فعال
- PostgreSQL جداگانه برای Test/Staging
- دسترسی n8n به آدرس webhook خودش
- PostgreSQL Credential
- OpenAI Credential فقط برای AI واقعی
- Google Calendar Credential فقط برای Calendar Sync

Credentialها عمداً داخل بسته قرار نگرفته‌اند. پس از Import، Credential مناسب محیط را روی Nodeها دوباره انتخاب کنید.

## مرحله ۱ — آماده‌سازی دیتابیس

1. `00_SETUP/00 - PostgreSQL Connection Test.json` را Import کنید، PostgreSQL Credential را انتخاب و اجرا کنید.
2. `00_SETUP/01 - Initialize Clinic Database.json` را Import و یک بار اجرا کنید.
3. فایل‌های `01_BOOTSTRAP` را دقیقاً با ترتیب زیر Import و یک بار اجرا کنید:

```text
13 → 15 → 17 → 19 → 22 → 24 → 26 → 29 → 31
→ 34 → 37 → 40 → 43 → 46 → 49 → 55 → 59 → 64
→ 70 → 77 → 82 → 93 → 101 → 107 → 113
```

ترتیب را بر اساس عدد بخوانید؛ مرتب‌سازی الفبایی ممکن است `101` را قبل از `13` نشان دهد.

فایل `00_SETUP/02 - Seed Clinic Demo Data.json` فقط برای دموی قدیمی و اختیاری است. برای تست یکپارچه، Workflow شماره 114 محیط کامل‌تری می‌سازد.

## مرحله ۲ — Import برنامه

پوشه‌های `02` تا `06` را Import کنید. Import به معنی Active کردن همهٔ Workflowها نیست.

پس از Import:

1. PostgreSQL Credential را روی تمام PostgreSQL Nodeها انتخاب کنید.
2. مقدار `webhook_base_url` را با محیط خود هماهنگ کنید.
3. در Docker معمولاً مقدار `http://n8n:5678/webhook` مناسب است؛ در n8n Cloud یا سرور مستقل از Domain واقعی استفاده کنید.
4. داخل Container از `localhost` استفاده نکنید، مگر n8n واقعاً در همان Container مقصد باشد.

## مرحله ۳ — تست Mock بدون OpenAI

در حالت Mock:

```text
106A = OFF
106D = OFF
119A = ON
119B Corrected V3 = ON
```

Workflowهای زیر را نیز Import و Active کنید:

```text
108  Unified Inbound Gateway
106E Conversation Orchestrator
52   Deterministic Safety Gate
106B Tool Router
106C Execute Routed Tool
```

همچنین Workflowهای دامنه‌ای که Tool Router برای سناریوی تست فراخوانی می‌کند باید Active باشند؛ برای تست‌های فعلی حداقل این موارد را آماده نگه دارید:

```text
11  Find Available Slots
12  Find Patient Appointments
47  Ask Clinic Information
```

سپس:

1. `114 - Seed Isolated Integration Test Environment - Corrected V3` را با `confirm_seed=true` اجرا کنید.
2. مقدار `webhook_base_url` در 114 و 115 را یکسان تنظیم کنید.
3. Workflow شماره 115 را برای هر case اجرا کنید:

```text
conversation_clinic_hours
conversation_patient_appointments
conversation_missing_information
```

4. نتیجه را با Workflow شماره 118 بررسی کنید.

نتیجهٔ مورد انتظار:

```text
test_status = passed
error_message = ""
تمام assertion_results[].passed = true
```

کوتاه‌ترین مسیر تست پس از Bootstrap:

```text
114 → 115 × 3 → 118
```

## مرحله ۴ — تست ساخت نوبت

Workflow شماره 09 اکنون به نسخه منتخب زیر متصل است:

```text
07 - Core - Create Appointment - Schedule Guarded V2
```

ابتدا شماره 07 را Import کنید. در Workflow شماره 09، Node با نام `Run Core Appointment Workflow` را باز کنید و بررسی کنید که همین Workflow انتخاب شده باشد؛ بعضی نسخه‌های n8n هنگام Import گروهی reference را مجدداً نگاشت می‌کنند.

برای Sync واقعی Calendar، ابتدا `00_SETUP/05 - Google Calendar Connection Test.json` را با Credential محیط اجرا کنید.

## مرحله ۵ — AI واقعی و Production

پس از قبولی Mock:

```text
119A = OFF
119B = OFF
106A = ON
106D = ON
```

OpenAI Credential را روی Nodeهای مدل در Workflowهای 106A و 106D انتخاب کنید. قابلیت‌های بالینی و Previsit ممکن است به OpenAI Credential جداگانه در Workflowهای 68 و 75 نیاز داشته باشند.

ورودی Provider واقعی باید ابتدا به Workflow شماره 112B برسد:

```text
Provider → 112B → 108 → 106E → Safety/Router/Tool → Response
```

## Workflowهایی که نباید هم‌زمان Active باشند

این Webhookها عمداً مشترک‌اند:

| واقعی | Mock | وضعیت مجاز |
|---|---|---|
| `106A` | `119A` | فقط یکی Active |
| `106D` | `119B` | فقط یکی Active |

فعال‌سازی هم‌زمان باعث Webhook Conflict می‌شود.

## Credential Matrix

| Credential | Mock | Production |
|---|---:|---:|
| PostgreSQL | لازم | لازم |
| OpenAI | لازم نیست | برای قابلیت‌های AI لازم |
| Google Calendar | فقط تست Sync | در صورت فعال‌بودن Sync لازم |
| Channel Provider | لازم نیست | برای کانال واقعی لازم |

## عیب‌یابی سریع

### Table or column does not exist

Workflow شماره 01 و سپس تمام Migrationهای `01_BOOTSTRAP` را با ترتیب اعلام‌شده اجرا کنید.

### Workflow not found در شماره 09

در Node `Run Core Appointment Workflow` نسخه `07 - Core - Create Appointment - Schedule Guarded V2` را دوباره انتخاب و Save کنید.

### Connection refused یا timeout

`webhook_base_url` باید از داخل محیط اجرای n8n قابل دسترس باشد. برای Workflow فعال از `/webhook/` استفاده کنید، نه `/webhook-test/`.

### Webhook conflict

وضعیت جفت‌های 106A/119A و 106D/119B را بررسی کنید.

### Credential error

Credentialهای exportشده قابل انتقال قطعی بین Instanceها نیستند. Credential محیط مقصد را دوباره روی Node مربوطه انتخاب کنید.

## قبل از Pilot

- هر سه تست Mock چند بار متوالی Pass شوند.
- ثبت نوبت تکراری و هم‌پوشانی پزشک رد شود.
- جداسازی اطلاعات دو کلینیک آزمایش شود.
- مسیرهای اورژانسی به Human Handoff برسند.
- Opt-out، Quiet Hours، Retry و Dead Letter تست شوند.
- Secret و اطلاعات حساس داخل Logها ذخیره نشوند.
- Backup و Restore دیتابیس عملاً آزمایش شود.

## تغییرات V4

- اضافه‌شدن مرحله صریح Setup و schema پایه
- حذف نسخه‌های قدیمی و تکراری از بسته اجرایی
- نگه‌داشتن نسخه‌های Corrected، Guarded و Final منتخب
- اصلاح reference Workflow شماره 09 به Workflow شماره 07 منتخب
- جایگزینی 119B قدیمی با `Corrected V3`
- حذف duplicate ID و webhookهای تصادفی
- اضافه‌شدن راهنمای Import، Activation، Credential و تست

