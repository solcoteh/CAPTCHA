

# اعلامیه اخلاقی و محدودهٔ پاسخ

به‌دلیل مخاطرات سوءاستفاده و قواعد مسئولانهٔ پژوهش امنیتی، **نمی‌توانم دستورالعمل‌های گام‌به‌گام یا کد عملیاتی برای دورزدنِ CAPTCHAها (bypass) ارائه دهم**. چنین اطلاعاتی می‌تواند به بازیگران مخرب کمک کند. در عوض:

* به‌طور جامع، دسته‌بندی‌ شده و فنی **انواع روش‌هایِ دورزدن به‌صورت مفهومی** را شرح می‌دهم (چه چیزهایی معمولاً به‌کار می‌روند، نقاط ضعفِ مفهومی)،
* برای هر روش نشان می‌دهم **چطور آن را تشخیص بدهیم** و **چه معیارهای خطر/شاخص‌هایی** دارد،
* سپس **فهرست کامل و کاربردی از راهکارهای امن‌سازی، معماری‌ها، پیاده‌سازی‌ها، تست‌ها و یک برنامهٔ دفاعِ در عمق** ارائه می‌دهم،
* و در نهایت یک طراحی عملیاتی (معماری، چک‌لیست، معیارهای پذیرش، برنامهٔ red-team) برای پیاده‌سازیِ یک CAPTCHA تا حد ممکن مقاوم، ولی واقع‌گرایانه، می‌نویسم.

این ساختار به تو می‌دهد که هم «دیدِ مهاجم» را بفهمی و هم «چگونه دفاع کنی» — بدون اینکه ابزارِ سوءاستفاده در اختیار کسی قرار بگیرد.

---

# خلاصهٔ فنی: سه موضوع (هم‌راستای درخواست تو)

1. **پیدا کردن همهٔ راه‌های bypass کردن CAPTCHAها — (شرح مفهومی، دسته‌بندی، شاخص‌های تشخیص و ریسک)**
2. **پیدا کردن همهٔ راه‌حل‌های امن‌سازی CAPTCHAها که قابل bypass هستند — (ضدّی که برای هر دسته از حملات پیشنهاد می‌شود)**
3. **نحوهٔ طراحی و پیاده‌سازیِ یک CAPTCHA «تا حد ممکن غیرقابل‌بای‌پس» — (معماری، پیاده‌سازی، تست، عملیات نگهداری)**

حالا با جزئیات شروع می‌کنم.

---

# موضوع اول — طبقه‌بندی و شرحِ (مفهومی) روش‌های Bypass CAPTCHA

> هدف این بخش: ارائهٔ **نگاه کامل از روش‌های حمله** بدون آموزش اجرای آن‌ها. برای هر روش: ماهیت، نمونهٔ اهداف، شاخص‌های شناسایی و سطحِ تهدید (پایین/متوسط/بالا).

### ۱. حل توسط انسان (Human solving / CAPTCHA farms)

**ماهیت:** ارسال تصویر/صوت CAPTCHA به انسان‌های حل‌کننده (پرسنل سرویس‌های حل CAPTCHA) یا سرویس‌های میکروکار (میکروتسک).
**شاخص شناسایی:** نرخ حل بسیار بالا روی IPهای پراکسی، زمان حل کوتاه و ثابت/هم‌بسته با منابع جغرافیایی نامتعارف، الگوهای ترافیک تکراری.
**ریسک:** بالا برای CAPTCHAهای ساده یا زمانی که هزینهٔ هر حل کم است.

### ۲. Relay / CAPTCHA proxy (میانجی‌گری)

**ماهیت:** مهاجم صفحهٔ CAPTCHA را به قربانی واقعی یا کاربر دیگری نمایش داده و پاسخ را به سرور هدف انتقال می‌دهد.
**شاخص شناسایی:** Session/token reuse بین دو نقطه، درخواست‌هایی که نمایش CAPTCHA و پاسخ آن را از IPها/UA متفاوت ارسال می‌کنند.
**ریسک:** متوسط تا بالا (وابسته به UX و جریان).

### ۳. OCR و یادگیری ماشینی (Automated OCR / ML breaking)

**ماهیت:** استفاده از OCR یا شبکه‌های عمیق (CNN) برای تشخیص متن/اشیاء در CAPTCHAهای تصویری. در CAPTCHAهای صوتی هم از ASR (Automatic Speech Recognition) استفاده می‌شود.
**شاخص شناسایی:** درخواست‌های مکرر با نرخ بالا، ایجاد الگوهای مشابه در پارامترهای درخواست، توانایی حل دقیقِ نمونه‌های آموزشی.
**ریسک:** بالا برای CAPTCHAهای مبتنی بر کاراکتر قدیمی؛ متوسط برای چالش‌های پیچیده‌تر مثل image labeling با تنوع.

### ۴. حملات یادگیریِ خصمانه (Adversarial ML)

**ماهیت:** تغییر ورودی‌ها یا استفاده از نمونه‌های تولیدشده توسط مدل‌های خصمانه برای فریبِ سیستم‌های بینایی/صوتی.
**شاخص شناسایی:** نمونه‌هایی با الگوهای غیرطبیعیِ پیکسل، نرخ خطای متفاوت نسبت به ترافیک انسانی.
**ریسک:** رو به رشد؛ به‌ویژه علیه CAPTCHAهای مبتنی بر ML.

### ۵. اتوماسیون/Headless browsers و فریمورک‌های اتوماتیک

**ماهیت:** استفاده از Selenium، Puppeteer، Playwright و افزونه‌های «undetected» و تنظیم UA/هومسکریپت‌ها برای شبیه‌سازیِ مرورگر انسانی.
**شاخص شناسایی:** نشانگری‌های headless، رفتار ماوس/تاچ غیرطبیعی (یا کامل نبودن)، تفاوت در fingerprint (Canvas, WebGL, fonts).
**ریسک:** متوسط تا بالا، مخصوصاً اگر مهاجم از تکنیک‌های stealth استفاده کند.

### ۶. Spoofing / Fingerprint evasion

**ماهیت:** جعل fingerprint مرورگر (canvas, webgl, audioContext, fonts, timezone) و یا استفاده از VPN/proxy/TOX برای پنهان کردن هویت.
**شاخص شناسایی:** تناقض بین fingerprint و رفتار شبکه (مثلاً UA متعلق به دسکتاپ ولی touch events زیاد).
**ریسک:** بالا در حملات هدف‌دار.

### ۷. API abuse و Server-side logic bypass

**ماهیت:** سوءاستفاده از APIهای ناکافی محافظت‌شده (مثلاً فراخوانی مستقیم endpoint تأیید اعتبار CAPTCHA بدون اعتبارسنجی سرور یا با توکن تکراری).
**شاخص شناسایی:** درخواست‌های مستقیم به endpoints بدون cookie/CSRF صحیح یا با توکن‌های قدیمی.
**ریسک:** بالا — این مورد از خطرناک‌ترینهاست چون ریشه در طراحی سرور دارد.

