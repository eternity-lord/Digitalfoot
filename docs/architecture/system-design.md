# طراحی سیستم — Digital Footprint Intelligence

> مرحله ۱ — ۲۰۲۶‑۱۰‑۱۴ · وضعیت: **تصمیم‌های معماری تثبیت شد** (لاگ کامل در `docs/decisions/adr-log.md`)
> مخزن در حال حاضر فقط `README.md` و `Rules` دارد؛ یعنی **هیچ کدی وجود ندارد**. این سند ساختار هدف را تعریف می‌کند تا مراحل بعد بدون بازطراحی پیش بروند.

---

## ۱. محدودیت‌های حاکم (Constraints)

| # | محدودیت | اثر بر طراحی |
| --- | --- | --- |
| C1 | فقط روی دستگاه؛ بدون backend (ADR‑002) | هیچ سروری برای داده کاربر؛ همه OAuth جریان‌های native/device‑flow؛ کلیدهای API توسط کاربر وارد می‌شوند |
| C2 | داده شخصی/حساس | رمزگذاری در حالت سکون، redaction در لاگ، حذف کامل، حداقل‌سازی |
| C3 | سهمیه و rate limit سرویس‌ها | لایه throttle + backoff + بودجه سهمیه per‑connector |
| C4 | محدودیت‌های اندروید (scoped storage، package visibility، special app‑ops، WorkManager) | استراتژی مجوز جدا برای هر connector + fallback |
| C5 | سیاست Google Play (مجوزهای حساس، Declaration Forms) | قابلیت‌های پرریسک پشت flag و fallback؛ مسیر انتشار sideload/F‑Droid هم پشتیبانی می‌شود |
| C6 | آرشیوهای چند گیگابایتی | streaming + checkpoint؛ ممنوعیت بارگذاری کامل در حافظه |
| C7 | قانون ۶ (جعل نکردن داده) | هر فیلد زمانی `kind` و `precision` و `derivation` دارد |
| C8 | قانون ۹ (ادامه از نقطه توقف) | ScanRun/ScanTask + cursor پایدار در Room |
| C9 | targetSdk 36 از ۲۰۲۶‑۰۸‑۳۱ برای انتشار در Play الزامی است | compileSdk/targetSdk = 36 |
| C10 | Room 2.x در حالت maintenance و Room 3.0 در راه است | انتخاب Room 2.8 با API‌های سازگار با مهاجرت (ADR‑012) |

---

## ۲. پشته فناوری (با راستی‌آزمایی نسخه‌ها در ۲۰۲۶‑۱۰‑۱۴)

| لایه | انتخاب | نسخه هدف | منبع راستی‌آزمایی |
| --- | --- | --- | --- |
| زبان | Kotlin | 2.4.x (پایدار: 2.4.21) | kotlinlang.org compatibility guide |
| Build | Gradle + AGP | Gradle ≥ 9.3.1، AGP 9.1.x | developer.android.com/build/releases |
| compile/target SDK | Android 16 | 36 | الزام Play از ۲۰۲۶‑۰۸‑۳۱ |
| min SDK | Android 8.0 | 26 | پوشش دستگاه + قابلیت‌های Keystore/Notification |
| UI | Jetpack Compose + Material 3 | Compose runtime 1.12.x | androidx release notes |
| ناوبری | Navigation Compose (type‑safe routes) | آخرین پایدار | androidx |
| داده ساختاریافته | Room 2.8.x (حالت سازگاری SupportSQLite) + SQLCipher | Room 2.8.5 / sqlcipher‑android | androidx.room + zetetic |
| ناهم‌زمانی | Coroutines + Flow | 1.10.x | kotlinx.coroutines |
| پس‌زمینه | WorkManager + Foreground Service (dataSync) | 2.10.x | androidx.work |
| شبکه | OkHttp + Retrofit + kotlinx.serialization | آخرین پایدار | square/jetbrains |
| ایمیل/بایگانی | Jakarta Mail (MIME) یا mime4j + `java.util.zip` + Apache Commons Compress | — | انتخاب در P1 با benchmark |
| DI | Hilt (در صورت سازگاری تأییدشده با AGP 9/KSP) وگرنه DI دستی با factory | — | باید در P0 راستی‌آزمایی شود (ADR‑013) |
| رمزنگاری | Android Keystore + Google Tink (AEAD) + SQLCipher | Tink 1.x | `androidx.security:security-crypto` **deprecated** است → استفاده نمی‌شود |
| تنظیمات | DataStore (Preferences + Proto) | 1.1.x+ | androidx.datastore |
| تست | JUnit5، Turbine، MockK، Robolectric، MockWebServer، WireMock، Compose UI Test, Room MigrationTestHelper, androidx.benchmark | — | — |
| CI | GitHub Actions (unit + Robolectric + lint + detekt + ktlint)؛ instrumented به‌صورت nightly/دستی | — | — |

> **یادداشت صداقت:** نسخه‌های دقیق در P0 با اجرای واقعی `./gradlew` قفل می‌شوند. اگر Hilt با AGP 9/KSP ناسازگار بود، DI دستی (graph در `:app`) جایگزین می‌شود و این تغییر در ADR ثبت می‌گردد.

---

## ۳. ساختار ماژول‌ها

