# منابع پژوهش — References

> مرحله ۱ — همه پیوندها در تاریخ **۲۰۲۶‑۱۰‑۱۴** بررسی شدند.
>
> **صداقت درباره روش پژوهش:** پژوهش با ابزار جست‌وجوی وب انجام شد. برخی نتایج از مستندات رسمی (primary) و برخی از مقالات فنی دست‌دوم (secondary) بودند. هر ادعایی که فقط از منبع دست‌دوم آمده با ⚠️ علامت خورده و **قبل از پیاده‌سازی باید با مستندات رسمی یا اجرای واقعی بسته شود** (فهرست V‑1 تا V‑12 در `docs/limitations.md`).
>
> این فایل فقط پیوند و ادعای مستند دارد؛ هیچ داده شخصی، توکن یا کلیدی در آن نیست.

---

## ۱. مستندات رسمی (primary) — مستقیم بررسی شدند

| موضوع | پیوند | ادعای پشتیبانی‌شده |
| --- | --- | --- |
| سهمیه Gmail API | https://developers.google.com/workspace/gmail/api/reference/quota | دو نوع سهمیه (per minute per project و per minute per user per project)؛ هزینه واحدی هر متد (`messages.list`=۵، `messages.get`=۵، `history.list`=۲، `messages.import/insert`=۲۵)؛ ۵۰۰ گیرنده در هر پیام |
| AccountManager و محدودیت visibility | https://developer.android.com/reference/android/accounts/AccountManager | از API 26 اپ‌ها فقط حساب‌هایی را می‌بینند که کاربر یا authenticator مجاز کرده یا امضای مشترک دارند؛ `GET_ACCOUNTS` کافی نیست؛ `VISIBILITY_*` |
| Microsoft Graph `signIn` | https://learn.microsoft.com/en-us/graph/api/resources/signin?view=graph-rest-1.0 | «برای دانلود sign‑in logs از طریق Graph باید لایسنس Entra ID P1 یا P2 داشته باشید» |
| HIBP API v3 | https://haveibeenpwned.com/api/v3 | `GET /breachedAccount/{email}` با هدر `hibp-api-key`؛ rate limit وابسته به اشتراک؛ `429` با `retry-after`؛ Pwned Passwords بدون rate limit و با k‑anonymity |
| HIBP subscription tiers | https://haveibeenpwned.com/API/Key | tierها با RPM مختلف (Pwned 1 … Pwned 5) و قیمت‌های ماهانه/سالانه |
| Play: مجوزهای حساس (SMS/Call Log) | https://support.google.com/googleplay/android-developer/answer/16558241 | فقط default handler برای SMS/Phone/Assistant؛ Declaration Form؛ ممنوعیت استفاده برای تبلیغات؛ توصیه به SMS Retriever API |
| Deprecated بودن Google Sign‑In | https://developers.google.com/android/reference/com/google/android/gms/auth/api/signin/package-summary | همه کلاس‌های `GoogleSignIn*` deprecated؛ جایگزین: Credential Manager (احراز هویت) و Google Identity Services (authorization) |
| AGP 9.1 release notes | https://developer.android.com/build/releases/agp-9-1-0-release-notes | AGP 9.1.1 (آوریل ۲۰۲۶)؛ پشتیبانی تا API 37؛ تغییر R8 (repackage به default package) |
| سازگاری AGP/Gradle | https://developer.android.com/build/releases/about-agp | حداقل Gradle لازم برای AGP 9.1 = 9.3.1 |
| Room release notes | https://developer.android.com/jetpack/androidx/releases/room | Room 2.8.5 پایدار (سپتامبر ۲۰۲۶)؛ minSdk 23؛ `room-sqlite-wrapper` |
| Room 3.0 | https://android-developers.googleblog.com/2026/03/room-30-modernizing-room.html | حذف SupportSQLite، coroutine‑first، Room 2.x در حالت maintenance تا پایدارشدن Room 3 |
| مهاجرت Room 2→3 | https://developer.android.com/training/data-storage/room/migration-2-to-3 | مسیر آمادگی: driver APIs و `InvalidationTracker.createFlow` |
| Compose runtime | https://developer.android.com/jetpack/androidx/releases/compose-runtime | پایدار 1.12.1 (اکتبر ۲۰۲۶)؛ 1.13.0‑beta01 |
| نسخه پایدار Kotlin | https://kotlinlang.org/docs/multiplatform/multiplatform-compatibility-guide.html | «نسخه پایدار فعلی Kotlin = 2.4.21» (صفحه به‌روزرسانی‌شده اکتبر ۲۰۲۶) |
| Discord — درخواست داده | https://support.discord.com/hc/en-us/articles/360004027692-Requesting-a-Copy-of-your-Data | مسیر User Settings → Data & Privacy → Request your data؛ انتخاب نوع داده در دسکتاپ؛ ارسال به ایمیل حساب؛ نیاز به تأیید ایمیل |
| Microsoft — آرشیو تاریخچه فعالیت | https://support.microsoft.com/en-US/accounts-billing/manage/open-your-activity-history-archive | privacy dashboard → Download your data → ZIP حاوی JSON برای هر نوع داده |
| Health Connect — مجوزها | https://support.google.com/android/answer/12201230 | کنترل per‑app و per‑data‑type؛ revoke؛ ذخیره روی دستگاه |
| BIND_NOTIFICATION_LISTENER_SERVICE | https://developer.android.com/reference/android/Manifest.permission#BIND_NOTIFICATION_LISTENER_SERVICE | مجوز خاص سیستمی برای NotificationListenerService |
| GDPR متن رسمی Art. 20 | https://gdpr-info.eu/art-20-gdpr/ | متن رسمی حق قابل‌حمل بودن داده (پیوند مرجع؛ در پژوهش از طریق منابع تحلیلی زیر تأیید شد) |

