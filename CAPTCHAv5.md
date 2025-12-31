

## مستند پژوهشی: تهدیدها، نقاط شکست و راهکارهای مقاوم‌سازی CAPTCHA و سامانه‌های ضدبات

**تاریخ:** ۳۱ دسامبر ۲۰۲۵
**هدف:** کاهش حملات خودکار (Bots)، اسپم، brute force، credential stuffing، ثبت‌نام انبوه و سوءاستفاده از فرم‌ها/Endpointها، با کمترین اصطکاک برای کاربران واقعی.

### خلاصه مدیریتی

1. CAPTCHA یک «دیوار نفوذناپذیر» نیست؛ یک **سیگنال/اصطکاک** در کنار کنترل‌های دیگر است. پژوهش‌ها نشان می‌دهند با رشد بینایی ماشین/یادگیری عمیق و سرویس‌های حل انسانی، اتکای صرف به CAPTCHA شکست می‌خورد. ([WIRED][1])
2. بزرگ‌ترین شکست‌های واقعی معمولاً از **پیاده‌سازی غلط** می‌آیند (اعتبارسنجی نکردن سمت سرور، امکان replay، لو رفتن secret، عدم محدودسازی نرخ و…)، نه صرفاً “ضعف معما”. اعتبارسنجی سمت سرور در reCAPTCHA/Turnstile/hCaptcha **اجباری/ضروری** است. ([Google for Developers][2])
3. «کپچای غیرقابل بایپس در همه شرایط» در عمل وجود ندارد؛ هدف درست، **افزایش هزینه مهاجم + کشف و مهار + کاهش سود حمله** با دفاعِ چندلایه و تطبیقی است.

---

# بخش ۱) مدل تهدید: بایپس CAPTCHA در سطح مفهومی (بدون آموزش اجرایی)

این بخش برای شناخت مسیرهای حمله است تا بتوانیم کنترل متناسب طراحی کنیم. (بدون ارائه دستورالعمل عملی)

## ۱.۱ کلاس‌های اصلی دور زدن (Attack Classes)

### A) حل انسانی و حملات رله (Human-in-the-loop / Relay)

* مهاجم به‌جای شکست الگوریتم، پاسخ را از **انسان** می‌گیرد (کارگر ارزان/فریلنسر/سرویس‌های حل).
* حتی CAPTCHAهای تعاملی/بازی‌محور هم می‌توانند با **رله جریانی** دور زده شوند. ([NDSS Symposium][3])
  **نشانه‌ها:** اختلاف غیرعادی بین زمان نمایش چالش و ارسال پاسخ، الگوهای زمانی غیرانسانی/غیرمحلی، نرخ موفقیت بالا از IPهای مشکوک، هم‌بستگی با Proxyهای مسکونی.

### B) حل خودکار با ML/OCR/CV/ASR

* متن‌محور (OCR/شبکه‌های عصبی)، تصویر‌محور (تشخیص شیء)، صوتی (ASR) در سال‌های اخیر به‌شدت پیشرفت کرده‌اند و بسیاری از کپچاهای کلاسیک را کم‌اثر کرده‌اند. ([CEUR-WS][4])

### C) سوءاستفاده از ضعف پیاده‌سازی و یکپارچه‌سازی (Integration/Implementation Flaws)

این شایع‌ترین و “ارزان‌ترین” مسیر است:

* **اعتبارسنجی نکردن سمت سرور**
* پذیرش توکن‌های منقضی/تکراری (Replay)
* لو رفتن Secret Key
* اعتبارسنجی نکردن «Action/Context» توکن و اتصال آن به عملیات درست
  مستندات رسمی تأکید می‌کنند توکن‌ها **یکبارمصرف** و **کوتاه‌عمر** هستند و باید سمت سرور validate شوند. ([Google for Developers][2])

### D) اتوماسیون مرورگر و تقلید رفتار کاربر

* اتوماسیون می‌تواند رفتارها را شبیه‌سازی کند، مخصوصاً وقتی CAPTCHA صرفاً یک ویجت UI باشد و کنترل‌های جانبی (ریسک‌سنجی/نرخ/سیگنال) ضعیف باشند.
* دفاع مؤثر معمولاً نیاز به **چند سیگنال** دارد (رفتاری، شبکه‌ای، دستگاهی، سابقه حساب).

