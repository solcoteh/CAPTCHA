# مستند پژوهشی و اجرایی

## امنیت CAPTCHA و دفاع در برابر تهدیدات اتوماسیون (Bot/Abuse)

**نسخه:** 2.0
**تاریخ:** 2025-12-31
**مخاطب:** تیم امنیت اپلیکیشن، توسعه، DevOps/SRE، Product
**هدف:** کاهش سوءاستفادهٔ اتوماسیونی (spam / credential stuffing / account creation abuse / scraping / OTP flooding) با یک رویکرد چندلایه و قابل‌پایش

---

## 0) خلاصه مدیریتی (Executive Summary)

CAPTCHA یک «کنترل ضد اتوماسیون» است، نه یک قفل غیرقابل شکست. واقعیت صنعت این است که **هیچ CAPTCHAای وجود ندارد که تحت همه شرایط “غیرقابل bypass” باشد**؛ چون مهاجم می‌تواند از **انسان در حلقه** (خدمات حل‌کننده انسانی / relay) یا از پیشرفت‌های ML/Computer Vision استفاده کند. بنابراین هدف حرفه‌ای و قابل دفاع این است که:

1. **هزینه و پیچیدگی حمله را بالا ببریم**،
2. **نرخ موفقیت اتوماسیون را کاهش دهیم**،
3. **سوءاستفاده در مقیاس را متوقف/کند کنیم**،
4. و با **مانیتورینگ و پاسخ**، حملات را زود تشخیص دهیم. ([Cloudflare][1])

> نتیجه: بهترین راهکار “یک کپچا خیلی سخت” نیست؛ بلکه **یک معماری چندلایه Risk-based + Step-up verification** است. ([OWASP][2])

---

## 1) دامنه، اصول و خط قرمزها

### 1.1 دامنه

این مستند روی **امن‌سازی و دفاع** تمرکز دارد:

* نحوهٔ طراحی، پیاده‌سازی و بهره‌برداری درست CAPTCHA
* کنترل‌های مکمل ضدبات (Rate limiting, reputation, risk scoring, step-up)
* مانیتورینگ، آلارم‌دهی، و پاسخ به رخداد
* کیفیت تجربه کاربر + دسترس‌پذیری (Accessibility)

### 1.2 خط قرمز (برای تیم امنیت)

هرگونه اقدام یا دانش عملیاتی برای دور زدن کنترل‌ها باید به‌صورت داخلی و تحت **چارچوب مجاز تست نفوذ/Red Team** و با **مجوز کتبی** انجام شود. در این مستند، روش‌های حمله فقط در **سطح تهدیدشناسی** و برای طراحی دفاع مطرح می‌شوند (بدون ارائه مراحل اجرایی/کد).

---

## 2) تعریف‌ها و مفاهیم کلیدی

* **CAPTCHA:** تستی برای تشخیص انسان از ماشین
* **Anti-bot / Bot Management:** مجموعه کنترل‌ها برای تشخیص/مهار ترافیک خودکار (نه فقط کپچا)
* **Risk Scoring:** امتیازدهی به ریسک درخواست بر اساس سیگنال‌های رفتاری/فنی و تصمیم‌گیری مرحله‌ای (Allow/Challenge/Block/Review) ([Google for Developers][3])
* **Step-up Verification:** افزایش سطح احراز/چالش وقتی ریسک بالاست (مثل MFA/WebAuthn/Email verification) ([NIST Publications][4])
* **Relay / Human-in-the-loop:** حل چالش توسط انسان و بازگرداندن پاسخ به ابزار اتوماسیون ([F5, Inc.][5])

---

## 3) تهدیدها: CAPTCHA دقیقاً برای چه چیزهایی استفاده می‌شود؟

بر اساس طبقه‌بندی تهدیدات اتوماسیونی، CAPTCHA معمولاً برای کنترل سناریوهای زیر به‌کار می‌رود:

* **Account Creation Abuse** (ثبت‌نام انبوه)
* **Credential Stuffing / Password Spraying** (حملات لاگین با دیتابیس پسوردهای لو رفته)
* **Spam فرم‌ها/کامنت‌ها/پیام‌ها**
* **OTP/SMS/Email Flooding** (مصرف منابع و آزار کاربر)
* **Scraping / Footprinting** (جمع‌آوری داده) ([OWASP][2])

