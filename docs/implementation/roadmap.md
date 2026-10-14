# نقشه راه پیاده‌سازی — Implementation Roadmap

> مرحله ۱ — ۲۰۲۶‑۱۰‑۱۴ · هر فاز یک خروجی **مشخص و تست‌پذیر** دارد و طبق قانون ۱۶/۱۷ `Rules` در پایان کامیت و پوش می‌شود.
>
> برآورد اندازه فازها با مقیاس نسبی است (`S` ≈ چند روز، `M` ≈ ۱–۲ هفته، `L` ≈ ۳–۴ هفته، `XL` ≈ بیش از یک ماه) و نه تعهد زمانی قطعی؛ چون سرعت واقعی باید از فاز P0 اندازه‌گیری شود.

---

## اصول حاکم بر ترتیب فازها

1. **اول زیرساخت صداقت:** مدل داده با provenance و `TimeFact` قبل از هر connector ساخته می‌شود، چون بعداً اضافه‌کردن منشأ به داده موجود ممکن نیست.
2. **اول منابع آفلاین و کم‌ریسک:** import فایل و منابع دستگاهی (بدون OAuth، بدون سیاست Play) بیشترین شاهد را با کمترین ریسک می‌دهند → ارزش سریع و قابل تست.
3. **هر connector = یک PR کوچک** با fixture، contract test و صفحه وضعیت خودش (قانون ۱۳).
4. **هیچ فاز «UI پوسته» وجود ندارد:** هر صفحه UI باید داده واقعی (یا fixture مستند) نشان دهد و وضعیت‌های خطا را پوشش دهد.
5. **دروازه امنیتی:** فاز ۰ تست‌های redaction را می‌سازد؛ از فاز ۱ به بعد هر PR باید آن‌ها را پاس کند.

---

## P0 — اسکلت پروژه و دروازه‌های کیفیت `M`

**هدف:** مخزن قابل ساخت، تست و CI باشد؛ تصمیم‌های فنی با اجرای واقعی راستی‌آزمایی شوند.

| خروجی | توضیح |
| --- | --- |
| ساختار Gradle چندماژولی | مطابق `system-design.md` بخش ۳؛ `libs.versions.toml` |
| پین نسخه‌ها | Kotlin 2.4.x، AGP 9.1.x، Gradle ≥ 9.3.1، compileSdk/targetSdk 36، minSdk 26 |
| CI (GitHub Actions) | `assembleDebug`, `testDebugUnitTest`, `lint`, `detekt`, `ktlint`, secret‑scan (gitleaks) |
| دروازه امنیتی اولیه | `LogRedactor` + تست `NoSecretInLogTest` با fixture |
| `core:model` | value types، enumها، `TimeFact`، `Derivation` + تست unit |
| `core:common` | `Result`/`Clock`/`Dispatchers`/`UnitIds` |
| راستی‌آزمایی Hilt/KSP | اگر ناسازگار بود → DI دستی و ثبت ADR‑013‑R1 |
| `.gitignore` | build، keystore، fixtures شخصی، `.idea`، خروجی تست |

**تست‌ها:** ساخت موفق، اجرای کامل unit tests، lint صفر خطا، secret scan صفر یافته.
**خروج از فاز:** `./gradlew build` سبز در CI + همه ADRهای P0 با وضعیت `Accepted`.

---

## P1 — هسته ذخیره‌سازی، رمزنگاری و لوله import `L`

**هدف:** کاربر بتواند یک فایل/آرشیو را انتخاب کند و شواهد با منشأ کامل ذخیره شوند — آفلاین.

| خروجی | جزئیات |
| --- | --- |
| `core:database` | Room 2.8 + SQLCipher + همه موجودیت‌های `data-model.md` + Migration 1 + ایندکس‌ها |
| `core:security` | Keystore master key، Tink keyset، TokenVault، blind index، RawBlobStore (streaming AEAD) |
| `core:import` | SAF picker (`ACTION_OPEN_DOCUMENT(_TREE)` + persistable)، hash، تشخیص چندتکه، `ImportedFile` با checkpoint |
| `parsers:mbox`, `parsers:eml`, `parsers:zip-archive`, `parsers:csv-generic`, `parsers:json-generic` | streaming، `ParseReport`، تحمل خرابی |
| `core:model` → ingest | نگاشت RawEvidence → Evidence/Identifier/Subject با dedup لایه ۱ و ۲ |
| `features:import` | انتخاب فایل، پیشرفت واقعی، گزارش «چند ورودی خوانده/رد/خطا شد» |
| `features:onboarding` | متن شفافیت، رضایت، انتخاب زبان، هشدار «کلید در Keystore است و با factory reset از دست می‌رود» |