## ۲. پیوندهای رسمی مرجع که باید در فاز اجرا مستقیم بازبینی شوند

| موضوع | پیوند مرجع |
| --- | --- |
| Google Takeout | https://takeout.google.com/settings/takeout |
| راهنمای صادرات داده گوگل | https://support.google.com/accounts/answer/3024190 |
| Google OAuth scopes (Gmail) | https://developers.google.com/identity/protocols/oauth2/scopes |
| Google Identity Services / Credential Manager | https://developer.android.com/identity/sign-in/credential-manager |
| Microsoft identity platform + MSAL Android | https://learn.microsoft.com/en-us/entra/identity-platform/msal-authentication-flows |
| Graph permissions reference | https://learn.microsoft.com/en-us/graph/permissions-reference |
| Play Console — target API level | https://support.google.com/googleplay/android-developer/answer/11926878 |
| Play Console — Package/App Visibility permission | https://play.google.com/intl/en-US/about/privacy-security-deception/permissions/ |
| Package visibility در اندروید | https://developer.android.com/training/package-visibility |
| Storage Access Framework | https://developer.android.com/training/data-storage/shared/documents-files |
| WorkManager | https://developer.android.com/develop/background-work/workmanager |
| Foreground service types (Android 14+) | https://developer.android.com/develop/background-work/services/fgs/service-types |
| UsageStatsManager | https://developer.android.com/reference/android/app/usage/UsageStatsManager |
| Android Keystore | https://developer.android.com/privacy-and-security/security-key-crypto |
| Google Tink | https://github.com/google/tink |
| SQLCipher for Android | https://www.zetetic.net/sqlcipher/ |
| GitHub REST — rate limits | https://docs.github.com/en/rest/using-the-rest-api/rate-limits-for-the-rest-api |
| GitHub REST — `/user`, `/user/emails`, `/user/installations`, search commits | https://docs.github.com/en/rest |
| Steam Web API | https://partner.steamgames.com/doc/webapi و https://steamcommunity.com/dev/apikey |
| YouTube Data API v3 quota | https://developers.google.com/youtube/v3/getting-started#quota |
| Stack Exchange API | https://api.stackexchange.com/docs |
| Last.fm API | https://www.last.fm/api |
| Spotify Web API | https://developer.spotify.com/documentation/web-api |
| Strava API | https://developers.strava.com/docs/reference/ |
| Fitbit API | https://dev.fitbit.com/ |
| Discord API — current user & connections | https://discord.com/developers/docs/resources/user |
| Twitch Helix | https://dev.twitch.tv/docs/api/reference/ |
| Mastodon API | https://docs.joinmastodon.org/api/ |
| Reddit Data API | https://support.reddithelp.com/hc/en-us/articles/26410290525844-Public-API-Terms-of-Use و https://www.reddit.com/dev/api/ |
| X API v2 | https://developer.x.com/en/docs/x-api و https://docs.x.com/x-api/fundamentals/rate-limits |
| RDAP | https://www.rdap.org/ |
| Wayback CDX API | https://github.com/internetarchive/wayback/tree/master/wayback-cdx-server |
| keys.openpgp.org API | https://keys.openpgp.org/about/api |
| Meta — Download your information | https://www.facebook.com/help/1701730696756992 |
| Apple — Data and Privacy | https://privacy.apple.com/ |
| Microsoft — Privacy dashboard | https://account.microsoft.com/privacy |
| Amazon — Privacy Central / Request your data | https://www.amazon.com/gp/help/customer/display.html?nodeId=GQ7KJ27YDJWWDNJS |
| LinkedIn — Get a copy of your data | https://www.linkedin.com/psettings/get-data-start |
| Google Password Manager (export) | https://passwords.google.com/ |
| SMS Retriever API | https://developers.android.com/identity/sms-retriever |

