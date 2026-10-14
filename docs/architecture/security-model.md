# مدل امنیتی — Security & Privacy Model

> مرحله ۱ — ۲۰۲۶‑۱۰‑۱۴
> اصل پایه: این اپ **بیشترین تراکم داده حساس کاربر روی یک دستگاه** را ایجاد می‌کند (ایمیل‌ها، رمزهای مدیریتی، پیامک‌ها، مکان، سلامت). بنابراین امنیت یک قابلیت نیست، یک پیش‌شرط انتشار است.
>
> قانون ۱۰ Rules: رمزها، توکن‌های OAuth، ایمیل‌ها و داده‌های شخصی در لاگ‌ها، گزارش خطا، Git و خروجی دیباگ افشا نمی‌شوند. این سند آن را به مکانیزم قابل تست تبدیل می‌کند.

---

## ۱. دارایی‌ها (Assets)

| شناسه | دارایی | حساسیت |
| --- | --- | --- |
| AS‑1 | توکن‌های OAuth / refresh token / کلیدهای API / رمز اپ / PAT | بحرانی — دسترسی مستقیم به حساب‌های کاربر |
| AS‑2 | پایگاه داده شواهد (ایمیل‌ها، حساب‌ها، رویدادها) | بحرانی |
| AS‑3 | blobهای خام (آرشیو ایمیل، فایل‌های واردشده) | بحرانی |
| AS‑4 | کلیدهای رمزنگاری (Keystore keyset) | بحرانی |
| AS‑5 | متادیتای استفاده (چه منبعی، کی اسکن شد) | بالا |
| AS‑6 | گزارش‌های خطا و لاگ‌ها | بالا (محل اصلی نشت) |
| AS‑7 | خروجی صادرشده (JSON/PDF گزارش) | بالا |

## ۲. مدل تهدید (STRIDE + سناریوهای واقعی)