```
Digitalfoot/
├── app/                              # تنها ماژول قابل نصب؛ graph و navigation
├── core/
│   ├── model/                        # دامنه خالص (بدون وابستگی اندروید)
│   ├── common/                       # Result، Dispatchers، Clock، Unit‑typed ids
│   ├── datastore/                    # DataStore proto/preferences
│   ├── database/                     # Room entities/DAO، SQLCipher، migrations
│   ├── security/                     # Keystore، Tink، redaction، blind index
│   ├── permissions/                  # ماشین وضعیت مجوزها (runtime/special/Play)
│   ├── network/                      # OkHttp، auth interceptor، throttle، retry
│   ├── connector-api/                # قرارداد Connector (بدون پیاده‌سازی)
│   ├── scan/                         # ScanOrchestrator، WorkManager، checkpoint
│   ├── correlation/                  # موتور همبستگی و dedup
│   ├── import/                       # SAF، archive reader، parser registry
│   └── reporting/                    # coverage، error report، export
├── connectors/
│   ├── device-packages/  device-usage/  device-contacts/  device-calendar/
│   ├── device-notifications/          (opt‑in)
│   ├── google-identity/  google-gmail/  google-people/  google-youtube/
│   ├── google-calendar/  google-drive/  google-takeout/
│   ├── ms-graph/  ms-export/
│   ├── github/  gitlab/  stackexchange/  codeforces/
│   ├── mastodon/  discord/  twitch/  reddit/
│   ├── steam/  roblox/
│   ├── spotify/  lastfm/  strava/  fitbit/
│   ├── hibp/  wayback/  rdap/  openpgp/
│   └── imap/                          (عمومی: Gmail/Outlook/Fastmail/…)
├── parsers/                           # پارسرهای فرمت‌محور و خالص (JVM-testable)
│   ├── mbox/  eml/  zip-archive/  takeout/  meta-archive/  amazon/  apple/
│   ├── linkedin/  discord-package/  microsoft-archive/  sms-xml/  calls-csv/
│   ├── password-csv/  onepassword/  bitwarden/  keepass/  totp-qr/  opml/
│   ├── cookies-txt/  ics/  vcard/  csv-generic/  json-generic/  html-generic/
│   └── telegram-json/  whatsapp-txt/
├── features/
│   ├── onboarding/  dashboard/  sources/  timeline/  import/  identity/
│   ├── coverage/  security-health/  settings/  deletion/
└── testing/
    ├── fixtures/                      # فایل‌های نمونه واقعی/ساختگی (بدون داده شخصی)
    ├── connector-contract-test/       # کیت تست مشترک همه connectorها
    └── fakes/                         # FakeConnector، FakeClock، FakePermissionGate
```

قواعد ماژول‌ها:
- `core:model` و `parsers/*` و `core:connector-api` **هیچ وابستگی اندرویدی ندارند** → تست خالص JVM.
- هر `connectors/*` فقط به `connector-api`، `model`، `common`، `network` وابسته است؛ هرگز به connector دیگر.
- هر connector باید ماژول مستقل، قرارداد مشخص، وضعیت دسترسی، خطای قابل گزارش و تست مناسب داشته باشد (قانون ۴) — یعنی الزاماً `test/` با fixture و اجرای `connector-contract-test`.

---

## ۴. نمای کلی معماری

```
┌──────────────────────────── Features (Compose) ───────────────────────────┐
│ Onboarding │ Sources │ Import │ Timeline │ Coverage │ Security │ Deletion  │
└───────────────▲──────────────────────▲───────────────────────▲────────────┘
                │ StateFlow<UiState>   │                       │
┌───────────────┴──────────────────────┴───────────────────────┴────────────┐
│                         Use Cases (core:*)                                 │
│  RunScan · ResumeScan · CancelScan · ImportFile · ResolveIdentities        │
│  MergeDuplicates · BuildTimeline · ComputeCoverage · PurgeData             │
└───▲───────────────▲────────────────▲──────────────────▲───────────────────┘
    │               │                │                  │
┌───┴────┐   ┌──────┴───────┐  ┌─────┴──────┐   ┌───────┴────────┐
│ Scan   │   │ Correlation  │  │ Import     │   │ Reporting      │
│Orch.   │   │ + Dedup      │  │ Pipeline   │   │ (coverage/err) │
└───▲────┘   └──────▲───────┘  └─────▲──────┘   └───────▲────────┘
    │               │                │                  │
┌───┴───────────────┴────────────────┴──────────────────┴───────────────────┐
│              Connector Registry  (contract: Connector)                     │
│  device-* │ google-* │ ms-* │ github │ steam │ hibp │ imap │ parsers-*     │
└───▲───────────────────────────────────────────────────────────────────────┘
    │ EvidenceSink (append-only) + CursorStore + ConsentGate + Throttle
┌───┴───────────────────────────────────────────────────────────────────────┐
│   Repositories → Room (SQLCipher) │ EncryptedBlobStore │ TokenVault(Tink)  │
│                 │ PermissionGate  │ AuditLog           │ Scheduler(Work)   │
└───────────────────────────────────────────────────────────────────────────┘
```

