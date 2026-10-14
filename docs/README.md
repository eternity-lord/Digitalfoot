# مستندات پروژه — Digital Footprint Intelligence

این پوشه تنها منبع حقیقت برای تصمیم‌های محصول، معماری و محدودیت‌هاست.
قرارداد ثابت پروژه در فایل [`Rules`](../Rules) قرار دارد و **همه مستندات باید با آن سازگار باشند**.

## وضعیت فعلی

| مورد | وضعیت |
| --- | --- |
| کد اپ | ❌ هنوز وجود ندارد (مخزن فقط `README.md` و `Rules` داشت) |
| مرحله ۱ (پژوهش + معماری) | ✅ انجام شد — ۲۰۲۶‑۱۰‑۱۴ |
| مرحله ۲ به بعد | ⏳ طبق `docs/implementation/roadmap.md` از فاز P0 |

## راهنمای مطالعه

| سند | محتوا | چه کسی/کی می‌خواند |
| --- | --- | --- |
| [`research/source-catalog.md`](research/source-catalog.md) | کاتالوگ کامل منابع با ۱۲ فیلد الزامی برای هر منبع + سه‌دسته‌بندی دسترسی | قبل از پیاده‌سازی هر connector |
| [`research/discovered-capabilities.md`](research/discovered-capabilities.md) | قابلیت‌های سه‌دسته + ۲۵ قابلیت جدید کشف‌شده با ارزش/منبع/محدودیت/هزینه/ریسک | تصمیم‌گیری درباره دامنه محصول |
| [`research/references.md`](research/references.md) | منابع رسمی و دست‌دوم پژوهش + ادعاهای رد‌شده | راستی‌آزمایی مجدد قبل از هر ادعا |
| [`architecture/system-design.md`](architecture/system-design.md) | ماژول‌ها، قرارداد Connector، موتور همبستگی، dedup، خط زمانی، مدیریت اسکن، گزارش پوشش، استراتژی تست | شروع هر فاز فنی |
| [`architecture/data-model.md`](architecture/data-model.md) | مدل دامنه یکپارچه، موجودیت‌ها، enumها، سیاست ذخیره داده حساس، مهاجرت‌ها | قبل از هر تغییر schema |
| [`architecture/security-model.md`](architecture/security-model.md) | مدل تهدید، رمزنگاری، TokenVault، redaction، انطباق Play/GDPR، سطوح حذف | قبل از هر PR (دروازه‌های امنیتی) |
| [`implementation/roadmap.md`](implementation/roadmap.md) | فازهای P0…P12 با خروجی، تست و ریسک | برنامه‌ریزی هر مرحله |
| [`implementation/acceptance-criteria.md`](implementation/acceptance-criteria.md) | معیارهای پذیرش تست‌پذیر برای هر فاز + قالب گزارش | پایان هر فاز |
| [`decisions/adr-log.md`](decisions/adr-log.md) | ۲۰ تصمیم معماری با زمینه/پیامد/وضعیت | وقتی سؤالی درباره «چرا این‌طور؟» پیش آمد |
| [`open-questions.md`](open-questions.md) | ۱۰ سؤال در انتظار تصمیم کاربر + ۱۲ مورد نیازمند راستی‌آزمایی فنی | قبل از شروع P0 |
| [`limitations.md`](limitations.md) | محدودیت‌ها در ۶ دسته + موارد رد‌شده آگاهانه | هر وقت «چرا این قابلیت نیست؟» پرسیده شد |
| [`reports/`](reports/) | گزارش پایان هر فاز (قالب در acceptance-criteria بخش ۱۲) | پایان هر فاز |

## قواعد نگهداری مستندات

1. **هیچ ادعای راستی‌آزمایی‌نشده‌ای** به‌عنوان قابلیت معرفی نمی‌شود؛ موارد مشکوک با ⚠️ و در `limitations.md` (بخش V) ثبت می‌شوند.
2. هر تغییر معماری = یک ADR جدید یا به‌روزرسانی وضعیت ADR قبلی.
3. هر محدودیت تازه کشف‌شده در اجرا = یک ردیف جدید در `limitations.md` با دلیل و جایگزین.
4. هر فاز = یک فایل گزارش در `docs/reports/`.
5. مستندات هم مثل کد کامیت و پوش می‌شوند (قانون ۱۶/۱۷ `Rules`).
