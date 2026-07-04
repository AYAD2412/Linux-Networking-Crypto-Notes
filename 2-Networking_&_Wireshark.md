# Networking & Wireshark — Practical Analysis Reference

مرجع عملي لمفاهيم الشبكات الأساسية، أدوات الفحص من الـ Linux CLI، والتحليل العملي باستخدام Wireshark.

---

## 1. Core Networking Concepts

### OSI Model vs TCP/IP Model

| OSI Model (7 Layers) | TCP/IP Model (4 Layers) | مثال |
|---|---|---|
| 7. Application | Application | HTTP, DNS, FTP |
| 6. Presentation | Application | Encryption, Encoding |
| 5. Session | Application | Session Management |
| 4. Transport | Transport | TCP, UDP |
| 3. Network | Internet | IP, ICMP |
| 2. Data Link | Network Access | Ethernet, ARP |
| 1. Physical | Network Access | Cables, Signals |

### مفهوم الـ Encapsulation

كل طبقة بتضيف الـ Header الخاص بيها فوق الداتا القادمة من الطبقة اللي فوقها، والترتيب بيبقى كالتالي وقت الإرسال:

```
Data → Segment (TCP/UDP Header) → Packet (IP Header) → Frame (Ethernet Header) → Bits
```

عند الاستقبال، بيحصل العكس تماماً (Decapsulation) — كل طبقة بتشيل الـ Header الخاص بيها وتمرر الباقي للطبقة اللي فوقها.

### Core Protocols

| Protocol | الوظيفة |
|---|---|
| IP | تحديد عنوان المصدر والوجهة وتوجيه الحزم |
| TCP | نقل موثوق للداتا مع ضمان الترتيب والوصول |
| UDP | نقل سريع للداتا بدون ضمان الوصول أو الترتيب |
| ICMP | رسائل تشخيصية للشبكة (مثل `ping`) |
| DNS | ترجمة أسماء الدومينات لعناوين IP |
| DHCP | توزيع عناوين IP تلقائياً على الأجهزة |
| ARP | ترجمة عنوان IP لعنوان MAC داخل نفس الشبكة المحلية |

### TCP vs UDP

| المعيار | TCP | UDP |
|---|---|---|
| الاتصال | Connection-oriented (3-Way Handshake) | Connectionless |
| الموثوقية | مضمون الوصول (Retransmission عند الفقد) | غير مضمون |
| الترتيب | الحزم بتوصل بالترتيب | مفيش ضمان للترتيب |
| السرعة | أبطأ نسبياً بسبب الفحوصات | أسرع |
| الاستخدام | HTTP, HTTPS, FTP, Email | DNS, Streaming, VoIP, Gaming |

---

## 2. Linux Networking CLI Tools

| Command | الوظيفة |
|---|---|
| `ip a` | عرض واجهات الشبكة وعناوين IP الحالية |
| `ifconfig` | نفس وظيفة `ip a` (أداة قديمة، مش موجودة في كل التوزيعات الحديثة) |
| `netstat -tulnp` | عرض الـ Ports المفتوحة والعمليات المرتبطة بيها |
| `ss -tulnp` | البديل الحديث لـ `netstat` (أسرع وأدق) |
| `ping <host>` | فحص الاتصال بجهاز معين |
| `traceroute <host>` | تتبع مسار الحزمة من جهازك للوجهة عبر كل الـ Routers |
| `nslookup <domain>` | الاستعلام عن عنوان IP الخاص بدومين معين |
| `dig <domain>` | استعلام DNS أكثر تفصيلاً من `nslookup` |

```bash
ip a                        # عرض عناوين IP الحالية
ss -tulnp                   # عرض الـ Ports المفتوحة والخدمات المرتبطة بيها
ping 8.8.8.8                # اختبار الاتصال بجوجل DNS
traceroute google.com       # تتبع مسار الحزم
dig google.com              # استعلام DNS تفصيلي
```

---

## 3. Wireshark Practical Analysis

### الواجهة الأساسية