### جریان داده اصلی
1. کاربر یک **Connection** می‌سازد (OAuth، کلید API، مجوز اندروید، یا فایل واردشده).
2. **ConsentGate** بررسی می‌کند رضایت معتبر (نسخه‌دار و زمان‌دار) برای آن منبع وجود دارد.
3. **ScanOrchestrator** یک `ScanRun` می‌سازد و آن را به `ScanTask`های کوچک (per cursor/page) تقسیم می‌کند.
4. **Connector** اجرا می‌شود، `Evidence` خام تولید می‌کند و در `EvidenceSink` می‌نویسد (append‑only).
5. **Ingestion** نرمال‌سازی، hashing، redaction و استخراج `Event`/`Account`/`Identifier` را انجام می‌دهد.
6. **Correlation + Dedup** هویت‌ها را به هم وصل می‌کند و تکراری‌ها را ادغام می‌کند.
7. **Reporting** پوشش، خطاها و صداقت نتایج را محاسبه می‌کند.
8. **UI** فقط از طریق Use Case و Flow به داده می‌رسد.

---

## ۵. قرارداد مشترک Connectorها (`core:connector-api`)

```kotlin
interface Connector {
    val descriptor: ConnectorDescriptor          // شناسه، نسخه، دسته، نیازمندی‌ها
    fun capabilities(): ConnectorCapabilities    // incremental، delta، pagination، offline
    fun accessState(ctx: ConnectorContext): AccessState
    suspend fun plan(request: ScanRequest): ScanPlan       // چند task، با چه cursorهایی
    fun execute(task: ScanTask, sink: EvidenceSink, progress: ProgressReporter): ConnectorResult
    suspend fun selfTest(ctx: ConnectorContext): SelfTestReport   // برای تست واقعی و عیب‌یابی
}

data class ConnectorDescriptor(
    val key: ConnectorKey,               // "google.gmail.api"
    val version: Int,                    // برای provenance
    val category: SourceCategory,        // EMAIL, IDENTITY, SOCIAL, COMMERCE, GAMING, DEV, DEVICE, ARCHIVE, PUBLIC_EVIDENCE
    val displayName: String,             // i18n key
    val requiresNetwork: Boolean,
    val requiresUserAction: UserActionKind?,   // OAUTH / API_KEY / FILE_IMPORT / SPECIAL_PERMISSION / RUNTIME_PERMISSION
    val androidPermissions: List<PermissionSpec>,
    val rateLimitHint: RateLimitHint?,   // برای throttle
    val privacyTier: PrivacyTier,        // NORMAL / SENSITIVE / CRITICAL
    val playPolicyRisk: PlayPolicyRisk,  // NONE / DECLARATION_REQUIRED / LIKELY_REJECTED
)

sealed interface AccessState {
    data object NotConfigured
    data object Ready
    data class MissingPermission(val permissions: List<String>)
    data class MissingUserAction(val action: UserActionKind)
    data class AuthExpired(val reauthHint: String)
    data class Blocked(val reason: BlockedReason)   // REGION, POLICY, TOS, LICENSE
    data class Unsupported(val reason: String)      // مثلاً API level پایین‌تر
}

sealed interface ConnectorResult {
    data class Completed(val cursor: Cursor?, val counts: EvidenceCounts)
    data class Suspended(val cursor: Cursor, val resumeAfter: Instant, val reason: SuspendReason)
    data class Failed(val error: ConnectorError)
    data class Partial(val cursor: Cursor?, val error: ConnectorError, val counts: EvidenceCounts)
}
```

### قواعد اجباری قرارداد
1. **بدون حالت در connector:** همه cursorها در `CursorStore` (Room) نگهداری می‌شوند، نه در حافظه و نه در `WorkManager.Data` (سقف ~۱۰KB).
2. **idempotent:** اجرای مجدد یک `ScanTask` با همان cursor نباید Evidence تکراری بسازد (کلید طبیعی = `sourceKey + externalId` یا `contentHash`).
3. **بدون نشت:** connector حق ندارد داده خام را لاگ کند؛ فقط `EvidenceSink` و `Redactor` مجاز به لمس محتوا هستند.
4. **توقف‌پذیر:** هر حلقه باید `ensureActive()` را صدا بزند و در مرز صفحه cursor را ذخیره کند.
5. **قابل تست:** هر connector باید `connector-contract-test` را پاس کند (شامل: plan پایدار، cursor قابل ادامه، رفتار ۴۲۹/۴۰۱/۵۰۰، رفتار فایل خراب، عدم نشت در لاگ).
6. **selfTest:** یک اجرای کوچک و بی‌خطر که وضعیت واقعی دسترسی را گزارش می‌کند (برای «تست‌ها را واقعاً اجرا کن» — قانون ۱۲).

### دسته‌بندی خطا (taxonomy مشترک)
`AUTH_REQUIRED · AUTH_EXPIRED · TOKEN_REVOKED · PERMISSION_DENIED · USER_CANCELLED · RATE_LIMITED · QUOTA_EXCEEDED · NETWORK_UNAVAILABLE · TLS_FAILURE · TIMEOUT · SERVER_ERROR · SCHEMA_MISMATCH · PARSE_ERROR · FILE_TOO_LARGE · FILE_CORRUPT · URI_PERMISSION_LOST · REGION_BLOCKED · POLICY_BLOCKED · DATA_NOT_AVAILABLE · INTERNAL · CANCELLED`

هر خطا سه ویژگی دارد: `retryable` (بله/خیر)، `backoff` (ثابت/نمایی)، `userMessageKey` (پیام صادقانه برای کاربر). هیچ خطایی «بی‌صدا» نادیده گرفته نمی‌شود؛ همه در `ErrorRecord` می‌مانند و در گزارش پوشش ظاهر می‌شوند.

---

## ۶. مدیریت منابع و مجوزها