---

## 4) چرا “CAPTCHA غیرقابل bypass” وجود ندارد؟

### 4.1 دلیل ۱: انسان در حلقه همیشه یک راه است

حتی اگر کپچا از نظر ML سخت شود، مهاجم می‌تواند چالش را به انسان بسپارد و پاسخ را به ربات برگرداند. این مدل به‌عنوان یک صنعت زیرزمینی شناخته می‌شود. ([F5, Inc.][5])

### 4.2 دلیل ۲: ML و اتوماسیون در حال پیشرفت است

CAPTCHAها «کاملاً ضدبات» نیستند و اصولاً هم به‌عنوان یک کنترل کامل در نظر گرفته نمی‌شوند. ([Cloudflare][1])

### 4.3 نتیجه عملی

هدف دفاعی شما باید **مدیریت ریسک** باشد:

* جلوگیری از سوءاستفاده در مقیاس
* کاهش False Positive برای کاربران واقعی
* افزایش هزینه مهاجم
* و طراحی پاسخ مرحله‌ای ([OWASP Cheat Sheet Series][6])

---

## 5) طبقه‌بندی سطح‌بالا از مسیرهای شکست CAPTCHA (فقط برای طراحی دفاع)

> این بخش برای اینکه تیم بداند “کجاها می‌شکند” و چطور باید محکم کند، نه برای اجرای حمله.

1. **حل چالش (ML/OCR/Automation)** روی CAPTCHAهای کلاسیک/ضعیف
2. **Human relay / CAPTCHA farms**
3. **شکست پیاده‌سازی**: اعتبارسنجی نکردن در سرور، توکن‌های قابل replay، نشت secret، Fail-open
4. **شکست پوشش**: endpointهای جانبی (API، موبایل، مسیرهای کمتر دیده‌شده) بدون کپچا
5. **شکست منطق کسب‌وکار**: مهاجم کار را از مسیر دیگری انجام می‌دهد که کنترل‌های ضدبات روی آن اعمال نشده ([OWASP][2])

---

## 6) اصول طلایی امن‌سازی CAPTCHA (مستقل از برند)

### 6.1 اعتبارسنجی فقط در سرور (Mandatory Server-side Verification)

**کلاینت قابل اعتماد نیست.** هر سازوکاری که فقط به “نمایش ویجت” یا “چک‌کردن JS” متکی باشد، از نظر امنیتی باگ طراحی دارد. Turnstile صراحتاً می‌گوید: بدون Siteverify حفاظت کامل نیست و توکن‌ها می‌توانند جعل شوند. ([Cloudflare Docs][7])
reCAPTCHA هم توضیح می‌دهد که verify باید از backend انجام شود. ([Google for Developers][8])
hCaptcha نیز verify سروری را به‌عنوان الگوی اصلی بیان می‌کند. ([hCaptcha Docs][9])

### 6.2 جلوگیری از Replay و استفاده مجدد

* توکن باید **یک‌بارمصرف** باشد
* TTL کوتاه باشد
* و در سرور، از مصرف دوباره جلوگیری شود
  reCAPTCHA می‌گوید توکن **Single-use** است و برای جلوگیری از replay منقضی می‌شود. ([Google for Developers][8])
  Turnstile نیز Single-use بودن و انقضای حدود ۵ دقیقه را تاکید می‌کند. ([Cloudflare Docs][7])

### 6.3 Binding به زمینه (Context Binding)

نتیجهٔ CAPTCHA را به زمینهٔ درخواست گره بزنید:

* مسیر/اکشن (action)
* session / CSRF token
* شناسه تراکنش (transaction id)
* و در صورت نیاز IP/UA به‌عنوان سیگنال ریسک (نه الزاماً شرط سخت)
  برای reCAPTCHA v3 اساساً مفهوم action و امتیازدهی ریسک وجود دارد و باید در backend تفسیر شود. ([Google for Developers][3])

### 6.4 Fail-Closed با مسیر جایگزین امن

