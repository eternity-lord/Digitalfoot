# کاتالوگ منابع ردپای دیجیتال — Source Catalog

> مرحله ۱ (پژوهش و معماری) — تاریخ پژوهش: ۲۰۲۶‑۱۰‑۱۴ (۱۴۰۴‑۰۷‑۲۲)
> این سند فهرست منابع داده‌ای است که می‌توانند درباره تاریخچه دیجیتال **صاحب دستگاه** و **حساب‌هایی که صراحتاً اجازه دسترسی داده** اطلاعات بدهند.
>
> **قواعد حاکم بر این سند (از `Rules`):**
> - هیچ منبعی صرفاً به علت دشواری پیاده‌سازی حذف نشده؛ محدودیت و راهکار مجاز آن ثبت شده است (قانون ۷).
> - هیچ API یا قابلیتی که وجود ندارد ادعا نشده است (قانون ۵).
> - تاریخ واقعی ثبت‌نام از شواهد غیرمستقیم جدا نگه داشته شده است (قانون ۶).
> - مواردی که از منابع رسمی به‌طور مستقیم راستی‌آزمایی نشده‌اند با علامت ⚠️ مشخص شده و در `docs/open-questions.md` ثبت شده‌اند.

## راهنمای ستون‌ها

| ستون | معنا |
| --- | --- |
| ID | شناسه پایدار منبع (در کد به‌عنوان `sourceKey` استفاده می‌شود) |
| نوع ردپا | چه چیزی از این منبع قابل استخراج است |
| روش مجاز دسترسی | تنها مسیر قانونی/مجاز (OAuth رسمی، API عمومی، فایل خروجی کاربر، مجوز اندروید) |
| ورود/OAuth | آیا نیاز به ورود کاربر یا جریان OAuth دارد |
| مجوز اندروید | مجوزهای لازم در سطح اپ اندروید |
| فرمت داده | قالب بازگشتی/فایل |
| تاریخ واقعی ثبت‌نام | `real` = فیلد رسمی تاریخ ساخت حساب وجود دارد، `indirect` = فقط شاهد غیرمستقیم، `none` = هیچ |
| محدودیت/سیاست | سهمیه، نیاز به تأیید، سیاست Play یا ToS سرویس |
| پیچیدگی | `XS` خیلی کم، `S` کم، `M` متوسط، `L` زیاد، `XL` خیلی زیاد |
| روش تست | چگونگی تست واقعی در مرحله پیاده‌سازی |
| وضعیت | `P#` = فاز پیشنهادی نقشه راه، `GATED` = مشروط به اقدام/تأیید کاربر، `BLOCKED` = غیرقابل دسترسی مستقیم (جایگزین دارد)، `REJECTED` = آگاهانه رد شده (دلیل در `docs/limitations.md`) |

---

## دسته A — ایمیل و هویت‌های آنلاین

| ID | منبع | نوع ردپا | روش مجاز دسترسی | ورود/OAuth | مجوز اندروید | فرمت | تاریخ واقعی ثبت‌نام | محدودیت/سیاست | پیچیدگی | روش تست | وضعیت | جایگزین |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| A1 | Gmail (Gmail API) | تمام پیام‌ها: ثبت‌نام، تأیید ایمیل، بازیابی رمز، ورود جدید، هشدار امنیتی، رسید خرید | OAuth 2.0 رسمی + `gmail.readonly` | بله (OAuth محدود/Restricted) | `INTERNET` | JSON (REST) + `history.list` برای دلتا | indirect (قدیمی‌ترین پیام از هر دامنه) | اسکوپ `gmail.readonly` در دسته **Restricted** است؛ نیازمند OAuth Verification و در صورت ارسال داده به سرور، ارزیابی امنیتی سالانه (CASA). سهمیه: ۱٬۲۰۰٬۰۰۰ واحد/دقیقه برای پروژه، ۶٬۰۰۰ واحد/دقیقه/کاربر، ۸۰٬۰۰۰٬۰۰۰ واحد/روز؛ `messages.list`=۵ واحد، `messages.get`=۵، `history.list`=۲. اپ تأییدنشده محدود به ۱۰۰ کاربر تست است | L | Fixture ضبط‌شده + MockWebServer + تست حساب واقعی کاربر با لاگ سهمیه | P4 | A2 یا A6 |
| A2 | Gmail/Outlook/سایر (IMAP) | همان A1 بدون نیاز به تأیید اپ گوگل | IMAP با `AUTHENTICATE XOAUTH2` یا App Password | بله (OAuth یا رمز اپ) | `INTERNET` | RFC 822 / MIME | indirect | Basic Auth در Exchange Online در حال حذف کامل است (دسامبر ۲۰۲۶ طبق اعلام مایکروسافت)؛ گوگل از ۲۰۲۲ «less secure apps» را بسته؛ توکن OAuth هر ۳۶۰۰ ثانیه منقضی می‌شود؛ توکن IMAP گوگل نیازمند اسکوپ `https://mail.google.com/` | L | تست با سرور IMAP محلی (GreenMail/Dovecot) + Fixture | P5 | A6 |
| A3 | Outlook/Hotmail/M365 (Microsoft Graph) | پیام‌ها، رویدادها، مخاطبان، فایل‌ها، پروفایل حساب | MSAL Android + Graph delegated (`Mail.Read`, `User.Read`, `Files.Read`, `Calendars.Read`, `Contacts.Read`) | بله | `INTERNET` | JSON + `@odata.nextLink` + `$deltaToken` | ⚠️ indirect (`createdDateTime` برای حساب شخصی MSA معمولاً خالی است — نیازمند راستی‌آزمایی) | نیاز به App Registration؛ `signInActivity` و `signInLogs` فقط برای tenant سازمانی با لایسنس **Entra ID P1/P2** و نقش Reports Reader + `AuditLog.Read.All` — برای حساب شخصی در دسترس نیست | L | MockWebServer با پاسخ‌های ضبط‌شده Graph + تست حساب واقعی | P5 | C3 |
| A4 | سایر ارائه‌دهندگان ایمیل (Fastmail, Yahoo, iCloud, ProtonMail Bridge, میزبان شخصی) | همان A1 | IMAP با App Password/OAuth اختصاصی هر سرویس | بله | `INTERNET` | RFC 822 | indirect | هر سرویس قواعد OAuth خودش را دارد؛ ProtonMail فقط از طریق Bridge (دسکتاپ) → در اندروید عملاً ممکن نیست ⚠️ | L | تست با Dovecot محلی | P6 | A6 |
| A5 | تحلیل ساختار ایمیل (سرصفحه‌ها) | تاریخ اولین تماس با هر سرویس، مسیر ارسال، DKIM/SPF/ARC، List‑Unsubscribe → کشف خبرنامه‌ها و اشتراک‌ها | پردازش محلی روی داده‌های A1–A4/A6 | خیر (روی داده مجاز) | خیر | MIME headers | indirect (با دقت و روش مستند) | فقط روی ایمیل‌هایی که کاربر مجوز خواندن داده؛ هیچ حدس بدون شاهد ثبت نمی‌شود | M | Golden-file test روی سرصفحه‌های نمونه | P2 | — |
| A6 | فایل MBOX/EML (Takeout یا هر کلاینت) | کل آرشیو ایمیل به‌صورت آفلاین | واردکردن فایل توسط کاربر از طریق SAF | خیر (اقدام کاربر) | دسترسی SAF (بدون مجوز ذخیره‌سازی) | MBOX / EML / ZIP | indirect | هیچ؛ کاملاً آفلاین و کم‌ریسک‌ترین مسیر | S | Fixture واقعی MBOX + تست فایل خراب/بزرگ | P1 | — |
| A7 | Have I Been Pwned — Breach Search | شواهد وجود حساب در نقض داده + تاریخ نقض + نام سرویس | API v3 با `hibp-api-key` (کلید کاربر) | خیر (کلید، نه OAuth) | `INTERNET` | JSON | indirect (`BreachDate` تاریخ نقض است نه ثبت‌نام) | کلید پولی: Pwned 1 (۱۰ درخواست/دقیقه)، Pwned 2 (~۱۹$/ماه، ۵۰ rpm)، تا Pwned 5 (۱۰۰۰ rpm)؛ هدر `User-Agent` توصیفی الزامی؛ ۴۲۹ با `retry-after` | S | MockWebServer + تست ۴۰۴/۴۲۹/۴۰۱ | P8 | J2 |
| A8 | HIBP Pwned Passwords | آیا رمز در داده‌های لو رفته بوده (بدون ارسال رمز) | API عمومی رایگان با k‑anonymity (۵ کاراکتر اول SHA‑1) | خیر | `INTERNET` | متن (suffix:count) | none | رایگان و بدون محدودیت نرخ؛ فقط با اقدام صریح کاربر و فقط روی رمزی که خودش انتخاب کرده | XS | تست بردارهای اعلام‌شده رسمی | P8 (GATED) | — |
| A9 | Gravatar | وجود پروفایل عمومی برای هش ایمیل + نام/لینک‌های connected | API عمومی `en.gravatar.com/{md5}.json` | خیر | `INTERNET` | JSON | none | فقط برای ایمیل خود کاربر و با رضایت صریح؛ ریسک نشت ایمیل در URL → باید لاگ نشود | S | Fixture + بررسی عدم حضور ایمیل در لاگ | P8 (GATED) | — |
| A10 | کلیدهای OpenPGP (keys.openpgp.org) | ایمیل‌های تأییدشده کاربر + تاریخ ساخت کلید | API رسمی/عمومی (HKP + JSON lookup) | خیر | `INTERNET` | JSON (key document) | indirect (تاریخ ساخت کلید) | سرویس عمومی؛ فقط ایمیل‌هایی که کاربر تأیید کرده برگردانده می‌شود | S | Fixture + تست شبکه | P8 | — |
| A11 | آدرس‌های ایمیل alias (`+`/catch‑all) | شناسایی اینکه کدام alias برای کدام سرویس ساخته شده | پردازش محلی روی A1–A6 | خیر | خیر | derived | indirect | هیچ حدسی ساخته نمی‌شود؛ فقط الگوهای مشاهده‌شده در فیلد گیرنده | S | Unit test روی قواعد نرمال‌سازی | P2 | — |
| A12 | Mozilla Monitor / Firefox Monitor | هشدار نقض داده | ⚠️ وضعیت سرویس در زمان پژوهش تأیید نشد | — | — | — | — | — | — | — | REJECTED (نیازمند راستی‌آزمایی) | A7 |