**تست‌ها:** golden-file برای MBOX/CSV/JSON؛ fuzzing (ZIP bomb، JSON عمیق، CSV ناقص)؛ `MigrationTestHelper`؛ تست idempotency (import دوباره همان فایل)؛ `NoSecretInDbTest`؛ تست resume (kill وسط import → ادامه از checkpoint)؛ تست از دست رفتن URI permission.
**خروج:** import یک آرشیو واقعی ۱GB+ بدون OOM و با گزارش دقیق.

---

## P2 — موتور همبستگی، dedup، خط زمانی و گزارش پوشش `L`

**هدف:** از شواهد پراکنده، حساب‌ها و خط زمانی **قابل توضیح** ساخته شود.

| خروجی | جزئیات |
| --- | --- |
| `core:correlation` | نرمال‌سازی شناسه‌ها، blocking، امتیازدهی وزنی، Union‑Find، `MergeLog`، واگردانی |
| قواعد استنتاج | `EMAIL_FIRST_CONTACT`, `EMAIL_SIGNUP_RECEIPT`, `APP_FIRST_INSTALL`, `ARCHIVE_FIRST_SNAPSHOT`, `BREACH_DATE`, `IDENTITY_CHANGE` با `ruleVersion` |
| `TimelineEvent` builder | ساخت رویداد فقط از Evidence موجود؛ بدون شاهد → بدون رویداد |
| `core:reporting` | `CoverageRecord`، `ErrorRecord`، ماتریس پوشش، متن صادقانه گزارش |
| `features:timeline` | فیلتر بر اساس `dateKind/confidence/kind`، نمایش برچسب اطمینان، بخش «بدون تاریخ» |
| `features:identity` | خوشه‌ها، پیشنهاد ادغام، تأیید/رد کاربر، تاریخچه تصمیم‌ها |
| `features:coverage` | ماتریس دسته × منبع + دلیل بررسی‌نشدن |
| `features:dashboard` | خلاصه: N حساب تأییدشده، M احتمالی، K غیرقابل دسترسی، J بررسی‌نشده |

**تست‌ها:** unit برای امتیازدهی (جدول موارد مرزی)، تست ادغام/واگردانی، golden-file برای قواعد استنتاج، snapshot test روی متن گزارش (جلوگیری از ادعای کاذب)، تست اینکه `ESTIMATED` بدون `derivation` رد می‌شود.
**خروج:** روی fixture ترکیبی (MBOX + CSV رمزها + یک آرشیو) خط زمانی و خوشه‌ها با توضیح قابل بازبینی ساخته شود.

---

## P3 — Connectorهای دستگاهی و مجوزها `L`

**هدف:** منابع محلی بدون OAuth: اپ‌های نصب‌شده، مجوزها، (opt‑in) usage stats، مخاطبان، فایل‌های محلی.

| خروجی | جزئیات |
| --- | --- |
| `core:permissions` | ماشین وضعیت مجوزها (runtime/special/settings)، rationale، تشخیص app‑op |
| `connectors:device-packages` | `firstInstallTime/lastUpdateTime`، نسخه، `checkPermission` برای مجوزهای اعطاشده (I10) |
| مسیر دوگانه package visibility | `QUERY_ALL_PACKAGES` (flavor `free`) و `<queries>` هدفمند (flavor `play`) |
| `connectors:device-usage` | opt‑in + تجمیع روزانه + نگهداری ۹۰ روزه |
| `connectors:device-contacts` | فقط identifierهای owner؛ فیلتر `third_party` |
| `parsers:sms-xml`, `parsers:calls-csv` | جایگزین مجاز `READ_SMS`/`READ_CALL_LOG` |
| `features:sources` | کارت هر منبع: وضعیت دسترسی، اقدام لازم، آخرین اجرا، خطاها، «چرا بررسی نشد» |