| # | تهدید | سناریو | کاهش |
| --- | --- | --- | --- |
| T‑1 | سرقت دستگاه / دسترسی فیزیکی | گوشی قفل‌نشده یا با رمز ضعیف | رمزگذاری کل DB با SQLCipher، قفل بیومتریک اختیاری، `FLAG_SECURE` روی صفحات حساس، انقضای خودکار قفل |
| T‑2 | استخراج با بکاپ ابری | `adb backup` / بکاپ گوگل | `allowBackup=false` + `dataExtractionRules` با رد کامل + کلید در Keystore (با `setUserAuthenticationRequired` در حالت سخت‌گیرانه) |
| T‑3 | اپ مخرب هم‌دستگاه | اپ دیگر داده ما را بخواند | app‑private storage، هیچ ContentProvider صادرشده، هیچ فایل در حافظه اشتراکی، `exported=false` برای همه کامپوننت‌ها مگر ضروری |
| T‑4 | نشت از طریق لاگ | `Log.d` با بدنه ایمیل یا توکن | `LogRedactor` با allowlist، ممنوعیت `Log.*` مستقیم (detekt rule سفارشی)، تست خودکار «هیچ رازی در logcat نیست» |
| T‑5 | نشت از طریق گزارش خطا | stacktrace با URL حاوی `key=...` | `ErrorRedactor` روی همه مسیرها؛ URLها فقط host+path؛ هدرها هرگز |
| T‑6 | فایل واردشده مخرب | ZIP bomb، JSON عمیق، XML XXE، فایل SQLite مخرب | محدودیت سخت نسبت فشرده‌سازی و تعداد ورودی، ممنوعیت entity خارجی در XML، سقف عمق/طول JSON، اجرای پارسر در sandbox منطقی با timeout، fuzzing در CI |
| T‑7 | سرقت توکن در حافظه | dump حافظه | نگه‌داری حداقلی، پاک‌کردن `CharArray`/`ByteArray` پس از استفاده، عدم serialize توکن در `WorkManager.Data` |
| T‑8 | MITM شبکه | وای‌فای عمومی | HTTPS اجباری، `networkSecurityConfig` بدون cleartext، اعتبارسنجی گواهی سیستمی، User‑Agent شناسا، ممنوعیت ارسال داده کاربر به هر host غیراز endpoint رسمی همان سرویس |
| T‑9 | OAuth فیشینگ / redirect دزدیده‌شده | اپ مخرب همان scheme را ثبت کند | PKCE اجباری، redirect با scheme اختصاصی + `AppAuth`/MSAL، تأیید `state`، نمایش اسکوپ‌های واقعی به کاربر قبل از تأیید |
| T‑10 | استفاده بیش از حد از سهمیه → افشای الگو | سرویس‌ها رفتار ما را ببینند | throttle، بودجه روزانه، User‑Agent صادقانه، بدون polling بی‌هدف |
| T‑11 | افشای هویت در URL (Gravatar/HIBP/OpenPGP) | ایمیل در query string | redaction در لاگ شبکه، عدم کش‌کردن آن URLها، استفاده از k‑anonymity برای رمزها |
| T‑12 | ادغام اشتباه هویت‌ها → اتهام نادرست | دو حساب متفاوت ادغام شوند | آستانه‌های محافظه‌کار، تأیید کاربر برای `CONFIRMED`، واگردانی کامل، `MergeLog` |
| T‑13 | داده اشخاص ثالث | مخاطبان/دوستان/گفتگوهای دیگران | ممنوعیت ذخیره انبوه، فیلتر `third_party`، فقط داده owner |
| T‑14 | انتشار در Git | fixture حاوی داده واقعی | `fixtures/` فقط داده ساختگی، pre‑commit secret scan (gitleaks/trufflehog)، `.gitignore` برای خروجی build و فایل‌های شخصی |
| T‑15 | زنجیره تأمین | وابستگی مخرب | lockfile نسخه‌ها، Renovate، `dependencyVerification` در Gradle، بررسی CVE در CI، حداقل وابستگی‌ها |
| T‑16 | دسترسی غیرمجاز به سرویس‌ها | scraping/enumeration | ممنوع در معماری (ADR‑011) + تست که هیچ connector بدون OAuth/کلید/فایل کاربر درخواست پروب هویت نمی‌زند |

---

## ۳. معماری رمزنگاری

```
        ┌──────────────────────────────────────────────┐
        │        Android Keystore (hardware‑backed)     │
        │  masterKey (AES‑256, no‑export, auth‑opt)     │
        └───────▲───────────────▲──────────────▲────────┘
                │ protects      │ protects     │ protects
        ┌───────┴──────┐ ┌──────┴───────┐ ┌────┴──────────────┐
        │ Tink keyset  │ │ SQLCipher    │ │ Blind‑index HMAC  │
        │ (AEAD)       │ │ DB key       │ │ key               │
        └───────▲──────┘ └──────▲───────┘ └────▲──────────────┘
                │               │              │
      ┌─────────┴──────┐  ┌─────┴──────┐  ┌────┴─────────┐
      │ TokenVault     │  │ Room DB    │  │ جست‌وجو/dedup │
      │ RawBlobStore   │  │ (SQLCipher)│  │ بدون رمزگشایی │
      └────────────────┘  └────────────┘  └──────────────┘
```