### ماشین وضعیت مجوز (`core:permissions`)
| نوع | نمونه | روش درخواست | fallback |
| --- | --- | --- | --- |
| Runtime (dangerous) | `READ_CONTACTS`, `READ_CALENDAR`, `CAMERA`, `POST_NOTIFICATIONS` | Activity Result API + rationale | قابلیت غیرفعال + توضیح |
| Special app‑op | `PACKAGE_USAGE_STATS` | هدایت به `ACTION_USAGE_ACCESS_SETTINGS` + تشخیص با `AppOpsManager.unsafeCheckOpNoThrow("android:get_usage_stats", …)` | فقط I1 |
| System settings grant | `BIND_NOTIFICATION_LISTENER_SERVICE` | `ACTION_NOTIFICATION_LISTENER_SETTINGS` + افشای برجسته جداگانه | خاموش می‌ماند |
| Install‑time + Play policy | `QUERY_ALL_PACKAGES` | Declaration Form در Play Console | `<queries>` هدفمند (I1b) |
| Health | `android.permission.health.READ_*` | دیالوگ اختصاصی Health Connect + privacy policy | غیرفعال |
| SAF | هیچ مجوزی لازم نیست | `ACTION_OPEN_DOCUMENT(_TREE)` + `takePersistableUriPermission` | کاربر فایل را کپی می‌کند |

### ConsentGate
- هر منبع یک `ConsentRecord` دارد: نسخه متن رضایت، زمان، دامنه (کدام داده)، آیا opt‑in بوده یا پیش‌فرض.
- برای `PrivacyTier.CRITICAL` (متن پیامک، رمزها، کوکی، سلامت، مکان) رضایت **جدا و صریح** لازم است و پیش‌فرض خاموش است.
- لغو رضایت → connector متوقف و (به انتخاب کاربر) داده آن منبع حذف می‌شود.

### مدیریت منابع اجرایی
- **Throttle:** هر connector یک `RateLimiter` (token bucket) با پارامترهای اعلام‌شده در `rateLimitHint` و سرصفحه‌های واقعی (`X-RateLimit-Remaining`, `retry-after`, `Retry-After`, Gmail quota units) دارد.
- **Quota budget:** بودجه روزانه per‑connector (مثلاً YouTube ۱۰٬۰۰۰ واحد/روز) در `QuotaLedger` ثبت و در UI نشان داده می‌شود؛ وقتی بودجه تمام شد connector `Suspended` می‌شود نه `Failed`.
- **Connection pool:** سقف همزمانی شبکه (پیش‌فرض ۴) و سقف همزمانی parse (پیش‌فرض ۲) تا حافظه منفجر نشود.
- **Memory discipline:** پارسرهای آرشیو streaming هستند (ورودی `InputStream`، خروجی `Sequence<Evidence>`)؛ ممنوعیت `readBytes()` روی فایل بزرگ؛ سقف سخت اندازه رکورد و عمق JSON.

---

## ۷. موتور همبستگی شواهد (Evidence Correlation Engine)

هدف: پاسخ به «این دو رکورد درباره یک حساب/یک هویت‌اند؟» با **نتیجه قابل توضیح** و بدون ادعای کاذب.

### ۷.۱ ورودی
`Identifier`های نرمال‌شده: email (lowercase, punycode, حذف `+tag` با ثبت نسخه اصلی), phone (E.164), username (lowercase, بدون leading `@`), URL (canonical, بدون query غیرضروری), name (نرمال‌سازی یونیکد + variant), externalId (per service).

### ۷.۲ مسدودسازی (blocking) برای جلوگیری از مقایسه O(n²)
کلیدهای بلوک: `email_hash`, `phone_e164`, `username_norm`, `domain`, `name_soundex`, `service_account`. فقط جفت‌های هم‌بلوک مقایسه می‌شوند.

### ۷.۳ امتیازدهی جفت (deterministic, explainable)
| ویژگی | وزن اولیه | توضیح |
| --- | --- | --- |
| تطابق کامل ایمیل | ۰٫۹۵ | قوی‌ترین شاهد |
| ایمیل با alias یکسان (`base+tag`) | ۰٫۸۵ | فقط وقتی `base` یکسان است |
| تطابق شماره (E.164) | ۰٫۹ | |
| تطابق username + همان سرویس | ۰٫۸ | |
| تطابق username بین سرویس‌ها | ۰٫۴ | رایج است → به‌تنهایی کافی نیست |
| تطابق نام نمایشی + دامنه یکسان | ۰٫۵ | |
| شواهد زمانی هم‌پوشان (±۲۴ ساعت روی همان سرویس) | ۰٫۳ | |
| اتصال اعلام‌شده سرویس (Discord connections, GitHub installations) | ۰٫۹ | منبع رسمی |
| تضاد (ایمیل متفاوت با username یکسان در همان سرویس) | −۰٫۶ | جلوی ادغام غلط |

خروجی: `score ∈ [0,1]` + `explanation: List<Feature(id, weight, evidenceIds)>`.

### ۷.۴ آستانه‌ها و وضعیت‌ها
- `score ≥ 0.90` → `LinkStatus.AUTO_LINKED` (همچنان قابل واگردانی)
- `0.65 ≤ score < 0.90` → `SUGGESTED` (نیازمند تأیید کاربر برای `CONFIRMED`)
- `score < 0.65` → `NOT_LINKED`
- هیچ ادغامی وضعیت حساب را به `CONFIRMED` نمی‌برد مگر: (الف) شاهد رسمی از خود سرویس، یا (ب) تأیید صریح کاربر، یا (پ) ≥۲ منبع مستقل با score ≥ ۰٫۸۵. این مستقیماً «اصل صداقت در پوشش داده» را اجرا می‌کند.