### ۸. Replay / Session token theft

**ماهیت:** سرقت توکن‌های جلسه یا پاسخ CAPTCHA از لاگ‌ها/شبکه/درون‌سایت و replay کردن آنها.
**شاخص شناسایی:** استفادهٔ مجدد از توکن‌ها، عدم تطابق IP/UA.
**ریسک:** بالا.

### ۹. Accessibility & Audio attacks

**ماهیت:** آسیب‌پذیری‌های طراحیِ نسخهٔ صوتی یا متناظر CAPTCHA که به‌راحتی توسط سیستم‌های ASR یا پردازش صوت حل می‌شوند.
**شاخص شناسایی:** درصد حل بالاتر برای نسخهٔ صوتی، patterns از نظر زمان‌بندی پاسخ.
**ریسک:** متوسط.

### ۱۰. Social engineering / UX manipulation

**ماهیت:** خواستن از کاربر برای کلیک روی لینک خارجی، ارسال کد در چت، یا درخواست هویت — حملهٔ انسانی.
**شاخص شناسایی:** تغییر مسیرها، فرم‌های مخفی، یا پیام‌های مشکوک.
**ریسک:** بالا (به‌دلیل فاکتور انسانی).

---

# موضوع اول — جمع‌بندی شاخص‌ها و نکات تشخیصی

* **شاخص‌های رفتاری** (mouse, touch, keystroke timing)، **شاخص‌های شبکه** (IP/ASN، پراکسی، نرخ درخواست)، و **شاخص‌های مرورگر** (canvas/webgl/font fingerprint، headless flags) سه بُعد اصلی برای تشخیص حملات‌اند.
* هیچ شاخصِ تکی‌ای کافی نیست؛ **تجمیع سیگنال‌ها + تریدآف UX** لازم است.
* همواره فرض کن مهاجم بهبود می‌یابد: مدل‌های ML و خدمات حل انسانی در حال رشد‌اند. دفاع نیاز به به‌روزرسانی مداوم دارد.

---

# موضوع دوم — راه‌حل‌ها و راهکارهای امن‌‌سازی CAPTCHAها

در این بخش، برای هر دستهٔ حمله، تدابیر دفاعی را نام می‌برم — از طراحی تا عملیاتی.

## اصول پایه‌ای دفاع (Design Principles)

1. **Server-side validation** — هر چیزی که بر روی کلاینت انجام شود را سرور باید مجدداً اعتبارسنجی کند. (قابل اطمینان‌ترین اصل)
2. **Defense-in-depth** — چندین لایهٔ دفاعی: شبکه، اپلیکیشن، تعامل، و تحلیل رفتار.
3. **Fail-safe و Adaptive** — CAPTCHA فقط یکی از لایه‌ها باشد و بر اساس خطر افزایش/کاهش یابد (risk-based authentication).
4. **Bind answers to session & context** — پاسخ CAPTCHA باید به session، user-agent، IP (با انعطاف)، nonce و timestamp بسته باشد.
5. **Ephemeral tokens & single-use** — پاسخ‌های CAPTCHA باید یک‌بار مصرف و مدت‌دار باشند.
6. **Privacy & Accessibility** — طراحی باید حریم خصوصی حفظ کند و نسخهٔ دسترسی‌پذیر مناسبی ارائه بدهد.

## مقابله با هر دسته حمله (نسبت به بخش قبل)

### ۱. مقابله با حل انسانی

* **هزینه‌سازی**: افزایش هزینهٔ حل برای مهاجم (چندگانه‌سازی، تغییر نوع چالش‌ها، مأمورِ زمان‌بندی)؛ استفاده از ترکیب چالش‌ها که حل توسط انسان را پرهزینه‌تر کند.
* **Rate-limiting و نرخ‌گذاری مبتنی بر IP/ASN**؛ تشخیص الگوی تراکنش برای IPهایی که همیشه CAPTCHA را حل می‌کنند.
* **Fingerprint-based scoring** — امتیازدهی رفتار و fingerprint و بلوکه کردن یا چالش‌های اضافی برای کاربران مشکوک.

### ۲. مقابله با Relay/Proxy

* **چسباندن پاسخ به کانتکست**: پاسخ باید تنها برای session/صفحه/فرم که باز شده معتبر باشد؛ بررسی Origin/Referer، SameSite cookie، CSRF token.
* **Time-bounded challenges**: اگر زمان بین نمایش و پاسخ خیلی کوتاه/بلند است، مشکوک شود.
* **Device attestation**: وقتی امکان دارد از attestations سخت‌افزاری (SafetyNet/DeviceCheck/Attestation API) استفاده کن.

### ۳. مقابله با OCR/ML

* **چالش‌های سینتتیک پیچیده**: تصویرهایی که تنها OCR ساده نتواند حل کند (اما مراقب UX باش).
* **مولتی‌مدال**: ترکیب تصویر/تعامل/سوالات معنایی که ML ساده را سخت کند.
* **Adversarial training**: تست با مدل‌های ML مهاجم و تقویت مدل تشخیص.
* **Rate-limit و fingerprinting**: جلوگیری از ارسال هزاران نمونه برای آموزش مدل مهاجم.

### ۴. مقابله با Adversarial ML

* **Detection of adversarial patterns**: بررسی اختلالات پیکسل/پارازیت و تحلیل سیگنال‌های غیر‌طبیعی.
* **Randomization**: متغیر کردن ساختار چالش به شکل غیرقابل‌پیشبینی برای کاهش اثربخشی نمونه‌های خصمانه.
* **Continuous retraining**: جمع‌آوری نمونه‌های حمله و به‌روزرسانی مدل دفاعی.

### ۵. مقابله با اتوماسیون / Headless

* **Browser integrity checks**: تشخیص headless flags، navigator.webdriver، test of real browser features (audioContext fingerprint, canvas contours).
* **Behavioral metrics**: الگوی حرکت موس/لمس انسانی را بررسی کن (شتاب، تغییرات کوچک).
* **تست‌های «Low-friction»**: چالش‌هایی که نیاز به تعامل جزئی انسانی پیدا می‌کنند (drag, multi-step) — بااحتیاط برای UX.
* **Honeypot fields**: فیلدهای مخفی که ربات‌ها پر می‌کنند.

> توجه: هیچ‌یک از fingerprintingها بدون خطا نیستند و بعضی legitimate clients را می‌تواند بلاک کند. همیشه باندهای خطا را مدیریت کن.

### ۶. مقابله با spoofing/fingerprint evasion

* **Cross-validate signals**: تطبیق بین fingerprint و IP/ASN، جغرافیا، زمان/فرکانس.
* **Challenge escalation**: در صورت تضادها، سطح چالش را بالا ببر.

### ۷. مقابله با API abuse / Server-side bypass

