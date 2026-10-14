# مدل داده یکپارچه — Unified Data Model

> مرحله ۱ — ۲۰۲۶‑۱۰‑۱۴ · پایگاه داده: Room 2.8 (حالت سازگاری SupportSQLite) روی SQLCipher
> اصل حاکم: **Evidence منبع حقیقت است.** هر موجودیت نمایشی (Account/Event/Cluster) از Evidence مشتق و قابل بازسازی است.
> هیچ فیلد زمانی بدون `kind` و `precision` وجود ندارد (اجرای قانون ۶ Rules).

---

## ۱. نقشه موجودیت‌ها

```
OwnerProfile 1 ── n Identifier 1 ── n IdentifierLink n ── 1 Subject
                        │                                     │
                        └────────── n Evidence n ─────────────┘
                                      │
                                      └── n TimelineEvent
Connection 1 ── n ScanRun 1 ── n ScanTask 1 ── 1 Cursor
Connection 1 ── 1 ConsentRecord (per scope/version)
Connection 1 ── 1 SourceCredentialsRef ── (TokenVault, خارج از DB)
Evidence n ── 1 RawBlobRef
IdentityCluster 1 ── n Subject ;  CorrelationLink n──n (Subject/Identifier)
ImportedFile 1 ── n Evidence
CoverageRecord / ErrorRecord / QuotaLedger / AuditLog / MergeLog / DuplicateGroup
```

---

## ۲. انواع پایه (value types در `core:model`)

```kotlin
@JvmInline value class OwnerId(val raw: String)          // همیشه یک owner (تک‌کاربره) در نسخه ۱
@JvmInline value class SubjectId(val raw: String)
@JvmInline value class EvidenceId(val raw: Long)
@JvmInline value class EventId(val raw: Long)
@JvmInline value class ConnectionId(val raw: String)
@JvmInline value class RunId(val raw: String)
@JvmInline value class TaskId(val raw: String)
@JvmInline value class ConnectorKey(val raw: String)     // "google.gmail.api"
@JvmInline value class ClusterId(val raw: String)
```

### ۲.۱ دسته‌بندی منبع
```kotlin
enum class SourceCategory {
    EMAIL, IDENTITY, GOOGLE, MICROSOFT, SOCIAL, MESSAGING, FORUM, CONTENT,
    COMMERCE, PAYMENT, SUBSCRIPTION, GAMING, EDUCATION, DEVELOPER, BROWSER,
    DEVICE, SMS_LIKE, NOTIFICATION, ARCHIVE_IMPORT, LOCAL_FILE,
    PUBLIC_EVIDENCE, BREACH_EVIDENCE, HEALTH, TRAVEL, FINANCE, TELECOM,
    AD_TECH, DELETED_ACCOUNT_EVIDENCE, OTHER
}
```

### ۲.۲ وضعیت اعتبارسنجی (دقیقاً مطابق Rules)
```kotlin
enum class ValidationStatus {
    DISCOVERED,      // شاهد وجود دارد، بررسی نشده
    PROBABLE,        // شواهد غیرمستقیم/متعارض
    CONFIRMED,       // شاهد رسمی سرویس یا تأیید صریح کاربر
    INACCESSIBLE,    // منبع وجود دارد ولی دسترسی مجاز ممکن نیست (دلیل ثبت می‌شود)
    NOT_CHECKED,     // هنوز اجرا نشده (با دلیل: مجوز/کلید/زمان)
    NO_RESULT,       // اجرا شد و چیزی پیدا نشد
    ERROR,           // اجرا ناموفق (به ErrorRecord ارجاع می‌دهد)
    SKIPPED_BY_USER, // کاربر آگاهانه رد کرد
    REVOKED,         // دسترسی قبلی داشت و لغو/منقضی شد
    DELETED_ACCOUNT_EVIDENCE  // شواهد حساب حذف/غیرفعال‌شده
}
```

### ۲.۳ اطمینان
```kotlin
enum class Confidence { CONFIRMED, HIGH, MEDIUM, LOW }
// CONFIRMED فقط برای فیلد رسمی سرویس یا تأیید کاربر.
// هر Confidence یک score عددی مستند دارد: 1.00 / 0.80 / 0.55 / 0.30
```