### ۷.۵ خوشه‌بندی
Union‑Find روی لینک‌های `AUTO_LINKED`/`CONFIRMED` → `IdentityCluster`. هر خوشه یک `clusterId` و یک `confidence` دارد. واگردانی ادغام = حذف لینک + بازسازی خوشه (نه حذف داده).

### ۷.۶ حلقه بازخورد
`MergeDecision(source, userAction, timestamp)` ذخیره می‌شود تا (۱) واگردانی ممکن باشد و (۲) در نسخه‌های بعد وزن‌ها بر اساس داده **خود کاربر روی دستگاه** تنظیم شوند (بدون ارسال به هیچ سروری).

### ۷.۷ بدون ML در نسخه ۱
تصمیم: نسخه ۱ فقط قوانین وزنی قابل توضیح (ADR‑007). مدل زبانی/یادگیری ماشین فقط در صورت نیاز و با داده روی دستگاه و با مستندسازی دقت/فراخوانی واقعی اضافه می‌شود.

---

## ۸. مدیریت تکراری‌ها (Deduplication)

| لایه | کلید | روش |
| --- | --- | --- |
| ۱. منبع یکسان | `(sourceKey, externalId)` | upsert؛ فیلدهای جدید با `observedAt` تازه‌تر برنده می‌شوند |
| ۲. محتوای یکسان از منابع مختلف | `contentHash = SHA‑256(canonicalJson(payload بدون فیلدهای ناپایدار))` | اولین رکورد برنده، بقیه `DuplicateOf` |
| ۳. ایمیل تکراری در چند آرشیو | `messageId` (RFC 822) + fallback `(from, date, subjectHash)` | ادغام با ثبت چند منشأ |
| ۴. حساب یکسان در چند منبع | موتور همبستگی (بخش ۷) | `IdentityCluster` |
| ۵. واردکردن مجدد همان فایل | `fileHash = SHA‑256(file)` در `ImportedFile` | هشدار به کاربر + حالت «به‌روزرسانی» به‌جای افزودن |

قواعد:
- **هرگز داده حذف نمی‌شود تا ادغام قطعی شود**؛ ادغام = ایجاد لینک، و `MergeLog` کامل نگه داشته می‌شود.
- tombstone برای داده حذف‌شده توسط کاربر (تا provenance باقی بماند بدون محتوا).
- تست: golden fixtures با تکراری عمدی + تست idempotency (اجرای دوباره همان task → صفر رکورد جدید).

---

## ۹. خط زمانی (Timeline)

### مدل رویداد
```kotlin
data class TimelineEvent(
    val id: EventId,
    val subjectRef: SubjectRef?,          // حساب/سرویس مرتبط (ممکن است null = دستگاه‌محور)
    val kind: EventKind,                  // ACCOUNT_CREATED, EMAIL_RECEIVED, SIGNUP_VERIFIED,
                                          // PASSWORD_RESET, NEW_LOGIN, SECURITY_ALERT, PURCHASE,
                                          // SUBSCRIPTION_RENEWED, SUBSCRIPTION_CANCELLED, POST_PUBLISHED,
                                          // APP_INSTALLED, APP_UPDATED, APP_USED, PERMISSION_GRANTED,
                                          // IDENTITY_CHANGED, ACCOUNT_LINKED, ACCOUNT_DELETED_EVIDENCE,
                                          // BREACH_DISCLOSED, ARCHIVE_SNAPSHOT, DATA_IMPORTED, UNKNOWN
    val occurredAt: Instant?,             // ممکن است ناشناخته باشد
    val datePrecision: DatePrecision,     // EXACT_INSTANT, EXACT_DATE, DAY, MONTH, YEAR, RANGE, UNKNOWN
    val dateKind: DateKind,               // REGISTERED, VERIFIED, RECEIVED, OBSERVED, ESTIMATED
    val derivation: Derivation?,          // ruleId + inputEvidenceIds + note (برای ESTIMATED الزامی)
    val confidence: Confidence,           // CONFIRMED / HIGH / MEDIUM / LOW
    val validationStatus: ValidationStatus,
    val evidenceIds: List<EvidenceId>,    // حداقل یک شاهد؛ رویداد بدون شاهد ساخته نمی‌شود
    val firstSeenRunId: RunId,
    val timezoneHint: String?             // فقط در صورت وجود در منبع
)
```

### قواعد زمانی (اجرای قانون ۶)
1. `occurredAt` **فقط** از فیلد رسمی منبع یا از شاهد مستقیم پر می‌شود.
2. اگر تاریخ از شاهد غیرمستقیم استنتاج شد: `dateKind = ESTIMATED` + `derivation` با `ruleId` و `inputEvidenceIds` اجباری است و `confidence ≤ MEDIUM`.
3. تفکیک سه زمان در همه رکوردها: `occurredAt` (کی اتفاق افتاد)، `observedAt` (منبع کی آن را ثبت کرد)، `ingestedAt` (ما کی دیدیم).
4. منطقه زمانی: همه‌چیز در DB به UTC؛ نمایش در منطقه زمانی دستگاه با برچسب «منبع بدون timezone».
5. رویداد بدون `occurredAt` در خط زمانی در بخش «بدون تاریخ» نشان داده می‌شود، نه با تاریخ جعلی.
6. نمایش UI همیشه `precision` و `confidence` را نشان می‌دهد (مثلاً «حدوداً مارس ۲۰۱۹ — شاهد: اولین ایمیل»).