### E) دور زدن از مسیر Accessibility و کانال‌های جایگزین

* بسیاری از کپچاها برای دسترس‌پذیری، مسیرهای جایگزین (مثل صوتی) دارند؛ اما Audio CAPTCHAها هم با پیشرفت ASR تضعیف شده‌اند. ([arXiv][5])
* از طرف دیگر، استانداردهای دسترس‌پذیری توصیه می‌کنند اگر CAPTCHA دارید، جایگزین‌هایی برای گروه‌های مختلف فراهم شود. ([w3.org][6])

---

# بخش ۲) واقعیت مهم: «امن‌سازی کامل که هیچ هکری نتواند بایپس کند» ممکن نیست

این جمله را بهتر است در گزارش با ادبیات حرفه‌ای این‌طور بنویسی:

> هیچ CAPTCHAای “اثباتاً غیرقابل دور زدن” نیست؛ چون مهاجم می‌تواند از انسان، اتوماسیون پیشرفته، یا ضعف‌های پیاده‌سازی استفاده کند. هدف طراحی دفاعی، **کاهش نرخ موفقیت حمله و افزایش هزینه/ریسک مهاجم** با کنترل‌های چندلایه و تطبیقی است.

این دیدگاه هم در ادبیات صنعتی و هم در پژوهش‌ها تکرار می‌شود. ([WIRED][1])

---

# بخش ۳) راهکارهای جامع سخت‌سازی CAPTCHA (برای انواع کپچاها)

## ۳.۱ اصول طلایی (برای همه کپچاها)

### اصل ۱: CAPTCHA هرگز “کنترل امنیتی تنها” نباشد

کنار CAPTCHA، این‌ها را باید داشته باشید:

* Rate limiting و کنترل‌های ضد brute-force / ضد credential stuffing ([OWASP Cheat Sheet Series][7])
* تشخیص ناهنجاری و مانیتورینگ درخواست‌ها (Bot telemetry) ([OWASP Foundation][8])
* برای عملیات حساس: **Step-up** (مثلاً MFA/OTP/WebAuthn) نه “کپچای سخت‌تر”.

### اصل ۲: اعتبارسنجی سمت سرور، غیرقابل مذاکره

نمونه‌های رسمی:

* reCAPTCHA: توکن **۲ دقیقه اعتبار** و **یکبارمصرف** است؛ باید سمت سرور verify شود تا replay نشود. ([Google for Developers][2])
* Turnstile: سمت سرور باید validate کند؛ توکن‌ها کوتاه‌عمر و single-use هستند و widget به‌تنهایی کافی نیست. ([Cloudflare Docs][9])
* hCaptcha: پس از چالش، توکن باید سمت سرور در endpoint رسمی verify شود. ([hCaptcha Docs][10])

### اصل ۳: توکن را به «کانتکست» ببندید

در لایه طراحی:

* توکن CAPTCHA باید فقط برای **همان عملیات** معتبر باشد (مثلاً “signup”، “login”، “reset-password”).
* باید با **session / CSRF token / nonce** و اطلاعات زمینه‌ای (IP ریسک‌دار، user-agent خانواده، زمان) هم‌بسته شود تا بازاستفاده سخت‌تر شود.

### اصل ۴: کلیدهای Secret را مثل کلید API حیاتی مدیریت کنید

* Secret داخل Frontend/موبایل قرار نگیرد.
* Rotate، محدودسازی دسترسی، جلوگیری از لیک در CI/CD و repo.
* مانیتورینگ استفاده غیرعادی.

---

## ۳.۲ کنترل‌های دفاعی مکمل (مخصوصاً برای مقابله با رله/حل انسانی)

### ضد “CAPTCHA farm” و رله

چون انسان حل می‌کند، باید رفتار و اقتصاد حمله را هدف بگیری:

* **افزایش هزینه حمله** با:

  * محدودسازی نرخ بر اساس IP/ASN/ریسک/حساب
  * “Proof-of-work سبک” یا اصطکاک محاسباتی برای درخواست‌های مشکوک (بدون اذیت کاربر سالم)
  * محدودسازی ساخت حساب و مرحله‌بندی (مثلاً ایمیل‌ولیدیشن + تاخیر هوشمند)
* **کشف رله** با:

  * ویژگی‌های زمانی (زمان پاسخ غیرعادی، الگوهای یکنواخت)
  * هم‌بستگی پاسخ‌ها بین چند حساب/چند IP
    پژوهش‌های relay/stream relay در کپچاهای تعاملی نشان می‌دهند باید روی “نشانه‌های آماری/رفتاری” کار کرد. ([NDSS Symposium][3])

---

## ۳.۳ کنترل‌های خاص بر اساس نوع CAPTCHA

### ۱) کپچای متنی (Text-based)

**مشکل اصلی:** OCR/Deep Learning در شکست این‌ها خیلی موفق شده. ([CEUR-WS][4])
**پیشنهاد دفاعی:**

* اگر مجبورید استفاده کنید: کوتاه، ساده، با نرخ خطای پایین برای انسان؛ اما **به‌عنوان لایه ثانویه**.
* ترجیحاً به سمت **risk-based** یا چالش‌های مدرن‌تر بروید.

### ۲) کپچای تصویری/انتخاب شیء

**مشکل:** مدل‌های تشخیص شیء/طبقه‌بندی رشد کرده‌اند. ([Seventh Sense Research Group][11])
**پیشنهاد دفاعی:**

* چالش را صرفاً “تشخیص شیء” نگذارید؛ با سیگنال‌های رفتاری/ریسکی ترکیب کنید.
* از پلتفرم‌هایی استفاده کنید که **چالش تطبیقی** دارند (نه static puzzle).

### ۳) کپچای صوتی (Audio)

**چالش:** دسترس‌پذیری مهم است، ولی ASR آن را تضعیف کرده. ([arXiv][5])
**پیشنهاد دفاعی:**

* اگر صوتی ارائه می‌دهید، طراحی باید همزمان “قابل فهم برای انسان” و “سخت برای ASR” باشد؛ پژوهش‌هایی دقیقاً روی همین موضوع کار کرده‌اند. ([arXiv][5])
* حتماً الزامات WCAG/دسترس‌پذیری را رعایت کنید (جایگزین‌های واقعی برای کاربران مختلف). ([w3.org][6])

### ۴) کپچای رفتاری/نامرئی/امتیازدهی ریسک

**مزیت:** اصطکاک کم؛ قابل ترکیب با سیاست‌های امنیتی.
**ریسک:** اگر فقط روی یک سیگنال تکیه کند، با اتوماسیون پیشرفته یا ترافیک انسانی ممکن است دور زده شود.
**پیشنهاد دفاعی:** آستانه‌ها، step-up، و “حاکمیت داده” (monitoring/feedback loop) حیاتی است.

### ۵) جایگزین‌های CAPTCHA: چالش‌های مرورگر و Bot Management

Cloudflare Turnstile نمونه‌ای از رویکرد «چالش‌های مرورگر + تله‌متری» برای کاهش معماهای آزاردهنده است. ([The Cloudflare Blog][12])

---

# بخش ۴) «چطور پیاده‌سازی کنیم که امن‌ترین حالت عملی را داشته باشد؟» (Blueprint اجرایی دفاعی)

این بخش را می‌توانی تقریباً مستقیم به تیم توسعه بدهی.

## ۴.۱ معماری پیشنهادی (Adaptive, Defense-in-Depth)

**لایه ۰ — بهداشت Endpoint**

* WAF/CDN rules برای الگوهای واضح bot
* محدودیت نرخ (Rate limit) بر اساس IP/ASN/Path/Account
* جلوگیری از enumeration (پیغام خطای یکنواخت، زمان پاسخ یکنواخت)

**لایه ۱ — سیگنال‌های تشخیص Bot**

* IP reputation، ASN، geo anomalies
* TLS/JA3-like fingerprints (در صورت امکان در لایه شبکه)
* رفتار (سرعت پر کردن فرم، ترتیب فیلدها، تعاملات)