### ۳.۱ انتخاب‌ها و دلیل
| تصمیم | انتخاب | دلیل |
| --- | --- | --- |
| کتابخانه رمزنگاری | **Google Tink (AEAD AES256‑GCM)** + `AndroidKeysetManager` | `androidx.security:security-crypto` از نسخه ۱٫۱٫۰ **deprecated** است و انتشار جدیدی نخواهد داشت؛ توصیه جاری گوگل DataStore + Tink + Keystore است |
| رمزگذاری پایگاه داده | **SQLCipher for Android** (`net.zetetic:sqlcipher-android`) با `SupportOpenHelperFactory` | رمزگذاری کل DB ساده‌تر و کم‌خطاتر از رمزگذاری فیلدبه‌فیلد است؛ جست‌وجو/ایندکس ممکن می‌ماند |
| trade‑off | Room در حالت سازگاری SupportSQLite (نه BundledSQLiteDriver) | SQLCipher با driver API جدید کار نمی‌کند → مهاجرت به Room 3.0 به تعویق می‌افتد (ADR‑012)؛ آگاهانه پذیرفته شد |
| کلید master | Keystore با `KEY_PROPERTIES.PURPOSE_ENCRYPT` + `PURPOSE_DECRYPT`, AES‑256‑GCM, `setUserAuthenticationRequired(false)` در حالت عادی و `true` در حالت سخت‌گیرانه (قابل انتخاب) | hard‑ware backing در TEE/StrongBox وقتی موجود باشد |
| جست‌وجو روی داده رمزنگاری‌شده | **blind index** = `HMAC_SHA256(hmacKey, normalize(value))` | امکان dedup/lookup بدون رمزگشایی؛ کلید HMAC جدا از کلید AEAD |
| ذخیره تنظیمات | DataStore Proto (بدون راز) | رازها هرگز در DataStore نیستند |
| hashing شناسه | SHA‑256 برای `content_hash` (غیرقابل برگشت برای محتوا) و HMAC برای blind index | SHA‑256 خالی روی ایمیل قابل rainbow‑table است → برای شناسه‌ها فقط HMAC استفاده می‌شود |
| رمز عبور صادرشده (گزارش) | AES‑GCM با کلید مشتق از عبارت عبور کاربر (Argon2/PBKDF2 با تعداد تکرار مستند) | export هرگز بدون عبارت عبور ممکن نیست |

### ۳.۲ چرخه عمر کلید
1. **ساخت:** اولین اجرا → `AndroidKeysetManager` یک keyset می‌سازد و با masterKey در Keystore محافظت می‌شود.
2. **چرخش:** چرخش سالانه/دستی؛ `Aead.encrypt` با keyset جدید + rewrap کل keyset‌ها؛ چون DB key جداست، چرخش DB key = rekey با SQLCipher (`PRAGMA rekey`) در پس‌زمینه با checkpoint.
3. **از دست رفتن:** اگر Keystore کلید را از دست بدهد (factory reset/تغییر قفل دستگاه در حالت auth‑required) → داده غیرقابل بازیابی است؛ این به کاربر در onboarding صریح گفته می‌شود و گزینه «export رمزگذاری‌شده» پیشنهاد می‌شود.
4. **حذف:** purge کامل = حذف DB + blobها + `KeysetHandle` + `delete` کلید Keystore + ثبت در `AuditLog`.

---

## ۴. TokenVault (مدیریت اعتبارنامه‌ها)

| قاعده | اجرا |
| --- | --- |
| فقط refresh token و API key ذخیره می‌شود | access token در حافظه با TTL |
| هرگز در DB، DataStore، فایل، لاگ، یا `WorkManager.Data` | `SourceCredentialsRef` فقط `vault_key` دارد |
| ذخیره با Tink AEAD در فایل app‑private | `files/vault/<key>.bin` |
| چرخش خودکار | refresh با backoff؛ در صورت `invalid_grant` → `AccessState.AuthExpired` + اطلاع به کاربر (بدون پاک‌کردن بی‌صدا) |
| لغو | دکمه «قطع اتصال» هم توکن را حذف می‌کند و هم (در سرویس‌هایی که API لغو دارند: گوگل `revoke`، GitHub `DELETE /applications/{client_id}/token`) توکن را سمت سرویس باطل می‌کند |
| نمایش | هرگز کامل نشان داده نمی‌شود (فقط ۴ کاراکتر آخر و زمان انقضا) |
| scope minimization | فقط اسکوپ‌های واقعاً لازم؛ فهرست اسکوپ‌ها قبل از OAuth به کاربر نشان داده می‌شود |

---

## ۵. Redaction (لایه‌ای و قابل تست)