### قواعد استنتاج (rule catalog — نمونه)
| ruleId | ورودی | خروجی | اطمینان |
| --- | --- | --- | --- |
| `EMAIL_FIRST_CONTACT` | قدیمی‌ترین ایمیل از دامنه سرویس | `ACCOUNT_CREATED ≥ occurredAt` | LOW/MEDIUM |
| `EMAIL_SIGNUP_RECEIPT` | ایمیل با الگوی «welcome/verify/confirm your email» + DKIM معتبر | `ACCOUNT_CREATED ≈ occurredAt` | MEDIUM/HIGH |
| `SERVICE_CREATED_AT` | فیلد رسمی (`created_at`, `timecreated`, `memberSince`) | `ACCOUNT_CREATED = occurredAt` | CONFIRMED |
| `ARCHIVE_FIRST_SNAPSHOT` | اولین رکورد Wayback | `ACCOUNT_CREATED ≤ occurredAt` | MEDIUM |
| `BREACH_DATE` | `BreachDate` HIBP | `ACCOUNT_CREATED ≤ breachDate` | MEDIUM |
| `APP_FIRST_INSTALL` | `firstInstallTime` | `APP_INSTALLED` (نه ثبت‌نام!) | HIGH برای نصب |
| `IDENTITY_CHANGE` | تفاوت snapshot پروفایل | `IDENTITY_CHANGED` با مقدار قبلی/جدید | HIGH |

---

## ۱۰. مدیریت اسکن و ادامه از نقطه توقف

### ساختار
```
ScanJob (منطقی، تعریف‌شده توسط کاربر: «اسکن کامل»، «اسکن Gmail»، «بازاسکن دوره‌ای»)
└── ScanRun (یک اجرا؛ وضعیت: PENDING/RUNNING/SUSPENDED/COMPLETED/FAILED/CANCELLED)
    └── ScanTask (یک صفحه/یک cursor/یک فایل؛ کوچک و idempotent)
        └── Cursor (JSON پایدار در Room)
```

### قواعد
1. **Checkpoint در مرز صفحه:** بعد از هر صفحه موفق، `Cursor` و `EvidenceCounts` در یک تراکنش atomic نوشته می‌شود.
2. **بازگشت از سقوط:** هنگام راه‌اندازی، `ScanRun`های `RUNNING` به `SUSPENDED` تبدیل و از آخرین cursor ادامه می‌یابند.
3. **بک‌گراند:** WorkManager با `Constraints(networkType, requiresCharging برای آرشیوهای بزرگ)` و `BackoffPolicy.EXPONENTIAL`. برای اسکن فعال و طولانی: `setExpedited` یا Foreground Service با `foregroundServiceType="dataSync"` + `FOREGROUND_SERVICE_DATA_SYNC` (محدودیت ~۶ ساعت در ۲۴ ساعت از اندروید ۱۵ → اسکن‌های بزرگ به قطعات WorkManager تقسیم می‌شوند).
4. **پیشرفت قابل مشاهده:** `setProgress` با `(runId, tasksDone, tasksTotal, currentSource, phase)` → نوار پیشرفت واقعی در UI + اعلان.
5. **لغو:** `CancellationTokenSource` مشترک؛ لغو باید cursor را نگه دارد (ادامه ممکن بماند) و در `AuditLog` ثبت شود.
6. **Delta sync:** برای منابع دارای delta رسمی از همان استفاده می‌شود: Gmail `history.list(startHistoryId)` (با fallback به `messages.list` هنگام `historyId expired`)، Graph `$deltaToken`، GitHub `updated_at`/`since`، IMAP `UID` + `MODSEQ`/`SEARCH SINCE`.
7. **بازاسکن دوره‌ای:** قابل تنظیم (پیش‌فرض خاموش)، با احترام به سهمیه و با گزارش «چه چیزی تغییر کرد».
8. **مقاومت در برابر قطع شبکه:** هر connector باید `Partial` برگرداند و cursor را حفظ کند؛ شبکه که آمد، ادامه از همان‌جا (نه از صفر).
9. **تست واقعی:** تست «kill فرایند وسط اسکن» (instrumented) و تست «قطع شبکه در صفحه N» (MockWebServer) اجباری است.

---

## ۱۱. گزارش پوشش و خطاها

### وضعیت‌های پوشش (دقیقاً همان شش وضعیت Rules + دو وضعیت فنی)
`DISCOVERED · PROBABLE · CONFIRMED · INACCESSIBLE · NOT_CHECKED · NO_RESULT` + `ERROR · SKIPPED_BY_USER`

### ماتریس پوشش
برای هر `SourceCategory` × هر منبع: تعداد شواهد، آخرین اجرا، وضعیت دسترسی، خطاهای باز، و «چرا بررسی نشد» (مثلاً «مجوز رد شد»، «کلید API وارد نشده»، «سرویس API ندارد → از مسیر درخواست داده استفاده کن»).

### خروجی‌ها
1. **صفحه Coverage:** درصد بررسی‌شده، دسته‌های بررسی‌نشده، و دلیل صادقانه هر کدام.
2. **صفحه Errors:** فهرست خطاها با منبع، نوع، زمان، retryable، و اقدام پیشنهادی.
3. **گزارش متنی/JSON قابل صدور** (بدون توکن، بدون رمز، بدون مقدار کوکی، بدون IP کامل مگر انتخاب کاربر).
4. **هیچ‌گاه** جمله‌ای مثل «همه حساب‌های شما پیدا شد» تولید نمی‌شود. متن پیش‌فرض: «N حساب با شاهد مستقیم، M مورد احتمالی، K دسته غیرقابل دسترسی، J دسته بررسی‌نشده».