## دسته B — حساب گوگل و سرویس‌های وابسته

| ID | منبع | نوع ردپا | روش مجاز دسترسی | ورود/OAuth | مجوز اندروید | فرمت | تاریخ واقعی ثبت‌نام | محدودیت/سیاست | پیچیدگی | روش تست | وضعیت | جایگزین |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| B1 | Google Sign‑In (Credential Manager) | هویت حساب گوگل روی دستگاه: ایمیل، نام، تصویر، `sub` | Credential Manager + Google Identity Services (جایگزین رسمی `GoogleSignIn` که **deprecated** و از `play-services-auth` حذف شده) | بله | `INTERNET` | JSON (credential response) | none | نیاز به OAuth Client ID اندروید (package + SHA‑1) و Web Client ID؛ جریان جدید `AuthorizationClient` برای اسکوپ‌های داده | S | تست روی device + Robolectric برای state | P4 | B2 |
| B2 | Google People API | مخاطبان گوگل، گروه‌ها، اطلاعات پروفایل کاربر | OAuth `contacts.readonly` | بله | `INTERNET` | JSON | none | اسکوپ Sensitive؛ سهمیه پیش‌فرض محدود؛ نیازمند OAuth verification برای انتشار | M | MockWebServer | P6 | B5 |
| B3 | YouTube Data API v3 | کانال کاربر (`channels.list?mine=true`)، **تاریخ ساخت کانال**، اشتراک‌ها (OPML در Takeout)، پلی‌لیست‌ها، ویدیوهای آپلودشده، لایک‌ها | OAuth `youtube.readonly` یا API Key برای داده عمومی | بله (برای `mine`) | `INTERNET` | JSON | **real** (فیلد `publishedAt` کانال) | سهمیه پیش‌فرض **۱۰٬۰۰۰ واحد/روز/پروژه**؛ هر متد ۱ تا ~۴۵۰ واحد؛ افزایش سهمیه نیازمند فرم audit؛ تاریخچه تماشا از API در دسترس نیست (فقط Takeout) | M | Fixture + تست سقف سهمیه | P6 | B5 |
| B4 | Google Calendar API | رویدادها: پرواز، هتل، دوره آموزشی، مصاحبه، قرارها + حساب‌های تقویم | OAuth `calendar.readonly` | بله | `INTERNET` | JSON (ICS در Takeout) | none | اسکوپ Sensitive/Restricted بسته به دامنه؛ سهمیه روزانه پروژه | M | Fixture | P7 | B5 |
| B5 | Google Takeout (واردکردن آرشیو) | بزرگ‌ترین منبع: Gmail MBOX، My Activity (Search/YouTube/Chrome)، Location History/Timeline، Chrome (Bookmarks/BrowserHistory/Autofill/Extensions)، Contacts (vCard/CSV)، Calendar (ICS)، Drive، Photos + JSON sidecar، Google Payments/Shopping History، Maps، Fit، Play Books، Groups، Chat | کاربر آرشیو را می‌سازد و اپ فایل را از SAF وارد می‌کند | اقدام کاربر | SAF | ZIP/TGZ حاوی JSON, MBOX, HTML, ICS, vCard, CSV | indirect (بسته به محصول) | Takeout از حساب می‌خواند نه از دستگاه: اگر Chrome sync خاموش باشد تاریخچه مرورگر در آرشیو نیست؛ آرشیو می‌تواند چند گیگابایت و چندتکه باشد؛ `archive_browser.html` فهرست قالب‌ها را می‌دهد | L | تست با آرشیوهای واقعی چندتکه + تست ZIP خراب/ناتمام | P1/P4 | — |
| B6 | Google Drive API | فایل‌ها، `createdTime`، اشتراک‌گذاری‌ها، نسخه‌ها | OAuth `drive.readonly` (یا اسکوپ‌های محدودتر پوشه‌ای) | بله | `INTERNET` | JSON | none | اسکوپ Restricted؛ سهمیه؛ دانلود فایل‌ها هزینه پهنای باند | L | Fixture | P9 | B5 |
| B7 | Google Play — تاریخچه خرید/اشتراک | خریدها، اشتراک‌ها، روش پرداخت | **API مصرف‌کننده وجود ندارد.** Play Billing فقط خریدهای داخل اپ خودمان را می‌دهد. مسیر مجاز: Takeout (Google Payments/Shopping History) یا ایمیل رسیدها | خیر | خیر | JSON/ایمیل | indirect | ادعای دسترسی API به خریدهای سایر اپ‌ها **نادرست** است | S | تست روی fixture رسید | P4 | B5 + A5 |
| B8 | Google Play Games Services | هویت بازی، دستاوردها، leaderboard | Play Games v2 sign‑in (OAuth داخلی) | بله | `INTERNET` | API client | indirect | فقط داده‌های بازی‌هایی که از PGS استفاده می‌کنند | M | تست device‑only | P9 | B5 |
| B9 | Health Connect (Fit/ساعت/اپ‌های سلامت) | رکوردهای سلامت + `dataOrigin` (کدام اپ نوشته) → شواهد نصب و فعالیت اپ‌ها | Health Connect SDK (اندروید ۱۴+ داخل سیستم؛ قبل از آن APK) | مجوز per‑data‑type با دیالوگ اختصاصی | `android.permission.health.READ_*` | رکوردهای SDK | none | داده حساس؛ نیازمند سیاست حریم خصوصی و رعایت سیاست Play؛ فقط در foreground قابل خواندن؛ اندروید ۹+ (API 28) | L | تست روی emulator با داده seed شده | P10 (GATED) | — |
| B10 | Google My Activity (بدون Takeout) | فعالیت جست‌وجو/یوتیوب/کروم | فقط از مسیر Takeout یا مشاهده کاربر در وب؛ **API عمومی ندارد** | — | — | — | — | — | — | — | BLOCKED | B5 |
| B11 | «اتصال اپ‌ها و سرویس‌های ثالث» حساب گوگل | فهرست اپ‌هایی که به حساب گوگل OAuth داده‌اند (شاهد قوی «اتصال حساب‌ها») | صفحه Security Checkup در وب؛ **API عمومی ندارد**؛ مسیر مجاز: کاربر خروجی/اسکرین‌شات/HTML ذخیره‌شده را وارد کند | اقدام کاربر | SAF | HTML/JSON دستی | none | ⚠️ قالب صفحه پایدار نیست → پارسر باید تحمل تغییر داشته باشد | M | Fixture صفحه ذخیره‌شده | P7 (GATED) | — |

## دسته C — مایکروسافت، Outlook و سرویس‌های مشابه