Wireshark بيقسم الشاشة لـ 3 أجزاء رئيسية:

1. **Packet List** — قائمة كل الحزم اللي اتلقطت.
2. **Packet Details** — تفاصيل الحزمة المختارة مقسمة حسب الطبقات (Layers).
3. **Packet Bytes** — الداتا الخام (Hex/ASCII) للحزمة.

### Packet Color Coding

Wireshark بيلوّن الحزم تلقائياً حسب النوع، وده بيسهّل التحليل السريع:

| اللون | المعنى الشائع |
|---|---|
| أخضر فاتح | TCP Traffic عادي |
| أزرق فاتح | UDP Traffic |
| أسود | حزم فيها أخطاء (Errors, Malformed Packets) |
| أحمر/أسود | TCP Errors (مثل Retransmissions) |

> يمكن تخصيص الألوان دي يدوياً من `View > Coloring Rules`.

### حفظ الملفات (.pcap)

```
File > Save As > اختيار الصيغة .pcap أو .pcapng
```

- `.pcap` — الصيغة القديمة والأكثر توافقاً مع أدوات التحليل الأخرى.
- `.pcapng` — الصيغة الحديثة (تدعم بيانات إضافية زي التعليقات).

### Display Filters Reference

| الفلتر | الاستخدام |
|---|---|
| `ip.addr == 192.168.1.1` | عرض كل الحزم من/لـ IP معين |
| `ip.src == 192.168.1.1` | عرض الحزم القادمة من IP معين فقط |
| `ip.dst == 192.168.1.1` | عرض الحزم المتجهة لـ IP معين فقط |
| `tcp.port == 80` | عرض حزم TCP على بورت 80 |
| `http.request.method == "POST"` | عرض طلبات POST فقط |
| `http.request.method == "GET"` | عرض طلبات GET فقط |
| `dns` | عرض كل حركة DNS |
| `tcp.flags.syn == 1` | عرض حزم بداية الاتصال (SYN) |
| `http contains "password"` | البحث عن كلمة معينة داخل حركة HTTP |

```bash
# مثال مركب: عرض كل طلبات POST المتجهة لسيرفر معين
ip.dst == 192.168.1.10 && http.request.method == "POST"
```

---

## 4. Advanced Analysis

### Follow TCP Stream

بيسمح بتجميع كل الحزم الخاصة باتصال TCP واحد وعرضها كمحادثة كاملة ومتسلسلة، بدل ما تدور على كل حزمة لوحدها.

**الخطوات:**

```
Right Click على الحزمة > Follow > TCP Stream
```

**الفرق بين HTTP وHTTPS عند استخدام Follow TCP Stream:**

| البروتوكول | ما يظهر في الـ Stream |
|---|---|
| HTTP | الداتا كاملة بصيغة Clear Text (Username, Password, Request Body) — قابلة للقراءة المباشرة |
| HTTPS | الداتا مشفّرة (TLS Encrypted) — بتظهر كـ Binary/Hex غير مفهوم بدون فك التشفير |

> هذا الفرق هو السبب الأساسي وراء أهمية التحول من HTTP إلى HTTPS في أي نظام يتعامل مع بيانات حساسة.

### Statistics Menu — تحديد الـ Conversations الأكثر استهلاكاً

```
Statistics > Conversations
```

- بيعرض جدول بكل أزواج الاتصال (Source/Destination) مع حجم الداتا المتبادلة ومدة الاتصال.
- مفيد لتحديد أي جهاز أو اتصال بيستهلك Bandwidth بشكل غير طبيعي (مؤشر محتمل على Data Exfiltration أو نشاط غير طبيعي).

يمكن أيضاً استخدام:

```
Statistics > Protocol Hierarchy
```

لمعرفة توزيع البروتوكولات داخل الـ Capture بالكامل (نسبة HTTP, DNS, TCP... إلخ).

---

**الجزء السابق:** [Linux Command Line](./1-Linux_Command_Line.md)
**الجزء التالي:** [Cryptography](./3-Cryptography.md)