**تست‌ها:** unit با `PackageManager`/`ContentResolver` fake؛ Robolectric برای flows مجوز؛ تست fallback وقتی `QUERY_ALL_PACKAGES` نیست؛ تست اینکه داده اشخاص ثالث ذخیره نمی‌شود؛ instrumented برای SAF.
**خروج:** اولین «گزارش پوشش» واقعی روی دستگاه تست با حداقل ۴ منبع محلی.

---

## P4 — مسیر گوگل: هویت، Gmail، Takeout `XL`

**هدف:** بزرگ‌ترین دسته شاهد (ایمیل و فعالیت گوگل) با معماری قابل تکرار برای بقیه OAuthها.

| خروجی | جزئیات |
| --- | --- |
| `core:network` | interceptor احراز هویت، throttle، خواندن هدرهای rate limit، allowlist دامنه |
| OAuth پایه | Credential Manager (Google Sign‑In جدید) + `AuthorizationClient` برای اسکوپ داده؛ PKCE؛ TokenVault |
| `connectors:google-identity` | حساب(ها)، ایمیل‌ها، `sub` |
| `connectors:google-gmail` | `messages.list` با queryهای هدفمند (`subject:(welcome OR verify OR confirm)`, `from:` دامنه‌های شناخته‌شده)، `messages.get` با `format=metadata` تا سهمیه کمتر مصرف شود، `history.list` برای delta، مدیریت `429` و `historyId expired` |
| `connectors:google-takeout` | پارسرهای `My Activity`، `Chrome/BrowserHistory.json`، `Chrome/Bookmarks.json`، `Contacts`، `Calendar`، `Google Payments/Shopping History`، `YouTube/history/watch-history.json` |
| `parsers:meta-archive` | آماده برای P6 ولی ساختار در همین فاز طراحی می‌شود |
| `features:security-health` | شروع: «حساب‌هایی که ایمیل تأیید/هشدار ورود جدید داشته‌اند» |

**ریسک‌ها و کاهش:** اسکوپ `gmail.readonly` **Restricted** است → برای انتشار عمومی نیاز به OAuth verification و (اگر داده به سرور برود) ارزیابی CASA دارد. چون ما backend نداریم و داده از دستگاه خارج نمی‌شود، مسیر «local‑only app» را در درخواست verification مستند می‌کنیم؛ در بدترین حالت fallback = IMAP با توکن OAuth (A2) یا import Takeout (A6/B5) که هیچ تأییدی نمی‌خواهد.
**تست‌ها:** MockWebServer با پاسخ‌های واقعی Gmail (fixture ضبط‌شده بدون داده شخصی)، تست سهمیه (۶٬۰۰۰ واحد/دقیقه/کاربر)، تست `history.list` و انقضای `historyId`، تست resume بعد از ۴۲۹، golden-file برای هر پارسر Takeout، تست device برای Credential Manager.
**خروج:** روی حساب واقعی داوطلب: کشف ≥ ۹۰٪ سرویس‌هایی که ایمیل ثبت‌نام دارند (با معیار پذیرش مستند در `acceptance-criteria.md`).

---

## P5 — مایکروسافت و آرشیوهای رسمی `L`

| خروجی | جزئیات |
| --- | --- |
| `connectors:ms-graph` | MSAL Android، `Mail.Read` (دلتا)، `User.Read`، `Calendars.Read`, `Contacts.Read`, `Files.Read` |
| `connectors:ms-export` | privacy dashboard archive (JSON) |
| `parsers:linkedin` | `Connections.csv` + آرشیو کامل |
| `connectors:imap` | IMAP عمومی با XOAUTH2 و App Password (با هشدار صریح) |
| صداقت محدودیت | sign‑in logs شخصی در دسترس نیست → `INACCESSIBLE` با دلیل و جایگزین |

**تست‌ها:** fixture Graph (با `$deltaToken`)، تست ۴۰۱/consent لازم، تست IMAP با GreenMail/Dovecot محلی، golden-file برای LinkedIn CSV.

