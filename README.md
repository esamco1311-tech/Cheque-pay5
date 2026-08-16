# طباعة الشيكات - تطبيق ويب (PWA + تسجيل دخول)

تطبيق واحد-صفحة (Single HTML file) لطباعة شيكات بنك مصر، بنك القاهرة، البنك الأهلي المصري، والعقاري المصري العربي، مع:
- معاينة حية للشيك فوق خلفية الشيك الحقيقي لكل بنك
- تفقيط تلقائي للمبلغ بالعربي
- حسابات متعددة لكل بنك مع رصيد وترقيم شيكات تلقائي
- سجل عمليات / كشف حساب قابل للتصدير Excel أو PDF
- بيان تفصيلي (خطاب) قابل للطباعة منفصل، مع تحقق تلقائي من مطابقته لمبلغ الشيك
- تسجيل تلقائي (Autocomplete) للحقول المتكررة
- **تسجيل دخول بكلمة سر بس (من غير إيميل أو تليفون) + مزامنة البيانات بين أي جهاز**
- PWA قابل للتثبيت كتطبيق على الموبايل/الكمبيوتر

## الملفات
- `index.html` — التطبيق نفسه
- `manifest.json` — بيانات تثبيت التطبيق (PWA)
- `sw.js` — Service Worker بسيط (offline caching)
- `icons/icon-192.png` و `icons/icon-512.png` — أيقونات التطبيق

**لازم كل الملفات دي تترفع مع بعض في نفس المجلد (root) بالظبط زي ما هما.**

---

## ⚠️ خطوة إلزامية قبل الرفع: تفعيل تسجيل الدخول والمزامنة (Firebase)

من غير الخطوة دي، البرنامج هيشتغل عادي بس **البيانات هتفضل محفوظة في المتصفح بتاعك بس** (زي الأول)، ومش هيظهر تسجيل دخول. لو عايز فعلاً تسجيل دخول بكلمة سر + مزامنة البيانات بين أي جهاز، لازم تعمل مشروع Firebase مجاني (بياخد ٥ دقايق، من غير أي فلوس أو بطاقة ائتمان):

### ١. اعمل مشروع Firebase
1. روح على https://console.firebase.google.com
2. سجّل دخول بحساب Google عادي.
3. دوس **"Add project" / "إضافة مشروع"**، اكتب أي اسم (مثلاً `cheque-app`)، وكمّل الخطوات (ينفع تلغي Google Analytics لو عايز، مش ضروري).

### ٢. فعّل تسجيل الدخول (Authentication)
1. من القائمة الجانبية: **Build → Authentication**.
2. دوس **"Get started"**.
3. من تبويب **"Sign-in method"**، اختار **Email/Password** وفعّله (Enable) واحفظ.

### ٣. فعّل قاعدة البيانات (Firestore)
1. من القائمة الجانبية: **Build → Firestore Database**.
2. دوس **"Create database"**.
3. اختار **"Start in production mode"** ودوس Next، واختار أي موقع سيرفر قريب (مثلاً `eur3` أو `me-west1`) ودوس **Enable**.
4. بعد ما يتعمل، روح تبويب **"Rules"** فوق، وامسح اللي موجود وحط بدله:
   ```
   rules_version = '2';
   service cloud.firestore {
     match /databases/{database}/documents {
       match /users/{userId}/{document=**} {
         allow read, write: if request.auth != null && request.auth.uid == userId;
       }
     }
   }
   ```
5. دوس **"Publish"**.

### ٤. هات بيانات المشروع (Config)
1. من القائمة الجانبية دوس على أيقونة الترس ⚙️ بجوار "Project Overview" → **Project settings**.
2. انزل لقسم **"Your apps"**، دوس على أيقونة الويب `</>`.
3. اكتب اسم مستعار للتطبيق ودوس **"Register app"** (متحتاجش Firebase Hosting، سيبها فاضية).
4. هيديك كود فيه object اسمه `firebaseConfig` فيه قيم زي دي:
   ```js
   const firebaseConfig = {
     apiKey: "AIza...",
     authDomain: "cheque-app-xxxx.firebaseapp.com",
     projectId: "cheque-app-xxxx",
     storageBucket: "cheque-app-xxxx.appspot.com",
     messagingSenderId: "123456789",
     appId: "1:123456789:web:abcdef"
   };
   ```