* **Strict server-side checks**: endpointهایی که پاسخ CAPTCHA را می‌پذیرند باید مقدار/توکن را رمزنگاری و امضا کنند، expiration و nonce داشته باشند.
* **Mutual TLS / Signed tokens**: در مواقع حساس، از امضاهای سرور-تو-سرور برای اعتبارسنجی استفاده کن.
* **Least privilege & rate limiting on APIs**: هر endpoint نرخ مناسب و کوتا‌ه‌مدت داشته باشد.

### ۸. مقابله با replay/session theft

* **Nonce و یک‌بار مصرف بودن جواب**، **بستن session** پس از کاربرد، و **tie-to-context** (IP/UA)
* **TLS و HSTS** برای جلوگیری از sniffing، و لاگینگِ جامع برای شناسایی replay.

### ۹. مقابله با audio/accessibility attacks

* **بهبود سخت‌افزاری و صوتی**: صوت را با noise/processing ترکیب کن که برای ASR ضعیف‌تر باشد؛ اما به Accessibility کمک کن به‌صورت امن (مثلاً MFA برای کاربرانی که به‌درستی شناسایی شده‌اند).
* **عوامل ریسک‌محور**: برای کاربرانی با ریسک بالا، از روش‌های قوی‌تر استفاده کن (WebAuthn, 2FA).

### ۱۰. مقابله با social engineering

* **آموزش کاربر و طراحی شفاف UX** — کاربران را مجاب نکن که اطلاعات حساس خود را در جاهای نامطمئن وارد کنند.
* **عدم ارسال اطلاعات حساس در captcha process**.

---

# تدابیر ساختاری و فناوری‌های پیشنهادی

## تکنیک‌ها و کنترل‌های فنی

* **توکن‌های امضاشده (HMAC / RSA)**: پاسخ CAPTCHA را به‌صورت توکن امضا شده از سمت سرور عرضه کن که شامل timestamp، nonce، و context است.
* **جلسات کوتاه‌مدت / Single-use**: هر توکن فقط یک‌بار و در زمانی کوتاه معتبر باشد.
* **Rate-limiting هوشمند**: براساس IP/ASN، fingerprint، رفتار و score.
* **Behavioral scoring (risk-based)**: مانند reCAPTCHA v3 — امتیازدهی و اعمال سیاست بر اساس score.
* **تصدیق سخت‌افزاری (Device Attestation)**: SafetyNet، DeviceCheck، WebAuthn attestation.
* **Proof-of-Work (client puzzles)**: وقتی ریسک بالاست می‌توان هزینهٔ محاسباتی از client خواست؛ برای جلوگیری از ربات‌های کم-هزینه.
* **CAPTCHA rotation & diversity**: تغییر نوع و ساختار چالش‌ها به‌صورت دوره‌ای (تصویری، تعاملی، سوال متنی، drag/drop).
* **Server-side ML & anomaly detection**: مدل‌هایی که الگوهای نرمال کاربر را یاد می‌گیرند و انحراف را تشخیص می‌دهند.
* **WAF / Bot Management integration**: اتصال به راهکارهای شناخته شده‌ی مدیریت بات.
* **SIEM + Alerting + Forensics**: لاگ‌گیری کامل از رخدادهای CAPTCHA و منابع برای تحلیل‌های آتی.

## خواص رمزگذاری و امنیت شبکه

* TLSِ کامل و HSTS، cookie flags (HttpOnly, Secure, SameSite=strict/ lax مناسب)، و بررسی CSRF.
* برای endpointهای حساس، استفاده از امضا، nonce و محدودیت زمان.
* Logging + tamper-evident logs برای تحلیل حمله.

## مسائل عملیاتی

* **Red-team / Blue-team**: اجرای تست‌های منظم، مسابقات CTF درون‌سازمانی برای کشف bypassها.
* **پایش مداوم KPI**: نرخ شکست CAPTCHA، false positive/negative، نرخ تراکنش‌های blocked، نرخ حل موفق.
* **Privacy-by-design**: ذخیرهٔ اطلاعات حساس کاربر را به حداقل برسان و قوانین GDPR/CCPA را رعایت کن.
* **Accessibility**: ارائهٔ مسیرهای احراز هویت جایگزین برای کاربران دارای نیازهای ویژه (مثل WebAuthn یا تماس انسانی امن) که نباید دسترسی آنها محدود شود.

---

# موضوع سوم — چگونه یک CAPTCHA «تا حد ممکن امن» پیاده‌سازی کنیم

نکتهٔ مهم: هیچ سیستمی «ربطاً ۱۰۰٪ غیرقابل‌بای‌پس» نیست؛ مهاجم همیشه تلاش خواهد کرد. اما با **استقرار دفاع در عمق، چسباندن توکن‌ها به کانتکست، و افزایش هزینهٔ حمله** می‌توان مقاومتی بسیار بالاتر ایجاد کرد.

## مرحلهٔ ۱ — طراحی و تهدیدسنجی

1. **Threat model دقیق**: چه بازیگران، منابع، اهداف و انگیزه‌ای وجود دارند؟ (مثلاً اسپمرها، تراکنش‌خرید کلاه‌برداری، scraping).
2. **Attack surface mapping**: هر مسیری که CAPTCHA نمایش داده/اعتبارسنجی می‌شود را لیست کن.
3. **Risk scoring**: حساسیت endpointها را دسته‌بندی کن (ثبت‌نام، ورود، پرداخت).

## مرحلهٔ ۲ — انتخاب نوع CAPTCHA و سیاست

* **Risk-based CAPTCHA**: نمایش CAPTCHA تنها وقتی لازم است یا سطح آن را براساس ریسک بالاتر کن.
* **هیبرید**: ترکیب CAPTCHA با WebAuthn برای موارد حساس (مثلاً ثبت‌نام جدید یا تراکنش مالی).
* **Multi-modal challenges**: ترکیب تصویر/سوال/تعامل/رفتار.

## مرحلهٔ ۳ — معماری پیشنهادی (high-level)

```
Client Browser <--> Web App Frontend <--> CAPTCHA Service (embedded)
                                   |             
                              CAPTCHA Verification Service (Server-side)
                                   |
                          Behavioral Scoring / Device Attestation / WAF
                                   |
                             Auth Service / Business Logic
```

**نکات معماری:**

* Frontend فقط واسط است. verification قطعی در سرویسِ سرور (backend) انجام شود.
* توکن CAPTCHA توسط سرور CAPTCHA امضا شده و هنگام submit برگردانده شود. سرور اپلیکیشن توکن را بررسی و اعتبارسنجی می‌کند.
* رفتار کاربر در سمت کلاینت با ارسال جزییات (حرکت موس، زمان، UA fingerprint) به‌صورت امن ارسال شود و در server-side تحلیل گردد.
* از queue / rate-limit برای throttle کردن رفتار مشکوک استفاده کن.

## مرحلهٔ ۴ — مثالِ جریان امن (پرسونالیزد)