### ۲.۴ معنا و دقت زمان
```kotlin
enum class DateKind {
    REGISTERED,   // تاریخ ساخت حساب (فیلد رسمی سرویس)
    VERIFIED,     // تأیید ایمیل/شماره
    RECEIVED,     // زمان دریافت ایمیل/پیامک
    OBSERVED,     // زمان مشاهده توسط منبع (مثلاً تاریخ پست، تاریخ نقض)
    ESTIMATED,    // استنتاج ما از شواهد (derivation اجباری)
    INGESTED,     // زمان ورود به پایگاه داده ما
    INSTALLED,    // تاریخ نصب اپ روی این دستگاه (نه ثبت‌نام!)
    UNKNOWN
}
enum class DatePrecision {
    EXACT_INSTANT, EXACT_DATE, DAY, MONTH, QUARTER, YEAR, DECADE, RANGE, UNKNOWN
}
data class TimeFact(
    val value: Instant?,          // null مجاز است
    val kind: DateKind,
    val precision: DatePrecision,
    val rangeStart: Instant? = null,
    val rangeEnd: Instant? = null,
    val sourceTimezone: String? = null,   // فقط اگر منبع داده باشد
    val derivation: Derivation? = null     // برای ESTIMATED اجباری (NOT NULL check)
)
data class Derivation(
    val ruleId: String,                  // "EMAIL_FIRST_CONTACT"
    val ruleVersion: Int,
    val inputEvidenceIds: List<Long>,
    val note: String?,
    val computedAt: Instant
)
```

> **قید پایگاه داده:** اگر `kind = ESTIMATED` آنگاه `derivation` باید غیرتهی باشد (بررسی در لایه DAO + تست unit).

---

## ۳. موجودیت‌ها

### ۳.۱ `OwnerProfile`
| فیلد | نوع | توضیح |
| --- | --- | --- |
| id | TEXT PK | `OwnerId` |
| display_name | TEXT? | واردشده توسط کاربر |
| created_at | INTEGER | UTC ms |
| locale | TEXT | `fa`, `en`, `de` |
| onboard_consent_version | INTEGER | نسخه متنی که کاربر پذیرفت |
| biometric_lock_enabled | INTEGER(bool) | |
| last_purge_at | INTEGER? | |

### ۳.۲ `Identifier`
| فیلد | نوع | توضیح |
| --- | --- | --- |
| id | INTEGER PK | |
| owner_id | TEXT FK | |
| kind | TEXT | `EMAIL, EMAIL_ALIAS, PHONE, USERNAME, PROFILE_URL, FULL_NAME, DOMAIN, EXTERNAL_ID, DEVICE_APP, TOTP_ISSUER, PGP_FINGERPRINT, AD_ACCOUNT_REF, OTHER` |
| value_cipher | BLOB | Tink AEAD روی مقدار نرمال‌شده (برای نمایش) |
| blind_index | TEXT | `HMAC_SHA256(keystoreKey, normalizedValue)` → جست‌وجو/dedup بدون رمزگشایی |
| normalized_note | TEXT | چه نرمال‌سازی‌ای اعمال شد (`lowercase`, `E164`, `punycode`, `alias_stripped`) |
| original_kept | INTEGER(bool) | آیا نسخه اصلی (مثلاً `+tag`) هم نگه داشته شد |
| first_seen_evidence_id | INTEGER | |
| first_seen_at | INTEGER | |
| last_seen_at | INTEGER | |
| privacy_tier | TEXT | `NORMAL/SENSITIVE/CRITICAL` |
| third_party | INTEGER(bool) | **true = متعلق به شخص دیگر است → هرگز ذخیره انبوه نمی‌شود** |

ایندکس‌ها: `UNIQUE(kind, blind_index)`، `INDEX(owner_id)`، `INDEX(third_party)`.

### ۳.۳ `Subject` (حساب/سرویس)
| فیلد | نوع | توضیح |
| --- | --- | --- |
| id | TEXT PK | |
| owner_id | TEXT FK | |
| service_key | TEXT | شناسه پایدار سرویس (`github`, `steam`, `digikala`) |
| service_display_name | TEXT | i18n |
| service_url | TEXT? | دامنه اصلی |
| cluster_id | TEXT? FK | خوشه هویت |
| primary_identifier_id | INTEGER? | |
| status | TEXT | `ACTIVE, INACTIVE_SUSPECTED, DELETED_EVIDENCE, UNKNOWN, USER_CLOSED` |
| validation_status | TEXT | از `ValidationStatus` |
| confidence | TEXT | |
| registered_time | BLOB(json) | `TimeFact` (ممکن است null) |
| first_evidence_time | BLOB(json) | `TimeFact` |
| last_evidence_time | BLOB(json) | `TimeFact` |
| evidence_count | INTEGER | کشود‌شده (denormalized) |
| notes | TEXT? | یادداشت کاربر |
| created_at / updated_at | INTEGER | |