اگر سرویس CAPTCHA یا verify دچار مشکل شد:

* **Fail-open** یعنی “بفرمایید داخل” → خطرناک
* **Fail-closed** یعنی “مسیر امن جایگزین” مثل step-up verification یا صف بررسی/محدودسازی موقت
  برای Turnstile هم تأکید شده که عدم verify عملاً پیاده‌سازی ناقص و آسیب‌پذیر است. ([Cloudflare Docs][10])

### 6.5 CAPTCHA جایگزین Rate Limiting نیست

به‌خصوص در credential stuffing، نباید روی یک حد حجمی قابل پیش‌بینی تکیه کرد و باید چند بازهٔ زمانی و چند سیگنال را با هم دید. ([OWASP Cheat Sheet Series][6])

### 6.6 مانیتورینگ solve-rate و نشانه‌های شکست

OWASP پیشنهاد می‌کند **نرخ حل کپچا** را مانیتور کنید: solve-rate غیرعادی بالا می‌تواند نشانهٔ اتوماسیون/حل خودکار باشد و solve-rate پایین می‌تواند به UX بد و ریزش کاربران اشاره کند. ([OWASP Cheat Sheet Series][6])
Turnstile Analytics هم روی داده‌های token validation برای وضعیت امنیتی و مشاهدهٔ الگوها تاکید دارد. ([Cloudflare Docs][11])

---

## 7) الگوی تصمیم‌گیری Risk-based (معماری پیشنهادی)

به‌جای اینکه همیشه CAPTCHA نشان دهید، یک خط لوله تصمیم‌گیری بسازید:

### 7.1 ورودی سیگنال‌ها

* نرخ درخواست در چند پنجره زمانی (burst + sustained) ([OWASP Cheat Sheet Series][6])
* reputation IP/ASN، الگوی جغرافیایی، الگوی user-agentهای غیرعادی
* رفتار درون‌صفحه (تعامل طبیعی)
* الگوهای شکست/موفقیت لاگین، تعداد اکانت‌های ساخته‌شده، نرخ ارسال OTP
* نتیجهٔ CAPTCHA (در صورت نمایش)

### 7.2 خروجی تصمیم‌ها

* **Allow**
* **Throttle** (کندسازی/صف)
* **Challenge** (CAPTCHA/JS challenge)
* **Step-up** (تأیید ایمیل، MFA، WebAuthn)
* **Block** (با سیاست قابل توضیح و قابل بازبینی)

### 7.3 چرا Step-up قوی ارزشمند است؟

برای برخی سناریوها (خصوصاً حساب کاربری)، بهترین “ضدبات” در نهایت **احراز هویت مقاوم در برابر فیشینگ** است؛ NIST به استفاده از روش‌های phishing-resistant تشویق می‌کند. ([NIST Publications][4])
WebAuthn هم استاندارد کلیدی احراز هویت کلیدعمومی origin-bound است. ([W3C][12])

---

## 8) راهنمای پیاده‌سازی امن (Vendor-agnostic + نکات برندهای رایج)

### 8.1 قواعد مشترک پیاده‌سازی

**بایدها**

* verify فقط در backend
* ثبت وضعیت توکن مصرف‌شده (برای جلوگیری از replay در سناریوهای حساس)
* binding به action/route/session
* rate limit روی endpoint حساس + endpoint verify
* لاگ و متریک استاندارد (بخش 10)

**نبایدها**

* نگهداری secret در frontend
* تصمیم امنیتی صرفاً در JS
* اعتماد به “موفقیت ظاهری” ویجت
* fail-open بدون کنترل جایگزین

---

### 8.2 Google reCAPTCHA (خصوصاً v3)

* reCAPTCHA v3 **امتیاز ریسک** برمی‌گرداند و باید در backend تفسیر شود و بر اساس آن تصمیم بگیرید. ([Google for Developers][3])
* verify توکن باید در backend انجام شود؛ توکن برای جلوگیری از replay منقضی می‌شود و single-use است. ([Google for Developers][8])

**الگوی توصیه‌شده**