| ID | منبع | نوع ردپا | روش مجاز دسترسی | ورود/OAuth | مجوز اندروید | فرمت | تاریخ واقعی ثبت‌نام | محدودیت/سیاست | پیچیدگی | روش تست | وضعیت | جایگزین |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| C1 | Microsoft Graph `/me` | پروفایل، UPN، displayName | MSAL + `User.Read` | بله | `INTERNET` | JSON | ⚠️ indirect | برای MSA شخصی بسیاری از فیلدهای directory خالی است | S | Fixture | P5 | C3 |
| C2 | Microsoft Graph — sign‑in logs | تاریخ/مکان/اپ ورودها | `AuditLog.Read.All` | بله | `INTERNET` | JSON | none | **فقط tenant سازمانی + لایسنس Entra ID P1/P2 + نقش Reports Reader**؛ برای حساب شخصی در دسترس نیست | L | Fixture سازمانی | BLOCKED (برای حساب شخصی) | C3 |
| C3 | Privacy Dashboard مایکروسافت (Download your data) | تاریخچه فعالیت، مرور Edge، مکان، اپ‌ها/سرویس‌ها، Cortana، Xbox | کاربر آرشیو JSON را از privacy.microsoft.com می‌گیرد و وارد می‌کند | اقدام کاربر | SAF | ZIP حاوی JSON | indirect | زمان آماده‌سازی متغیر؛ قالب‌ها بدون نسخه رسمی منتشرشده ⚠️ | M | Fixture واقعی | P5 | — |
| C4 | Xbox / حساب بازی مایکروسافت | گیم‌اسکورها، بازی‌ها، دوستان | **API عمومی برای داده شخصی وجود ندارد**؛ مسیر: C3 یا ایمیل‌ها | — | — | — | none | APIهای غیررسمی Xbox خلاف ToS است → استفاده نمی‌شود | — | — | BLOCKED | C3 |
| C5 | Skype / Teams مصرف‌کننده | تاریخچه چت، مخاطبان | Graph (محدود برای مصرف‌کننده) یا خروجی رسمی حساب | بله/اقدام کاربر | `INTERNET` | JSON | none | دسترسی به تاریخچه پیام مصرف‌کننده از Graph محدود است ⚠️ | L | Fixture | P9 | C3 |
| C6 | LinkedIn (مالکیت مایکروسافت) | Connections.csv (نام، URL، ایمیل در صورت اشتراک، شرکت، موقعیت، **تاریخ اتصال**)، آرشیو کامل: پیام‌ها، پست‌ها، تاریخچه جست‌وجو، لاگین IP | «Get a copy of your data» → ZIP → واردکردن | اقدام کاربر | SAF | CSV/HTML/JSON | indirect (تاریخ اتصال واقعی است) | آرشیو کامل ۲۴–۷۲ ساعت؛ ایمیل فقط اگر طرف مقابل اجازه داده باشد؛ API عمومی برای داده شخصی ندارد | S | Fixture واقعی CSV | P5 | — |

## دسته D — شبکه‌های اجتماعی، پیام‌رسان‌ها، انجمن‌ها و پلتفرم‌های محتوا

| ID | منبع | نوع ردپا | روش مجاز دسترسی | ورود/OAuth | مجوز اندروید | فرمت | تاریخ واقعی ثبت‌نام | محدودیت/سیاست | پیچیدگی | روش تست | وضعیت | جایگزین |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| D1 | Facebook / Instagram (Meta) | پست‌ها، کامنت‌ها، فالوورها، لایک‌ها، تبلیغات/علاقه‌ها، لاگین‌ها | «Download your information» در Accounts Center → JSON/HTML → واردکردن | اقدام کاربر | SAF | ZIP با JSON/HTML | indirect | Graph API برای داده شخصی کاربر از سال ۲۰۲۰ در دسترس توسعه‌دهندگان عادی نیست؛ فایل‌ها تکه‌تکه‌اند (`followers_1.json`, `followers_2.json`)؛ رسانه‌ها حجم را بسیار زیاد می‌کنند | M | Fixture واقعی + تست چندتکه | P6 | — |
| D2 | X (Twitter) | پروفایل، پست‌ها، فالوها | API v2 با OAuth 2.0 user context (`/2/users/me`) یا آرشیو رسمی حساب | بله / اقدام کاربر | `INTERNET` | JSON / JS‑zip archive | indirect | **از ۲۰۲۶ مدل pay‑per‑use**: ~$0.01 برای user lookup و ~$0.005 برای هر پست خوانده‌شده؛ tier رایگان برای ثبت‌نام جدید وجود ندارد؛ سقف ماهانه reads | M | Fixture + تست ۴۰۲/۴۲۹ | P8 (GATED: کلید/اعتبار کاربر) | آرشیو account.js/tweets.js |
| D3 | Reddit | پروفایل، پست‌ها/کامنت‌ها، ساب‌ردیت‌ها، تاریخچه | OAuth2 رسمی (`/api/v1/me`, `/user/{me}/submitted`) | بله | `INTERNET` | JSON | indirect | tier رایگان ۱۰۰ QPM برای استفاده غیرتجاری، اما **از اواخر ۲۰۲۵ ثبت خودکار OAuth client بسته شده و نیازمند تأیید دستی است**؛ استفاده تجاری ~$0.24/۱۰۰۰ فراخوانی | M | Fixture + تست ۴۲۹ | P8 (GATED) | «request your data» ردیت |
| D4 | Mastodon / Fediverse | پروفایل، دنبال‌شوندگان، پست‌ها، سرور | API رسمی باز + OAuth (`/api/v1/accounts/verify_credentials`) | بله | `INTERNET` | JSON | indirect (`created_at` پروفایل اغلب real است ⚠️) | رایگان؛ نیاز به انتخاب سرور توسط کاربر؛ rate limit per‑instance | S | Fixture | P6 | — |
| D5 | Discord | حساب، سرورها، **اتصالات حساب‌های دیگر** (`/users/@me/connections`: Steam, Twitch, Xbox, …)، بسته داده کامل | OAuth رسمی برای `/users/@me*` + «Request all of my data» | بله / اقدام کاربر | `INTERNET` | JSON (بسته داده: `account/`, `messages/`, `servers/`, `activity/`) | indirect | Message Content intent برای بات‌ها privileged است؛ self‑bot خلاف ToS → استفاده نمی‌شود؛ بسته داده تا ۳۰ روز و فقط پیام‌های خود کاربر | M | Fixture + MockWebServer | P6 | — |
| D6 | Telegram | چت‌ها، کانال‌ها، مخاطبان | خروجی رسمی Telegram Desktop (`ChatExport_*/result.json` یا HTML) → واردکردن | اقدام کاربر | SAF | JSON/HTML | indirect | export فقط در دسکتاپ؛ Secret Chat هرگز صادر نمی‌شود؛ TDLib روی اندروید ممکن است ولی سنگین (XL) و نیازمند شماره کاربر | M (import) / XL (TDLib) | Fixture واقعی result.json | P6 | — |
| D7 | WhatsApp | چت‌های صادرشده | «Export chat» هر گفتگو → `.txt` (بدون رسانه تا ~۴۰٬۰۰۰ پیام) → پارسر | اقدام کاربر | SAF | TXT (+ZIP با رسانه) | indirect | هر گفتگو جدا صادر می‌شود؛ قالب متن بدون قرارداد رسمی و وابسته به زبان/نسخه ⚠️ → پارسر باید محافظه‌کار باشد | M | Fixture چندزبانه | P7 (GATED) | — |
| D8 | Signal | تاریخچه چت | **هیچ API/خروجی متنی رسمی ندارد**؛ فقط فایل پشتیبان رمزنگاری‌شده با passphrase | اقدام کاربر | SAF | `.backup` رمزنگاری‌شده | none | رمزگشایی پشتیبان توسط اپ ثالث ریسک امنیتی جدی دارد | XL | — | REJECTED (در `limitations.md`) | شواهد ایمیل/پیامک |
| D9 | Twitch | پروفایل، دنبال‌شونده‌ها، اشتراک‌ها، کلیپ‌ها | Helix API رسمی + OAuth user token | بله | `INTERNET` | JSON | indirect | نیاز به Client ID اپ؛ rate limit ۸۰۰/دقیقه ⚠️ | S | Fixture | P7 | — |
| D10 | TikTok | ویدیوها، لایک‌ها، پروفایل | «Download your data» → JSON → واردکردن | اقدام کاربر | SAF | ZIP/JSON | indirect | API عمومی برای داده شخصی ندارد؛ آماده‌سازی چند روز | M | Fixture | P7 | — |
| D11 | Snapchat | خاطرات، دوستان، لاگین‌ها | «My Data» export | اقدام کاربر | SAF | ZIP/JSON | indirect | — ⚠️ قالب نیازمند راستی‌آزمایی | M | Fixture | P8 | — |
| D12 | Pinterest / Tumblr / Reddit‑like forums | پین‌ها/پست‌ها/پروفایل | خروجی رسمی داده یا API باز (Tumblr API v2) | اقدام کاربر/بله | SAF/`INTERNET` | JSON/ZIP | indirect | Tumblr API v2 رسمی است ⚠️ نیازمند بررسی سهمیه جاری | M | Fixture | P8 | — |
| D13 | انجمن‌های Discourse | پست‌ها، موضوعات، فعالیت کاربر | API عمومی Discourse (`/u/{username}/activity.json`, `/u/{username}/emails` فقط برای خودش با توکن) | بله برای داده خصوصی | `INTERNET` | JSON | indirect | هر نمونه قوانین خودش را دارد؛ فقط نمونه‌هایی که کاربر حساب دارد | M | Fixture | P8 | شواهد ایمیل |
| D14 | انجمن‌های vBulletin/phpBB/XenForo | پروفایل و پست‌ها | **API رسمی ندارد** | — | — | — | none | خزش/اسکریپ چنین سایت‌هایی خلاف ToS است | — | — | BLOCKED | شواهد ایمیل (A1–A6) |