ایندکس‌ها: `UNIQUE(service_key, primary_identifier_id)`، `INDEX(cluster_id)`، `INDEX(validation_status)`.

### ۳.۴ `Evidence` (append‑only — منبع حقیقت)
| فیلد | نوع | توضیح |
| --- | --- | --- |
| id | INTEGER PK auto | |
| source_key | TEXT | کدام connector/parser |
| source_version | INTEGER | نسخه connector (برای provenance) |
| connection_id | TEXT FK | |
| run_id | TEXT FK | کدام اجرا |
| task_id | TEXT FK | |
| parser_id / parser_version | TEXT/INT | برای مسیر import |
| kind | TEXT | `MESSAGE, PROFILE, ACTIVITY, PURCHASE, SUBSCRIPTION, POST, INSTALL, PERMISSION_GRANT, USAGE_EVENT, NOTIFICATION_META, BREACH_RECORD, ARCHIVE_SNAPSHOT, DOMAIN_RECORD, KEY_RECORD, CONNECTION_LINK, FILE_RECORD, SECURITY_EVENT, OTHER` |
| external_id | TEXT? | شناسه در منبع (messageId, repo id, …) |
| content_hash | TEXT | SHA‑256 payload نرمال‌شده |
| payload_cipher | BLOB | Tink AEAD روی payload نرمال‌شده (ساختار JSON مستند per kind) |
| payload_schema | TEXT | نسخه schema payload |
| blob_ref_id | INTEGER? FK | ارجاع به داده حجیم خام |
| occurred_at / date_kind / date_precision | INTEGER/TEXT/TEXT | `TimeFact` تخت‌شده برای ایندکس |
| observed_at | INTEGER? | زمان ثبت در منبع |
| ingested_at | INTEGER | زمان ورود به DB |
| occurred_range_start / occurred_range_end | INTEGER? | برای `RANGE` |
| derivation | BLOB(json)? | برای ESTIMATED |
| confidence | TEXT | |
| validation_status | TEXT | |
| subject_id | TEXT? FK | بعد از همبستگی پر می‌شود |
| identifier_ids | TEXT(json) | |
| sensitive_fields_redacted | INTEGER(bool) | آیا قبل از ذخیره redaction انجام شد |
| dedup_group_id | TEXT? | |
| deleted | INTEGER(bool) | tombstone (محتوا پاک، provenance باقی) |

ایندکس‌ها: `UNIQUE(source_key, external_id)` (partial: وقتی external_id ناتهی)، `INDEX(content_hash)`، `INDEX(subject_id)`، `INDEX(occurred_at)`، `INDEX(run_id)`، `INDEX(kind, validation_status)`.

> **قاعده:** هیچ `UPDATE` روی `payload_cipher` یک Evidence موجود انجام نمی‌شود. تغییر = Evidence جدید + `supersedes_id`.

### ۳.۵ `RawBlobRef` (داده حجیم خام)
| فیلد | نوع | توضیح |
| --- | --- | --- |
| id | INTEGER PK | |
| relative_path | TEXT | مسیر در app‑private encrypted store |
| size_bytes | INTEGER | |
| sha256 | TEXT | |
| encryption | TEXT | `TINK_AEAD_STREAM` |
| mime | TEXT | |
| retention_policy | TEXT | `KEEP, DELETE_AFTER_PARSE, DELETE_ON_SOURCE_DELETE, USER_CHOICE` |
| created_at | INTEGER | |

### ۳.۶ `TimelineEvent`
مطابق بخش ۹ `system-design.md` (فیلدهای `id, subject_id, kind, occurred_at, date_precision, date_kind, derivation, confidence, validation_status, first_seen_run_id, timezone_hint, deleted`).
ایندکس: `INDEX(occurred_at)`, `INDEX(subject_id, kind)`, `INDEX(kind, confidence)`, `INDEX(date_kind)`.
جست‌وجوی متنی: جدول FTS5 مجازی `timeline_fts(title, summary)` که **فقط** متن غیرحساس (عنوان/خلاصه تولیدشده بدون بدنه) در آن می‌رود.