### تست
- Snapshot test روی متن‌های گزارش (تا ادعای کاذب وارد نشود).
- تست unit که اگر `NOT_CHECKED > 0` باشد گزارش حتماً آن را ذکر کند.

---

## ۱۲. امنیت، رمزگذاری و حذف داده

خلاصه تصمیم‌ها (جزئیات کامل: `docs/architecture/security-model.md`):
- DB با **SQLCipher** و کلید محافظت‌شده در **Android Keystore** (hardware‑backed در صورت وجود).
- blobهای خام (فایل‌های واردشده، بدنه ایمیل) با **Tink AEAD** رمزگذاری و در app‑private storage.
- توکن‌های OAuth/کلیدهای API در **TokenVault** (Tink + Keystore)؛ هرگز در SharedPreferences معمولی، هرگز در لاگ، هرگز در Git.
- جست‌وجو روی داده رمزنگاری‌شده با **blind index** (HMAC با کلید Keystore) + **FTS5** روی متن مجاز (بدون رمز/بدنه حساس).
- `allowBackup=false` + `dataExtractionRules` برای جلوگیری از خروج داده با بکاپ ابری.
- `FLAG_SECURE` روی صفحات حساس + قفل بیومتریک اختیاری.
- **Redaction لایه‌ای:** `LogRedactor` (allowlist)، `ErrorRedactor`، `ExportRedactor`.
- **حذف:** سه سطح — حذف یک شاهد، حذف یک منبع (همه داده + توکن + cursor)، «پاک‌سازی کامل» (wipe DB + blob + keyset با confirm دومرحله‌ای) + `AuditLog` بدون محتوا.

---

## ۱۳. مدل داده یکپارچه

تعریف کامل موجودیت‌ها، فیلدها، ایندکس‌ها و مهاجرت‌ها در `docs/architecture/data-model.md`. خلاصه موجودیت‌های اصلی:

`OwnerProfile · Subject(ServiceAccount) · Identifier · Evidence · RawBlobRef · TimelineEvent · Connection · ConsentRecord · SourceCredentialsRef · ScanJob · ScanRun · ScanTask · Cursor · CorrelationLink · IdentityCluster · MergeLog · DuplicateGroup · ImportedFile · CoverageRecord · ErrorRecord · QuotaLedger · AuditLog · Settings`

اصل: **Evidence منبع حقیقت است** (append‑only) و هر چیزی که کاربر می‌بیند (Account, Event, Cluster) از آن مشتق و قابل بازسازی است. این یعنی اگر الگوریتم همبستگی عوض شود، داده خام از دست نمی‌رود.

---

## ۱۴. معماری اتصال آنلاین

| سناریو | الگو |
| --- | --- |
| OAuth با redirect (Google, Microsoft, GitHub, Discord, Twitch, Mastodon, Spotify) | `CustomTabs` + `AppAuth`/MSAL + redirect scheme اختصاصی؛ PKCE اجباری؛ refresh token در TokenVault |
| Device flow (GitHub برای دستگاه بدون مرورگر راحت) | `OAuth2 Device Flow` رسمی |
| کلید API کاربر (Steam, HIBP, Last.fm, Riot, GitLab PAT) | ورودی دستی + اعتبارسنجی فوری با `selfTest` |
| IMAP با XOAUTH2 | توکن OAuth از همان provider + `AUTHENTICATE XOAUTH2`؛ App Password فقط با هشدار صریح |
| فایل (Takeout, آرشیوها, CSV, XML, MBOX) | SAF + streaming + hash + checkpoint |
| همه | یک `HttpClient` مشترک با: interceptor احراز هویت، retry با backoff، خواندن هدرهای rate limit، timeouts سخت، User‑Agent شناسا (الزام HIBP)، **بدون** certificate pinning سخت برای سرویس‌های ثالث (pinning فقط برای endpointهای ثابت خودمان — که وجود ندارند چون backend نداریم) |

---

## ۱۵. معماری داده محلی و پردازش آفلاین

1. `ImportRequest` (URI, hash, size, parserId, options) → `ImportedFile` با وضعیت.
2. `ArchiveReader` به‌صورت lazy پیمایش می‌کند (ZIP/TGZ؛ پشتیبانی از آرشیو چندتکه با ترتیب‌دهی `z01..zip`).
3. `ParserRegistry` بر اساس مسیر/پسوند/MAGIC parser انتخاب می‌کند؛ اگر ناشناخته بود → `GenericParser` + «mapping assistant» (K2).
4. هر parser خروجی `Sequence<RawEvidence>` + `ParseReport` (تعداد خوانده‌شده، ردشده، خطاها، نسخه parser) می‌دهد.
5. در پایان، `ImportedFile.status = COMPLETED|PARTIAL|FAILED` و کاربر **دقیقاً** می‌بیند چند درصد فایل پردازش شد و چرا بقیه نشد.
6. حذف فایل منبع توسط کاربر هیچ اثری بر داده استخراج‌شده ندارد (چون در DB رمزنگاری‌شده است) مگر کاربر صریحاً «حذف همراه با فایل» را بخواهد.