---

## P6 — توسعه‌دهندگان، اجتماعی با API رسمی، و واردکننده‌های پرحجم `L`

| خروجی | جزئیات |
| --- | --- |
| `connectors:github` | `/user` (`created_at`)، `/user/emails`، `/repos`، `/events`، `/user/installations`؛ Device Flow؛ secondary rate limits |
| `connectors:stackexchange` | `creation_date` |
| `connectors:mastodon`, `connectors:discord`, `connectors:twitch` | پروفایل + `/users/@me/connections` (شاهد قوی account linking) |
| `parsers:password-csv`, `parsers:bitwarden`, `parsers:onepassword`, `parsers:keepass` | فقط metadata (ADR‑004) + تست عدم نشت |
| `parsers:amazon`, `parsers:apple`, `parsers:discord-package` | آرشیوهای رسمی |
| `features:import` v2 | «mapping assistant» برای آرشیو ناشناخته (K2) |

**تست‌ها:** contract test برای هر connector؛ golden-file برای هر پارسر؛ تست redaction رازها (دروازه انتشار)؛ تست rate limit.

---

## P7 — بازی‌ها، موسیقی/ورزش، 2FA و درخواست داده `L`

| خروجی | جزئیات |
| --- | --- |
| `connectors:steam` | `GetPlayerSummaries` (`timecreated`!)، `GetOwnedGames`، `GetSteamLevel`؛ کلید کاربر |
| `connectors:roblox`, `connectors:lastfm`, `connectors:spotify`, `connectors:strava`, `connectors:fitbit` | تاریخ واقعی عضویت در Last.fm/Fitbit؛ recently played در Spotify |
| `parsers:totp-qr` | protobuf `otpauth-migration://` → فقط issuer/account |
| `parsers:telegram-json`, `parsers:whatsapp-txt`, `parsers:google-calendar-html` | |
| `features:data-request` | دستیار درخواست داده (K1) با کاتالوگ سرویس‌ها + نامه آماده (fa/en/de) + وضعیت پیگیری |
| `connectors:google-youtube`, `connectors:google-calendar` | از B3/B4 |

**تست‌ها:** fixture protobuf (ساختگی)، golden-file تلگرام/واتس‌اپ، تست کلید Steam نامعتبر، تست «پروفایل خصوصی» Steam → `NO_RESULT` صادقانه.

---

## P8 — شواهد عمومی و نقض داده `M`

| خروجی | جزئیات |
| --- | --- |
| `connectors:hibp` | کلید کاربر، k‑anonymity برای رمزها، مدیریت ۴۲۹ با `retry-after` |
| `connectors:wayback` | CDX برای URLهای تأییدشده توسط کاربر |
| `connectors:rdap`, `connectors:openpgp` | تاریخ ثبت دامنه، ایمیل‌های تأییدشده |
| `parsers:cookies-txt`, `parsers:opml`, `parsers:firefox-*` | فقط metadata برای کوکی |
| `connectors:reddit`, `connectors:gitlab`, `connectors:x-user` | با هشدار هزینه/تأیید |
| `features:security-health` v2 | امتیاز ریسک هر حساب (نقض + رمز لو رفته + 2FA) |

**تست‌ها:** fixture CDX/RDAP/HIBP، تست ۴۰۴ (بدون نقض) در مقابل ۴۲۹، تست اینکه هش ایمیل در لاگ نیست.

---

## P9 — منابع مشروط و سنگین `L`

`connectors:google-drive`, `connectors:ms-skype`, `connectors:notion/figma/slack/trello`, `connectors:kaggle/dockerhub/hf/pypi`, `connectors:riot`, `connectors:moodle`, `connectors:battlenet`, `parsers:e10-bank-csv/mt940`, `connectors:epic/psn/nintendo` (از مسیر import).

**تست‌ها:** contract test + fixture per service + تست انقضای توکن.

---

## P10 — منابع پرریسک (opt‑in سخت‌گیرانه) `L`

`connectors:device-notifications` (allowlist + metadata‑only)، `connectors:health-connect`، `connectors:crypto-exchange` (کلید read‑only)، `parsers:location-history` (تجمیعی)، `features:deletion` (سطوح L1–L5 + ابطال سمت سرویس).