### ۵.۱ قواعد عمومی
- **allowlist نه denylist:** فقط فیلدهایی که صریحاً مجازند لاگ می‌شوند.
- الگوهای اجباری mask: `Bearer\s+\S+`, `key=\S+`, `token=\S+`, `hibp-api-key`, `Authorization`, ایمیل → `a***@d***`, شماره → `+**…123`, URL query → حذف کامل، IP → `x.x.*.*`، مسیر فایل → فقط نام.
- ممنوعیت لاگ بدنه پیام/ایمیل/پیامک؛ فقط `messageId` و طول.

### ۵.۲ سه Redactor
| Redactor | کجا | خروجی مجاز |
| --- | --- | --- |
| `LogRedactor` | Timber/logcat | سطح `INFO` در release؛ فقط تگ + رخداد + شناسه‌ها |
| `ErrorRedactor` | `ErrorRecord.redacted_context` | `messageKey` + پارامترهای غیرحساس |
| `ExportRedactor` | گزارش JSON/PDF | بدون توکن/رمز/کوکی/secret/بدنه؛ مکان تجمیعی؛ IP اختیاری |

### ۵.۳ تست‌های اجباری (دروازه انتشار)
1. `NoSecretInLogTest`: اجرای سناریوهای واقعی روی emulator + scan کامل logcat با regex رازها → باید صفر باشد.
2. `NoSecretInDbTest`: dump DB و جست‌وجوی الگوهای رمز/توکن در همه ستون‌ها (به‌جز vault) → صفر.
3. `PasswordCsvImportTest`: import یک CSV رمز → هیچ رمزی در DB/لاگ/export نباشد.
4. `CookiesImportTest`: import cookies.txt → هیچ `value` ذخیره نشود.
5. `TotpQrTest`: اسکن fixture → secret در هیچ خروجی نباشد.
6. `ExportRedactionTest`: صدور گزارش در حالت‌های مختلف و diff با فهرست ممنوعه.
7. `BackupDisabledTest`: بررسی manifest و dataExtractionRules.

---

## ۶. مجوزها و انطباق با سیاست Google Play

| مجوز/API | سیاست | تصمیم ما |
| --- | --- | --- |
| `QUERY_ALL_PACKAGES` | Play: فقط اگر عملکرد اصلی اپ نیاز به دیدن همه اپ‌ها داشته باشد + Declaration Form + ویدیو؛ در غیر این صورت حذف از manifest | کشف اپ‌های نصب‌شده **یکی از عملکردهای اصلی** است، ولی ریسک رد شدن بالاست → معماری دو مسیره: اگر تأیید شد کامل، وگرنه `<queries>` هدفمند (~۲۰۰ پکیج شناخته‌شده) و اعلام صادقانه پوشش کمتر در گزارش |
| `PACKAGE_USAGE_STATS` | special app‑op؛ داده حساس؛ افشای برجسته لازم | opt‑in، افشای جداگانه، ذخیره تجمیعی |
| `BIND_NOTIFICATION_LISTENER_SERVICE` | نیازمند فعال‌سازی دستی کاربر و افشای برجسته | opt‑in، allowlist پکیج، فقط metadata، بدنه دور ریخته می‌شود |
| `READ_SMS` / `READ_CALL_LOG` | فقط default handler | **استفاده نمی‌شود**؛ جایگزین: import فایل توسط کاربر |
| `READ_CONTACTS` / `READ_CALENDAR` | sensitive اما مجاز با افشا | opt‑in + فقط داده owner |
| `health.READ_*` | سیاست داده سلامت | فاز آخر، opt‑in، فقط `dataOrigin` و شمارش |
| `AD_ID` | اگر برای تبلیغات استفاده نشود نباید درخواست شود | **استفاده نمی‌شود** |
| `CAMERA` | عادی | فقط برای اسکن QR خروجی 2FA |
| `POST_NOTIFICATIONS` | عادی (API 33+) | برای پیشرفت اسکن |
| `MANAGE_EXTERNAL_STORAGE` | Play محدود کرده | **استفاده نمی‌شود**؛ فقط SAF |
| User Data policy | افشای برجسته و رضایت، ممنوعیت فروش داده | صفحه شفافیت در onboarding + `ConsentRecord` + عدم وجود هر شبکه به سرور خودمان |