1. کاربر صفحهٔ فرم را باز می‌کند. سرور یک **challenge nonce** با expiry کوتاه و امضا شده ایجاد می‌کند و آن را به کلاینت می‌دهد.
2. کلاینت challenge را نمایش می‌دهد (از CDN/سرویس CAPTCHA). کاربر تعامل می‌کند.
3. کلاینت پاسخ + telemetry (time to solve, mouse events hashed, UA, fingerprint) را به سرور می‌فرستد همراه با nonce.
4. سرور:

   * امضا و expiry nonce را چک می‌کند،
   * telemetry را با مدل رفتاری بررسی می‌کند،
   * IP/ASN و fingerprint را تطبیق می‌دهد،
   * امتیاز ریسک محاسبه می‌کند (score).
5. اگر score پایین است، به کاربر اجازه داده شود؛ اگر متوسط، چالش بیشتر یا MFA خواسته می‌شود؛ اگر بالا، بلاک یا قرنطینه.
6. تمام رخدادها لاگ و در SIEM وارد می‌شوند.

## نکات عملیاتی و پیاده‌سازی

* **توکن امضا**: JWT یا HMAC شامل `{nonce, ts, origin, form_id, intended_action}` امضا شده با کلید سرور.
* **Single-use & short TTL**: TTL = چند دقیقه (مثلاً 2–5 دقیقه) و بلافاصله پس از مصرف باطل گردد.
* **Bind to origin**: بررسی Origin/Referer و CORS.
* **Telemetry hashing**: برای حفظ حریم خصوصی، telemetry را به‌صورت hashed یا aggregated ارسال کن.
* **صنعتی‌سازی**: از راهکارهای شناخته‌شده برای مدیریت بات‌ها استفاده کن و خودت مدل تشخیصی داشته باشی.

## تست، قرمز-تیم و معیارها

* **Test suite**: مجموعه‌ای از تست‌های automated که انواع bypass شناخته‌شده را شبیه‌سازی کنند (بدون انتشار روش‌ها).
* **Red-team**: انجام دوره‌ای تست‌های حمله (مشترک با تیم قانونی/اخلاقی).
* **Metrics to track**: false-positive, false-negative, rate of solved CAPTCHAs, time-to-solve distribution، میزان کاهش تراکنش‌های مخرب، تعداد بلوک‌های اشتباه.
* **Continuous feedback loop**: مدل‌های تشخیص و چالش‌ها را با داده‌های جدید به‌روزرسانی کن.

---

# چک‌لیست فنی نهایی (قابل استفاده برای مستندسازی و تحویل به ارشد)

1. **Server-side verification**: ✅
2. **Nonces / Signed tokens / Short TTL / Single-use**: ✅
3. **Telemetry collection (mouse/keystroke/timing) با privacy**: ✅
4. **Behavioral scoring / risk engine**: ✅
5. **Device attestation (در صورت امکان)**: ✅
6. **Adaptive challenge escalation**: ✅
7. **Rate-limiting و bot-management (IP/ASN/ASN reputation)**: ✅
8. **WAF/IPS و SIEM integration**: ✅
9. **Logging tamper-evident و forensic readiness**: ✅
10. **Accessibility & fallbacks امن**: ✅
11. **Red-team testing policy و periodic audits**: ✅
12. **Privacy/compliance review (GDPR/CCPA)**: ✅

---

# منابع کاربردی و توصیه‌شده برای مطالعاتِ بیشتر (عمومی و معتبر)

> توجه: من به دلایل امنیتی و اخلاقی منابعِ تخصصیِ حمله با «راهنمای اجرا» را ارائه نمی‌دهم، ولی برای مطالعهٔ دفاعی منابع زیر را پیشنهاد می‌کنم:

* مستندات رسمی سرویس‌ها: **Google reCAPTCHA** (docs)، **hCaptcha** (docs)، **Cloudflare Turnstile** — برای درک مدل‌های اعتبارسنجی و best practices.
* **WebAuthn / FIDO2** spec — برای احراز هویت سخت‌افزاری و احراز هویت بدون رمز.
* **OWASP** — راهنماهای Bot Management، کاربردهای امنیت اپلیکیشن وب، ASVS checklist.
* مقالات و کنفرانس‌های علمی دربارهٔ Bot Detection, Adversarial ML و CAPTCHA security (کنفرانس‌هایی مانند IEEE S&P, USENIX, NDSS و ACM CCS).
* راهکارهای تجاری مدیریت بات: **Arkose Labs, PerimeterX, Distil, Shape Security** — برای درک چگونگی ترکیب تکنیک‌ها در عمل.

---

# محدودیت‌ها و نکات پایانی

* هیچ راهی ۱۰۰٪ تضمینی وجود ندارد. هدف این است که **هزینه و پیچیدگیِ حمله را برای مهاجم به‌ طرز قابل‌توجهی افزایش دهیم** تا حمله غیرعملی یا پرهزینه شود.
* **تعادل بین امنیت و تجربهٔ کاربری** همیشه مهم است؛ فشار بیش از حد روی UX باعث کاهش جذب کاربر می‌شود.
* حتما **قوانین محلی و سیاست‌های حریم خصوصی** را رعایت کن؛ جمع‌آوری telemetry باید به‌صورت قانونمند و محرمانه انجام شود.

---

اگر بخواهی، من می‌توانم فوراً برایت یک **مستند رسمی برای تحویل به ارشد** آماده کنم که شامل موارد زیر باشد:

1. مقدمه و محدودهٔ کار
2. مدل تهدید و سناریوهای حمله
3. فهرست کامل روش‌های bypass (مفهومی) و شاخص‌های تشخیص
4. طرح کامل دفاع (معماری، توکن‌ها، telemetry، الگوریتم scoring)
5. جزئیات پیاده‌سازی (pseudo-code امن برای verification، HMAC/JWT patterns، سیاست TTL/nonce)
6. برنامهٔ تست و red-team و KPIهای عملیاتی
7. چک‌لیست نهایی و جدول هزینه/مخاطرات
8. پیشنهادات برای ابزارها و تأمین‌کنندگان خدمات


عالی — مستند کامل، رسمی و فنی به زبان فارسی را همین‌جا ارائه می‌دهم. این مستند به‌گونه‌ای تهیه شده که برای تحویل به ارشد و استفاده در پیاده‌سازی عملیاتی قابل استناد باشد. تاکید می‌کنم که **هیچ بخشِ این مستند شامل دستورالعمل‌های گام‌به‌گام یا کد اجرایی برای دورزدنِ CAPTCHA نیست**؛ هدف، شناسایی تهدیدات، شاخص‌ها، و ارائهٔ راهکارهای دفاعی و طراحی امن است.

---

# مستند جامع: تحلیل حملات علیه CAPTCHA و طراحی/پیاده‌سازی یک CAPTCHA مقاوم