* تعیین اکشن‌های واضح (login / signup / reset / comment / checkout)
* تعریف آستانه‌ها بر اساس دادهٔ واقعی و نرخ خطا
* برای ریسک بالا: step-up (MFA/WebAuthn) به جای تکرار کپچا

---

### 8.3 Cloudflare Turnstile

* Turnstile تأکید می‌کند **server-side validation اجباری است** و توکن‌ها می‌توانند جعل شوند؛ همچنین tokenها **single-use** و با TTL محدود هستند. ([Cloudflare Docs][7])
* Turnstile Analytics می‌تواند داده‌های token validation برای دید امنیتی بدهد. ([Cloudflare Docs][11])

**الگوی توصیه‌شده**

* verify سروری در همان مسیر تراکنش
* fail-closed + fallback امن
* مانیتورینگ کِی و کجا challenge زیاد می‌شود (نشانه حمله یا UX بد)

---

### 8.4 hCaptcha

* الگوی hCaptcha واضح است: کاربر توکن می‌گیرد، سرور توکن را به endpoint siteverify می‌فرستد و نتیجه را مبنای تصمیم قرار می‌دهد. ([hCaptcha Docs][9])

---

## 9) الگوی “تقریباً سخت‌ترین حالت” در دنیای واقعی (بدون ادعای غیرقابل شکست)

### 9.1 هدف

حتی اگر CAPTCHA حل شد، مهاجم نتواند **در مقیاس** سوءاستفاده کند.

### 9.2 اجزا

1. **Rate limiting تطبیقی** (مبتنی بر رفتار + چند پنجره زمانی) ([OWASP Cheat Sheet Series][6])
2. **Risk scoring** (سیگنال‌های passive/active) ([Google for Developers][3])
3. **CAPTCHA فقط برای ریسک متوسط** (نه برای همه کاربران)
4. **Step-up قوی برای ریسک بالا**: WebAuthn/MFA و سیاست phishing-resistant ([NIST Publications][4])
5. **پایش solve-rate و token validation** ([OWASP Cheat Sheet Series][6])
6. **بستن مسیرهای جانبی** (API/موبایل/endpointهای “فراموش‌شده”) با همان سیاست

---

## 10) مانیتورینگ، متریک‌ها و آلارم‌ها (کلید موفقیت عملیاتی)

### 10.1 متریک‌های پایه

* CAPTCHA solve rate / fail rate ([OWASP Cheat Sheet Series][6])
* نرخ challenge نمایش داده‌شده به کل ترافیک (به تفکیک endpoint)
* latency verify (اثر روی UX و timeouts)
* توزیع جغرافیایی/ASN/IP reputation
* conversion rate در مسیرهایی که کپچا دارند (اثر روی بیزینس)

### 10.2 آلارم‌های پیشنهادی

* solve-rate غیرعادی بالا (مشکوک به automation/human relay) ([OWASP Cheat Sheet Series][6])
* افزایش ناگهانی challenge در یک endpoint خاص (شروع حمله یا misconfiguration)
* افزایش timeout/خطای verify (مشکل سرویس/شبکه، ریسک fail-open)

### 10.3 لاگ‌های لازم

* correlation id برای هر درخواست
* تصمیم نهایی (allow/challenge/step-up/block) + دلیل (risk signals)
* نتیجه verify (بدون ذخیرهٔ secret یا داده حساس)

---

## 11) چک‌لیست اجرایی برای تیم توسعه/DevOps

### 11.1 چک‌لیست پیاده‌سازی

* [ ] verify سروری اجباری است (reCAPTCHA/Turnstile/hCaptcha) ([Google for Developers][8])
* [ ] توکن single-use و TTL رعایت می‌شود (جلوگیری از replay) ([Google for Developers][8])
* [ ] binding به action/route/session/CSRF انجام شده ([Google for Developers][3])
* [ ] rate limiting روی endpointهای حساس فعال است و چند بازه زمانی را پوشش می‌دهد ([OWASP Cheat Sheet Series][6])
* [ ] fail-open نداریم؛ fallback امن داریم ([Cloudflare Docs][10])
* [ ] متریک‌ها و آلارم‌ها مطابق بخش 10 پیاده‌سازی شده‌اند ([Cloudflare Docs][11])