## دسته E — فروشگاه‌ها، پرداخت، سفارش‌ها و اشتراک‌ها

| ID | منبع | نوع ردپا | روش مجاز دسترسی | ورود/OAuth | مجوز اندروید | فرمت | تاریخ واقعی ثبت‌نام | محدودیت/سیاست | پیچیدگی | روش تست | وضعیت | جایگزین |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| E1 | Amazon | سفارش‌ها، آدرس‌ها، خریدهای دیجیتال، Alexa | «Your Account → Manage your data → Request your data» (ZIP شامل `Retail.OrderHistory.1.csv` و…) | اقدام کاربر | SAF | ZIP/CSV/JSON | indirect | صفحه «Download order reports» در ۲۰۲۳ حذف شد؛ آماده‌سازی از چند ساعت تا چند هفته؛ لینک منقضی می‌شود؛ API مصرف‌کننده وجود ندارد | S | Fixture واقعی CSV | P6 | — |
| E2 | eBay | سفارش‌ها، پیشنهادهای خرید | **API شخصی برای تاریخچه خرید مصرف‌کننده وجود ندارد**؛ مسیر: ایمیل رسیدها یا صفحه حساب توسط کاربر | — | — | — | indirect | Order API مخصوص فروشنده/اپ‌های تأییدشده است | — | — | BLOCKED | A1–A6 (رسیدها) |
| E3 | PayPal | تراکنش‌ها، اشتراک‌ها | دانلود CSV/Custom Statement از حساب توسط کاربر → واردکردن | اقدام کاربر | SAF | CSV | indirect | Transaction Reporting API نیازمند اعتبارنامه تجاری است؛ کلید API شخصی در اپ اندروید ذخیره نمی‌شود | S | Fixture | P7 | A5 (ایمیل‌های PayPal) |
| E4 | Google Payments / Shopping History | خریدها، اشتراک‌ها، رسیدها | بخشی از Takeout (B5) | اقدام کاربر | SAF | JSON/HTML | indirect | — | S | Fixture | P4 | — |
| E5 | Apple (App Store/iCloud/سرویس‌ها) | تاریخچه خرید، اشتراک‌ها، دستگاه‌ها، یادداشت‌ها/مخاطبان iCloud | privacy.apple.com → «Request a copy of your data» → ZIP | اقدام کاربر | SAF | ZIP (CSV/JSON/VCF/ICS) | indirect | تا ~۷ روز؛ ممکن است تأیید هویت اضافی بخواهد؛ محتوای iCloud Mail در آرشیو نیست | S | Fixture | P7 | — |
| E6 | تشخیص اشتراک از ایمیل | اشتراک‌های فعال/لغوشده، دوره تمدید، مبلغ، ارز | پارسر روی A1–A6 (رسیدها، «your subscription will renew», «payment failed») | خیر | خیر | derived | none | فقط الگوهای مستند؛ هر برچسب «اشتراک فعال» باید شاهد داشته باشد | M | Golden-file + precision/recall روی مجموعه برچسس‌خورده | P3 | — |
| E7 | فروشگاه‌های ایرانی/محلی (دیجی‌کالا، اسنپ، تپسی، دیوار، بازار، …) | سفارش‌ها، حساب‌ها، تراکنش‌ها | **API عمومی ندارند**؛ مسیر: ایمیل/پیامک دریافتی کاربر یا فایل خروجی که خود سرویس می‌دهد | اقدام کاربر | SAF | CSV/JSON/PDF | indirect | بسیاری فقط پنل وب دارند؛ درخواست داده بر اساس قانون حمایت از مصرف‌کننده/حریم خصوصی محلی ⚠️ نیازمند بررسی حقوقی | M | Fixture | P7 | A5 + I5 |
| E8 | صرافی‌های رمزارز (Binance, Coinbase, Kraken) | موجودی، تراکنش‌ها، تاریخ ساخت حساب | API رسمی با کلید **read‑only** که کاربر خودش می‌سازد | اقدام کاربر (کلید) | `INTERNET` | JSON | **real** در برخی (تاریخ ساخت حساب) | نگهداری کلید API صرافی روی دستگاه ریسک بالا؛ فقط read‑only؛ IP‑whitelist توصیه می‌شود | L | Fixture + تست انقضا/۴۰۱ | P10 (GATED) | E6/A5 |
| E9 | بانک‌ها (EU/PSD2) | تراکنش‌ها، حساب‌ها | PSD2 AIS فقط از طریق TPP دارای مجوز → **برای اپ اندروید مستقل مجاز نیست** | — | — | — | — | نیاز به مجوز تنظیم‌کننده (BaFin/EBA) | — | — | BLOCKED | واردکردن صورت‌حساب CSV/PDF توسط کاربر (E10) |
| E10 | صورت‌حساب بانکی/کارت (فایل کاربر) | تراکنش‌ها → کشف سرویس‌های پرداخت‌شونده | واردکردن CSV/PDF/MT940 از بانک کاربر | اقدام کاربر | SAF | CSV/PDF/MT940 | none | PDF پارس‌شدنی نیست به‌طور قابل اتکا → فقط CSV/MT940 پشتیبانی رسمی می‌شود؛ داده بسیار حساس | M | Fixture | P8 (GATED) | E6 |

## دسته F — بازی‌ها، لانچرها و پلتفرم‌های گیمینگ

| ID | منبع | نوع ردپا | روش مجاز دسترسی | ورود/OAuth | مجوز اندروید | فرمت | تاریخ واقعی ثبت‌نام | محدودیت/سیاست | پیچیدگی | روش تست | وضعیت | جایگزین |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| F1 | Steam | پروفایل، **`timecreated` (تاریخ واقعی ساخت حساب)**، بازی‌های مالکیت‌شده + ساعات، سطح، دوستان، دستاوردها | Steam Web API رسمی با کلید شخصی کاربر (`ISteamUser/GetPlayerSummaries/v2`, `IPlayerService/GetOwnedGames/v1`, `GetRecentlyPlayedGames`, `GetSteamLevel`) | اقدام کاربر (ساخت کلید) | `INTERNET` | JSON/XML/VDF | **real** | کلید رایگان از steamcommunity.com؛ پروفایل private/friends‑only داده کامل نمی‌دهد مگر کلید متعلق به همان SteamID باشد؛ ~۱۰۰٬۰۰۰ فراخوانی/روز ⚠️ (نیازمند راستی‌آزمایی) | S | Fixture + MockWebServer + تست پروفایل خصوصی | P7 | — |
| F2 | Epic Games | حساب، بازی‌ها، خریدها | **API عمومی شخصی ندارد**؛ «Download my data» رسمی | اقدام کاربر | SAF | ZIP/JSON | indirect | — ⚠️ | M | Fixture | P8 | A5 |
| F3 | PlayStation Network | تروفی‌ها، خریدها | **API عمومی ندارد**؛ درخواست داده Sony | اقدام کاربر | SAF | ZIP | indirect | — ⚠️ | M | Fixture | P8 | A5 |
| F4 | Nintendo Account | بازی‌ها، اشتراک Switch Online | **API عمومی ندارد**؛ درخواست داده | اقدام کاربر | SAF | ZIP/CSV | indirect | — ⚠️ | M | Fixture | P8 | A5 |
| F5 | Riot Games | اکانت، PUUID، تاریخچه بازی | Riot API رسمی با Development/Production key (کاربر باید کلید بسازد) | اقدام کاربر (کلید) | `INTERNET` | JSON | indirect | Development key محدود و منقضی‌شونده؛ Production key نیازمند ثبت‌نام توسعه‌دهنده؛ پیدا کردن PUUID نیازمند region + game name | L | Fixture | P9 (GATED) | A5 |
| F6 | Battle.net / Blizzard | حساب و پروفایل | OAuth رسمی فقط `oauth/userinfo` (داده بسیار محدود) | بله | `INTERNET` | JSON | none | دسترسی به بازی‌ها/کاراکترها از API عمومی ممکن نیست | M | Fixture | P9 | A5 |
| F7 | Roblox | **تاریخ ساخت حساب (`created`)**، نام کاربری، دوستان | API عمومی `users.roblox.com/v1/users/{id}` (بدون احراز هویت برای داده عمومی) | خیر | `INTERNET` | JSON | **real** | فقط داده عمومی؛ کشف ID از نام کاربری با endpoint جست‌وجوی رسمی ⚠️ محدودیت نرخ سخت‌گیرانه | S | Fixture | P8 | — |
| F8 | GOG / itch.io / Humble | کتابخانه بازی | itch.io API رسمی برای توسعه‌دهندگان؛ GOG/Humble → درخواست داده یا ایمیل رسید | اقدام کاربر | SAF/`INTERNET` | JSON/CSV | indirect | — ⚠️ | M | Fixture | P9 | A5 |
| F9 | Minecraft/Mojang | پروفایل، skin، UUID | زنجیره احراز هویت Xbox Live (مسیر لانچرها) — نیمه‌رسمی | اقدام کاربر | `INTERNET` | JSON | none | استفاده از جریان غیرمستند ریسک ToS دارد → فقط با اطلاع‌رسانی صریح و در صورت نبود جایگزین ⚠️ | L | Fixture | P10 (GATED) | A5 |
| F10 | لانچرهای اندروید/بازی‌های نصب‌شده | چه بازی‌هایی نصب بوده و از چه زمانی | I1 (PackageManager: `firstInstallTime`) | مجوز | package visibility | derived | indirect (تاریخ نصب روی **این** دستگاه، نه ثبت‌نام) | تفکیک «تاریخ نصب» از «تاریخ ثبت‌نام» الزامی است (قانون ۶) | S | Unit test با PackageManager fake | P3 | — |

