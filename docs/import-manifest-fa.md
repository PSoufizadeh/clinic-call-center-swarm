# Import Manifest — Workflow V4

## معنی وضعیت‌ها

| وضعیت | معنی |
|---|---|
| `RUN_ONCE` | Import، اتصال Credential و یک بار اجرای دستی |
| `IMPORT` | Import شود؛ Activation به کاربرد محیط بستگی دارد |
| `ACTIVATE` | پس از تنظیم محیط فعال شود |
| `MOCK_ONLY` | فقط در تست Mock فعال شود |
| `OPTIONAL` | فقط در صورت نیاز Import یا اجرا شود |

## پوشه‌ها

| مسیر | وضعیت | توضیح |
|---|---|---|
| `00_SETUP/00` | `RUN_ONCE` | تست PostgreSQL قبل از هر کار |
| `00_SETUP/01` | `RUN_ONCE` | ساخت جداول پایه؛ اجباری در دیتابیس تازه |
| `00_SETUP/02` | `OPTIONAL` | دادهٔ دموی ساده؛ با 114 هم‌زمان لازم نیست |
| `00_SETUP/05` | `OPTIONAL` | تست اتصال Google Calendar |
| `01_BOOTSTRAP/*` | `RUN_ONCE` | Migrationها با ترتیب عددی README |
| `02_ENTRYPOINTS/*` | `ACTIVATE` | ورودی‌های اصلی؛ پس از تنظیم امنیت و URL |
| `03_SCHEDULERS/*` | `OPTIONAL` | پس از پایان تست و کنترل اثرات فعال شود |
| `04_WORKERS/*` | `IMPORT` | Worker لازم متناسب با قابلیت فعال شود |
| `05_CLINICAL/*` | `IMPORT` | قابلیت‌های بالینی و RAG |
| `05_COMMUNICATION/*` | `IMPORT` | Consent و Messaging |
| `05_CONVERSATION/106A` | `ACTIVATE` | فقط AI واقعی؛ با 119A هم‌زمان نباشد |
| `05_CONVERSATION/106D` | `ACTIVATE` | فقط AI واقعی؛ با 119B هم‌زمان نباشد |
| `05_CONVERSATION/106B,106C,106E,50,52` | `ACTIVATE` | اجزای مسیر مکالمه مطابق محیط |
| `05_DOMAIN_*/*` | `IMPORT` | Workflowهای دامنه؛ موارد فراخوانی‌شده Active شوند |
| `05_PATIENT/*` | `IMPORT` | عملیات بیمار و Previsit |
| `06_ADMIN_*/*` | `IMPORT` | Administration، Security و Observability |
| `07_TESTS/09` | `OPTIONAL` | Self-test ثبت نوبت؛ reference شماره 07 بررسی شود |
| `07_TESTS/114` | `RUN_ONCE` | Seed محیط تست ایزوله |
| `07_TESTS/115` | `OPTIONAL` | اجرای E2E Mock برای هر case |
| `07_TESTS/118` | `OPTIONAL` | گزارش تست |
| `07_TESTS/119A` | `MOCK_ONLY` | جایگزین 106A در Mock |
| `07_TESTS/119B Corrected V3` | `MOCK_ONLY` | جایگزین 106D در Mock |
| `07_TESTS/121,122` | `MOCK_ONLY` | Adapter و Callback آزمایشی |

## نسخه‌های کنارگذاشته‌شده از V4

این فایل‌ها پس از مقایسه با نسخه‌های نهایی، وارد بسته اجرایی V4 نشده‌اند:

| نسخه قدیمی | جایگزین منتخب |
|---|---|
| `03 - Find Available Appointment Slots` | `11 - Find Available Slots - Schedule Guarded V2` |
| `04 - Create Demo Appointment` | `07 - Core - Create Appointment - Schedule Guarded V2` و تست 09 |
| `06 - Create Appointment with Calendar Sync` | `07 - Schedule Guarded V2` |
| `07 - Core - Create Appointment` | `07 - Schedule Guarded V2` |
| نسخه تکراری `07 Schedule Guarded` در Initial | نسخه موجود در `05_DOMAIN_APPOINTMENT` |
| `08 - Test Core Appointment Workflow` | Self-test شماره 09 و E2E شماره 115 |
| نسخه تکراری شماره 09 در Initial | نسخه اصلاح‌شده در `07_TESTS` |
| `10 - Core - Find Available Slots` | نسخه Standalone/Guarded شماره 11 |
| نسخه‌های تکراری 11 و 12 در Initial | نسخه‌های موجود در `05_DOMAIN_APPOINTMENT` |
| `119B Credential Free` قدیمی | `119B Corrected V3` |

فایل‌های 00، 01، 02 و 05 از مجموعه Initial نگه داشته شدند، چون نقش Setup یا تست اتصال دارند و جایگزین مستقیم Workflow عملیاتی نیستند.