**دروازه:** هیچ‌کدام بدون پاس‌کردن تست‌های redaction و صفحه افشای برجسته جداگانه فعال نمی‌شوند.

---

## P11 — گزارش‌گیری، صدور، بومی‌سازی و سخت‌سازی `M`

| خروجی | جزئیات |
| --- | --- |
| `features:export` | JSON/PDF رمزگذاری‌شده با عبارت عبور، بدون راز |
| بومی‌سازی کامل | fa/en/de + RTL + کاتالوگ سرویس‌های آلمانی و ایرانی |
| سخت‌سازی عملکرد | Macrobenchmark: import ۲GB، خط زمانی ۱۰۰k رویداد |
| دسترسی‌پذیری | contentDescription، کنتراست، فونت بزرگ |
| مستندسازی کاربر | راهنمای هر منبع + «چرا این منبع در دسترس نیست» |

---

## P12 — انطباق انتشار و بتا `M`

| خروجی | جزئیات |
| --- | --- |
| دو flavor | `play` (با fallback مجوزها) و `free` (قابلیت کامل) |
| Declaration Forms | متن و ویدیوی `QUERY_ALL_PACKAGES` (در صورت نیاز)، افشای User Data |
| سیاست حریم خصوصی | PRIVACY.md + صفحه داخلی |
| OAuth verification | آماده‌سازی مستندات local‑only برای اسکوپ‌های Restricted |
| بتا | ۱۰ داوطلب، گزارش باگ، معیارهای پذیرش واقعی |
| تست نفوذ سبک | بازبینی دستی redaction، backup، exported components، deep links |

---

## وابستگی‌های بحرانی بین فازها

```
P0 → P1 → P2 → {P3, P4}
P4 → P5 → P6 → P7 → P8 → P9 → P10 → P11 → P12
P2 (correlation) پیش‌نیاز همه connectorهاست.
P1 (import) پیش‌نیاز همه parserهاست.
```

## ریسک‌های سطح پروژه

| ریسک | احتمال | اثر | کاهش |
| --- | --- | --- | --- |
| رد شدن `QUERY_ALL_PACKAGES` در Play | بالا | متوسط | fallback `<queries>` + flavor دوگانه |
| طولانی شدن OAuth verification گوگل برای اسکوپ Restricted | بالا | بالا | مسیر IMAP + Takeout import به‌عنوان جایگزین کامل از P4 |
| تغییر قالب آرشیوهای رسمی (Takeout/Meta/Apple) | متوسط | متوسط | پارسر نسخه‌دار + شکست صادقانه + fixture regression |
| هزینه APIهای پولی (X, HIBP) | قطعی برای X | کم | کلید/اعتبار توسط کاربر؛ پیش‌فرض خاموش |
| حجم داده → کندی/ANR | متوسط | بالا | streaming، Paging 3، تجمیع، benchmark در P11 |
| تغییر Room 3.0 | متوسط | کم | ADR‑012 + لایه DAO باریک |
| رد شدن در Play به‌خاطر مجوزهای حساس | متوسط | بالا | انتشار `free` از F‑Droid به‌عنوان مسیر اصلی جایگزین |
| داده شخصی داوطلبان در fixtureها | متوسط | بحرانی | فقط fixture ساختگی + secret scan + بازبینی PR |

## تعریف «تمام» (Definition of Done) برای هر فاز

1. کد + تست‌ها در CI سبز.
2. تست‌های واقعی اجرا شده و نتیجه (پاس/شکست/اجراننشده) در گزارش مرحله ثبت شده.
3. مستندات به‌روزرسانی (کاتالوگ، محدودیت‌ها، ADRها).
4. هیچ راز/داده شخصی در مخزن یا لاگ.
5. گزارش پایان مرحله: فایل‌های تغییرکرده، قابلیت‌های تکمیل‌شده، تست‌های اجراشده، خطاهای باقی‌مانده، محدودیت‌های فنی، دستور ادامه مرحله بعد.
6. کامیت‌های کوچک و توصیفی + پوش موفق روی شاخه کاری (قانون ۱۶/۱۷).