## دسته G — آموزش، دانشگاه، ابزارهای توسعه و مخازن کد

| ID | منبع | نوع ردپا | روش مجاز دسترسی | ورود/OAuth | مجوز اندروید | فرمت | تاریخ واقعی ثبت‌نام | محدودیت/سیاست | پیچیدگی | روش تست | وضعیت | جایگزین |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| G1 | GitHub | پروفایل، **`created_at` (تاریخ واقعی ساخت حساب)**، ایمیل‌ها (با `user:email`)، مخازن، رویدادها، stars، گیت‌هاب اپ‌های نصب‌شده (`/user/installations`) | OAuth App یا Device Flow یا PAT کاربر | بله | `INTERNET` | JSON | **real** | ۵٬۰۰۰ درخواست/ساعت (authenticated)، ۶۰/ساعت بدون احراز هویت؛ Search API محدودیت جدا (~۳۰/دقیقه)؛ secondary rate limits | M | Fixture + MockWebServer + تست ۴۰۳/secondary limit | P6 | — |
| G2 | GitLab (gitlab.com و خودمیزبان) | پروفایل `created_at`، پروژه‌ها، رویدادها، MRها | OAuth2 یا PAT؛ پشتیبانی از نمونه خودمیزبان | بله | `INTERNET` | JSON | **real** | نمونه خودمیزبان نیاز به trust کاربر دارد؛ rate limit per‑instance | M | Fixture | P7 | — |
| G3 | Bitbucket | پروفایل، مخازن | OAuth consumer رسمی | بله | `INTERNET` | JSON | indirect ⚠️ | — | M | Fixture | P8 | — |
| G4 | Stack Exchange / Stack Overflow | پروفایل، **`creation_date`**، reputation، سایت‌های فعال | API رسمی + OAuth (یا key بدون OAuth با سهمیه کمتر) | بله | `INTERNET` | JSON (gzip) | **real** | ۳۰۰ درخواست/روز با OAuth، ۱۰٬۰۰۰ با app key ⚠️ نیازمند راستی‌آزمایی؛ throttle per‑IP | S | Fixture | P6 | — |
| G5 | جست‌وجوی کامیت GitHub با `author-email` | مخازن/سازمان‌هایی که ایمیل کاربر در تاریخچه عمومی آن‌هاست | Search API رسمی (`/search/commits?q=author-email:"…"`) | بله | `INTERNET` | JSON | indirect (تاریخ کامیت) | فقط روی داده عمومی؛ محدودیت نرخ search؛ باید با رضایت صریح کاربر و فقط برای ایمیل خودش | M | Fixture + تست ۴۲۲ | P8 (GATED) | — |
| G6 | npm | پکیج‌ها، نام کاربری، ایمیل عمومی | Registry API عمومی (`registry.npmjs.org/-/user/org.couchdb.user:{name}`) | خیر | `INTERNET` | JSON | indirect ⚠️ | داده عمومی | XS | Fixture | P8 | — |
| G7 | PyPI | پکیج‌ها، نویسنده | JSON API عمومی (`pypi.org/pypi/{project}/json`) | خیر | `INTERNET` | JSON | indirect | — | XS | Fixture | P9 | — |
| G8 | Docker Hub | ایمیج‌ها، نام کاربری | REST API عمومی | خیر | `INTERNET` | JSON | indirect ⚠️ | — | S | Fixture | P9 | — |
| G9 | Hugging Face | پروفایل، مدل‌ها، دیتاست‌ها | API عمومی `/api/users/{name}` + توکن برای داده خصوصی | بله (اختیاری) | `INTERNET` | JSON | indirect ⚠️ | — | S | Fixture | P9 | — |
| G10 | Codeforces / AtCoder | **تاریخ ثبت‌نام (Codeforces `registrationTime`)**، ریتینگ | Codeforces API رسمی (کلید اختیاری) | خیر | `INTERNET` | JSON | **real** | کلید API از حساب کاربر ساخته می‌شود | S | Fixture | P8 | — |
| G11 | Kaggle | پروفایل، کرنل‌ها، دیتاست‌ها | Kaggle API رسمی با API token کاربر | اقدام کاربر | `INTERNET` | JSON | indirect ⚠️ | — | M | Fixture | P9 | — |
| G12 | Coursera / edX / Udemy | دوره‌ها، گواهی‌ها، تاریخ ثبت‌نام | Coursera API فقط برای شرکای سازمانی؛ edX/Udemy → درخواست داده یا ایمیل گواهی | اقدام کاربر | SAF | JSON/CSV/PDF | indirect | API مصرف‌کننده وجود ندارد؛ PDF گواهی پارس‌پذیری ضعیفی دارد → از ایمیل استفاده می‌شود | S | Fixture | P7 | A5 |
| G13 | Moodle / LMS دانشگاه | دوره‌ها، تکالیف، آخرین ورود | Moodle Web Service با توکن کاربر (`core_webservice`) | اقدام کاربر (توکن) | `INTERNET` | JSON/XML | indirect | هر نمونه تنظیمات متفاوت دارد؛ توکن باید توسط کاربر از صفحه Security keys گرفته شود | M | Fixture | P9 (GATED) | A5 |
| G14 | Notion / Figma / Slack / Trello / Jira | حساب‌های کاری، فضای‌های کاری، فعالیت | OAuth رسمی هر سرویس (Notion `search`, Figma `/v1/me`, Slack `users.profile:read`, Trello `/1/members/me`, Atlassian OAuth 2.0 (3LO)) | بله | `INTERNET` | JSON | indirect | هر کدام app registration جدا می‌خواهند؛ Slack برای Workspace apps نیاز به نصب توسط ادمین دارد | L | Fixture per service | P9 | — |
| G15 | دامنه‌های ثبت‌شده کاربر | **تاریخ ثبت دامنه (real)**، رجیسترار، وضعیت | RDAP رسمی (`rdap.org`) / WHOIS | خیر | `INTERNET` | JSON | **real** | RDAP جایگزین رسمی WHOIS است؛ بسیاری از دامنه‌ها WHOIS privacy دارند؛ باید فهرست دامنه‌ها را کاربر بدهد یا از ایمیل استخراج شود | S | Fixture RDAP | P8 | — |

## دسته H — مرورگرها، تاریخچه وب، بوکمارک، کوکی و رمزهای ذخیره‌شده