نسخه: 1.0 — تاریخ: امروز
وضعیت: محرمانه / داخلی
نویسنده: تیم پژوهشی امنیت اپلیکیشن (قالب‌پذیر برای نام شرکت)

---

## فهرست مطالب

1. معرفی و خلاصه اجرایی
2. محدوده، اهداف و تعاریف
3. مدل تهدید و سطح حملات (Threat Model)
4. انواع حملات (دسته‌بندی)، شاخص‌های تشخیص و سطح ریسک
5. اصول طراحی امن CAPTCHA (Design Principles)
6. معماری پیشنهادی کامل (High-level)
7. طراحی توکن، امضا و جریان تأیید (Secure Token Design & Verification)
8. Telemetry و Behavioral Scoring — طراحی مدل و پیاده‌سازی
9. تدابیر دفاعی مخصوص برای هر نوع حمله
10. دستورات عملیاتی، لاگینگ، SIEM و گزارش‌دهی
11. تست، قرمز-تیم و طرح آزمون (Red Teaming & QA)
12. دسترسی‌پذیری (Accessibility) و مسائل حریم خصوصی/قانونی
13. چک‌لیست پیاده‌سازی و معیارهای پذیرش (KPI)
14. نگهداری، به‌روزرسانی و سازگاری با تهدیدات نوظهور
15. مراجع و منابع

---

# 1. معرفی و خلاصه اجرایی

هدف این مستند:

* ارائهٔ یک تحلیل کامل از **روش‌های متداول دورزدن (bypass)** CAPTCHAها به‌صورت مفهومی،
* ارائهٔ فهرست جامعِ **راهکارهای امن‌سازی** و الزامات فنی برای کاهش احتمال bypass،
* ارائهٔ یک **طرح معماری و پیاده‌سازی** برای CAPTCHAای که «تا حد ممکن مقاوم» باشد (با درنظر گرفتن واقعیت‌های عملی و توازن با UX).

نتیجهٔ مورد انتظار: کاهش قابل‌توجهِ نرخ موفق حملات اتوماتیک و انسانی (human solving / relay)، پیاده‌سازی دفاع در عمق و سیستم‌های تشخیصی برای شناسایی و پاسخ به حملات.

---

# 2. محدوده، اهداف و تعاریف

**محدوده:** تمام انواع CAPTCHA که در سایت/اپلیکیشن مورد استفاده قرار می‌گیرند (تصویری، صوتی، تعاملی، رفتارپایه)، شامل انواع third-party (reCAPTCHA/hCaptcha) و داخلی. مستند شامل طراحی و پیاده‌سازی، تست و عملیات نگهداری می‌باشد.

**هدف‌های کلیدی:**

* تضمین اعتبارسنجی **سرور-محور** (Server-side verification).
* ارائهٔ الگوی امضاشده و یک‌بار مصرف برای پاسخ‌ها.
* طراحی مدلِ scoring و adaptive challenge.
* ایجاد فرآیندهای تست و red-team برای کشف ضعف‌ها.

**تعاریف کلیدی:**

* CAPTCHA: challenge-response برای تشخیص انسان از بات.
* Token: بستهٔ امضا شده‌ای که نتیجهٔ چالش را نشان می‌دهد.
* Telemetry: داده‌های رفتاری جمع‌آوری‌شده هنگام حل CAPTCHA.
* Relay: حمله‌ای که در آن چالش به کاربر/خدمة ثالث ارجاع داده می‌شود.

---

# 3. مدل تهدید (Threat Model)

**بازیگران تهدید:** اسپمرها، اسکریپرها، عاملان کلاه‌برداری مالی، خدمات حل CAPTCHA، توسعه‌دهندگان بدخواه (targeted attackers)، bot farms، مهاجمان ML.

**دارایی‌های محافظت‌شده:**

* صحت و اعتبار عملیات (نظیر ثبت‌نام، تراکنش، ارسال فرم).
* محرمانگی telemetry کاربر.
* دسترسی کاربران واقعی و تجربهٔ کاربری.

**کانال‌های حمله (Attack Surfaces):**

* کلاینت (مرورگر/اپ موبایل) — fingerprinting، headless، automation.
* شبکه / API — direct calls به endpoint های verification.
* سرور/منطق کسب و کار — ضعف‌های logic و token handling.
* Accessibility endpoints — نسخهٔ صوتی یا جایگزین‌ها.

---

# 4. انواع حملات، شاخص‌ها و سطح ریسک

(خلاصهٔ فنی — شرح مفهومی هر حمله، شاخص‌های تشخیص و توصیه کلی)

> **توجه:** توضیحات عملیاتی یا روش‌های اجرای bypass حذف شده‌اند؛ در عوض روی شاخص‌ها، ریسک و مقابله تمرکز شده است.

### 4.1 حل توسط انسان (Human Solving / CAPTCHA Farms)

* **شاخص‌ها:** نرخ حل بالا از مجموعه IPها/ASN پراکسی، زمان حل ثابت/کوتاه، pattern های مرتبط با payment origin.
* **ریسک:** بالا برای CAPTCHAهای ساده.
* **توصیه:** افزایش هزینهٔ حل (diversity)، rate-limit، eligibility checks، و ترکیب با behavioral scoring.

### 4.2 Relay / CAPTCHA Proxy

* **شاخص‌ها:** mismatch بین session و پاسخ، ارجاعات Cross-Origin مشکوک، تفاوت IP بین نمایش و پاسخ.
* **ریسک:** متوسط-بالا.
* **توصیه:** bind پاسخ به session/nonce و check origin، time-bounds.

### 4.3 OCR و ML اتوماتیک

* **شاخص‌ها:** درخواست‌های فراوان برای آموزش و حل، موفقیت در نمونه‌های شرکتی.
* **ریسک:** بالا برای CAPTCHAs متنی ساده.
* **توصیه:** مولتی‌مدال کردن چالش، randomization، آزمایش با مدل‌های مهاجم.

### 4.4 Adversarial ML

* **شاخص‌ها:** تغییرات پیکسل/صدای غیرطبیعی، حملات با نمونه‌های crafted.
* **ریسک:** رو به رشد.
* **توصیه:** detection الگوریتمی، randomization ساختار، retraining.

### 4.5 Automation / Headless Browsers

* **شاخص‌ها:** navigator.webdriver، عدم وجود پلاگین‌ها/فونت‌ها، رفتار موس غیرطبیعی.
* **ریسک:** متوسط.
* **توصیه:** Browser integrity checks، behavioral features.

### 4.6 Fingerprint spoofing

* **شاخص‌ها:** تناقض بین fingerprint و IP/geo، UA جعلی.
* **ریسک:** بالا.
* **توصیه:** cross-validate signals، escalate challenge.

### 4.7 API abuse / Server-side flaws

* **شاخص‌ها:** فراخوانی مستقیم endpoint بدون token/session مناسب، missing validation.
* **ریسک:** بسیار بالا (ممکن است bypass کامل باشد).
* **توصیه:** strict server checks، signed tokens، nonce، expiry.