**مسیر انتشار دوگانه (ADR‑014):** build `play` (با fallback مجوزها و حذف قابلیت‌های پرریسک) و build `free` (F‑Droid/sideload با قابلیت کامل و هشدار صریح). تفاوت‌ها با product flavor و نه کد شرطی پراکنده.

---

## ۷. حداقل‌سازی و نگهداری داده

| داده | نگهداری پیش‌فرض |
| --- | --- |
| metadata شواهد | دائم (تا حذف توسط کاربر) |
| بدنه کامل ایمیل/پیام | فقط در صورت opt‑in؛ وگرنه snippet ≤ ۲۰۰ کاراکتر |
| فایل خام واردشده | `DELETE_AFTER_PARSE` مگر کاربر «نگهداری آرشیو» را بخواهد |
| Usage events خام | تجمیع روزانه پس از ۹۰ روز |
| Notification metadata | ۹۰ روز |
| ErrorRecord | ۳۰ روز |
| AuditLog | ۱۸۰ روز (بدون محتوا) |
| خط زمانی مکان | خاموش به‌صورت پیش‌فرض؛ در صورت فعال، تجمیع شهری |

---

## ۸. حذف داده (حق فراموشی در سطح اپ)

| سطح | چه حذف می‌شود | تأیید |
| --- | --- | --- |
| L1 — حذف یک شاهد | یک `Evidence` + blob مرتبط + رویدادهای مشتق بازسازی می‌شوند | یک تأیید |
| L2 — حذف یک منبع | همه Evidence/Event/Subject/Cursor/Connection + توکن‌ها در Vault + `ConsentRecord` | یک تأیید + نمایش تعداد رکوردها |
| L3 — حذف دسته‌ای | مثلاً «همه داده مکان» یا «همه داده پیامک» | یک تأیید |
| L4 — پاک‌سازی کامل | DB + blobها + keysetها + DataStore + کلیدهای Keystore | **تأیید دومرحله‌ای با تایپ عبارت** + گزینه export رمزگذاری‌شده قبل از حذف |
| L5 — حذف و ابطال سمت سرویس | فراخوانی revoke گوگل/GitHub/Discord و… قبل از حذف محلی | opt‑in در L2 |

پس از هر حذف: `AuditLog` (بدون محتوا) + بازسازی `CoverageRecord` (منبع به `NOT_CHECKED` برمی‌گردد با دلیل «توسط کاربر حذف شد») تا گزارش پوشش دروغ نگوید.

---

## ۹. امنیت شبکه

- HTTPS-only؛ `cleartextTrafficPermitted=false`.
- `networkSecurityConfig`: اعتماد فقط به CAهای سیستمی؛ ممنوعیت `debug-overrides` در release.
- **بدون certificate pinning برای سرویس‌های ثالث** (باعث شکست در صورت چرخش گواهی آن‌ها می‌شود و چون backend خودمان نداریم، pinning بی‌مورد است).
- timeouts: connect ۱۵s، read ۳۰s، write ۳۰s، کل هر task ≤ ۱۰ دقیقه با checkpoint.
- retry: نمایی با jitter، فقط برای `retryable`ها؛ هرگز برای ۴۰۱/۴۰۳.
- `User-Agent` صادقانه و قابل شناسایی (الزام HIBP و حسن رفتار با APIها).
- allowlist دامنه‌ها در سطح `core:network`: هر connector دامنه‌های مجاز خود را اعلام می‌کند؛ درخواست به دامنه خارج از فهرست → `POLICY_BLOCKED` + `AuditLog`.

## ۱۰. امنیت کد و ساخت