**لایه ۲ — چالش تطبیقی**

* کاربران کم‌ریسک: بدون اصطکاک یا چالش سبک
* کاربران مشکوک: CAPTCHA/Turnstile/hCaptcha + محدودیت نرخ شدیدتر
* کاربران پرریسک: Step-up (MFA/Email/Phone/WebAuthn) یا مسدودسازی موقت

**لایه ۳ — کنترل‌های پس از اقدام**

* برای signup: تأیید ایمیل + محدودیت ساخت حساب
* برای login: سیاست ضد credential stuffing + قفل موقت/تاخیر تصاعدی ([OWASP Cheat Sheet Series][7])
* برای عملیات حساس (برداشت، تغییر ایمیل، reset): MFA/WebAuthn

## ۴.۲ جریان درست اعتبارسنجی (Critical Flow)

1. کاربر فرم را submit می‌کند و توکن challenge تولید می‌شود
2. سرور **قبل از هر کاری** توکن را با Provider verify می‌کند
3. سرور فقط در صورت success، منطق اصلی را اجرا می‌کند
   این همان چیزی است که در راهنمای Turnstile هم به‌عنوان «Proper flow» آمده است. ([Cloudflare Docs][13])

## ۴.۳ چک‌لیست “اشتباهات مرگبار” (Do-Not-Do)

* ❌ تکیه به “تیک خوردن” در UI بدون verify سمت سرور ([Cloudflare Docs][9])
* ❌ قبول کردن توکن‌های قدیمی/تکراری (Replay) ([Google for Developers][2])
* ❌ لو دادن Secret در frontend/mobile یا repo
* ❌ گذاشتن CAPTCHA روی فرم، ولی باز گذاشتن API پشت آن بدون همان کنترل‌ها (API bypass)
* ❌ نبود rate limit و مانیتورینگ

---

# بخش ۵) پاسخ مستقیم به «موضوع سوم»: کپچایی که هیچ‌کس نتواند bypass کند؟

اگر کارفرما دقیقاً همین جمله را گفته، بهترین پاسخ حرفه‌ای این است:

* **کپچای صددرصد غیرقابل بایپس وجود ندارد** (به‌خصوص به خاطر حل انسانی/رله و ضعف‌های پیاده‌سازی).
* اما می‌توان سامانه‌ای طراحی کرد که **در عمل** بایپس را برای مهاجم **غیر اقتصادی، پرریسک و کم‌نتیجه** کند.

## نسخه پیشنهادی “حداکثر امنیت عملی” برای شرکت شما

1. استفاده از یک سرویس مدرن (Turnstile / reCAPTCHA Enterprise / hCaptcha Enterprise) با validate سمت سرور ([Cloudflare Docs][9])
2. Adaptive challenges + Risk scoring (اصطکاک برای مشکوک‌ها)
3. Rate limiting + تشخیص credential stuffing + تاخیر تصاعدی ([OWASP Cheat Sheet Series][7])
4. برای عملیات حساس: Step-up با MFA/WebAuthn (نه کپچای سخت‌تر)
5. ضد رله: تحلیل زمانی/رفتاری + مانیتورینگ/لیست سیاه پویا ([NDSS Symposium][3])
6. دسترس‌پذیری و رعایت WCAG (جایگزین معتبر برای کاربران خاص) ([w3.org][6])

---

# بخش ۶) برنامه تست و ارزیابی (برای اینکه گزارش شما «قابل دفاع» باشد)

## ۶.۱ KPIهای پیشنهادی

* نرخ موفقیت bot قبل/بعد
* نرخ false positive (گیر افتادن کاربر واقعی)
* نرخ تکمیل فرم (Conversion)
* هزینه متوسط حمله (request cost) و زمان کشف

## ۶.۲ سناریوهای تست (بدون ارائه روش اجرایی)

* ترافیک انسانی واقعی vs ترافیک خودکار
* سناریوهای رله انسانی (برای سنجش نشانه‌های زمانی)
* تلاش برای replay/ارسال توکن تکراری/منقضی (برای اطمینان از یکبارمصرف بودن)

---

# پیوست A) نقشه تطبیق نوع کپچا به ریسک و دفاع

