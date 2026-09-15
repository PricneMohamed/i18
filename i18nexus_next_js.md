# سجل المحادثة الكامل: إعداد وتكامل i18nexus مع Next.js (سؤال وجواب)

توثيق شامل لكامل جلسة العمل والمحادثة سؤالاً بإجابة، موضحاً المفاهيم، الأكواد، واستكشاف الأخطاء وإصلاحها (Troubleshooting) على كل من خادم وسيرفر Next.js App Router (Server & Client Components) وتكامل التدويل وتغيير اللغات.

---

### س1:
> **`I18NEXUS_API_KEY="9a8d9263-a264-42e4-b249-b9357121d486"`**  
> **`I18NEXUS_PAT="your_personal_access_token"` اضيف ايه؟**

#### الإجابة:
في مكان `I18NEXUS_PAT` تضع **Personal Access Token** الخاص بحسابك على منصة i18nexus، وليس مفتاح المشروع.

**خطوات استخراجه:**
1. ادخل إلى حسابك في موقع [i18nexus.com](https://i18nexus.com).
2. اضغط على صورة ملفك أو اسمك للدخول إلى **Profile Settings** أو **Account Settings**.
3. ابحث عن قسم **Personal Access Tokens** أو **API Tokens**.
4. اضغط على **Generate New Token**.
5. انسخ الرمز وضعه مكان `"your_personal_access_token"`.

**الفرق بينهما:**
* **`I18NEXUS_API_KEY`**: مفتاح خاص بالمشروع نفسه لتحديد أي مشروع يتم مزامنة ملفاته.
* **`I18NEXUS_PAT`**: رمز تحقق لحسابك الشخصي يُستخدم غالباً عند تنفيذ أوامر الـ CLI.

---

### س2:
> **`I18NEXUS_API_KEY` اجيبه منين؟**

#### الإجابة:
يُستخرج من داخل إعدادات المشروع نفسه:
1. ادخل إلى المشروع المطلوب من لوحة التحكم (Dashboard).
2. اضغط على **Settings** (⚙️).
3. انتقل إلى قسم **API & CLI** أو **General Settings**.
4. انسخ القيمة الموجودة في حقل **Project API Key** إلى ملف البيئة `.env`.

---

### س3:
> **ازاي اعمل مشروع ب i18؟**

#### الإجابة:
1. **إنشاء المشروع على المنصة:** إنشاء مشروع واختيار اللغة الأساسية واللغات المستهدفة ونسخ الـ API Key.
2. **تثبيت أداة الـ CLI وسحب الملفات:**
   ```bash
   npm install -g i18nexus-cli
   ```
   في ملف `.env`:
   ```env
   I18NEXUS_API_KEY="your_api_key_here"
   ```
   أمر التنزيل:
   ```bash
   i18nexus pull
   ```
3. **تهيئة مكتبة التدويل (نموذج React التقليدي):**
   ```bash
   npm install i18next react-i18next i18next-http-backend
   ```
   إنشاء ملف `i18n.js` واستدعاء `useTranslation` داخل المكونات.

---

### س4:
> **i18n.js في root؟ و بعدين؟**

#### الإجابة:
المكان الأفضل داخل `src/i18n.js` أو في الـ `root`.
الخطوات التالية:
1. **استيراده في Entry Point:** إضافة `import './i18n';` في أعلى ملف `src/main.jsx` أو `src/index.js`.
2. **التأكد من مسار الـ JSON:** وجود الملفات داخل المسار العام `public/locales/{lng}/{ns}.json`.
3. **الاستخدام بالمكونات:** استخدام `useTranslation` واستدعاء `i18n.changeLanguage('ar')`.

---

### س5:
> **بستخدم NEXTJS**

#### الإجابة:
Next.js يعتمد على الرندرة بالسيرفر (SSR / Server Components)، لذا يختلف الإعداد:
1. **تثبيت الحزم المناسبة:**
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
3. **محرك السيرفر `src/app/i18n/index.js`** لتهيئة الترجمة على الخادم.
4. **هيكلة المسارات:** وضع الصفحات داخل المجلد الديناميكي `src/app/[lng]/page.jsx`.

---

### س6:
> **خطأ: `React Hook "useTranslation" cannot be called in an async function.` في `page.jsx`**

#### الإجابة:
* **السبب:** قواعد React و ESLint تمنع تسمية أو استدعاء دوال تبدأ بـ `use` (Hooks) داخل دوال غير متزامنة (`async function`).
* **الحل:** إعادة تسمية الدالة المخصصة في `src/app/i18n/index.js` من `useTranslation` إلى `getTranslation` واستدعاؤها كدالة عادية:
  ```javascript
  const { t } = await getTranslation(lng, 'common');
  ```

---

### س7:
> **بعدين هل فيه middleware بيوجهني للغة معينة؟**

#### الإجابة:
نعم، الـ **Middleware** يفحص الرابط والكوكي و`Accept-Language` ويوجه الزائر تلقائياً (Redirect):
1. تثبيت الحزمة:
   ```bash
   npm install accept-language
   ```
2. إنشاء `src/middleware.js`:
   ```javascript
   import { NextResponse } from 'next/server';
   import acceptLanguage from 'accept-language';
   import { fallbackLng, languages } from './app/i18n/settings';

   acceptLanguage.languages(languages);

   export const config = {
     matcher: ['/((?!api|_next/static|_next/image|assets|images|favicon.ico|.*\\..*).*)'],
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
> **Build Error: `Can't resolve './src/app/i18n/settings'` داخل `./src/middleware.js`**

#### الإجابة:
* **السبب:** الملف موجود بالفعل داخل `src/`، فالمسار النسبي المكتوب يبحث عن `src/src/...`.
* **الحل:** تعديل مسار الاستيراد إلى:
  ```javascript
  import { fallbackLng, languages } from './app/i18n/settings';
  // أو عبر الـ alias:
  import { fallbackLng, languages } from '@/app/i18n/settings';
  ```

---

### س9:
> **المتصفح يظهر اسم المفتاح `meta_title` بدلاً من النص المترجم**

#### الإجابة:
يعني أن محرك الترجمة لم يعثر على القيمة للأسباب التالية:
1. عدم تطابق اسم الـ Namespace المستدعى مع اسم ملف الـ JSON.
2. خطأ في مسار قراءة الملفات في الباك إند.
3. المفتاح غير موجود بنفس حالة الأحرف داخل الملف.

---

### س10:
> **الملف يحتوي على `{ "meta_title": "أهلاً بكم في صفحتي الرئيسية" }` وما زال لا يترجم على `/en`**

#### الإجابة:
عند فتح `localhost:3000/en`، يبحث المحرك في `public/locales/en/` وليس `ar/`. تأكد من وضع المفتاح المقابل بالإنجليزية في ملف `en`، أو تجربة فتح الرابط المباشر `localhost:3000/ar`.

---

### س11:
> **Build Error: `server relative imports are not implemented yet` عند محاولة `import(...)` مسار ديناميكي**

#### الإجابة:
* **السبب:** بيئة بناء Next.js تمنع استيراد الـ JSON عبر مسارات ديناميكية على السيرفر.
* **الحل الجذري:** القراءة المباشرة من نظام التشغيل بواسطة `fs.readFileSync` و `path.join`:
  ```javascript
  // src/app/i18n/index.js
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
          return JSON.parse(fs.readFileSync(filePath, 'utf-8'));
        })
      )
      .init(getOptions(lng, ns));
    return i18nInstance;
  };

  export async function getTranslation(lng, ns, options = {}) {
    const i18nextInstance = await initI18next(lng, ns);
    await i18nextInstance.changeLanguage(lng);
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
> **ليه مش شغال رغم إنشاء الملفات؟ (ظهور اسم الملف كـ `Home.json`)**

#### الإجابة:
1. اسم الملف كان بحرف كبير `Home.json`، بينما الكود يستدعي `'common'`. تم تعديل الاستدعاء إلى:
   ```javascript
   const { t } = await getTranslation(lng, 'Home');
   ```
2. استبدال الـ dynamic import بحل `fs.readFileSync` لحل مشكلة السيرفر.

---

### س13:
> **النص مش بيتغير للعربي على الرابط `/ar` ويفضل ثابت إنجليزي**

#### الإجابة:
1. في **Next.js 15+** أصبحت الـ `params` عبارة عن **Promise**، وكان يجب عمل `await` لها:
   ```javascript
   const resolvedParams = await params;
   const lng = resolvedParams?.lng;
   ```
2. إضافة سطر تأكيد تغيير لغة الـ instance في الخادم:
   ```javascript
   await i18nextInstance.changeLanguage(lng);
   ```

---

### س14:
> **شرح تفصيلي لدالة تبديل اللغة:**
> ```javascript
> const switchLanguage = () => {
>   const segments = pathname.split('/');
>   segments[1] = targetLng;
>   const newPath = segments.join('/') || `/${targetLng}`;
>   router.push(newPath);
> };
> ```

#### الإجابة:
وظيفتها تبديل اللغة مع الحفاظ على الصفحة الحالية للزائر:
1. `pathname.split('/')`: تقسيم الرابط مثل `/ar/services` إلى مصفوفة: `['', 'ar', 'services']`.
2. `segments[1] = targetLng`: استبدال كود اللغة القديم بالجديد (مثلاً تحويل `ar` إلى `en`).
3. `segments.join('/')`: إعادة دمج عناصر المصفوفة بسلاش لتصبح المسار الجديد: `/en/services`.
4. `router.push(newPath)`: توجيه العميل للصفحة دون إعادة تحميل كاملة للموقع (Client-side Navigation).

---

### س15:
> **ازاي استخدم `t()` داخل مكونات العميل `"use client"` (مثل الـ Navbar)؟**

#### الإجابة:
مكونات العميل لا يمكنها قراءة الملفات عبر `fs`، بل تعتمد على عميل `react-i18next`.

1. **إنشاء ملف إعداد العميل `src/app/i18n/client.js`:**
   ```javascript
   'use client';
   import i18next from 'i18next';
   import { initReactI18next, useTranslation as useTranslationOrg } from 'react-i18next';
   import resourcesToBackend from 'i18next-resources-to-backend';
   import { getOptions } from './settings';
   import { useParams } from 'next/navigation';
   import { useEffect } from 'react';

   i18next
     .use(initReactI18next)
     .use(
       resourcesToBackend(
         (language, namespace) => import(`../../../public/locales/${language}/${namespace}.json`)
       )
     )
     .init({
       ...getOptions(),
       lng: undefined,
       detection: {
         order: ['path', 'htmlTag', 'cookie', 'navigator'],
       },
     });

   export function useTranslation(ns, options = {}) {
     const params = useParams();
     const lng = params?.lng || 'en';
     const ret = useTranslationOrg(ns, options);
     const { i18n } = ret;

     useEffect(() => {
       if (i18n.resolvedLanguage !== lng) {
         i18n.changeLanguage(lng);
       }
     }, [lng, i18n]);

     return ret;
   }
   ```

2. **الاستدعاء داخل مكون `Navbar.jsx`:**
   ```jsx
   "use client";
   import { useTranslation } from '@/app/i18n/client';

   export default function Navbar() {
     const { t } = useTranslation('Home');

     return (
       <header>
         <span>{t('brand_name')}</span>
         <button>{t('book_now')}</button>
       </header>
     );
   }
   ```