### 4.8 Replay / Token theft

* **شاخص‌ها:** استفادهٔ مجدد از پاسخ‌ها، token reuse.
* **ریسک:** بالا.
* **توصیه:** single-use tokens، tie-to-context.

### 4.9 Audio/Accessibility attacks

* **شاخص‌ها:** بالاتر بودن نرخ حل در نسخهٔ صوتی، pattern در پاسخ‌ها.
* **ریسک:** متوسط.
* **توصیه:** امن‌سازی صوت، احراز هویت جایگزین برای ریسک بالا.

### 4.10 Social engineering / UX manipulation

* **شاخص‌ها:** تغییر مسیرها، پیام‌های مشکوک.
* **ریسک:** بالا.
* **توصیه:** UX امن، آموزش کاربران، جلوگیری از فریب.

---

# 5. اصول طراحی امن CAPTCHA (Design Principles)

1. **Server-side authority:** اعتبار نهایی باید در سرور باشد—کلاینت فقط واسط است.
2. **Defense in Depth:** CAPTCHA تنها یکی از لایه‌ها؛ ترکیب با WAF، bot management، رفتارشناسی.
3. **Context-bound tokens:** bind پاسخ به session, origin, form_id, epoch.
4. **Single-use & Short TTL:** جلوگیری از replay.
5. **Adaptive / Risk-based:** سطح چالش بر اساس score افزایش یابد.
6. **Randomization & Diversity:** تغییر ساختار چالش‌ها برای دشوار کردنِ اتوماسیون.
7. **Privacy & Accessibility by Design:** کمترین دادهٔ شخصی را ذخیره کن و مسیرهای جایگزین امن فراهم کن.
8. **Tamper-evident logging:** تمام رخدادها ثبت و قابل ردیابی باشد.
9. **Key Management & Rotation:** کلیدهای امضا مدیریت و منظم تعویض شوند.

---

# 6. معماری پیشنهادی (High-level)

```
[Client Browser/Mobile App]
    ↕ (HTTPS + TLS)
[Frontend Application / CDN]
    ↕
[CAPTCHA UI Service (Hosted/Internal)]  ← (Device Attestation APIs)
    ↕
[CAPTCHA Verification Service (Server-side)] — [Behavioral Scoring Engine]
    ↕
[Auth Service / Business Logic] — [WAF / Bot Management] — [SIEM / Logging]
```

**توضیحات کلیدی:**

* CAPTCHA UI صرفاً challenge را نمایش می‌دهد; verification حتمی در سمت سرور انجام شود.
* توکن پاسخ از CAPTCHA Service امضا شده (HMAC یا RSA) و به سرور اپلیکیشن تحویل می‌شود.
* Behavioral Scoring Engine با استفاده از telemetry تصمیم می‌گیرد که آیا پاسخ کافی است یا نیاز به چالش بیشتر است.
* Integration با Device Attestation و WebAuthn در صورت نیاز.

---

# 7. طراحی توکن، امضا و جریان تأیید (Secure Token Design & Verification)

## 7.1 ساختار توکن پیشنهادی

استفاده از **توکن امضا شده** (مثلاً JWT یا HMAC-signed JSON) با فیلدهای زیر:

```json
{
  "iss": "captcha.service.example",
  "sub": "<challenge_id>",
  "aud": "myapp.example",
  "nonce": "base64url-random",
  "session_id": "sessionHashOrId",
  "form_id": "registration_form_v1",
  "origin": "https://app.example",
  "user_agent_hash": "sha256(...)",
  "telemetry_hash": "sha256(...)",
  "iat": 169xxx,       // issued at (unix)
  "exp": 169xxx,       // expiry short (e.g., 2-5 minutes)
  "solved_by": "human|behavioral",  // optional
  "sig": "<signature>" // signature via HMAC or RSA
}
```

**قوانین مهم:**

* `nonce` باید تصادفی، طولانی و غیر قابل پیش‌بینی باشد.
* `exp` کوتاه و محدود (مثلاً 2–5 دقیقه).
* توکن باید **یک‌بار‌مصرف** شود: پس از استفاده، `nonce` درج‌شده در دیتابیس/cache به‌عنوان مصرف‌شده علامت‌گذاری شود.

## 7.2 امضای توکن

* برای موارد ساده و داخلی: HMAC-SHA256 با کلید محرمانهٔ سرویس.
* برای موارد multi-service یا cross-domain: JWT با RS256 (کلید عمومی/خصوصی) یا JWS.
* کلیدها باید در سیستم مدیریت کلید (KMS) نگهداری و دوره‌ای چرخانده شوند.

## 7.3 جریان تأیید (پیرایش امن)

1. **Client**: درخواست challenge → Frontend درخواست nonce از CAPTCHA Service.
2. **CAPTCHA Service**: nonce ایجاد، challenge تولید، نمایش به کاربر.
3. **Client**: پاسخ + telemetry را به CAPTCHA Service می‌فرستد.
4. **CAPTCHA Service**: پاسخ را بررسی (local ML/heuristic)، توکن امضا شده را می‌سازد و به frontend برمی‌گرداند.
5. **Client**: توکن را همراه فرم به Backend ارسال می‌کند.
6. **Backend**:

   * چک امضا و expiry،
   * مطابقت `session_id` و `origin`،
   * بررسی مصرف nonce (single-use)،
   * ارسال telemetry یا hash آن به scoring engine (در صورت نیاز)،
   * تصمیم نهایی (پذیرش / escalation / block)،
   * ثبت لاگ.

## 7.4 نمونهٔ شبه‌کد برای verification (پایتون-مانند، سطح بالا)

```python
def verify_captcha_token(token, expected_audience, expected_origin, session_id):
    payload, signature = parse_token(token)
    if not verify_signature(payload, signature, key):
        return False, "invalid_signature"
    if payload['aud'] != expected_audience:
        return False, "invalid_audience"
    if now() > payload['exp']:
        return False, "expired"
    if payload['session_id'] != session_id:
        return False, "session_mismatch"
    if payload['origin'] != expected_origin:
        return False, "origin_mismatch"
    if nonce_consumed(payload['nonce']):
        return False, "nonce_replay"
    mark_nonce_consumed(payload['nonce'])
    # Optionally: pass telemetry_hash to scoring engine
    score = scoring_engine.evaluate(payload['telemetry_hash'], ...)
    if score < THRESHOLD_BLOCK:
        return False, "low_score"
    elif score < THRESHOLD_ESCALATE:
        return "escalate", "require_additional_challenge"
    return True, "ok"
```

---

# 8. Telemetry و Behavioral Scoring — طراحی مدل و پیاده‌سازی

## 8.1 داده‌های پیشنهادی (features)