* **متنی:** ضعیف‌تر در برابر ML → فقط به‌عنوان لایه ثانویه + rate limiting
* **تصویری:** بهتر از متنی در برخی سناریوها، اما همچنان شکننده در برابر CV → تطبیقی + سیگنال رفتاری
* **صوتی:** لازم برای دسترس‌پذیری، اما شکننده در برابر ASR → طراحی مقاوم + گزینه‌های دسترس‌پذیری ([arXiv][5])
* **رفتاری/نامرئی:** UX عالی → نیازمند مانیتورینگ و step-up برای پرریسک‌ها
* **Bot management/challenges مرورگر:** مناسب برای جایگزینی/کاهش CAPTCHA کلاسیک ([The Cloudflare Blog][12])

---

## جمع‌بندی نهایی برای ارائه به ارشد

اگر بخواهم یک جمله‌ای “حرفه‌ای و دقیق” جمع‌بندی کنم:

> بهترین دفاع در برابر bypass کپچا، ساختن یک سامانه **ضدبات تطبیقی و چندلایه** است: validate سمت سرور + توکن کوتاه‌عمر و یکبارمصرف + rate limiting + سیگنال‌های ریسکی + step-up برای عملیات حساس. CAPTCHA به‌تنهایی کافی نیست و «غیرقابل بایپس مطلق» در دنیای واقعی قابل تضمین نیست. ([Cloudflare Docs][9])

---

اگر خواستی، همین مستند را در پیام بعدی **به قالب یک گزارش شرکتی** تبدیل می‌کنم (فهرست، شماره‌گذاری رسمی، بخش “Scope/Assumptions”، و بخش “Recommendations by Priority”) و طوری می‌نویسم که آماده کپی‌کردن داخل Word باشد.

[1]: https://www.wired.com/story/captcha-automation-broken-history-fix?utm_source=chatgpt.com "Captcha is dying. This is how it's being reinvented for the AI age"
[2]: https://developers.google.com/recaptcha/docs/verify?utm_source=chatgpt.com "Verifying the user's response | reCAPTCHA"
[3]: https://www.ndss-symposium.org/wp-content/uploads/2017/09/02_3-paper.pdf?utm_source=chatgpt.com "Dynamic Cognitive Game CAPTCHA Usability and ..."
[4]: https://ceur-ws.org/Vol-1885/93.pdf?utm_source=chatgpt.com "Breaking CAPTCHAs with Convolutional Neural Networks"
[5]: https://arxiv.org/abs/2203.05408?utm_source=chatgpt.com "Attacks as Defenses: Designing Robust Audio CAPTCHAs Using Attacks on Automatic Speech Recognition Systems"
[6]: https://www.w3.org/WAI/WCAG21/Techniques/general/G144.html?utm_source=chatgpt.com "Ensuring that the web page contains another CAPTCHA ..."
[7]: https://cheatsheetseries.owasp.org/cheatsheets/Credential_Stuffing_Prevention_Cheat_Sheet.html?utm_source=chatgpt.com "Credential Stuffing Prevention - OWASP Cheat Sheet Series"
[8]: https://owasp.org/www-project-automated-threats-to-web-applications/?utm_source=chatgpt.com "OWASP Automated Threats to Web Applications"
[9]: https://developers.cloudflare.com/turnstile/get-started/server-side-validation/?utm_source=chatgpt.com "Validate the token · Cloudflare Turnstile docs"
[10]: https://docs.hcaptcha.com/?utm_source=chatgpt.com "hCaptcha Docs"
[11]: https://www.internationaljournalssrg.org/IJEEE/2024/Volume11-Issue6/IJEEE-V11I6P130.pdf?utm_source=chatgpt.com "Breaking CAPTCHA Using Deep Learning Techniques and ..."
[12]: https://blog.cloudflare.com/turnstile-private-captcha-alternative/?utm_source=chatgpt.com "Announcing Turnstile, a user-friendly, privacy-preserving ..."
[13]: https://developers.cloudflare.com/turnstile/tutorials/login-pages/?utm_source=chatgpt.com "Protect your forms · Cloudflare Turnstile docs"