| ID | منبع | نوع ردپا | روش مجاز دسترسی | ورود/OAuth | مجوز اندروید | فرمت | تاریخ واقعی ثبت‌نام | محدودیت/سیاست | پیچیدگی | روش تست | وضعیت | جایگزین |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| H1 | Chrome sync (از Takeout) | `BrowserHistory.json`، `Bookmarks.json`، `Autofill` (آدرس‌ها)، `Extensions`، تنظیمات | Takeout → واردکردن (B5) | اقدام کاربر | SAF | JSON | none | فقط اگر Chrome sync روشن بوده؛ Takeout از حساب می‌خواند نه دستگاه | M | Fixture واقعی | P4 | — |
| H2 | Google Password Manager | **فهرست سایت‌هایی که کاربر در آن حساب دارد** (url + username + password) | خروجی CSV رسمی از passwords.google.com/Chrome → واردکردن | اقدام کاربر | SAF | CSV | indirect | فایل **متن‌ساده** و بسیار حساس است → سیاست اپ: فقط `url` و `username` ذخیره می‌شود، ستون `password` هرگز ذخیره/لاگ/نمایش داده نمی‌شود (ADR‑004) | S | Fixture + تست اینکه هیچ رمزی در DB/لاگ نیست | P6 (GATED) | — |
| H3 | Firefox (دسکتاپ) | بوکمارک‌ها (HTML)، تاریخچه (`places.sqlite`)، رمزها (CSV) | کاربر فایل را صادر و منتقل می‌کند → SAF | اقدام کاربر | SAF | HTML/SQLite/CSV | none | Firefox for Android امکان صدور ندارد؛ `places.sqlite` ممکن است قفل باشد → نیاز به کپی | M | Fixture | P8 | H1 |
| H4 | Safari (Apple) | بوکمارک‌ها | خروجی بوکمارک + privacy.apple.com | اقدام کاربر | SAF | HTML/ZIP | none | تاریخچه مرورگر صادر نمی‌شود | S | Fixture | P8 | — |
| H5 | Edge | علاقه‌مندی‌ها، رمزها (CSV)، تاریخچه + privacy dashboard | خروجی رسمی یا C3 | اقدام کاربر | SAF | HTML/CSV/JSON | none | — | S | Fixture | P7 | — |
| H6 | مرورگرهای اندروید (Chrome/Firefox/Brave روی دستگاه) | تاریخچه، کوکی، رمزها به‌صورت مستقیم | **غیرممکن**: داده در پوشه خصوصی اپ دیگر است و با scoped storage در دسترس نیست | — | — | — | — | هر تلاش برای خواندن آن = دسترسی غیرمجاز (قانون ۱۱) | — | — | BLOCKED | H1/H2 |
| H7 | کوکی‌های صادرشده با افزونه (`cookies.txt`) | دامنه‌های لاگین‌شده (شاهد قوی وجود حساب) | فایل Netscape cookies.txt توسط کاربر | اقدام کاربر | SAF | TXT | none | **مقادیر کوکی = توانایی ربودن نشست.** سیاست اپ: فقط `domain` و `expiration` ذخیره می‌شود؛ `value` هرگز (ADR‑004) | S | Fixture + تست عدم ذخیره value | P8 (GATED) | — |
| H8 | OPML اشتراک‌های RSS (Feedly/Inoreader/NewsBlur) | فهرست سایت‌ها و خبرنامه‌های دنبال‌شده | خروجی OPML یا API رسمی OAuth (Feedly/Inoreader) | بله/اقدام کاربر | `INTERNET` | OPML/XML/JSON | none | — | S | Fixture | P8 | — |

## دسته I — دستگاه اندروید (محلی، با مجوز)

| ID | منبع | نوع ردپا | روش مجاز دسترسی | ورود/OAuth | مجوز اندروید | فرمت | تاریخ واقعی ثبت‌نام | محدودیت/سیاست | پیچیدگی | روش تست | وضعیت | جایگزین |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| I1 | PackageManager — بسته‌های نصب‌شده | چه اپ‌هایی نصب‌اند، **`firstInstallTime`**، `lastUpdateTime`، نسخه، امضا | API رسمی اندروید | خیر | package visibility: `<queries>` هدفمند یا `QUERY_ALL_PACKAGES` | derived | none (تاریخ نصب روی این دستگاه) | از اندروید ۱۱ فهرست اپ‌ها فیلتر است؛ `QUERY_ALL_PACKAGES` در Play **ممنوع مگر عملکرد اصلی اپ به آن نیاز داشته باشد** + Declaration Form + ویدیو؛ Play فهرست اپ‌های نصب‌شده را داده شخصی حساس می‌داند؛ فروش/استفاده برای تبلیغات ممنوع | M | Unit test با PackageManager fake + تست fallback | P3 | I1b |
| I1b | `<queries>` هدفمند | همان I1 برای ~۲۰۰ پکیج شناخته‌شده سرویس‌ها | manifest `<queries>` | خیر | بدون مجوز خاص | derived | none | مسیر fallback اگر Declaration Form رد شود؛ پوشش کمتر ولی بدون ریسک Play | S | تست manifest merger | P3 | — |
| I2 | UsageStatsManager | خط زمانی استفاده از اپ‌ها (`queryEvents`: MOVE_TO_FOREGROUND با timestamp)، مدت استفاده | API رسمی | خیر | `PACKAGE_USAGE_STATS` (special app‑op؛ کاربر در Settings فعال می‌کند) | derived | none | از runtime permission نیست → باید کاربر را به `ACTION_USAGE_ACCESS_SETTINGS` هدایت کرد؛ داده بسیار حساس؛ نیازمند افشای برجسته و رضایت؛ فقط با هدف نمایش به خود کاربر | L | تست device‑only + fake events در Robolectric | P3 (GATED) | I1 |
| I3 | ContactsContract | مخاطبان + **نام/نوع حساب‌های همگام‌شده روی دستگاه** (raw contacts دارای account name/type هستند) → کشف ایمیل‌های دیگر کاربر | API رسمی | خیر | `READ_CONTACTS` | derived | none | مجوز sensitive؛ فقط برای شناسایی هویت‌های **خود کاربر** استفاده می‌شود؛ مخاطبان اشخاص ثالث ذخیره انبوه نمی‌شوند (ADR‑011) | M | Fixture + Robolectric | P3 (GATED) | — |
| I4 | CalendarContract | تقویم‌ها (نام حساب هر تقویم)، رویدادها، شرکت‌کنندگان | API رسمی | خیر | `READ_CALENDAR` | derived | none | مجوز sensitive | M | Fixture | P7 (GATED) | B4 |
| I5 | پیامک‌ها | OTP، کد تأیید، هشدار بانک، پیام سرویس‌ها | `READ_SMS` در Play **فقط برای اپ پیش‌فرض پیامک** مجاز است → برای ما ممکن نیست | — | — | — | — | سیاست Play: اپ باید default SMS/Phone/Assistant handler باشد + Declaration Form؛ SMS Retriever API فقط OTP اپ خودمان را می‌دهد | — | — | BLOCKED | **جایگزین مجاز:** کاربر با اپ دلخواه خودش خروجی `SMS Backup & Restore` (XML) می‌گیرد و ما فایل را وارد می‌کنیم (I5b) |
| I5b | فایل XML پیامک‌ها (SMS Backup & Restore) | همان I5 بدون مجوز حساس | واردکردن فایل توسط کاربر از SAF | اقدام کاربر | SAF | XML | indirect | قالب مستند و پایدار؛ فقط پیام‌های کاربر؛ پارسر باید فقط فرستنده/متن/زمان را بگیرد و متن را طبق سیاست نگه دارد | S | Fixture واقعی XML | P6 | — |
| I6 | Call logs | تماس‌ها | `READ_CALL_LOG` فقط برای default Phone/Assistant handler → ممکن نیست | — | — | — | — | همان سیاست Play | — | — | BLOCKED | واردکردن CSV توسط کاربر (I6b) |
| I6b | فایل CSV تماس‌ها | همان I6 | واردکردن فایل کاربر | اقدام کاربر | SAF | CSV | indirect | — | XS | Fixture | P9 | — |
| I7 | NotificationListenerService | اعلان‌های ورود جدید، OTP، سفارش، هشدار امنیتی → شواهد بلادرنگ فعالیت حساب | API رسمی | خیر | `BIND_NOTIFICATION_LISTENER_SERVICE` (کاربر در Settings دو مرحله‌ای فعال می‌کند) | derived | none | بسیار حساس؛ Play افشای برجسته و رضایت جدا می‌خواهد؛ سیاست اپ: **خاموش به‌صورت پیش‌فرض**، فقط allowlist پکیج که کاربر انتخاب می‌کند، فقط `package + title + timestamp` ذخیره می‌شود و متن بدنه دور ریخته می‌شود (ADR‑006) | L | تست service با fake notifications + تست redaction | P10 (GATED) | I5b/A5 |
| I8 | Storage Access Framework | واردکردن هر فایل/پوشانه‌ای که کاربر انتخاب می‌کند | `ACTION_OPEN_DOCUMENT` / `ACTION_OPEN_DOCUMENT_TREE` + `takePersistableUriPermission` | اقدام کاربر | بدون مجوز ذخیره‌سازی | هر قالب | — | grant پیش‌فرض تا ریستارت اعتبار دارد؛ با `takePersistableUriPermission` پایدار می‌شود؛ از اندروید ۱۱ picker ریشه داخلی و `Download` را نمی‌دهد؛ اگر فایل جابه‌جا/حذف شود grant از دست می‌رود؛ `MANAGE_EXTERNAL_STORAGE` استفاده نمی‌شود (Play آن را محدود کرده) | M | تست device + fake ContentResolver | P1 | — |
| I9 | AccountManager | حساب‌های روی دستگاه | از API 26 فقط حساب‌هایی که authenticator آن‌ها همان امضا را دارد یا کاربر با `newChooseAccountIntent()` انتخاب کرده | اقدام کاربر | `GET_ACCOUNTS` (عملاً بی‌اثر) | derived | none | `GET_ACCOUNTS` دیگر کافی نیست؛ انتظار دریافت فهرست کامل حساب‌ها **نادرست** است | S | تست device | P4 | B1 |
| I10 | مجوزهای اعطاشده به سایر اپ‌ها | «اپ X مجوز دوربین/موقعیت/میکروفون دارد» | `PackageManager.checkPermission(perm, pkg)` + `getPackageInfo(GET_PERMISSIONS)` | خیر | package visibility (I1/I1b) | derived | none | فقط برای پکیج‌های قابل مشاهده؛ نیازمند همان استراتژی package visibility | M | Unit test | P3 | — |
| I11 | شناسه تبلیغات (Advertising ID) | شناسه بازاریابی | `AdvertisingIdClient` | خیر | `com.google.android.gms.permission.AD_ID` | — | none | سیاست Play: اگر برای تبلیغات استفاده نشود نباید درخواست شود؛ کاربر می‌تواند ریست/opt‑out کند؛ ارزش واقعی برای این محصول پایین و ریسک سیاستی بالا | — | — | REJECTED | — |
| I12 | ANDROID_ID (SSAID) | شناسه دستگاه per‑app‑signing | `Settings.Secure` | خیر | خیر | — | none | از اندروید ۸ به ازای هر امضا متفاوت است → برای همبستگی بین‌سرویسی بی‌فایده است | — | — | REJECTED | — |
| I13 | پشتیبان‌گیری سیستم (Google One/Device backup) | داده اپ‌های دیگر | **غیرقابل دسترسی برای اپ ثالث** | — | — | — | — | — | — | — | BLOCKED | B5 |
| I14 | Digital Wellbeing / App Ops history | تاریخچه دسترسی اپ‌ها به مجوزها | API عمومی ندارد (فقط سیستم) | — | — | — | — | — | — | — | BLOCKED | I2 |

