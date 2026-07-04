# Cryptography — Core Concepts Reference

مرجع لمفاهيم التشفير الأساسية وربطها بحماية البيانات في الشبكات وأنظمة Linux.

---

## 1. The Core Concepts: CIA Triad

كل قرار أمني بيدور حول 3 مبادئ أساسية:

| المبدأ | المعنى | مثال |
|---|---|---|
| Confidentiality | ضمان إن الداتا متشافة عن أي جهة غير مصرح لها | التشفير (Encryption) |
| Integrity | ضمان إن الداتا ما اتغيرتش أثناء النقل أو التخزين | Hashing |
| Availability | ضمان إن الداتا/الخدمة متاحة وقت الحاجة ليها | Backups, Redundancy |

---

## 2. Symmetric vs Asymmetric Encryption

### Symmetric Encryption

- بيستخدم **مفتاح واحد** لعملية التشفير وفك التشفير.
- أمثلة: AES, DES.
- سريع نسبياً، ومناسب لتشفير كميات كبيرة من الداتا.
- المشكلة: لازم الطرفين يتفقوا على المفتاح ويتبادلوه بأمان قبل البدء.

```
Plaintext --[Same Key]--> Ciphertext --[Same Key]--> Plaintext
```

### Asymmetric Encryption

- بيستخدم **زوج مفاتيح**: Public Key و Private Key.
- أمثلة: RSA, ECC.
- الـ Public Key بيتوزع بحرية، والـ Private Key بيفضل سري.
- أبطأ من الـ Symmetric، فبيتستخدم غالباً في **Key Exchange** مش تشفير الداتا الكبيرة مباشرة.

```
Encrypt with Public Key --> Decrypt only with matching Private Key
```

### مقارنة سريعة

| المعيار | Symmetric | Asymmetric |
|---|---|---|
| عدد المفاتيح | مفتاح واحد | زوج مفاتيح (Public/Private) |
| السرعة | سريع | أبطأ |
| الاستخدام الشائع | تشفير الداتا الفعلية | تبادل المفاتيح، التوقيع الرقمي |
| مثال | AES | RSA |

---

## 3. Hashing vs Encryption

نقطة مهمة لازم توضيحها بدقة: **الـ Hashing مش تشفير.**

| المعيار | Encryption | Hashing |
|---|---|---|
| قابل للعكس؟ | نعم (بمفتاح مناسب) | لا — One-Way Function |
| الهدف | Confidentiality | Integrity |
| أمثلة | AES, RSA | SHA-256, MD5 |
| الاستخدام الشائع | حماية محتوى الداتا | التحقق من عدم تغيّر الداتا (مثل تخزين كلمات المرور، توقيع الملفات) |

```bash
# مثال: توليد SHA-256 hash لملف على Linux
sha256sum file.txt
```

> ملاحظة: MD5 و SHA-1 بقوا غير آمنين حالياً بسبب Collision Attacks، والمعيار الحالي هو SHA-256 أو أعلى.

---

## 4. Real-World Application

### HTTPS/TLS Handshake

عملية الـ TLS Handshake بتجمع بين المفهومين (Asymmetric + Symmetric) في تسلسل واحد:

| الخطوة | الوظيفة |
|---|---|
| 1. Client Hello | العميل بيبعت للسيرفر الـ TLS Version المدعومة والـ Cipher Suites |
| 2. Server Hello + Certificate | السيرفر بيرد بالـ Public Key بتاعه (جوه الـ SSL Certificate) |
| 3. Key Exchange | يتم الاتفاق على Session Key باستخدام Asymmetric Encryption |
| 4. Symmetric Encryption | من هنا وبعد كده، كل الداتا بتتشفر بالـ Session Key باستخدام Symmetric Encryption (أسرع) |

بكده الاتصال بياخد **أمان الـ Asymmetric** في مرحلة الاتفاق، و**سرعة الـ Symmetric** في نقل الداتا الفعلية.

### SSH Keys

بديل عن الدخول بـ Password على سيرفرات Linux، باستخدام نفس مبدأ الـ Asymmetric Encryption:

```bash
# توليد زوج مفاتيح SSH على الجهاز المحلي
ssh-keygen -t rsa -b 4096

# نسخ الـ Public Key للسيرفر
ssh-copy-id user@server_ip

# بعد كده الدخول يتم بدون Password
ssh user@server_ip
```

**كيف يعمل:**

| المفتاح | مكانه |
|---|---|
| Private Key | يفضل على جهازك فقط، ما بيتشاركش أبداً |
| Public Key | بينقل للسيرفر ويتخزن في `~/.ssh/authorized_keys` |

عند محاولة الدخول، السيرفر بيتحقق إن الجهاز اللي بيحاول يدخل معاه الـ Private Key المطابق للـ Public Key المخزن، من غير ما يحتاج نقل أي Password عبر الشبكة.

---

**الجزء السابق:** [Networking & Wireshark](./2-Networking_%26_Wireshark.md)
**رجوع للفهرس:** [README](./README.md)