### ۳.۷ `Connection`
| فیلد | نوع | توضیح |
| --- | --- | --- |
| id | TEXT PK | |
| source_key | TEXT | |
| label | TEXT | نام کاربرپسند («حساب Gmail کاری») |
| auth_kind | TEXT | `OAUTH, API_KEY, IMAP_OAUTH, IMAP_APP_PASSWORD, ANDROID_PERMISSION, FILE_IMPORT, MANUAL, NONE` |
| account_hint_cipher | BLOB? | مثلاً ایمیل حساب (رمزنگاری‌شده) |
| access_state | TEXT | از `AccessState` |
| scopes | TEXT(json) | اسکوپ‌های واقعاً اعطاشده |
| enabled | INTEGER(bool) | |
| last_success_at / last_error_at | INTEGER? | |
| created_at | INTEGER | |
| consent_id | INTEGER FK | |

### ۳.۸ `ConsentRecord`
| فیلد | نوع | توضیح |
| --- | --- | --- |
| id | INTEGER PK | |
| connection_id | TEXT? FK | |
| source_key | TEXT | |
| consent_text_version | INTEGER | |
| scope | TEXT | چه داده‌ای |
| privacy_tier | TEXT | |
| granted | INTEGER(bool) | |
| granted_at / revoked_at | INTEGER? | |
| affirmative_action | TEXT | `CHECKBOX, SETTINGS_GRANT, FILE_PICK, SCAN_QR, OAUTH, TYPED_CONFIRM` |

### ۳.۹ `SourceCredentialsRef`
| فیلد | نوع | توضیح |
| --- | --- | --- |
| id | INTEGER PK | |
| connection_id | TEXT FK | |
| vault_key | TEXT | کلید در TokenVault (خارج از DB) |
| kind | TEXT | `OAUTH_REFRESH, OAUTH_ACCESS, API_KEY, APP_PASSWORD, PAT, COOKIE_REF(ممنوع), NONE` |
| expires_at | INTEGER? | |
| last_rotated_at | INTEGER? | |

> **هیچ رازی در این جدول نیست**؛ فقط ارجاع. جزئیات در `security-model.md`.

### ۳.۱۰ `ScanJob` / `ScanRun` / `ScanTask` / `Cursor`
```
ScanJob(id, name, trigger[SCHEDULED|MANUAL|PERIODIC|RESUME], source_keys json, options json, enabled, created_at)
ScanRun(id, job_id, started_at, finished_at?, state[PENDING|RUNNING|SUSPENDED|COMPLETED|FAILED|CANCELLED],
        executor[FOREGROUND_SERVICE|WORK_MANAGER|IMMEDIATE],
        tasks_total, tasks_done, tasks_failed, network_failures, last_suspended_reason?, cancelled_by_user bool)
ScanTask(id, run_id, source_key, connection_id, attempt, cursor_id, state[PENDING|RUNNING|DONE|FAILED|SKIPPED],
         started_at, finished_at, items_read, items_written, items_rejected, error_id?,
         priority, deadline_at?, expedited bool)
Cursor(id, source_key, connection_id, kind[PAGE_TOKEN|UID|HISTORY_ID|DELTA_LINK|SINCE_TS|FILE_OFFSET|ENTRY_PATH|OFFSET_LIMIT|NONE],
       payload_cipher BLOB, etag?, expires_at?, updated_at, checkpoint_run_id)
```
قواعد: نوشتن `Cursor` + `ScanTask.state=DONE` + `Evidence`ها در **یک تراکنش** (atomicity برای قانون ۹).