## ۳. منابع دست‌دوم (secondary) — با ادعای مشخص و ⚠️ نیازمند راستی‌آزمایی

| ادعا | منبع |
| --- | --- |
| ⚠️ تغییر سهمیه Gmail از ۱ مه ۲۰۲۶: ۶٬۰۰۰ واحد/دقیقه/کاربر (قبلاً ۱۵٬۰۰۰)، ۸۰M/روز، سقف همزمانی منتشرنشده، watch ≤ ۱ رویداد/ثانیه | https://www.usecarly.com/blog/gmail-api-limits/ · https://www.unipile.com/gmail-api-limits/ |
| ⚠️ طبقه‌بندی اسکوپ‌های گوگل (non‑sensitive/sensitive/restricted) و نیاز به ارزیابی امنیتی سالانه برای اسکوپ Restricted وقتی داده به سرور می‌رود | https://developer.nylas.com/docs/cookbook/email/messages/list-messages-google/ · https://www.usecarly.com/blog/gmail-api-limits/ |
| ⚠️ deprecated بودن کامل Jetpack Security Crypto از ۱٫۱٫۰ و توصیه DataStore + Tink + Keystore | https://blog.includesecurity.com/2026/08/encryptedsharedpreferences-is-dead-heres-what-you-should-use-instead/ · https://blog.kinto-technologies.com/posts/2025-06-16-encrypted-shared-preferences-migration/ · https://medium.com/@n20/encryptedsharedpreferences-is-deprecated-what-should-android-developers-use-now-7476140e8347 |
| ⚠️ سیاست Play درباره `QUERY_ALL_PACKAGES`: Declaration Form، ویدیو، توجیه عملکرد اصلی، و موارد مجاز (device search، antivirus، file manager، browser) | https://orangeoma.zendesk.com/hc/en-us/articles/360021412159 · https://ptkd.com/journal/android-package-visibility-query-all-packages · https://www.xda-developers.com/google-is-restricting-which-apps-can-see-the-other-installed-apps-on-a-device/ |
| ⚠️ دسترسی به `signInActivity`/sign‑in logs نیازمند لایسنس P1/P2 و نقش Reports Reader + `AuditLog.Read.All` | https://practical365.com/entra-id-sign-ins/ · https://learn.microsoft.com/en-us/answers/questions/163553 · https://learn.microsoft.com/en-us/answers/questions/428350 |
| ⚠️ از API 26 `getAccountsByType("com.google")` بدون انتخاب کاربر خالی برمی‌گردد | https://stackoverflow.com/questions/35050548 · https://stackoverflow.com/questions/46496069 |
| ⚠️ محتوای Takeout: Gmail=MBOX، Contacts=vCard/CSV، Calendar=ICS، فعالیت‌ها=JSON، Photos/Drive=فایل اصلی + JSON sidecar، `archive_browser.html` | https://takeoutreader.com/how-does-google-takeout-work · https://www.forensicsware.com/blog/open-files-downloaded-from-google-takeout/ · https://hackmag.com/security/google-takeout |
| ⚠️ Chrome sync پیش‌شرط حضور تاریخچه در Takeout؛ مسیرهای `Chrome/BrowserHistory.json` و `My Activity/Chrome/MyActivity.json` | https://takeoutreader.com/google-takeout-chrome-history · https://webapps.stackexchange.com/questions/137335 · https://github.com/positron48/youtube-history-analyzer |
| ⚠️ Takeout شامل Bookmarks/Autofill/History/Extensions/Settings کروم است | https://learn.microsoft.com/en-us/answers/questions/2363022 (گزارش کاربر از آرشیو واقعی) |
| ⚠️ X API از ۲۰۲۶ pay‑per‑use شد (بدون tier رایگان برای ثبت‌نام جدید؛ ~$0.005/read، ~$0.01/user lookup) | https://www.outstand.so/blog/x-api-pricing · https://postproxy.dev/blog/x-api-pricing-2026/ · https://elfsight.com/blog/how-to-get-x-twitter-api-key-in-2026/ |
| ⚠️ Reddit: 100 QPM رایگان غیرتجاری، ~$0.24/۱۰۰۰ فراخوانی تجاری، و بسته‌شدن ثبت خودکار client از اواخر ۲۰۲۵ | https://www.socialcrawl.dev/blog/reddit-data-api-2026 · https://snitchfeed.com/blog/reddit-api-pricing |
| ⚠️ Telegram export فقط در Desktop؛ `ChatExport_*/result.json`؛ Secret Chat صادر نمی‌شود | https://www.mosaicchats.com/blog/how-to-export-telegram-chat · https://pcg-telegram.com/blogs/957582268/ |
| ⚠️ Steam: `timecreated` در GetPlayerSummaries و محدودیت پروفایل خصوصی در GetOwnedGames | https://rdrr.io/github/josegallegos07/steamR/man/get_player_summaries.html · https://pkg.go.dev/github.com/xemkayx/steam-api/pkg/steamclient |
| ⚠️ الزام target API 36 از ۲۰۲۶‑۰۸‑۳۱ برای اپ‌ها و به‌روزرسانی‌های Play (تلفن/تبلت/foldable/Auto)؛ استثناها Wear 35 و TV/XR 34 | https://primetestlab.com/blog/google-play-target-api-level-2026 · https://www.testerscommunity.com/blog/update-your-target-api-level-by-august-31-2026 · https://ecorpit.com/android-target-api-36-play-store-deadline-migration-2026/ |
| ⚠️ GitHub: ۵٬۰۰۰ درخواست/ساعت authenticated و ۶۰ بدون احراز هویت؛ `/user/emails` با اسکوپ `user:email` | https://github.com/orgs/community/discussions/63553 · https://docs2.lfe.io/v3/users/emails/ |
| ⚠️ مسیر و قالب export رمزهای Chrome (CSV متن‌ساده از Google Password Manager) | https://www.chromethemer.com/export-passwords-from-chrome-securely/ · https://passlock.to/blog/how-to-export-passwords-from-chrome/ · https://www.dashlane.com/blog/how-to-export-google-chrome-passwords-to-csv |
| ⚠️ Meta Accounts Center → Download your information (JSON کامل‌تر از HTML) و تکه‌شدن فایل‌ها (`followers_1.json`, `followers_2.json`) | https://memoracy.com/posts/download-facebook-instagram-data-archive · https://insta-checkmate.com/en/blog/instagram-data-download |
| ⚠️ LinkedIn: `Connections.csv` با ستون‌های First/Last Name, URL, Email (مشروط), Company, Position, Connected On؛ آرشیو کامل ۲۴–۷۲ ساعت | https://getboothiq.com/blog/how-to-export-linkedin-contacts · https://connectsafely.ai/articles/export-linkedin-contacts-connections-guide-2026 · https://socialguardian.app/blog/linkedin-data-export/ |
| ⚠️ Amazon: حذف «Download order reports» در مارس ۲۰۲۳ و جایگزینی با «Request your data» (ZIP شامل `Retail.OrderHistory.1.csv`) | https://amazaspend.com/blog/how-to-download-your-amazon-order-history · https://tiller.com/how-to-download-your-amazon-order-history-report/ · https://www.orderproanalytics.com/blog/how-to-download-your-amazon-order-history-report |
| ⚠️ SAF: `ACTION_OPEN_DOCUMENT(_TREE)` + `takePersistableUriPermission`؛ از اندروید ۱۱ picker ریشه داخلی/`Download` را نمی‌دهد؛ محدودیت‌های `MANAGE_EXTERNAL_STORAGE` | https://capawesome.io/blog/android-scoped-storage-in-capacitor-apps/ · https://medium.com/microsoft-mobile-engineering/scoped-storage-in-android-10-android-11-28d58d989f3c |
| ⚠️ `PACKAGE_USAGE_STATS` یک app‑op است و با `ACTION_USAGE_ACCESS_SETTINGS` + `AppOpsManager.unsafeCheckOpNoThrow("android:get_usage_stats", …)` بررسی می‌شود | https://proandroiddev.com/accessing-app-usage-history-in-android-79c3af861ccf · https://stackoverflow.com/questions/26431795 |
| ⚠️ محتوای بسته داده Discord (account/, messages/, servers/, activity/) و فقط پیام‌های خود کاربر | https://memoryclone.com/guide-request-discord-data-package.php · https://takeoutday.org/guides/how-to-export-discord-data |
| ⚠️ Apple privacy.apple.com → Request a copy of your data؛ تا ~۷ روز؛ ممکن است تأیید هویت اضافی بخواهد | https://www.idownloadblog.com/2024-03-21/get-personal-data-from-apple/ · https://discussions.apple.com/thread/256035247 |
| ⚠️ Health Connect: اندروید ۹+ (APK)، از اندروید ۱۴ داخل سیستم؛ مجوز per data type؛ `dataOrigin` در metadata رکوردها | https://openwearables.io/blog/google-health-connect-integration-android-health-data-for-developers · https://www.kodeco.com/35028713-health-connect-android-api |
| ⚠️ حذف Basic Auth در Exchange Online (IMAP/POP/SMTP) و کار نکردن App Password پس از آن؛ XOAUTH2 به‌عنوان مسیر جایگزین | https://www.getmailbird.com/microsoft-modern-authentication-enforcement-email-guide/ · https://www.unipile.com/imap-server-connection-guide/ · https://github.com/mhoenes/sortroom/issues/45 |
| ⚠️ قالب رشته `AUTHENTICATE XOAUTH2` و TTL ۳۶۰۰ ثانیه‌ای توکن | https://www.unipile.com/imap-server-connection-guide/ |
| ⚠️ YouTube Data API: ۱۰٬۰۰۰ واحد/روز/پروژه به‌صورت پیش‌فرض و افزایش با فرم audit | https://www.getphyllo.com/post/youtube-api-limits-how-to-calculate-api-usage-cost-and-fix-exceeded-api-quota · https://www.blotato.com/blog/youtube-api-pricing |
| ⚠️ GDPR Art. 20 در مقابل Art. 15: قالب ساختاریافته/رایج/ماشین‌خوان، فقط داده «ارائه‌شده توسط کاربر»، فقط مبنای رضایت/قرارداد، مهلت یک ماه | https://www.legiscope.com/blog/data-portability-right.html · https://gdprlocal.com/right-to-data-portability/ · https://luxgap.com/lois/rgpd/art-20/ |
| ⚠️ WhatsApp export به‌صورت `.txt` در اندروید و سقف ~۴۰٬۰۰۰ پیام بدون رسانه | https://www.mosaicchats.com/blog/how-to-export-whatsapp-chat · https://www.ravsio.com/blog/how-to-export-whatsapp-chats-without-losing-media |
| ⚠️ NotificationListenerService نیازمند فعال‌سازی دو مرحله‌ای در Settings و حساسیت بالای آن | https://dl.acm.org/doi/10.1145/3728898 · https://stackoverflow.com/questions/42451538 |

