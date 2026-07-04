# 🐧 Linux Command Line — Practical Reference

مرجع عملي لأوامر الـ Linux مرتب على أساس **سيناريو الاستخدام اليومي** لـ Analyst، مش مجرد رص أوامر عشوائي.

---

## 1️⃣ Navigating & File System

### الأوامر الأساسية

| Command | الوظيفة |
|---------|---------|
| `pwd` | يعرض المسار الحالي (Print Working Directory) |
| `cd` | التنقل بين الـ Directories |
| `ls -la` | عرض كل الملفات (حتى الـ Hidden) مع التفاصيل الكاملة (Permissions, Owner, Size, Date) |
| `mkdir` | إنشاء Directory جديد |
| `rm -rf` | حذف Directory وكل محتوياته بالقوة (**خطير جداً، لازم حذر شديد**) |

```bash
cd /var/log        # الانتقال لمسار معين
ls -la              # عرض كل الملفات جوه الـ Directory الحالي
mkdir new_project   # إنشاء مجلد جديد
rm -rf old_folder   # حذف مجلد بالكامل - لا رجعة فيه
```

### Relative Path vs Absolute Path

| النوع | التعريف | مثال |
|-------|---------|------|
| **Absolute Path** | مسار كامل يبدأ من الـ Root (`/`) | `/home/user/documents/file.txt` |
| **Relative Path** | مسار نسبي لمكانك الحالي | `documents/file.txt` أو `../file.txt` |

> 💡 `.` بيرمز للمجلد الحالي، و `..` بيرمز للمجلد الأب (Parent Directory).

---

## 2️⃣ Text Manipulation & Log Parsing

الجزء ده هو الأهم عملياً لأي Analyst بيتعامل مع Logs كبيرة.

### قراءة وفلترة الملفات

| Command | الاستخدام |
|---------|-----------|
| `cat file` | عرض محتوى الملف كامل (مناسب للملفات الصغيرة فقط) |
| `less file` | عرض الملف صفحة بصفحة (مناسب للملفات الكبيرة) |
| `head -n 20 file` | عرض أول 20 سطر |
| `tail -n 20 file` | عرض آخر 20 سطر |
| `tail -f file` | متابعة الملف Live أول بأول (مفيد لمتابعة Logs بتتحدث باستمرار) |

```bash
tail -f /var/log/auth.log   # متابعة محاولات تسجيل الدخول لحظة بلحظة
```

### أدوات الـ Parsing القوية

| Tool | الوظيفة |
|------|---------|
| `grep` | البحث عن نص/pattern معين جوه الملف |
| `awk` | معالجة النصوص وتقسيمها لأعمدة (Columns) |
| `sed` | البحث والاستبدال (Find & Replace) داخل الملفات |
| `cut` | استخراج جزء معين من كل سطر (بناءً على Delimiter) |
| `sort` | ترتيب الأسطر |
| `uniq` | إزالة الأسطر المكررة (لازم الداتا تكون Sorted الأول) |

```bash
grep "404" access.log                     # استخراج كل الأسطر اللي فيها 404
awk '{print $1}' access.log               # طباعة العمود الأول بس (غالباً الـ IP)
sed 's/error/ERROR/g' file.log            # استبدال كل كلمة error بـ ERROR
cut -d ' ' -f1 access.log                 # قص العمود الأول بناءً على المسافة كـ Delimiter
```

### 🎯 تطبيق عملي: استخراج الـ Unique IP Addresses من Log كبير

```bash
cat access.log | awk '{print $1}' | sort | uniq -c | sort -nr
```

**شرح الخطوات:**

| الخطوة | الوظيفة |
|--------|---------|
| `cat access.log` | قراءة الملف |
| `awk '{print $1}'` | استخراج العمود الأول (الـ IP Address) من كل سطر |
| `sort` | ترتيب الـ IPs أبجدياً/رقمياً (ضروري قبل `uniq`) |
| `uniq -c` | حذف المكرر وعد عدد مرات تكرار كل IP |
| `sort -nr` | ترتيب النتيجة تنازلياً حسب عدد التكرار (الأكتر تكراراً في الأول) |

> ✅ الأمر ده بيدّيك أكتر الـ IPs اللي بتحاول تدخل على السيرفر، وده مفيد جداً في كشف الـ Brute Force Attacks.

---

## 3️⃣ System Information & Resources

| Command | الوظيفة |
|---------|---------|
| `top` | مراقبة العمليات والموارد Live (CPU, RAM) |
| `htop` | نسخة محسّنة ومرئية أكتر من `top` |
| `df -h` | عرض مساحة التخزين المتاحة (Human-readable) |
| `free -m` | عرض استخدام الـ RAM بالـ Megabytes |

```bash
df -h        # مساحة الأقراص
free -m      # حالة الذاكرة
top          # مراقبة العمليات لحظياً
```

---

## 4️⃣ Permissions & Ownership

### Permission Matrix

كل ملف في Linux ليه 3 أنواع صلاحيات لـ 3 فئات:

| الفئة | الرمز |
|-------|-------|
| Owner (المالك) | `u` |
| Group (المجموعة) | `g` |
| Others (الباقي) | `o` |

| الصلاحية | الرمز | القيمة الرقمية |
|----------|-------|----------------|
| Read | `r` | 4 |
| Write | `w` | 2 |
| Execute | `x` | 1 |

مثال: `rwxr-xr--` تعني:
- Owner: `rwx` (Read+Write+Execute = 7)
- Group: `r-x` (Read+Execute = 5)
- Others: `r--` (Read فقط = 4)

= الصلاحية الرقمية الكاملة: **754**

### أوامر التعديل

```bash
chmod 755 script.sh          # تعديل الصلاحيات رقمياً
chmod u+x script.sh          # إضافة صلاحية Execute للـ Owner فقط
chown user:group file.txt    # تغيير مالك الملف والمجموعة
```

---

## 5️⃣ Users & Privilege Escalation

| Command | الوظيفة |
|---------|---------|
| `whoami` | عرض اسم المستخدم الحالي |
| `id` | عرض الـ UID, GID, والمجموعات اللي المستخدم تابع لها |
| `su` | التبديل لمستخدم تاني (Switch User) |
| `sudo` | تنفيذ أمر بصلاحيات الـ Root |

```bash
whoami
id
sudo -l          # عرض الأوامر المسموح للمستخدم الحالي ينفذها كـ Root
```

### ⚠️ خطورة الـ Misconfigured Sudo Permissions

لو مستخدم عادي معاه صلاحية `sudo` على أداة معينة بدون قيود (زي `find`, `vim`, `less`, `awk`)، ده ممكن يُستغل في **Privilege Escalation** — يعني المستخدم يوصل لصلاحيات Root من غير ما يكون مفروض له.

**مثال بسيط للاستغلال (لو `vim` مسموح بـ sudo بدون قيود):**

```bash
sudo vim -c ':!/bin/sh'
```

الأمر ده بيفتح Shell بصلاحيات Root من جوه الـ Vim، وده بيحصل بسبب سوء ضبط ملف `/etc/sudoers`.

> 🔒 **القاعدة الأمنية:** دايماً اضبط صلاحيات الـ `sudo` بأقل حد ممكن (Principle of Least Privilege)، وراجع ملف `/etc/sudoers` بأمر `visudo` بس (عشان يتفحص الأخطاء قبل الحفظ).

---

⬅️ **الجزء اللي بعده:** [Networking & Wireshark](./2-Networking_%26_Wireshark.md)