### ۳.۱۱ `CorrelationLink` / `IdentityCluster` / `MergeLog` / `DuplicateGroup`
```
CorrelationLink(id, left_type[SUBJECT|IDENTIFIER], left_id, right_type, right_id,
                score REAL, features json, status[AUTO_LINKED|SUGGESTED|CONFIRMED|REJECTED|UNLINKED],
                rule_version INT, created_at, updated_at, decided_by[SYSTEM|USER], decided_at?)
IdentityCluster(id, member_count, confidence, created_at, updated_at, split_from_id?)
MergeLog(id, cluster_id?, action[LINK|UNLINK|MERGE_SUBJECT|SPLIT|REBUILD], actor[SYSTEM|USER],
         before json, after json, reason, created_at)
DuplicateGroup(id, canonical_evidence_id, member_ids json, method[EXTERNAL_ID|CONTENT_HASH|MESSAGE_ID|FUZZY], created_at)
```
قاعده: `UNLINK` داده حذف نمی‌کند؛ فقط لینک را `UNLINKED` می‌کند و خوشه بازسازی می‌شود.

### ۳.۱۲ `ImportedFile`
| فیلد | نوع | توضیح |
| --- | --- | --- |
| id | INTEGER PK | |
| uri_string | TEXT | URI انتخابی کاربر (بدون محتوا) |
| display_name | TEXT | |
| sha256 | TEXT | برای تشخیص واردکردن مجدد |
| size_bytes | INTEGER | |
| archive_layout | TEXT(json) | فهرست ورودی‌ها و چندتکه بودن |
| parser_id / parser_version | TEXT/INT | |
| state | TEXT | `PENDING, SCANNING, PARSING, COMPLETED, PARTIAL, FAILED, CANCELLED` |
| entries_total / entries_done / entries_failed | INTEGER | |
| checkpoint | TEXT(json) | مسیر/آفست آخرین ورودی موفق |
| report | TEXT(json) | گزارش پارس (خطاها، انواع نادیده‌گرفته‌شده) |
| started_at / finished_at | INTEGER? | |

### ۳.۱۳ `CoverageRecord` / `ErrorRecord` / `QuotaLedger` / `AuditLog`
```
CoverageRecord(id, category, source_key, connection_id?, validation_status, evidence_count,
               subjects_count, last_run_at?, last_success_at?, reason_key?, reason_detail?, updated_at)
ErrorRecord(id, run_id?, task_id?, source_key, error_kind, retryable bool, http_status?, quota_unit_cost?,
            message_key, redacted_context json, occurred_at, resolved_at?, retry_count, user_visible bool)
QuotaLedger(id, source_key, window[DAILY|MINUTE|MONTHLY], period_key, units_used, units_limit?,
            updated_at)
AuditLog(id, at, actor[USER|SYSTEM|CONNECTOR], action, target_type, target_id, outcome, redacted_context json)
```
`AuditLog` برای «چه کسی/چه چیزی کی به چه داده‌ای دست زد» و برای اثبات حذف کامل است — بدون محتوا.

### ۳.۱۴ `Settings`
DataStore Proto (نه Room): ترجیحات UI، زبان، تم، بازاسکن دوره‌ای، سطح جزئیات خط زمانی، وضعیت opt‑inهای حساس.

---

## ۴. سیاست ذخیره‌سازی فیلدهای حساس

| داده | ذخیره می‌شود؟ | چگونه |
| --- | --- | --- |
| ایمیل/شماره/نام کاربری کاربر | بله | `value_cipher` (Tink) + `blind_index` (HMAC) |
| بدنه کامل ایمیل | اختیاری (پیش‌فرض: فقط metadata + snippet کوتاه) | در `RawBlobRef` رمزنگاری‌شده با `DELETE_AFTER_PARSE` اگر کاربر نخواهد |
| رمز عبور (از CSV مدیریت‌کننده رمز) | **هرگز** | فقط `url` + `username`؛ تست خودکار عدم وجود |
| مقدار کوکی | **هرگز** | فقط `domain` + `expiration` |
| secret دو عاملی (TOTP) | **هرگز** | فقط `issuer` + `accountName` |
| توکن OAuth / refresh | خارج از DB در TokenVault | Tink + Keystore |
| IP در سرصفحه ایمیل | پیش‌فرض: فقط کشور/شبکه تجمیعی | ذخیره کامل فقط با opt‑in |
| متن پیامک | بله (در صورت import) | رمزنگاری‌شده + `SENSITIVE` + حذف انتخابی |
| مکان دقیق | پیش‌فرض خاموش | در صورت فعال: تجمیع روزانه/شهر، نه نقطه دقیق |
| داده سلامت | پیش‌فرض خاموش | فقط `dataOrigin` + شمارش، نه مقادیر |
| داده اشخاص ثالث (مخاطبان دیگران) | **هرگز به‌صورت انبوه** | فقط identifierهای متعلق به owner با `third_party=false` |