### 11.2 چک‌لیست امنیت عملیاتی

* [ ] Secretها در Secret Manager/ENV و خارج از کد/فرانت‌اند
* [ ] WAF/Rate limit/CDN ruleها با اپلیکیشن هم‌راستا هستند
* [ ] پوشش APIها و نسخه موبایل بررسی شده (فقط وب‌فرم نیست)

---

## 12) تست و ارزیابی (QA امنیت + Red Team داخلی)

### 12.1 تست‌های ضروری (بدون ورود به bypass عملیاتی)

* تست “عدم امکان انجام عملیات حساس بدون verify سروری”
* تست replay و استفاده مجدد توکن (باید fail شود) ([Google for Developers][8])
* تست مسیرهای جانبی: API، موبایل، endpointهای قدیمی
* تست failover: وقتی سرویس verify کند/قطع می‌شود، آیا سیاست fail-closed اجرا می‌شود؟ ([Cloudflare Docs][10])

### 12.2 معیار پذیرش

* نرخ موفقیت سوءاستفاده کاهش ملموس
* افزایش کنترل‌شده friction (challenge rate معقول)
* عدم ریزش شدید conversion

---

## 13) دسترس‌پذیری (Accessibility) و تجربه کاربری (UX)

CAPTCHA می‌تواند مانع جدی برای کاربران دارای معلولیت باشد. توصیه‌های W3C/WCAG روی ارائه راهکارهای جایگزین، توضیح مناسب برای محتوای غیرمتنی، و کاهش اصطکاک تاکید دارند. ([W3C][13])

**الگوی عملی پیشنهادی**

* به‌جای کپچاهای سخت و پرتکرار، از risk-based استفاده کنید (challenge فقط وقتی لازم است). ([Google for Developers][3])
* برای کاربران مشکل‌دار/false positive، مسیر پشتیبانی و step-up جایگزین داشته باشید.

---

## 14) حریم خصوصی و کاهش داده‌برداری

کنترل‌های ضدبات معمولاً با سیگنال‌های رفتاری کار می‌کنند؛ پس باید:

* داده‌های جمع‌آوری‌شده را حداقل کنید
* retention کوتاه و مبتنی بر نیاز داشته باشید
* برای کاربران/قوانین محلی، اطلاع‌رسانی شفاف و ارزیابی اثر حریم خصوصی انجام دهید
  در بحث‌های صنعتی، رویکردهای جدیدتر مثل Privacy Pass برای کاهش اصطکاک و حفظ حریم خصوصی مطرح‌اند. ([Datatracker][14])

---

## 15) جایگزین‌ها و مکمل‌های CAPTCHA (وقتی CAPTCHA به‌تنهایی کافی نیست)

### 15.1 Step-up مقاوم در برابر فیشینگ

برای سناریوهای حساب کاربری، بهترین دفاع پایدار، رفتن به سمت احراز هویت قوی (مثل WebAuthn/Passkeys) است. ([NIST Publications][4])

### 15.2 روش‌های مکمل در فرم‌ها

* rate limiting
* reputation
* رفتارشناسی (بدون اتکا صرف به یک سیگنال) ([OWASP Cheat Sheet Series][6])
* روش‌های ضداسپم در سطح فرم (مانند کنترل‌های کم‌اصطکاک) — با این شرط که جایگزین امنیتی اصلی نشوند. ([W3C][15])

---

## 16) شبه‌کد تصمیم‌گیری امن (الگوی معماری)

```text
ورودی: درخواست + سیگنال‌ها (IP/ASN/reputation, نرخ, session, device signals, token CAPTCHA)

risk = compute_risk(request)

اگر risk == LOW:
    allow

اگر risk == MEDIUM:
    اگر captcha_token موجود است:
        ok = server_side_verify(token, expected_action, expected_context)
        اگر ok و binding برقرار:
            allow
        else:
            step-up یا deny
    else:
        challenge (نمایش CAPTCHA)

اگر risk == HIGH:
    step-up قوی (WebAuthn/MFA/Email verification) یا throttle/block
```

(این الگو با فلسفهٔ risk-scoring و تصمیم‌گیری مرحله‌ای سازگار است.) ([Google for Developers][3])

