# Digitalfoot

**Digital Footprint Intelligence** — اپ اندرویدی واقعی، کامل و قابل توسعه برای کشف، گردآوری، نرمال‌سازی، ارتباط‌دهی و نمایش ردپاهای دیجیتال **صاحب دستگاه** و حساب‌هایی که او صراحتاً اجازه دسترسی به آن‌ها را داده است.

این پروژه یک نمونه اولیه، دمو یا فهرست برنامه‌های نصب‌شده نیست؛ یک محصول داده‌محور با منشأ (provenance) کامل، اطمینان سنجیده و گزارش صادقانه است.

## وضعیت

| مورد | وضعیت |
| --- | --- |
| مرحله ۱ — پژوهش، نیازمندی‌ها و معماری | ✅ تکمیل شد (۲۰۲۶‑۱۰‑۱۴) |
| مرحله ۲ به بعد — پیاده‌سازی (فازهای P0…P12) | ⏳ شروع نشده |
| کد اپ | ❌ هنوز وجود ندارد |

## قوانین پروژه

قرارداد ثابت و الزام‌آور پروژه در فایل [`Rules`](Rules) است. هر تغییر (کد یا مستندات) باید با آن سازگار باشد و پس از پایان هر درخواست، کامیت و پوش شود (قانون ۱۶ و ۱۷ همان فایل).

## مستندات

همه مستندات در [`docs/`](docs/) قرار دارند؛ نمایه کامل در [`docs/README.md`](docs/README.md).

| سند | محتوا |
| --- | --- |
| [`docs/research/source-catalog.md`](docs/research/source-catalog.md) | کاتالوگ منابع ردپا با ۱۲ فیلد الزامی برای هر منبع |
| [`docs/research/discovered-capabilities.md`](docs/research/discovered-capabilities.md) | دسته‌بندی سه‌گانه قابلیت‌ها + قابلیت‌های جدید کشف‌شده |
| [`docs/research/references.md`](docs/research/references.md) | منابع رسمی و دست‌دوم پژوهش + ادعاهای رد‌شده |
| [`docs/architecture/system-design.md`](docs/architecture/system-design.md) | معماری سیستم، قرارداد Connectorها، موتور همبستگی، اسکن، تست |
| [`docs/architecture/data-model.md`](docs/architecture/data-model.md) | مدل دامنه یکپارچه و موجودیت‌های پایگاه داده |
| [`docs/architecture/security-model.md`](docs/architecture/security-model.md) | مدل تهدید، رمزنگاری، redaction، حذف داده، انطباق Play/GDPR |
| [`docs/implementation/roadmap.md`](docs/implementation/roadmap.md) | نقشه راه فازهای P0…P12 |
| [`docs/implementation/acceptance-criteria.md`](docs/implementation/acceptance-criteria.md) | معیارهای پذیرش تست‌پذیر هر فاز |
| [`docs/decisions/adr-log.md`](docs/decisions/adr-log.md) | لاگ تصمیم‌های معماری (ADR) |
| [`docs/open-questions.md`](docs/open-questions.md) | موارد مبهم در انتظار تصمیم |
| [`docs/limitations.md`](docs/limitations.md) | محدودیت‌های فنی، سیاستی و موارد رد‌شده آگاهانه |
| [`docs/reports/`](docs/reports/) | گزارش پایان هر مرحله |

## اصول غیرقابل مذاکره

1. **داده فقط روی دستگاه** — هیچ سرور، تحلیل یا همگام‌سازی ابری متعلق به ما وجود ندارد.
2. **فقط مسیرهای مجاز** — OAuth رسمی، API عمومی، فایل خروجی متعلق به کاربر، مجوزهای صریح اندروید. بدون scraping، بدون شمارش خودکار نام کاربری، بدون دورزدن محدودیت سرویس‌ها.
3. **بدون جعل داده** — تاریخ ثبت‌نام، مشاهده، دریافت و استنتاج‌شده هرگز با هم اشتباه گرفته نمی‌شوند.
4. **رازی ذخیره نمی‌شود** — رمز عبور، مقدار کوکی و secret دو‌عاملی هرگز در پایگاه داده، لاگ یا خروجی نمی‌آیند.
5. **گزارش صادقانه** — تفکیک «کشف‌شده / احتمالی / تأییدشده / غیرقابل دسترس / بررسی‌نشده / بدون نتیجه» و پرهیز از ادعای کامل‌بودن.