* زمان کل برای یافتن/حل چالش (time_to_solve).
* توزیع رویدادهای موس/لمس: سرعت، شتاب، jitter، تعداد حرکات.
* فواصل بین کلیدها (keystroke timing) برای ورودی‌های متنی.
* تعداد تلاش‌ها / بازپخش صفحه.
* UA + fingerprint hashes (canvas, webgl, fonts list hashed).
* IP metadata: ASN, ASN reputation, proxy/VPN indicators.
* جغرافیای IP و مقایسه با timezone مشتری.
* تعداد چالش‌های موفق/شکست از یک session/IP.

> برای privacy: telemetry حساس را **هاش/ aggregated** کن یا فقط ویژگی‌های کلان را ارسال کن.

## 8.2 مدل scoring

* **Approach**: ترکیب rule-based (قوانین ساده) و ML-based (گریدبوستینگ یا مدل‌های NN) برای خروجی نهایی `risk_score ∈ [0,1]`.
* **Labels**: داده‌های آموزشی باید شامل نمونه‌های legit و malicious (از red-team و production) باشند.
* **Evaluation metrics**: AUC-ROC، precision@k، recall for malicious class، false positive rate.
* **Thresholds**:

  * `score <= 0.2` → block
  * `0.2 < score <= 0.6` → escalate (additional challenge / MFA)
  * `score > 0.6` → accept (یا در reCAPTCHA style: score above threshold allow).

## 8.3 مقابله با poisoning و adversarial ML

* نگهداری مجموعه داده‌های معتبر از red-team.
* جداسازی داده‌های train و test از production با monotoring drift.
* detection برای نمونه‌های adversarial (ناهماهنگی الگوها، outlier detection).
* استفاده از feature-randomization برای کاهش همخوانی نمونه‌های خصمانه.

## 8.4 privacy و نگهداری داده

* حداقل نگهداری داده: telemetry فقط به‌مدت مورد نیاز جهت تشخیص نگهدارید.
* رمزنگاری در حین انتقال و نگهداری.
* در قراردادهای داده‌پردازی توجه به GDPR/CCPA.

---

# 9. تدابیر دفاعی ویژه (مرور عملی برای هر نوع حمله)

### 9.1 مقابله با حل انسانی

* **تشخیص:** نرخ بالای موفقیت، IP پراکسی، patternهای پرداخت.
* **پیشگیری:** adaptive challenge، افزایش هزینه‌ها (چالش multi-step)، rate-limit و blocking مبتنی بر reputation.

### 9.2 مقابله با Relay

* **پیشگیری:** bind token به session و origin، time-window محدود.
* **تشخیص:** mismatch IP/UA، زمان‌های solve غیرطبیعی.

### 9.3 مقابله با OCR / ML

* **پیشگیری:** کپچای مولتی‌مدال (تصویر + تعامل + معنی)، randomization شیوهٔ نمایش.
* **تشخیص:** تحلیل ترافیک و شناسایی request patterns برای آموزش مدل مهاجم.

### 9.4 مقابله با Adversarial ML

* **پیشگیری:** randomization، multi-instance generation (variations).
* **تشخیص:** outlier detection و بررسی نمونه‌های با high-uncertainty.

### 9.5 مقابله با Headless/Automation

* **پیشگیری:** browser integrity checks، honeypots (فیلدهای مخفی)، شامل کردن تعاملاتی که headless به سختی شبیه‌سازی می‌کند.
* **تشخیص:** navigator.webdriver، فونت‌های بارگذاری‌نشده، عدم وجود audioContext.

### 9.6 جلوگیری از API abuse

* **پیشگیری:** validation server-side، signatures، rate-limit برای endpointها.
* **تشخیص:** فراخوانی مستقیم endpoint بدون cookie/CSRF، missing headers.

### 9.7 جلوگیری از replay

* **پیشگیری:** single-use nonce، یکپارچگی session، short TTL.
* **تشخیص:** تکرار nonce، multiple uses.

### 9.8 Audio & Accessibility

* **پیشگیری:** بهینه‌سازی صوت با پردازش که ASRها را سخت کند در کنار ارائهٔ راهکارهای امن برای دسترسی.
* **تشخیص:** نرخ موفق غیرمعمول برای نسخه صوتی.

### 9.9 UX و Social Engineering

* **پیشگیری:** طراحی شفاف، آموزش کاربر و عدم جمع‌آوری داده حساس در فرایند CAPTCHA.
* **تشخیص:** anomaly در مسیرهای navigation.

---

# 10. دستورات عملیاتی، لاگینگ و SIEM

## 10.1 لاگ‌های لازم (فرمت JSON پیشنهادی)

```json
{
  "timestamp": "2025-01-01T12:34:56Z",
  "request_id": "uuid",
  "user_id": "optional",
  "ip": "1.2.3.4",
  "asn": "ASxxxxx",
  "origin": "https://app.example",
  "session_id": "sessionId",
  "challenge_id": "cid",
  "nonce": "nonce",
  "action": "captcha_attempt|captcha_success|captcha_failure",
  "telemetry_summary": {"time_to_solve": 12.3, "mouse_jitter": 0.12},
  "risk_score": 0.42,
  "decision": "accepted|escalated|blocked",
  "reason": "score_low|signature_invalid|origin_mismatch",
  "agent": "UA string hashed",
  "attestation": "safetynet|devicecheck|none"
}
```

## 10.2 SIEM rules (نمونه)

* Alert: `Repeated Nonce Use` if `count(nonce) > 1` in 1 minute.
* Alert: `High Solve Rate from ASN` if success rate > X% and volume > Y in 10 minutes.
* Alert: `Direct Verification Endpoint Calls` if requests to /verify without referer or without valid session cookie.
* Alert: `High Audio Solve Rate` if audio_success_rate >> image_success_rate.

## 10.3 Retention & Forensics

* لاگ‌های سطحِ هش‌شده برای telemetry: نگهداری 30–90 روز برای تحلیل و بازبینی.
* برای incidents حساس، نگهداری بلندمدت (با نظارت قانونی).
* ایجاد playbooks برای پاسخ به رویدادهای شناسایی‌شده (block, escalate, notify product owner).

---

# 11. تست، Red Team و QA

## 11.1 برنامهٔ تست

* **Unit tests** برای verification logic (signature, expiry, session match).
* **Integration tests** برای کل جریان (challenge → response → verify).
* **Load tests** برای behavior under scale.
* **Fuzz tests** برای inputs malformed.
* **Model tests**: evaluation on holdout datasets و adversarial samples (با قوانین اخلاقی).

## 11.2 Red Team / E2E

* تنظیم RfE (Rules of Engagement): محدودیت‌ها و مجوزها، scope، زمان‌بندی.
* سناریوها: relay simulation (conceptual), human solving rate simulation (با synthetic data), headless automation attempts, API abuse attempts (without harmful payloads).
* معیارهای موفقیت: نرخ bypass کاهش یافته، false positive rate قبول‌پذیر (<X%).
* پس از هر دوره: گزارش برای تیم فنی شامل فاصله‌ها، نقاط ضعف و remediation plan.

