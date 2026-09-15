# سجل المحادثة الكامل: إعداد وتكامل i18nexus مع Next.js (سؤال وجواب)

توثيق شامل لكامل جلسة العمل والمحادثة سؤالاً بإجابة، موضحاً المفاهيم، الأكواد، وسيناريوهات استكشاف الأخطاء وإصلاحها (Troubleshooting) حتى تشغيل التدويل بالكامل بنجاح.

---

### س1:
> **`I18NEXUS_API_KEY="9a8d9263-a264-42e4-b249-b9357121d486"`**  
> **`I18NEXUS_PAT="your_personal_access_token"` اضيف ايه؟**

#### الإجابة:
في مكان `I18NEXUS_PAT` تضع **Personal Access Token** الخاص بحسابك على منصة i18nexus، وليس مفتاح المشروع.

**خطوات استخراجه من حسابك:**
1. ادخل إلى حسابك في موقع [i18nexus.com](https://i18nexus.com).
2. اضغط على صورتك الشخصية أو اسمك أعلى يمين الشاشة للدخول إلى **Profile Settings** أو **Account Settings**.
3. ابحث عن قسم **Personal Access Tokens** أو **API Tokens**.
4. اضغط على **Generate New Token** (أو Create Token).
5. انسخ الرمز الناتج وضعه مكان `"your_personal_access_token"`.

**الفرق بين المتغيرين:**
* **`I18NEXUS_API_KEY`**: مفتاح خاص بالمشروع نفسه (Project API Key) ليحدد أي مشروع يتم سحب أو رفع الترجمات منه.
* **`I18NEXUS_PAT`**: رمز تحقق خاص بحسابك الشخصي يُستخدم غالباً عند تنفيذ أوامر الـ CLI (مثل سحب ملفات الترجمة تلقائياً أو مزامنتها عبر سطر الأوامر).

---

### س2:
> **`I18NEXUS_API_KEY` اجيبه منين؟**

#### الإجابة:
الـ **API Key** الخاص بالمشروع يُستخرج من داخل إعدادات المشروع نفسه:
1. افتح موقع [i18nexus.com](https://i18nexus.com) وسجّل الدخول.
2. ادخل إلى **المشروع** المطلوب من لوحة التحكم (Dashboard).
3. اضغط على **Settings** (أيقونة الترس ⚙️) الخاصة بالمشروع.
4. انتقل إلى قسم **API & CLI** أو **General Settings**.
5. ستجد حقل **Project API Key**، اضغط على زر **Copy** وانسخه إلى ملف بيئة العمل `.env`.

---

### س3:
> **ازاي اعمل مشروع ب i18؟**

#### الإجابة:
خطوات إنشاء وربط مشروع يعتمد على `i18nexus` مع مكتبة التدويل `i18next`:

1. **إنشاء المشروع على المنصة:**
   * أنشئ حساباً وادخل على i18nexus واضغط **Create Project**.
   * حدد اللغة الأساسية (مثل الإنجليزية أو العربية) واللغات المستهدفة.
   * انسخ مفتاح المشروع `Project API Key` من الإعدادات.

2. **تثبيت أداة الـ CLI وسحب الملفات:**
   ```bash
   npm install -g i18nexus-cli
   ```
   أنشئ ملف `.env` في المسار الرئيسي:
   ```env
   I18NEXUS_API_KEY="your_api_key_here"
   ```
   ثم شغّل أمر السحب:
   ```bash
   i18nexus pull
   ```
   *تقوم الأداة بتنزيل ملفات الـ JSON داخل مجلد `public/locales`.*

3. **إعداد واستخدام مكتبة الترجمة (نموذج React التقليدي):**
   ```bash
   npm install i18next react-i18next i18next-http-backend
   ```
   إنشاء ملف تهيئة `i18n.js`:
   ```javascript
   import i18n from 'i18next';
   import { initReactI18next } from 'react-i18next';
   import HttpBackend from 'i18next-http-backend';

   i18n
     .use(HttpBackend)
     .use(initReactI18next)
     .init({
       fallbackLng: 'en',
       lng: 'en',
       backend: {
         loadPath: '/locales/{{lng}}/{{ns}}.json',
       },
       interpolation: {
         escapeValue: false,
       },
     });

   export default i18n;
   ```

---

### س4:
> **i18n.js في root؟ و بعدين؟**

#### الإجابة:
مكانه الأفضل والأنظم يكون داخل مجلد **`src/`** (مثلاً: `src/i18n.js`) أو في الـ `root`. الخطوات التالية بعد إنشائه:

1. **استيراد الملف في نقطة البداية (Entry Point):**
   * في React (Vite): أضف `import './i18n';` في أعلى ملف `src/main.jsx`.
   * في React (CRA): أضف `import './i18n';` في أعلى ملف `src/index.js`.
2. **التأكد من مسار الملفات:**
   التأكد من أن ملفات الـ JSON موجودة داخل مسار متاح للتحميل عبر الويب مثل:
   ```text
   public/locales/en/default.json
   public/locales/ar/default.json
   ```
3. **استخدام الترجمة داخل المكونات:**
   استيراد واستخدام `useTranslation` وتغيير اللغة:
   ```jsx
   import { useTranslation } from 'react-i18next';

   function App() {
     const { t, i18n } = useTranslation();
     return (
       <div>
         <h1>{t('welcome_message')}</h1>
         <button onClick={() => { i18n.changeLanguage('ar'); document.dir = 'rtl'; }}>العربية</button>
         <button onClick={() => { i18n.changeLanguage('en'); document.dir = 'ltr'; }}>English</button>
       </div>
     );
   }
   ```

---

### س5:
> **بستخدم NEXTJS**

#### الإجابة:
في **Next.js (App Router)**، يختلف الأمر عن React التقليدي لأن Next.js يعتمد الرندرة بالسيرفر (SSR / Server Components).

**الخطوات المعيارية لـ Next.js App Router:**
1. **تثبيت الحزم اللازمة:**
   ```bash
   npm install i18next react-i18next i18next-resources-to-backend
   ```
2. **ملف الإعدادات `src/app/i18n/settings.js`:**
   ```javascript
   export const fallbackLng = 'en';
   export const languages = [fallbackLng, 'ar'];
   export const defaultNS = 'common';

   export function getOptions(lng = fallbackLng, ns = defaultNS) {
     return {
       supportedLngs: languages,
       fallbackLng,
       lng,
       fallbackNS: defaultNS,
       defaultNS,
       ns,
     };
   }
   ```
3. **محرك السيرفر `src/app/i18n/index.js`:**
   تهيئة مكتبة i18next للتعامل مع مكونات السيرفر.
4. **هيكلة مسارات Next.js الديناميكية:**
   وضع المسار الأساسي داخل مجلد `src/app/[lng]/page.jsx` و `src/app/[lng]/layout.jsx`.

---

### س6:
> *(إرسال لقطة شاشة توضح خطأ في الكود)*  
> **`React Hook "useTranslation" cannot be called in an async function.` في ملف `page.jsx`**

#### الإجابة:
* **سبب الخطأ:** محرك قواعد React / ESLint يمنع تسمية أو استدعاء الدوال التي تبدأ بـ `use` (Hooks) داخل دوال غير متزامنة (`async function`).
* **الحل الجذري:**
  1. إعادة تسمية الدالة المخصصة في `src/app/i18n/index.js` من `useTranslation` إلى `getTranslation`.
  2. في صفحة `page.jsx`، استيراد واستدعاء الاسم الجديد:
     ```jsx
     import { getTranslation } from '../i18n';

     export default async function HomePage({ params: { lng } }) {
       const { t } = await getTranslation(lng, 'common');
       // ...
     }
     ```

---

### س7:
> **بعدين هل فيه middleware بيوجهني للغة معينة؟**

#### الإجابة:
نعم، الـ **Middleware** هو المسؤول الأساسي في Next.js عن:
1. فحص مسار الرابط (Pathname) والتأكد هل يحتوي على لغة مدعومة أم لا.
2. فحص تفضيل لغة المستخدم عبر (Cookies) ثم رأس لغة المتصفح (`Accept-Language`).
3. توجيه الزائر تلقائياً (Redirect) إلى الرابط المناسب مثل `/ar` أو `/en`.

**طريقة الإعداد:**
1. تثبيت مكتبة فحص لغة الطلب:
   ```bash
   npm install accept-language
   ```
2. إنشاء ملف الـ Middleware:
   ```javascript
   import { NextResponse } from 'next/server';
   import acceptLanguage from 'accept-language';
   import { fallbackLng, languages } from './app/i18n/settings';

   acceptLanguage.languages(languages);

   export const config = {
     matcher: ['/((?!api|_next/static|_next/image|assets|favicon.ico|sw.js).*)'],
   };

   export function middleware(req) {
     let lng;
     if (req.cookies.has('i18next')) lng = acceptLanguage.get(req.cookies.get('i18next').value);
     if (!lng) lng = acceptLanguage.get(req.headers.get('Accept-Language'));
     if (!lng) lng = fallbackLng;

     if (
       !languages.some((loc) => req.nextUrl.pathname.startsWith(`/${loc}`)) &&
       !req.nextUrl.pathname.startsWith('/_next')
     ) {
       return NextResponse.redirect(new URL(`/${lng}${req.nextUrl.pathname}`, req.url));
     }

     const response = NextResponse.next();
     const currentLng = languages.find((loc) => req.nextUrl.pathname.startsWith(`/${loc}`));
     if (currentLng) {
       response.cookies.set('i18next', currentLng);
     }
     return response;
   }
   ```

---

### س8:
> **Build Error**  
> **`Module not found: Can't resolve './src/app/i18n/settings'` داخل `./src/middleware.js (3:1)`**

#### الإجابة:
* **سبب الخطأ:** ملف `middleware.js` تم وضعه داخل مجلد `src/`، لذلك كتابة `./src/...` تبحث عن مجلد `src` متكرر وغير موجود.
* **الحل:** تعديل سطر الاستيراد داخل `src/middleware.js` إلى المسار النسبي الصحيح:
  ```javascript
  import { fallbackLng, languages } from './app/i18n/settings';
  ```
  أو عبر مسار الـ Alias:
  ```javascript
  import { fallbackLng, languages } from '@/app/i18n/settings';
  ```

---

### س9:
> *(إرسال لقطة شاشة من المتصفح تظهر ظهور النص `meta_title` في الصفحة بدلاً من الترجمة الحقيقية)*

#### الإجابة:
ظهور اسم المفتاح مباشرة كـ `meta_title` يعني أن محرك الترجمة فشل في جلب القيمة المقابلة له من ملف الـ JSON، للأسباب التالية:
1. **عدم تطابق اسم الـ Namespace:** الاستدعاء يطلب `'common'` بينما الملف الفعلي قد يحمل اسماً آخر (مثل `Home.json` أو `default.json`).
2. **فشل مسار الـ dynamic import:** ملف الـ JSON لا يتم قراءته من المسار الصحيح.
3. **المفتاح غير موجود** أو غير مكتوب بنفس حالة الأحرف بالضبط في ملف الـ JSON المستهدف.

---

### س10:
> **`{ "meta_title": "أهلاً بكم في صفحتي الرئيسية" }`**

#### الإجابة:
* إذا كان هذا هو محتوى ملفك، فتأكد أن الرابط المطلوب يطابق اللغة؛ عند فتح `localhost:3000/en`، سيبحث التطبيق داخل ملف الإنجليزية `locales/en/...` وليس ملف العربية.
* يجب التأكد من تطابق اسم الملف مع اسم الـ Namespace الممرر في دالة `getTranslation(lng, '...')`.
* تجربة الدخول المباشر إلى `localhost:3000/ar` للتحقق من قراءة القيمة العربية.

---

### س11:
> **Build Error**  
> **`Module not found: Can't resolve './locales/' <dynamic> '/' <dynamic> '.json'`**  
> **`server relative imports are not implemented yet. Please try an import relative to the file you are importing from.`**

#### الإجابة:
* **سبب الخطأ:** في Next.js (خاصة مع بيئات البناء الحديثة مثل Turbopack وWebpack Server Targets)، لا يُدعم استيراد المسارات الديناميكية المتغيرة `import(...)` من مجلد `public` مباشرة في السيرفر.
* **الحل الجذري:** استبدال الـ Dynamic Import بالقراءة المباشرة من نظام الملفات عبر مكتبة `fs` و `path` المدمجتين في Node.js داخل ملف `src/app/i18n/index.js`:
  ```javascript
  import { createInstance } from 'i18next';
  import { initReactI18next } from 'react-i18next/initReactI18next';
  import resourcesToBackend from 'i18next-resources-to-backend';
  import fs from 'fs';
  import path from 'path';
  import { getOptions } from './settings';

  const initI18next = async (lng, ns) => {
    const i18nInstance = createInstance();
    await i18nInstance
      .use(initReactI18next)
      .use(
        resourcesToBackend((language, namespace) => {
          const filePath = path.join(
            process.cwd(),
            'public',
            'locales',
            language,
            `${namespace}.json`
          );
          const fileContent = fs.readFileSync(filePath, 'utf-8');
          return JSON.parse(fileContent);
        })
      )
      .init(getOptions(lng, ns));
    return i18nInstance;
  };

  export async function getTranslation(lng, ns, options = {}) {
    const i18nextInstance = await initI18next(lng, ns);
    return {
      t: i18nextInstance.getFixedT(
        lng,
        Array.isArray(ns) ? ns[0] : ns,
        options.keyPrefix
      ),
      i18n: i18nextInstance,
    };
  }
  ```

---

### س12:
> *(إرسال لقطة شاشة توضح مسار الملفات: الملف اسمه `Home.json` مع وجود خط أحمر على الـ dynamic import في السطر 14)*  
> **ليه مش شغال؟**

#### الإجابة:
المشكلة كانت محددة بوضوح في صورتين:
1. **اسم الملف الفعلي هو `Home.json` (بحرف H كبير):** بينما كان الكود يستدعي `'common'`.
   * **الحل:** تعديل الاستدعاء في `page.jsx` إلى:
     ```jsx
     const { t } = await getTranslation(lng, 'Home');
     ```
2. **السطر 14 (`import(...)`):** به خطأ استيراد نسبي، ويجب تبديله فوراً بحل `fs.readFileSync` المذكور أعلاه لتجاوز قيود Next.js.
3. تعديل `defaultNS` في `src/app/i18n/settings.js` إلى `'Home'`.

---

### س13:
> *(إرسال لقطة شاشة تظهر فتح الرابط `localhost:3000/ar` لكن النص يظهر بالإنجليزية `Welcome to my home page`)*  
> **مش بتتغير للعربي**

#### الإجابة:
السبب وراء ثبات اللغة على الإنجليزية يعود لنقطتين رئيسيتين:
1. **`params` أصبحت Promise في إصدارات Next.js الحديثة (Next.js 15+):**  
   عدم عمل `await` لـ `params` جعل قيمة `lng` دائماً `undefined`، وبالتالي كان المحرك يعود للغة الافتراضية (English fallback).
2. **مثيل الترجمة يحتاج لـ `changeLanguage` صراحة بالسيرفر.**

**الحل المطبق:**
* في `src/app/[lng]/page.jsx`:
  ```jsx
  import { getTranslation } from '../i18n';
  import Link from 'next/link';

  export default async function HomePage({ params }) {
    // فك الـ Promise لضمان استخراج كود اللغة
    const resolvedParams = await params;
    const lng = resolvedParams?.lng;

    const { t } = await getTranslation(lng, 'Home');

    return (
      <main style={{ padding: '2rem' }}>
        <h1>{t('meta_title')}</h1>

        <div style={{ marginTop: '1rem', display: 'flex', gap: '1rem' }}>
          <Link href="/ar">العربية</Link>
          <Link href="/en">English</Link>
        </div>
      </main>
    );
  }
  ```
* في `src/app/i18n/index.js`، إضافة سطر تأكيد اللغة:
  ```javascript
  export async function getTranslation(lng, ns, options = {}) {
    const i18nextInstance = await initI18next(lng, ns);
    await i18nextInstance.changeLanguage(lng); // تأكيد ضبط اللغة للطلب الحالي

    return {
      t: i18nextInstance.getFixedT(
        lng,
        Array.isArray(ns) ? ns[0] : ns,
        options.keyPrefix
      ),
      i18n: i18nextInstance,
    };
  }
  ```

---

### س14:
> **اشتغلت، حطلي كل المعلومات والمشاكل الي حصلت معايا وكل الشات في ملف md**

#### النتيجة:
تم توثيق كل خطوة والمفاهيم التقنية وحلول الأخطاء بالكامل بصيغة سؤال وجواب مفصلة لتكون دليلاً تقنياً ومرجعاً شاملاً للمشروع.