---

## ۵. نسخه‌بندی و مهاجرت

1. `PRAGMA user_version` توسط Room مدیریت می‌شود؛ هر schema change یک `Migration` + تست `MigrationTestHelper` دارد.
2. `Evidence.payload_schema` و `Cursor.kind` نسخه‌دار هستند تا تغییر پارسر داده قدیمی را بی‌اعتبار نکند.
3. `ConnectorDescriptor.version` در هر Evidence ثبت می‌شود → می‌توان نتیجه را بر اساس نسخه connector بازتولید کرد.
4. قوانین استنتاج (`ruleId + ruleVersion`) ذخیره می‌شوند → اگر قاعده عوض شد، رویدادهای قدیمی قابل بازسازی‌اند بدون از دست رفتن شاهد.
5. مهاجرت مخرب (حذف ستون حاوی داده کاربر) فقط با تأیید صریح کاربر و ثبت در `AuditLog`.

## ۶. نگاشت به SQL (نمونه DDL خلاصه)

```sql
CREATE TABLE evidence (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  source_key TEXT NOT NULL,
  source_version INTEGER NOT NULL,
  connection_id TEXT,
  run_id TEXT,
  task_id TEXT,
  parser_id TEXT, parser_version INTEGER,
  kind TEXT NOT NULL,
  external_id TEXT,
  content_hash TEXT NOT NULL,
  payload_cipher BLOB NOT NULL,
  payload_schema TEXT NOT NULL,
  blob_ref_id INTEGER,
  occurred_at INTEGER, date_kind TEXT NOT NULL, date_precision TEXT NOT NULL,
  occurred_range_start INTEGER, occurred_range_end INTEGER,
  observed_at INTEGER,
  ingested_at INTEGER NOT NULL,
  derivation TEXT,
  confidence TEXT NOT NULL,
  validation_status TEXT NOT NULL,
  subject_id TEXT,
  identifier_ids TEXT NOT NULL,
  sensitive_fields_redacted INTEGER NOT NULL DEFAULT 0,
  dedup_group_id TEXT,
  deleted INTEGER NOT NULL DEFAULT 0,
  FOREIGN KEY(blob_ref_id) REFERENCES raw_blob_ref(id) ON DELETE SET NULL
);
CREATE UNIQUE INDEX idx_evidence_source_external
  ON evidence(source_key, external_id) WHERE external_id IS NOT NULL;
CREATE INDEX idx_evidence_hash ON evidence(content_hash);
CREATE INDEX idx_evidence_time ON evidence(occurred_at);
CREATE INDEX idx_evidence_subject ON evidence(subject_id, kind);
CREATE INDEX idx_evidence_run ON evidence(run_id);
```

## ۷. پرس‌وجوهای اصلی و بودجه عملکردی

| پرس‌وجو | ایندکس | بودجه |
| --- | --- | --- |
| خط زمانی با فیلتر (۱۰٬۰۰۰ رویداد) | `idx_event_time` + Paging 3 | < ۱۰۰ ms برای صفحه اول |
| «آیا ایمیل X را قبلاً دیده‌ام؟» | `blind_index` UNIQUE | < ۵ ms |
| dedup هنگام ingest (۱۰۰۰ رکورد/ثانیه) | `content_hash` | ≥ ۱۰۰۰ رکورد/ثانیه روی دستگاه میان‌رده |
| گزارش پوشش | `CoverageRecord` کوچک | < ۵۰ ms |
| جست‌وجوی متنی | FTS5 | < ۲۰۰ ms |

## ۸. چیزهایی که عمداً در مدل داده **نیستند**

- هیچ فیلد «تعداد کل حساب‌های اینترنتی کاربر» (ادعای غیرقابل اثبات).
- هیچ جدول «رمز عبور»، «کوکی»، «secret 2FA».
- هیچ فیلد «تاریخ ثبت‌نام» بدون `DateKind`.
- هیچ موجودیت مربوط به اشخاص ثالث به‌عنوان هدف اصلی.
- هیچ ستون «امتیاز اعتبار» که از سرویس‌های غیرمجاز leak خریده شده باشد.