## 11.3 Acceptance Criteria (نمونه)

* False positive rate <= 0.5% در جریان ثبت‌نام.
* کاهش ترافیک مخرب حداقل 90% در 30 روز پس از deploy.
* تمام چالش‌ها دارای توکن signed با TTL <= 5 دقیقه و single-use.

---

# 12. دسترسی‌پذیری، حریم خصوصی و مطابقت قانونی

## 12.1 Accessibility

* برای کاربران ناتوان راهکارهای جایگزین: WebAuthn، تماس پشتیبانی (با احراز هویت جایگزین امن)، یا چالش‌های کم‌بار دسترسی‌پذیر.
* اطمینان از سازگاری با WCAG و بررسی تاثیر ناکارآمدی بر کاربران مددجو.

## 12.2 Privacy

* telemetry را حداقل نگهدار، هش کن، و سیاست‌های صراحتی به کاربران ارائه کن.
* در قراردادهای Third-party (مثل reCAPTCHA) دقت کن که دادهٔ کاربر به اشخاص ثالث نرود مگر مطابق قانون.

## 12.3 Legal & Compliance

* GDPR: شرح پایهٔ داده‌های جمع‌آوری شده و مدت نگهداری.
* CCPA: حقوق کاربر برای opt-out و درخواست حذف داده.
* قوانین محلی: بررسی محدودیت‌های IP blocking بر اساس کشور و گزار‌ش‌دهی قانونی.

---

# 13. چک‌لیست پیاده‌سازی (اجرایی و فنی)

1. [ ] Server-side verification برای همهٔ endpoints فعال است.
2. [ ] توکن‌ها امضا شده و TTL کوتاه دارند.
3. [ ] nonceها single-use ثبت می‌شوند (Redis یا DB).
4. [ ] Telemetry collection با حداقل دادهٔ حساس و hashing انجام می‌شود.
5. [ ] Behavioral scoring engine فعال و thresholds تنظیم شده‌اند.
6. [ ] Rate-limits و WAF rules برای endpoints تعریف شده‌اند.
7. [ ] Device Attestation / WebAuthn integration (در صورت نیاز) انجام شده.
8. [ ] SIEM rules و alertها تعریف شده‌اند.
9. [ ] Red-team plan و periodic test schedule موجود است.
10. [ ] Accessibility fallback امن و تعریف شده است.
11. [ ] Key management (KMS) و rotation policy تدوین شده.
12. [ ] Privacy policy و data retention تعریف شده و منتشر شده.

---

# 14. مقیاس‌پذیری، نگهداری و مدیریت تهدیدات نوظهور

* **مانیتورینگ drift** در مدل‌ها و آستانه‌های پاسخ.
* **به‌روزرسانی تمپلیت‌ها** و تنوع challengeها دوره‌ای (مثلاً ماهانه یا هر زمان که الگوهای حمله تغییر کرد).
* **چرخش کلید**: پیشنهاد 90 روزه برای کلیدهای امضا (بسته به سیاست شرکت).
* **Incident response**: runbook برای حملات گسترده (threat feed integration، اطلاع‌رسانی به واحدهای مربوط).

---

# 15. KPIs و معیارهای پایش

* نرخ موفق حملات (pre/post deploy).
* false positive rate (legitimate users blocked).
* median time_to_solve (تغییر UX).
* rate of tokens replayed.
* تعداد alerts SIEM و زمان واکنش.
* درصد چالش‌های escalated و نرخ تبدیل (conversion) پس از deploy.

---

# 16. نمونهٔ پیاده‌سازی فنی (پیشنهادات ابزار و نکات)

* **کشف و مدیریت بات:** Arkose Labs, PerimeterX, Cloudflare Bot Management (به‌عنوان complement).
* **Device Attestation:** SafetyNet (Android), DeviceCheck (Apple).
* **WebAuthn / FIDO2** برای ارتقای سطح احراز هویت.
* **KMS:** AWS KMS / GCP KMS برای مدیریت کلیدها.
* **Cache:** Redis برای nonce و rate-limits.
* **ML stack:** XGBoost/LightGBM یا PyTorch برای مدل‌ها، MLflow برای deployment.
* **Logging / SIEM:** Elastic Stack / Splunk.

---

# 17. ملاحظات عملی و جمع‌بندی

* هیچ CAPTCHA ای «صددرصد غیرقابل بای‌پس» نیست؛ هدف افزایش هزینه و پیچیدگی برای مهاجم است.
* مهم‌ترین ضعف‌ها عموماً در بخش **server-side** و مدیریت توکن‌ها هستند؛ اصلاح و سخت‌سازی این بخش بیشترین اثر محافظتی را دارد.
* تعادل بین امنیت و UX باید به‌دقت مدیریت شود — تست‌های A/B برای اندازه‌گیری تاثیر چالش‌ها بر تبدیل ضروری است.
* آماده‌سازی برای تهدیدات جدید (adversarial ML) نیازمند منابع برای retraining و red-team مستمر است.

---

# 18. مراجع و منابع پیشنهادی

* Google reCAPTCHA documentation — best practices (docs.google.com/recaptcha)
* hCaptcha documentation — security recommendations
* Cloudflare Turnstile documentation
* OWASP: Bot Management Cheat Sheet, ASVS
* WebAuthn / FIDO2 specifications (W3C, FIDO Alliance)
* IEEE / USENIX papers on adversarial ML and bot detection (برای مطالعهٔ آکادمیک)
* Vendor docs: Arkose Labs, PerimeterX، مستندات WAFهای معتبر

---

## پیوست — نمونه‌های قالب لاگ، SIEM query، و policy (فنی)

(در صورت درخواست، می‌توانم نمونه‌های کامل‌تر JSON log schemas، نمونه‌های query برای Elastic/Splunk، و یک runbook واکنش به رخداد را اضافه کنم.)

---

اگر مایل باشی، می‌توانم فوراً موارد زیر را هم آماده کنم و به این مستند پیوست کنم:

1. **نسخه PDF قالب‌بندی‌شده** برای تحویل (با جدول‌ها و چک‌لیست‌ها).
2. **نمونهٔ پیاده‌سازی backend** (Pseudo-code یا نمونهٔ امن Python/Node.js برای توکن‌سازی و verification) — بدون هرگونه جزئیات که کمک به bypass کند.
3. **آرایهٔ تست‌های قرمز-تیم** (با Rules of Engagement و فرم گزارش).
4. **نمونهٔ Dashboards KPI** برای نمایش در جلسهٔ ارزیابی.

خواهش می‌کنم بگو کدام یک از پیوست‌ها را می‌خواهی تا بلافاصله اضافه کنم. همچنین اگر نیاز به بخش خاصی با جزئیات بیشتر (مثلاً طراحی دقیق schema توکن، نمونهٔ کنفیگ Redis برای nonce، یا نمونهٔ SIEM queries) داری، آن را прямо بگو تا همان‌را کامل و دقیق اضافه کنم.