## دسته J — شواهد عمومی و تاریخی (فقط برای دارایی‌های خود کاربر)

| ID | منبع | نوع ردپا | روش مجاز دسترسی | ورود/OAuth | مجوز اندروید | فرمت | تاریخ واقعی ثبت‌نام | محدودیت/سیاست | پیچیدگی | روش تست | وضعیت | جایگزین |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| J1 | Wayback Machine (CDX API) | **اولین تاریخ آرشیوشده پروفایل کاربر** → شاهد قوی وجود حساب در آن تاریخ | API عمومی رسمی `web.archive.org/cdx/search/cdx` | خیر | `INTERNET` | متن/JSON | indirect (با دقت timestamp) | رایگان؛ rate limit عرفی؛ باید فقط URLهایی پرسیده شود که کاربر تأیید کرده مربوط به خودش است | S | Fixture CDX + تست ۴۲۹ | P8 (GATED) | — |
| J2 | RDAP/WHOIS | تاریخ ثبت/به‌روزرسانی/انقضای دامنه کاربر | RDAP رسمی | خیر | `INTERNET` | JSON | **real** | privacy proxy تاریخ را پنهان نمی‌کند اما هویت را پنهان می‌کند | S | Fixture | P8 | — |
| J3 | Certificate Transparency (crt.sh) | زیردامنه‌های دامنه کاربر | API عمومی CT logs | خیر | `INTERNET` | JSON | none | ارزش شخصی کم؛ فقط اگر کاربر دامنه دارد | S | Fixture | P9 | — |
| J4 | Keybase proofs | لینک حساب‌های اجتماعی به یک هویت | API عمومی `keybase.io/_/api/1.0/user/lookup.json` | خیر | `INTERNET` | JSON | none | ⚠️ سرویس در حالت نگهداری است؛ ممکن است حذف شود → به‌عنوان منبع «احتمالی» و با fallback | S | Fixture | P9 (GATED) | — |
| J5 | About.me / Linktree / Carrd / وب‌سایت شخصی | هویت‌های منتشرشده توسط خود کاربر | کاربر URL می‌دهد؛ اپ صفحه را می‌گیرد و لینک‌ها را استخراج می‌کند | اقدام کاربر | `INTERNET` | HTML | none | فقط URLهای اعلام‌شده توسط کاربر؛ بدون خزش خودکار | M | Fixture HTML | P8 (GATED) | — |
| J6 | **شمارش نام کاربری در سایت‌ها (username enumeration)** | حدس وجود حساب از روی نام کاربری | خزش/پروب خودکار صفحات ورود یا ثبت‌نام | — | — | — | — | **خلاف ToS و robots.txt اکثر سایت‌ها، نرخ خطای مثبت بالا، و از نظر قانونی پرریسک (دسترسی غیرمجاز).** طبق قانون ۵ و ۱۱ پیاده‌سازی نمی‌شود | — | — | REJECTED | جایگزین: کاربر خودش لینک پروفایل را وارد می‌کند یا اپ صفحه را **با کلیک کاربر** در مرورگر باز می‌کند و نتیجه را کاربر دستی ثبت می‌کند (بدون پروب خودکار) |
| J7 | خزش موتورهای جست‌وجو | کشف پروفایل‌ها | — | — | — | — | — | خلاف ToS موتورهای جست‌وجو | — | — | REJECTED | کاربر URL را دستی اضافه می‌کند |
| J8 | سرویس‌های پولی leak (DeHashed, LeakCheck, IntelX) | داده نشتی | API پولی | — | — | — | — | ToS محدودکننده، داده اغلب از منابع غیرقانونی، ریسک حقوقی و اخلاقی | — | — | REJECTED | A7 (HIBP با کلید کاربر) |

## دسته K — درخواست رسمی داده از سرویس‌ها (ستون فقرات استراتژیک)

| ID | منبع | نوع ردپا | روش مجاز دسترسی | ورود/OAuth | مجوز اندروید | فرمت | تاریخ واقعی ثبت‌نام | محدودیت/سیاست | پیچیدگی | روش تست | وضعیت | جایگزین |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| K1 | «دستیار درخواست داده» (Data Subject Request Assistant) | هر سرویسی که API ندارد: کاربر از داخل اپ مسیر دقیق درخواست داده (GDPR Art. 15/20 یا صفحه رسمی سرویس) را می‌گیرد، نامه آماده دریافت می‌کند، و پس از رسیدن آرشیو آن را وارد می‌کند | صفحه راهنمای داخلی + ایمیل پیش‌نویس‌شده + SAF import | اقدام کاربر | `INTERNET`, SAF | — | indirect | GDPR Art. 20: داده باید در قالب ساختاریافته، رایج و ماشین‌خوان (JSON/CSV/XML) و ظرف یک ماه داده شود؛ Art. 15 حق دسترسی گسترده‌تر است و داده استنتاج‌شده را هم شامل می‌شود | M | تست unit برای کاتالوگ + E2E دستی با ۳ سرویس واقعی | P2 | — |
| K2 | Importer عمومی آرشیوهای ناشناخته | تشخیص ساختار ZIP/JSON/CSV/MBOX و نگاشت با کمک کاربر | SAF + پارسرهای عمومی + «mapping assistant» | اقدام کاربر | SAF | ZIP/JSON/CSV/XML/HTML/MBOX | indirect | پارسر عمومی بدون نگاشت کاربر فقط «شواهد خام با منشأ» تولید می‌کند، نه ادعای تفسیرشده | L | fuzzing + golden files | P3 | — |
| K3 | کاتالوگ قالب‌های رسمی صادرات | دانش اینکه هر سرویس چه فایل‌هایی می‌دهد (Takeout, Meta, Amazon, Apple, LinkedIn, Discord, Microsoft, Snapchat, TikTok, …) | مستندات داخلی اپ (بدون تماس شبکه) | خیر | خیر | — | — | قالب‌ها تغییر می‌کنند → هر پارسر باید نسخه‌بندی و تحمل تغییر داشته باشد و شکست را صادقانه گزارش کند | M | Fixture per service + تست regression | P2→P8 | — |

## دسته L — دسته‌های کشف‌شده در پژوهش (خارج از فهرست اولیه)