### ٥. حط القيم دي في ملف `index.html`
1. افتح `index.html` بأي محرر نصوص (Notepad، VS Code، إلخ).
2. دور على السطر ده تقريبًا في أول الكود (Ctrl+F وابحث عن `FIREBASE_CONFIG`):
   ```js
   const FIREBASE_CONFIG = {
     apiKey: "PASTE_YOUR_API_KEY",
     authDomain: "PASTE_YOUR_PROJECT.firebaseapp.com",
     projectId: "PASTE_YOUR_PROJECT_ID",
     storageBucket: "PASTE_YOUR_PROJECT.appspot.com",
     messagingSenderId: "PASTE_YOUR_SENDER_ID",
     appId: "PASTE_YOUR_APP_ID"
   };
   ```
3. استبدل كل قيمة بالقيمة المقابلة اللي نسختها من Firebase في الخطوة اللي فاتت.
4. احفظ الملف.

### ٦. أول استخدام بعد الرفع
- افتح رابط الموقع، هيظهرلك شاشة تسجيل الدخول.
- دوس على **"أول مرة تستخدم البرنامج؟ اضغط هنا لتعيين كلمة سر"**.
- اكتب كلمة سر (٦ أحرف على الأقل) وأكّدها واحفظ.
- من بعد كده، أي جهاز تفتح بيه نفس الرابط وتسجل دخول بنفس كلمة السر، هتلاقي نفس البيانات (الحسابات، السجل، إلخ) متزامنة تلقائيًا.

⚠️ **مفيش استرجاع تلقائي لكلمة السر** لأنه مفيش إيميل حقيقي مربوط بالحساب (بالتصميم، عشان تسجيل الدخول يبقى بكلمة سر بس). لو نسيت كلمة السر، الحل الوحيد إنك تدخل على Firebase Console → Authentication → تمسح المستخدم وتعمل "تعيين كلمة سر" تاني من الصفر (وقتها هتفقد الوصول للبيانات القديمة إلا لو عندك نسخة احتياطية Excel من السجل).

---

---

## ⚠️ تحديث مهم: إصلاح مشكلة "Fix the links to your icons"

لو ظهرلك في PWABuilder تحذيرات حمرا عن الأيقونات ("Fix the links to your icons" / "Fix the icon types")، السبب غالبًا إن مجلد `icons/` القديم ماترفعش صح (مشكلة شائعة عند رفع مجلدات فرعية من الموبايل).

**الحل:** النسخة دي بقت الأيقونات فيها في **نفس المجلد الرئيسي** (زي `index.html` بالظبط) من غير أي مجلد فرعي: `icon-192.png`, `icon-512.png`, `icon-maskable-192.png`, `icon-maskable-512.png`.

**لازم:**
1. لو رفعت مجلد `icons/` قبل كده في الـ Repository، **امسحه بالكامل**.
2. ارفع الملفات الأربعة الجديدة دي مباشرة في المجلد الرئيسي (Root) جنب `index.html` و`manifest.json`.
3. استنى دقيقة لحد ما GitHub Pages يحدّث النسخة، وبعدين جرب PWABuilder تاني (ممكن يحتاج "Rescan" أو تمسح الكاش بتاعه بزيادة `?v=2` آخر الرابط).

## طريقة النشر على GitHub Pages

1. اعمل Repository جديد على GitHub (public).
2. ارفع كل الملفات مع بعض (`index.html` بعد ما تحط فيه بيانات Firebase، `manifest.json`, `sw.js`, ومجلد `icons` بكل اللي جواه).
3. من **Settings → Pages**: اختار **Source: Deploy from a branch**، **Branch: main**، المجلد **/(root)**، واحفظ.
4. بعد دقيقة أو اتنين، هيديك رابط `https://<username>.github.io/<repo-name>/`.

## استخدام PWABuilder
بعد النشر، روح على pwabuilder.com وحط رابط الموقع بتاعك — المفروض يلاقي الـ manifest والـ service worker تلقائيًا.

## ملاحظة عن الأمان
النظام ده مناسب لاستخدام شخصي/مكتب صغير (شخص واحد أو فريق صغير بيستخدم نفس كلمة السر). مش بديل عن نظام حسابات بنكي حقيقي أو نظام محاسبة معتمد.