---

## ۴. ادعاهایی که در پژوهش **رد** شدند

| ادعای رایج | چرا رد شد |
| --- | --- |
| «با یک API می‌شود همه حساب‌های اینترنتی کاربر را پیدا کرد» | هیچ چنین API وجود ندارد؛ بیشتر سرویس‌ها API شخصی ندارند (L2‑6…L2‑9) |
| «می‌شود تاریخچه مرورگر/رمزهای مرورگر اندروید را خواند» | scoped storage + sandbox (L1‑1) |
| «می‌شود با `GET_ACCOUNTS` فهرست حساب‌های دستگاه را گرفت» | از API 26 ممکن نیست (L1‑6) |
| «می‌شود با App Password به IMAP مایکروسافت وصل شد» | Basic Auth در حال حذف کامل است (L5‑11) |
| «می‌شود تاریخچه تماشا در یوتیوب را از API گرفت» | فقط Takeout (L4‑8) |
| «`QUERY_ALL_PACKAGES` با یک مجوز ساده گرفته می‌شود» | نیازمند Declaration Form و توجیه عملکرد اصلی؛ رد شدن رایج است (L2‑3) |
| «sign‑in logs مایکروسافت برای هر حسابی قابل خواندن است» | نیاز به tenant + لایسنس P1/P2 (L2‑8) |
| «می‌شود با شمارش نام کاربری، حساب‌ها را در صدها سایت کشف کرد» | خلاف ToS و پرریسک قانونی → آگاهانه رد شد (L6‑1) |