| ID | منبع | نوع ردپا | روش مجاز دسترسی | ورود/OAuth | مجوز اندروید | فرمت | تاریخ واقعی ثبت‌نام | محدودیت/سیاست | پیچیدگی | روش تست | وضعیت | جایگزین |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| L1 | Spotify | پروفایل، پلی‌لیست‌ها، **recently played با timestamp دقیق**، دنبال‌شونده‌ها | Web API رسمی + OAuth | بله | `INTERNET` | JSON | indirect | `/me/player/recently-played` فقط ~۲۰ مورد آخر؛ سهمیه نرم | S | Fixture | P7 | — |
| L2 | Last.fm | **`registered` (تاریخ واقعی ساخت حساب)**، scrobble‌ها، هنرمندان | API رسمی با کلید کاربر | اقدام کاربر | `INTERNET` | JSON/XML | **real** | کلید رایگان | S | Fixture | P8 | — |
| L3 | Strava | ورزش‌ها، مسیرها، باشگاه‌ها | API رسمی OAuth | بله | `INTERNET` | JSON | indirect | نیاز به app approval؛ rate limit ۱۰۰/۱۵دقیقه و ۱۰۰۰/روز ⚠️ | M | Fixture | P9 | B9 |
| L4 | Fitbit | پروفایل با **`memberSince`**، فعالیت‌ها | API رسمی OAuth 2.0 | بله | `INTERNET` | JSON | **real** | از ۲۰۲۵ مهاجرت به Google account؛ ثبت اپ جدید ممکن است محدود باشد ⚠️ | M | Fixture | P9 | B9 |
| L5 | Google Authenticator export (TOTP) | **فهرست سرویس‌هایی که کاربر رویشان 2FA دارد** (issuer + account) | کاربر QR «Export accounts» را در اپ ما اسکن می‌کند (payload `otpauth-migration://` = protobuf رسمی گوگل) | اقدام کاربر | `CAMERA` (فقط برای اسکن) | protobuf/base64 | none | **secret هرگز ذخیره/لاگ نمی‌شود**؛ فقط issuer و account name (ADR‑004) | M | Fixture protobuf + تست عدم نشت secret | P7 (GATED) | — |
| L6 | صادرات مدیریت‌کننده رمز (Bitwarden/1Password/KeePass/Dashlane/Enpass/NordPass) | فهرست سایت‌ها + نام کاربری → نقشه حساب‌ها | فایل خروجی رسمی هر ابزار | اقدام کاربر | SAF | JSON/CSV/1PUX/KDBX | indirect | KDBX نیاز به رمز master دارد → رمزگشایی در دستگاه؛ **فقط metadata ذخیره می‌شود، رمزها هرگز** | M | Fixture per format + تست redaction | P6 (GATED) | H2 |
| L7 | Google Takeout — Autofill/آدرس‌ها | آدرس‌های postal کاربر، شماره‌ها | B5 | اقدام کاربر | SAF | JSON | none | بسیار حساس؛ فقط با رضایت جداگانه | S | Fixture | P7 (GATED) | — |
| L8 | Alexa (در آرشیو Amazon) | درخواست‌های صوتی، دستگاه‌ها | E1 | اقدام کاربر | SAF | ZIP/JSON | none | حجم زیاد و حساس | M | Fixture | P8 | — |
| L9 | Xing / سرویس‌های آلمانی (Telekom, Otto, Zalando, Check24, Klarna, Deutsche Post) | حساب‌ها، سفارش‌ها، قراردادها | خروجی رسمی داده یا K1 (درخواست GDPR) | اقدام کاربر | SAF | ZIP/CSV/PDF | indirect | کاربر در آلمان است → GDPR کاملاً اعمال می‌شود؛ بسیاری فقط پرتال وب دارند | M | Fixture | P8 | K1 |
| L10 | اپراتور مخابرات / سیم‌کارت | تاریخ فعال‌سازی خط، تغییر اپراتور | پرتال کاربر یا قبض/ایمیل | اقدام کاربر | SAF | PDF/CSV | **real** (در قبض/پرتال) | API عمومی ندارد | S | Fixture | P9 | A5 |
| L11 | Passkeyها / WebAuthn | چه سرویس‌هایی passkey دارند | **API برای شمارش passkey وجود ندارد** (Credential Manager فقط برای ساخت/استفاده است) | — | — | — | — | — | — | — | BLOCKED | L5/L6 |
| L12 | Google Play — نظرات/امتیازهای کاربر | چه اپ‌هایی را کاربر ارزیابی کرده | API عمومی ندارد | — | — | — | — | — | — | — | BLOCKED | A5 (ایمیل‌های «thanks for your review») |
| L13 | داده هدف‌گیری تبلیغاتی (Google Ad Center / Meta Ad Preferences / TikTok Ads) | علایق تبلیغاتی، تبلیغ‌دهندگانی که داده آپلود کرده‌اند | فقط از طریق آرشیو رسمی حساب (Meta: «Ads information»، Google: Takeout) | اقدام کاربر | SAF | JSON/HTML | none | API عمومی ندارد | M | Fixture | P8 | — |
| L14 | فایل‌های محلی کاربر (ZIP/HTML/JSON/CSV/XML/MBOX/PDF/SQLite) | هر چیزی که کاربر در دستگاه دارد: قبض‌ها، بلیت‌ها، اسناد، آرشیو چت | SAF import + پارسرهای فرمت‌محور | اقدام کاربر | SAF | همه قالب‌های بالا | indirect | PDF به‌طور قابل اتکا پارس نمی‌شود → فقط متن قابل استخراج با منشأ «unreliable»؛ SQLite/`places.sqlite` نیاز به کپی دارد | L | fuzzing + golden files | P1/P3 | — |

---

## جمع‌بندی سه‌دسته‌بندی (خلاصه)

| دسته | منابع |
| --- | --- |
| **قابل پیاده‌سازی مستقیم و قانونی** (بدون اقدام اضافه کاربر جز ورود/مجوز) | A1, A2, A3, A5, A7*, A8, A10, B1, B2, B3, B4, B6, C1, D4, D5, D9, F1*, G1, G2, G3, G4, G6, G7, G10, G15, H1, I1, I1b, I10, J1, J2, J3, L1, L2 (*با کلید کاربر) |
| **قابل پیاده‌سازی با اقدام یا تأیید کاربر** (واردکردن فایل، ساخت کلید، فعال‌سازی مجوز خاص، رضایت جداگانه) | A6, A9, B5, B7, B8, B9, B11, C3, C6, D1, D2, D3, D6, D7, D10, D11, D12, D13, E1, E3, E4, E5, E6, E7, E8, E10, F2, F3, F4, F5, F7, F8, G5, G8, G9, G11, G12, G13, G14, H2, H3, H4, H5, H7, H8, I2, I3, I4, I5b, I6b, I7, I8, J4, J5, K1, K2, K3, L5, L6, L7, L8, L9, L10, L13, L14 |
| **غیرقابل دسترسی مستقیم + جایگزین مجاز** | C2 (→C3), C4 (→C3), B10 (→B5), D8 (رد شده؛ →شواهد ایمیل), E2 (→ایمیل رسیدها), E9 (→E10), H6 (→H1/H2), I5 (→I5b), I6 (→I6b), I13 (→B5), I14 (→I2), L11 (→L5/L6), L12 (→A5), D14 (→شواهد ایمیل) |
| **آگاهانه رد شده (به دلیل قانون/سیاست/ریسک)** | A12, I11, I12, J6, J7, J8, D8 → دلیل کامل در `docs/limitations.md` |

## خلاصه منابع دارای «تاریخ واقعی ثبت‌نام»

این فیلد در مدل داده با `DateKind.REGISTERED` و `DatePrecision.EXACT` و `Confidence.CONFIRMED` ذخیره می‌شود و **هرگز** با تاریخ مشاهده یا دریافت اشتباه گرفته نمی‌شود:

| منبع | فیلد رسمی |
| --- | --- |
| GitHub | `user.created_at` |
| GitLab | `user.created_at` |
| Stack Exchange | `user.creation_date` |
| Steam | `players[].timecreated` |
| Roblox | `user.created` |
| Last.fm | `user.registered` |
| Fitbit | `user.memberSince` |
| YouTube (کانال) | `channel.publishedAt` |
| RDAP/WHOIS (دامنه) | `events[eventAction=registration].eventDate` |
| Codeforces | `registrationTime` |
| قبض/پرتال اپراتور | تاریخ فعال‌سازی خط |

بقیه منابع فقط **شاهد غیرمستقیم** می‌دهند (مثل قدیمی‌ترین ایمیل از دامنه سرویس، اولین پست، تاریخ نقض داده، اولین تصویر آرشیوشده در Wayback). این‌ها با `DateKind.OBSERVED` یا `ESTIMATED` + `derivation` ثبت می‌شوند.

## ارجاعات رسمی استفاده‌شده در پژوهش

فهرست کامل پیوندها و تاریخ دسترسی در `docs/research/references.md`.