---

## 17) منابع منتخب (برای استناد در تحویل به ارشد)

* OWASP Automated Threats to Web Applications (طبقه‌بندی تهدیدات اتوماسیونی، شامل CAPTCHA Defeat) ([OWASP][2])
* OWASP Credential Stuffing Prevention Cheat Sheet (مانیتور solve-rate، نرخ‌ها و رویکرد چندسیگنالی) ([OWASP Cheat Sheet Series][6])
* Google reCAPTCHA v3 + تفسیر امتیاز و تصمیم‌گیری در backend ([Google for Developers][3])
* Google reCAPTCHA Verify (backend verify، single-use، جلوگیری از replay) ([Google for Developers][8])
* Cloudflare Turnstile (الزام server-side validation، TTL، single-use) ([Cloudflare Docs][7])
* hCaptcha Docs (الگوی siteverify در سرور) ([hCaptcha Docs][9])
* Cloudflare Learning Center (CAPTCHAها کامل و بی‌نقص نیستند) ([Cloudflare][1])
* F5 Labs (واقعیت Human CAPTCHA Solvers و اثر آن بر امنیت) ([F5, Inc.][5])
* NIST SP 800-63B (راهنمای احراز هویت و تاکید بر phishing-resistant) ([NIST Publications][4])
* W3C WebAuthn (استاندارد احراز هویت کلیدعمومی origin-bound) ([W3C][12])
* IETF Privacy Pass (معماری و پروتکل‌های توکن‌های حریم‌خصوصی‌محور) ([Datatracker][14])
* W3C/WCAG (دسترس‌پذیری و چالش‌های CAPTCHA) ([W3C][13])

---

[1]: https://www.cloudflare.com/learning/bots/how-captchas-work/?utm_source=chatgpt.com "How CAPTCHAs work | What does CAPTCHA mean?"
[2]: https://owasp.org/www-project-automated-threats-to-web-applications/?utm_source=chatgpt.com "OWASP Automated Threats to Web Applications"
[3]: https://developers.google.com/recaptcha/docs/v3?utm_source=chatgpt.com "reCAPTCHA v3"
[4]: https://nvlpubs.nist.gov/nistpubs/SpecialPublications/NIST.SP.800-63B-4.pdf?utm_source=chatgpt.com "NIST.SP.800-63B-4.pdf"
[5]: https://www.f5.com/labs/articles/i-was-a-human-captcha-solver?utm_source=chatgpt.com "I Was a Human CAPTCHA Solver"
[6]: https://cheatsheetseries.owasp.org/cheatsheets/Credential_Stuffing_Prevention_Cheat_Sheet.html?utm_source=chatgpt.com "Credential Stuffing Prevention - OWASP Cheat Sheet Series"
[7]: https://developers.cloudflare.com/turnstile/get-started/server-side-validation/?utm_source=chatgpt.com "Validate the token · Cloudflare Turnstile docs"
[8]: https://developers.google.com/recaptcha/docs/verify?utm_source=chatgpt.com "Verifying the user's response | reCAPTCHA"
[9]: https://docs.hcaptcha.com/?utm_source=chatgpt.com "hCaptcha Docs"
[10]: https://developers.cloudflare.com/turnstile/get-started/?utm_source=chatgpt.com "Get started · Cloudflare Turnstile docs"
[11]: https://developers.cloudflare.com/turnstile/turnstile-analytics/token-validation/?utm_source=chatgpt.com "Token validation - Turnstile"
[12]: https://www.w3.org/TR/webauthn-2/?utm_source=chatgpt.com "Web Authentication: An API for accessing Public Key ..."
[13]: https://www.w3.org/TR/WCAG21/?utm_source=chatgpt.com "Web Content Accessibility Guidelines (WCAG) 2.1"
[14]: https://datatracker.ietf.org/doc/rfc9576/?utm_source=chatgpt.com "RFC 9576 - The Privacy Pass Architecture"
[15]: https://www.w3.org/WAI/GL/wiki/Captcha_Alternatives_and_thoughts?utm_source=chatgpt.com "Captcha Alternatives and thoughts - WCAG WG"