---

## ۱۶. معماری UI (Compose)

- الگو: `ViewModel` + `StateFlow<UiState>` + رویدادهای یک‌بارمصرف با `Channel`.
- صفحه‌ها: Onboarding (شفافیت و رضایت)، Dashboard (خلاصه + پوشش)، Sources (اتصال/وضعیت/خطا)، Import (پیشرفت و گزارش پارس)، Timeline (فیلتر بر اساس kind/confidence/dateKind)، Identity (خوشه‌ها + ادغام دستی + واگردانی)، Coverage (ماتریس صادقانه)، Security Health، Settings، Deletion Center.
- هر سطح اطمینان یک برچسب بصری ثابت دارد؛ «احتمالی» هرگز مثل «تأییدشده» نمایش داده نمی‌شود.
- دسترسی‌پذیری: حداقل کنتراست، برچسب contentDescription، پشتیبانی RTL (فارسی/عربی) از روز اول.
- i18n: `fa`, `en`, `de` در نسخه ۱ (با ساختار قابل گسترش).

---

## ۱۷. استراتژی تست

| سطح | ابزار | چه چیزی | دروازه |
| --- | --- | --- | --- |
| Unit (JVM) | JUnit5, MockK, Turbine, kotlinx‑coroutines‑test | مدل دامنه، قواعد استنتاج، امتیازدهی همبستگی، dedup، throttle، redaction | ≥۸۰٪ روی `core:*` و `parsers/*` |
| Parser golden-file | fixtures واقعی/ساختگی (بدون داده شخصی) | MBOX, Takeout JSON, Meta JSON, Amazon CSV, LinkedIn CSV, Discord package, SMS XML, password CSV, cookies.txt, ICS, vCard, OPML, KDBX | ۱۰۰٪ fixtures در CI اجرا می‌شوند |
| Fuzzing | ورودی مخرب/ناتمام | ZIP bomb، JSON عمیق، CSV ناقص، MBOX خراب، Unicode | هیچ crash؛ همه `PARSE_ERROR` با گزارش |
| Contract test | `connector-contract-test` | هر connector: plan/cursor/idempotency/error mapping/no‑leak | هر connector جدید بدون پاس‌کردن merge نمی‌شود |
| Network integration | MockWebServer/WireMock | ۴۰۱/۴۰۳/۴۲۹/۵۰۰، سهمیه، delta، pagination | سناریوهای ثبت‌شده در fixture |
| DB | Room `MigrationTestHelper` + SQLCipher | مهاجرت‌ها، ایندکس‌ها، تراکنش‌های atomic | تست مهاجرت برای هر تغییر schema |
| Android integration | Robolectric + instrumented (emulator) | SAF، PackageManager، UsageStats، NotificationListener، WorkManager resume | تست «kill وسط اسکن» و «قطع شبکه» |
| UI | Compose UI Test + screenshot (Paparazzi/Roborazzi) | جریان‌های رضایت، برچسب اطمینان، RTL | سناریوهای بحرانی |
| امنیت | تست‌های «هیچ رازی در لاگ/DB/export نیست»، `allowBackup` بررسی، lint امنیتی | redaction | **بلوکه‌کننده**: شکست = عدم انتشار |
| عملکرد | Macrobenchmark | زمان اولین اسکن، مصرف حافظه هنگام import ۲GB | بودجه‌های تعریف‌شده در acceptance-criteria |

**قانون اجرایی (قانون ۱۲):** نتیجه واقعی هر اجرای تست در گزارش پایان مرحله می‌آید (تعداد پاس/شکست/اجراننشده) و هیچ تستی بدون دلیل مستند غیرفعال نمی‌شود.

---

## ۱۸. تصمیم‌های کلیدی (خلاصه؛ متن کامل در `docs/decisions/adr-log.md`)

| ADR | تصمیم |
| --- | --- |
| 001 | Kotlin + Jetpack Compose + معماری لایه‌ای/ماژولار + Coroutines/Flow + Room + WorkManager |
| 002 | فقط روی دستگاه؛ هیچ backend/تحلیلی/ابر همگام‌سازی |
| 003 | SQLCipher + Keystore + Tink؛ `androidx.security-crypto` استفاده نمی‌شود (deprecated) |
| 004 | سیاست «metadata‑only» برای رازها: رمز/کوکی/secret هرگز ذخیره نمی‌شود |
| 005 | package visibility: تلاش برای `QUERY_ALL_PACKAGES` + fallback `<queries>` |
| 006 | Notification Listener خاموش به‌صورت پیش‌فرض + allowlist + redaction بدنه |
| 007 | همبستگی با قوانین وزنی قابل توضیح؛ بدون ML در نسخه ۱ |
| 008 | Evidence append‑only به‌عنوان منبع حقیقت؛ مشتقات بازسازی‌پذیر |
| 009 | هر منبع = یک ماژول با قرارداد و تست مشترک |
| 010 | cursor در Room نه در WorkManager.Data؛ checkpoint در مرز صفحه |
| 011 | ممنوعیت scraping/enumeration و ممنوعیت جمع‌آوری داده اشخاص ثالث |
| 012 | Room 2.8 در حالت سازگاری SupportSQLite برای SQLCipher |
| 013 | DI: Hilt اگر سازگاری تأیید شد، وگرنه DI دستی |
| 014 | انتشار دوگانه: Play (با fallback مجوزها) و F‑Droid/sideload (بدون محدودیت Play) |
