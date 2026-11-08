# DomainSniffer 2.0 — © أفلاطون (Aflatoon / Plato)

dylib مستقل يراقب اتصالات اللعبة ويعرضها بقائمة عائمة بتصميم Cyber-Terminal.
**لا يحتاج Substrate ولا ElleKit ولا أي تفعيل إضافي**: محرك الـ hook مدمج داخل الملف
(إعادة ربط رموز Mach-O لدوال C + ObjC runtime لـ NSURLSession).

## ما يلتقطه
- DNS: `getaddrinfo` و `gethostbyname`
- HTTP / رفع / تحميل / WebSocket عبر `NSURLSession`
- اتصالات IP مباشرة: `connect` و `sendto` (UDP)
- يميّز المحجوب من NextDNS (رد 0.0.0.0 أو ::) والفاشل

## 🌐 DNS تلقائي → 8.8.8.8 (جديد)
- مفعّل افتراضياً، وزر `DNS 8.8.8.8` في القائمة يشغّله ويوقفه (يُحفظ في `settings.json`).
- `getaddrinfo` و `gethostbyname`: تُحلّ الأسماء مباشرة عبر 8.8.8.8 (محلّل مدمج + كاش حسب TTL). إذا تعذّر الوصول لـ 8.8.8.8 يرجع تلقائياً لـ DNS النظام ويتجاوزه 20 ثانية.
- `connect` و `sendto`: أي اتصال بمنفذ 53 (UDP/TCP) إلى أي سيرفر DNS يتحوّل إلى 8.8.8.8 (أو `2001:4860:4860::8888` لـ IPv6)، والسجل يعرض العنوان الأصلي.
- قواعد RULES اليدوية لها الأولوية على الوضع التلقائي.
- ملاحظة: مع هذا الوضع لن يعمل كشف الحجب عبر NextDNS (لأن الردود تأتي من Google)؛ أوقف الزر إذا أردت ذلك.

## الميزات
- زر عائم متوهّج قابل للسحب يفتح القائمة.
- إحصائيات حيّة: TOTAL / DOMAINS / IPS / BLOCKED، بحث، وفلاتر.
- الضغط على أي سطر: حجب في NextDNS / استبدال / نسخ.
- 🔑 NEXTDNS KEY: مفتاح API يُكتب مرة وحدة ويُحفظ في `Documents/DomainSniffer/settings.json`.
- 🔁 RULES: قواعد استبدال (`1.1.1.1 ← 8.8.8.8`, `*.example.com ← 1.2.3.4`) بمفتاح تشغيل لكل قاعدة، وتُحفظ بعد الخروج من اللعبة.
- حفظ ومشاركة وتصدير Denylist.

## البناء
GitHub Actions (ملف `.github/workflows/main.yml`) أو محلياً:
```
export THEOS=~/theos
make package
```
الناتج: `.theos/obj/DomainSniffer.dylib`

## الحقن (بدون جيلبريك)
حقن `DomainSniffer.dylib` في الـ IPA بـ Sideloadly أو ESign أو Feather. لا حاجة لخيار Substrate.

## ملاحظات
- المفتاح محفوظ كنص عادي داخل Documents.
- قواعد الـ IP تعمل على `connect`/`sendto`؛ اتصالات HTTPS عبر CFNetwork قد لا تمر بها.