- R8/ProGuard با rules سخت‌گیرانه؛ حذف کامل `BuildConfig.DEBUG` paths در release.
- detekt + ktlint + Android Lint + `lint-security` در CI (بلوکه‌کننده).
- Gradle `dependencyVerification` (checksum) + Renovate + بررسی CVE.
- ممنوعیت reflection برای دسترسی به APIهای محدودشده/غیرعمومی (greylist).
- بدون استفاده از APIهای hidden (`@hide`) — ریسک رد شدن در Play و شکست در نسخه‌های بعدی اندروید.
- `exported=false` برای همه receiver/service/provider مگر آنچه ذاتاً باید exported باشد (NotificationListenerService).
- deep link: فقط scheme اختصاصی OAuth؛ بدون `android:host="*"` و بدون پذیرش intent ناشناس برای import (فقط از SAF داخلی).

## ۱۱. انطباق GDPR (چون کاربر در آلمان است)

| اصل | اجرا در اپ |
| --- | --- |
| قانونمندی/رضایت (Art. 6(1)(a)) | `ConsentRecord` برای هر منبع؛ پردازش فقط داده **خود کاربر** و با هدف شخصی (household exemption Art. 2(2)(c) نیز قابل استناد است، ولی ما محافظه‌کارانه رضایت می‌گیریم) |
| حداقل‌سازی (Art. 5(1)(c)) | سیاست metadata‑only، عدم ذخیره رازها، تجمیع داده حساس |
| شفافیت (Art. 5(1)(a), 12–14) | صفحه «چه داده‌ای، از کجا، چرا» برای هر منبع + نمایش `derivation` برای هر تاریخ استنتاج‌شده |
| حق دسترسی/قابل‌حمل (Art. 15, 20) | دستیار درخواست داده (K1) + صدور JSON ساختاریافته از داده خود اپ |
| حق حذف (Art. 17) | سطوح حذف L1–L5 |
| امنیت پردازش (Art. 32) | رمزنگاری کامل، Keystore، تست‌های redaction |
| انتقال به کشور ثالث | **هیچ انتقالی وجود ندارد** (بدون backend)؛ APIهای ثالث فقط با درخواست کاربر و به endpoint رسمی همان سرویس |
| Data Protection Impact | چون پردازش محلی و شخصی است DPIA رسمی لازم نیست، ولی یک ارزیابی ریسک داخلی در همین سند ثبت شده |

## ۱۲. پاسخ به رخداد (در سطح اپ)

| رخداد | اقدام |
| --- | --- |
| نشت توکن (لو رفتن لاگ/فایل) | ابطال سمت سرویس + چرخش keyset + اطلاع صریح در اپ + ثبت در AuditLog |
| باگ redaction کشف‌شده | patch + پاک‌سازی logcat ممکن نیست → اطلاع صادقانه به کاربر در changelog |
| فایل مخرب باعث crash | قرنطینه آن `ImportedFile` (state=FAILED) + گزارش بدون payload |
| تغییر سیاست سرویس (مثل حذف API) | connector به `INACCESSIBLE` با دلیل + پیشنهاد مسیر جایگزین (نه شکست بی‌صدا) |

## ۱۳. چیزهایی که این اپ **هرگز** انجام نمی‌دهد

1. داده کاربر را به هیچ سرور متعلق به ما نمی‌فرستد (چنین سروری وجود ندارد).
2. رمز عبور، مقدار کوکی، secret 2FA یا کلید خصوصی را ذخیره نمی‌کند.
3. به حساب اشخاص ثالث دسترسی پیدا نمی‌کند یا داده آن‌ها را جمع نمی‌کند.
4. محدودیت دسترسی سرویس‌ها را دور نمی‌زند (بدون scraping، بدون enumeration، بدون self‑bot، بدون reverse‑engineering API غیررسمی).
5. ادعای «همه حساب‌های شما پیدا شد» نمی‌کند.
6. داده را برای تبلیغات یا فروش استفاده نمی‌کند.
7. بدون رضایت صریح، منبع حساس را فعال نمی‌کند.
8. از مجوزهای محدودشده Play به‌عنوان راه فرار از طراحی درست استفاده نمی‌کند.
