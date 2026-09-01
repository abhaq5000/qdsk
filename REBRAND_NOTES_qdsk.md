# qdsk — ملاحظات إعادة العلامة التجارية (مبني على RustDesk / AGPL-3.0)

هذا المستودع نسخة مُعاد تسميتها من RustDesk باسم **qdsk**، مع تعديل وظيفي واحد
(رقم ID مميّز لكل مستخدم Windows). **سيرفرات الربط تبقى سيرفرات RustDesk العامة**
(`rs-ny.rustdesk.com`) كما هو مطلوب — لم يُغيَّر أي شيء يخص السيرفر.

## التعديلات المُطبَّقة

### 1) العلامة التجارية (Rebranding)
| الملف | التغيير |
|---|---|
| `libs/hbb_common/src/config.rs` | `APP_NAME` = `"qdsk"` ، `ORG` = `"com.tatweersoft"` |
| `flutter/windows/runner/Runner.rc` | بيانات المنتج (ProductName/CompanyName/OriginalFilename…) → qdsk / Tatweersoft |

- `APP_NAME` يتحكّم تلقائيًا في: عنوان النوافذ، أيقونة شريط المهام، اسم الخدمة،
  مجلّد الإعدادات (`%APPDATA%\qdsk`)، أنابيب IPC، صندوق "حول". `main.cpp` يقرأ الاسم
  ديناميكيًا من المكتبة، فلا حاجة لتعديله.
- اسم ملف الـexe يبقى داخليًا `rustdesk.exe` (لأن `build.py` والمثبّت MSI يفترضانه)؛
  أعِد تسميته إلى `qdsk.exe` بعد التنزيل إن رغبت.
- **الأيقونة/الشعار**: لم تُستبدل (تحتاج ملف فنّي `.ico`). استبدل
  `flutter/windows/runner/resources/app_icon.ico` بأيقونتك قبل البناء إن أردت شعارًا خاصًّا.

### 2) التعديل الوظيفي: ID مميّز لكل مستخدم Windows
- المكان: `libs/hbb_common/src/config.rs` → دالة `get_auto_id()`.
- الأصل: يشتقّ الرقم من عنوان MAC فقط (ثابت للجهاز) ⇒ كل المستخدمين نفس الرقم.
- بعد التعديل (Windows فقط): الرقم = تجزئة SHA-256 لـ `MAC + اسم المستخدم` ⇒
  كل مستخدم Windows يحصل على رقم **مميّز وثابت**. سلوك Linux/macOS لم يتغيّر.

**نطاق العمل:**
- ✅ يعمل عند تشغيل qdsk **محمولًا/لكل مستخدم** (كل مستخدم في جلسته) — كل مستخدم رقم خاص.
- ⚠️ عند التثبيت كخدمة نظام (SYSTEM) يكون مسار الإعدادات مشتركًا (`ServiceProfiles\LocalService`)
  فيصير الرقم واحدًا للجهاز بحكم معمارية الخدمة. لِـ ID لكل مستخدم استخدم النسخة المحمولة.

### 3) دمج الـsubmodule (Vendoring)
- كان `libs/hbb_common` وحدة فرعية (submodule) تشير لريبو RustDesk، فتعديلاتنا لم تكن
  لتصل للبناء السحابي. تمّ دمج ملفاته مباشرةً في المستودع وحُذف `.gitmodules`، فأصبح
  المستودع مكتفيًا ذاتيًا وتصل تعديلاته إلى GitHub Actions.

## الدفع إلى GitHub وتشغيل البناء السحابي

> البناء المحلي على ويندوز غير عملي هنا (يتطلّب vcpkg + LLVM + محرّك Flutter المخصّص من
> RustDesk)، لذا نستخدم GitHub Actions الجاهزة في المستودع.

```bash
cd e:/rstdsk/client

# 1) أنشئ مستودعًا جديدًا على GitHub (مثال: qdsk) من واجهة GitHub، ثم:
git remote remove origin 2>/dev/null
git remote add origin https://github.com/<حسابك>/qdsk.git

# 2) اعتمد التعديلات ثم ادفعها
git add -A
git commit -m "Rebrand to qdsk; per-user Windows id; vendor hbb_common"
git push -u origin master
```

### تشغيل البناء وتنزيل الـexe
1. افتح المستودع على GitHub → تبويب **Actions**.
2. فعّل الـWorkflows إن طُلب، ثم اختر **"Flutter Nightly Build"** → **Run workflow**
   (تشغيل يدوي `workflow_dispatch`).
3. انتظر انتهاء مهمّة `x86_64-pc-windows-msvc` (قد تستغرق عشرات الدقائق).
4. من صفحة التشغيل، نزّل الـArtifact باسم **`rustdesk-unsigned-windows-x86_64`** —
   يحوي تطبيق qdsk لويندوز (غير موقَّع). خطوات التوقيع تُتخطّى تلقائيًا لغياب أسرار التوقيع.
5. (اختياري) أعِد تسمية `rustdesk.exe` بداخله إلى `qdsk.exe`.

> بديل: ادفع وسمًا بصيغة إصدار (مثل `git tag 1.5.0 && git push origin 1.5.0`) لتشغيل
> **"Flutter Tag Build"** الذي يبني كل المنصّات (Windows/Android/Linux/macOS) وينشئ Release.

## ملاحظة الترخيص
RustDesk تحت رخصة AGPL-3.0. إعادة التسمية والاستضافة الذاتية مسموحة بشرط إبقاء الكود
مفتوحًا لمستخدمي الخدمة وإزالة علامة "RustDesk" التجارية (وهو ما تمّ). حُفِظ إسناد المصدر في
`LegalCopyright` وهذا الملف.
