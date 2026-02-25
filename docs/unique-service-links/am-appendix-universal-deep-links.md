Հավելված․ Hartak.am-ի համար Mobile Universal Links և App Links
===============================================================

**Ինտեգրման փաստաթուղթ – Mobile հավելված** \
Փետրվար 2026 v1.0

## Բովանդակություն

- 1. Շրջանակ
- 2. Ծառայության հասանելիության սցենարներ
- 3. Բջջային հավելվածների ընդհանուր պայմանագիր
- 4. iOS – Universal Links
  - 4.1. Պահանջներ
  - 4.2. AASA ֆայլը ծառայություն մատուցողի հոստում
  - 4.3. iOS հավելվածի կարգավորում (Xcode)
  - 4.4. Universal Links-ի մշակում հավելվածում
- 5. Android – App Links
  - 5.1. Պահանջներ
  - 5.2. Digital Asset Links ֆայլը ծառայություն մատուցողի հոստում
  - 5.3. AndroidManifest – HTTP(S) deep links (App Links)
  - 5.4. Հղման մշակում Activity-ում
- 6. Նույնականացման փոխանցում բջջային հավելվածներին
  - 6.1. Կարո՞ղ է Hartak/YesEm նույնականացումը ուղղակի փոխանցվել բջջային հավելվածին
  - 6.2. Առաջարկվող անվտանգ մոտեցումներ
- 7. Թեստավորման ստուգաթերթ

## 1. Շրջանակ

Այս հավելվածը նկարագրում է, թե **ինչպես կարող են բջջային հավելվածները բացվել Hartak.am-ից եկող հղումներով**՝ օգտագործելով ժամանակակից HTTPS հղումներ։

- **Կանոնիկ HTTPS եզակի ծառայության հղումներ** (ինչպես սահմանված է հիմնական փաստաթղթում)։

Նպատակն այն է, որ երբ օգտատերը սեղմում է այս հղումներից մեկը, և պաշտոնական բջջային հավելվածը տեղադրված է, հավելվածը **բացվի ճիշտ էկրանով**։ Հակառակ դեպքում հղումը պետք է սահուն փոխանցվի բրաուզերին և շարունակի սովորական վեբ հոսքը YesEm-ի միջոցով։

Օրինակներում ենթադրում ենք, որ՝

- **Ծառայություն մատուցողի հիմնական URL**՝ `https://serviceportal.am`
- **Եզակի ծառայության հղման ձևաչափ**՝ `https://serviceportal.am/services/{serviceId}`

Որտեղ՝

- `serviceId`-ը Ազգային ծառայությունների կատալոգի նույնացուցիչն է (տես [հիմնական փաստաթուղթը](./am.md))։

Այս դոմենները, սխեմաները և ID-ները հարմարեցրեք ձեր իրական միջավայրին։

Կարևոր կանոն․ դոմենի վավերացման ֆայլերը պետք է տեղադրվեն **ծառայություն մատուցողի դոմենների տակ**, ոչ թե `hartak.am`-ում։

Մոբայլ պլատֆորմների յուրահատկություններ՝

- **iOS** վավերացման համար օգտագործում է միայն `apple-app-site-association` (AASA)։
- **Android** վավերացման համար օգտագործում է միայն `/.well-known/assetlinks.json` (Digital Asset Links)։

## 2. Ծառայության հասանելիության սցենարներ

Ծառայություն մատուցողները կարող են աշխատել հետևյալ մոդելներից մեկով՝

1. **Միայն վեբ**
   - HTTPS հղումը բացում է ծառայության վեբ տաբերակը։
   - Օգտատերը շարունակում է YesEm-ով և հետո անցնում է մատուցվող վեբ ծառայություն էջ։
   - App Link կարգավորում չի պահանջվում։

2. **Միայն բջջային հավելված** (iOS, Android կամ երկուսն էլ)
   - Պահպանեք Hartak-ից եկող նույն կանոնիկ HTTPS հղման ձևաչափը (`/services/{serviceId}`)։
   - Եթե հավելվածը տեղադրված է և դոմենի վավերացումը կարգավորված է, հավելվածը բացվում է անմիջապես։
   - iOS-ի համար պետք է AASA, Android-ի համար՝ `assetlinks.json`։
   - Եթե հավելվածը տեղադրված չէ, բացեք ծառայություն մատուցողի կողմից վերահսկվող վեբ fallback էջ, որը բացատրում է՝
     - ինչպես տեղադրել հավելվածը, և/կամ
     - ինչպես շարունակել բրաուզերով, եթե կա ծառայության վեբ տարբերակ։

3. **Վեբ և բջջային**
   - Պահպանեք մեկ կանոնիկ HTTPS հղում (`/services/{serviceId}`) որպես հանրային մուտքի կետ։
   - Տեղադրված հավելված ունեցող օգտատերերը անմիջապես ուղղորդվում են native էկրաններ Universal Links/App Links միջոցով։
   - Հավելված չունեցող օգտատերերը շարունակում են բրաուզերով՝ YesEm և վեբ տարբերակ։

## 3. Բջջային հավելվածների ընդհանուր պայմանագիր

Վեբի և բջջայինի միջև միասնական պայմանագիր պահպանելու համար՝

- **Մեկ հղման քաղաքականություն**․ Hartak-ը պահում և հրապարակում է միայն մեկ կանոնիկ հղման ձևաչափ՝ `/services/{serviceId}`։
- Բջջային հավելվածները պետք է՝
  - ընդունեն `/services/{serviceId}` ձևաչափի կանոնիկ HTTPS հղումները,
  - կարողանան կամ parse անել `serviceId`, կամ ամբողջ URL-ը ուղարկել backend և այնտեղ լուծել routing/ownership-ը,
  - ցանկության դեպքում օգտագործեն **լրացուցիչ query պարամետրեր** (օր. `?source=hartak`) միայն անալիտիկայի համար,
  - անհայտ կամ չաջակցվող հղման դեպքում բացեն ընդհանուր էկրան կամ սահուն փոխանցեն բրաուզերին։

## 4. iOS – Universal Links

### 4.1. Պահանջներ

- iOS 9+։
- Ծառայություն մատուցողի դոմենը (օր. `serviceportal.am`) պետք է HTTPS-ով հոսթի **`apple-app-site-association` (AASA)** ֆայլ։
- iOS հավելվածում պետք է միացված լինի **Associated Domains** հնարավորությունը նույն դոմենի համար։
- Android-ի `assetlinks.json`-ը iOS-ում չի օգտագործվում։

### 4.2. AASA ֆայլը ծառայություն մատուցողի հոստում

`apple-app-site-association` ֆայլը տեղադրեք ծառայություն մատուցողի հոստում՝

`https://serviceportal.am/apple-app-site-association`
  - Առանց ֆայլի ընդլայնման։
  - Content-Type՝ `application/json`։
  - Պետք է սպասարկվի այն նույն ծառայություն մատուցողի դոմենից, որը առկա է հղման մեջ։

Օրինակ բովանդակություն՝

```json
{
  "applinks": {
    "apps": [],
    "details": [
      {
        "appID": "ABCDE12345.com.example.taxapp",
        "paths": [
          "/services/*"
        ]
      }
    ]
  }
}
```

- **`appID`** = `<TEAM_ID>.<BUNDLE_ID>`, օրինակ՝ `ABCDE12345.com.example.taxapp`։
- **`paths`**-ը սահմանում է, թե որ URL-ները բացվեն հավելվածում․ այստեղ նպատակաուղղվում ենք `/services/` կանոնիկ հղումներին։

### 4.3. iOS հավելվածի կարգավորում (Xcode)

1. Բացեք iOS հավելվածը Xcode-ում։
2. Ընտրեք հավելվածի **target → Signing & Capabilities → + Capability → Associated Domains**։
3. Ավելացրեք՝

   - `applinks:serviceportal.am`

4. Վերակառուցեք (rebuild) և տեղադրեք հավելվածը սարքում։

### 4.4. Universal Links-ի մշակում հավելվածում

Իրականացման մանրամասների համար օգտվեք Apple-ի պաշտոնական փաստաթղթերից՝

- Supporting Universal Links in your app:  
  <a href="https://developer.apple.com/documentation/xcode/supporting-universal-links-in-your-app" target="_blank" rel="noopener noreferrer">https://developer.apple.com/documentation/xcode/supporting-universal-links-in-your-app</a>
- `UISceneDelegate` continuation API (`scene(_:continue:)`):  
  <a href="https://developer.apple.com/documentation/uikit/uiscenedelegate/scene(_:continue:)" target="_blank" rel="noopener noreferrer">https://developer.apple.com/documentation/uikit/uiscenedelegate/scene(_:continue:)</a>

## 5. Android – App Links

### 5.1. Պահանջներ

- Android 6.0+ (App Links-ի ավելի լավ աշխատանքի համար)։
- Ծառայություն մատուցողի դոմենը պետք է հասանելի լինի HTTPS-ով։
- Վավերացված App Links-ի համար հոսթեք **Digital Asset Links** ֆայլը `https://serviceportal.am/.well-known/assetlinks.json` հասցեում։
- iOS AASA-ն Android-ում չի օգտագործվում։

### 5.2. Digital Asset Links ֆայլը ծառայություն մատուցողի հոստում

`assetlinks.json` ֆայլը տեղադրեք այստեղ՝ `https://serviceportal.am/.well-known/assetlinks.json`  
Այն պետք է սպասարկվի նույն ծառայություն մատուցողի դոմենից, որն օգտագործվում է հղումներում։

Օրինակ բովանդակություն՝

```json
[
  {
    "relation": ["delegate_permission/common.handle_all_urls"],
    "target": {
      "namespace": "android_app",
      "package_name": "com.example.taxapp",
      "sha256_cert_fingerprints": [
        "12:34:56:78:90:AB:CD:EF:12:34:56:78:90:AB:CD:EF:12:34:56:78:90:AB:CD:EF:12:34:56:78:90:AB:CD:EF"
      ]
    }
  }
]
```

### 5.3. AndroidManifest – HTTP(S) deep links (App Links)

Ձեր հավելվածի `AndroidManifest.xml`-ում հայտարարեք `Activity`, որը կարող է մշակել եզակի ծառայության հղումները՝

```xml
<!-- AndroidManifest.xml -->
<manifest package="com.example.serviceapp"
    xmlns:android="http://schemas.android.com/apk/res/android">

    <application
        android:label="Service App"
        android:icon="@mipmap/ic_launcher">

        <activity
            android:name=".ServiceEntryActivity"
            android:exported="true">

            <!-- App Link for unique service links -->
            <intent-filter android:autoVerify="true">
                <action android:name="android.intent.action.VIEW" />

                <category android:name="android.intent.category.DEFAULT" />
                <category android:name="android.intent.category.BROWSABLE" />

                <data
                    android:scheme="https"
                    android:host="serviceportal.am"
                    android:pathPrefix="/services" />
            </intent-filter>

        </activity>

    </application>
</manifest>
```

Սա Android-ին թույլ է տալիս հավելվածը առաջարկել որպես handler, օրինակ՝

- `https://serviceportal.am/services/12345`

Եթե App Links-ի վավերացումը ճիշտ է կարգավորված, Android-ը կարող է բացել հավելվածը անմիջապես՝ առանց chooser dialog-ի։

### 5.4. Հղման մշակում Activity-ում

Իրականացման մանրամասների համար օգտվեք Android-ի պաշտոնական փաստաթղթերից՝

- Android App Links overview and implementation guide:  
  <a href="https://developer.android.com/training/app-links" target="_blank" rel="noopener noreferrer">https://developer.android.com/training/app-links</a>
- Add intent filters and configure web domain association:  
  <a href="https://developer.android.com/training/app-links/add-applinks" target="_blank" rel="noopener noreferrer">https://developer.android.com/training/app-links/add-applinks</a>
- Verify Android App Links and test behavior:  
  <a href="https://developer.android.com/training/app-links/verify-android-applinks" target="_blank" rel="noopener noreferrer">https://developer.android.com/training/app-links/verify-android-applinks</a>

## 6. Նույնականացման փոխանցում բջջային հավելվածներին

### 6.1. Կարո՞ղ է Hartak/YesEm նույնականացումը ուղղակի փոխանցվել բջջային հավելվածին

Ոչ՝ որպես «raw web session» անվտանգ և տեղափոխելի ձևով։ Բրաուզերի cookie-ները և հավելվածի session-ները տարբեր միջավայրեր են, ուստի հավելվածը չպետք է ստանա կամ կրկին օգտագործի բրաուզերի session cookie-ները։

### 6.2. Առաջարկվող անվտանգ մոտեցումներ

1. **YesEm-ով նույնականացում անմիջապես բջջային հավելվածում**
   - Օգտատերը բացում է հավելվածը։
   - Հավելվածը սկսում է YesEm նույնականացում։
   - Նույնականացուման հաջող ավարտից հետո հավելվածը օգագործում է օգտատիրոջ նույնականացված տվյալները։

2. **YesEm login-ից հետո շարունակել վեբից հավելված**
   - Օգտատերը սկսում է կանոնիկ HTTPS ծառայության հղումից։
   - Վերահասցեավորմում է բրաուզերում հասնում է YesEm։
   - Հաջող login-ից հետո authorization result-ը վերադարձվում է բջջային հավելվածին (Universal Link/App Link կամ callback URI)։
   - Հավելվածը փոխանակում է authorization code-ը և ստեղծում սեփական app session։

## 7. Թեստավորման ստուգաթերթ

Նախքան թողարկումը՝

- **Դոմենի կարգավորում**
  - iOS՝ AASA ֆայլը տեղադրված և հասանելի է `https://serviceportal.am/apple-app-site-association` հասցեում՝ ծառայություն մատուցողի հոստում։
  - Android՝ Digital Asset Links ֆայլը տեղադրված և հասանելի է `https://serviceportal.am/.well-known/assetlinks.json` հասցեում՝ նույն ծառայություն մատուցողի հոստում։
  - iOS/Android վավերացման ֆայլերը չպետք է հոսթվեն `hartak.am`-ում, եթե հղումները իրականում բացվում չեն `hartak.am`-ից։

- **iOS**
  - Տեղադրեք հավելվածը իրական սարքում (ոչ միայն սիմուլյատորում)։
  - Սեղմեք կանոնիկ հղումները՝
    - Notes, email կամ browser-ից։
  - Հաստատեք, որ՝
    - `/services/{serviceId}` հղումները բացում են հավելվածը։
    - `/services/*`-ից դուրս հղումները բացվում են միայն բրաուզերում։

- **Android**
  - Տեղադրեք հավելվածը signed build-ով։
  - Սեղմեք կանոնիկ հղումները՝
    - Email, messaging apps, browser-ից։
  - Վավերացրեք, որ՝
    - հավելվածը առաջարկվում է որպես handler (կամ բացվում է անմիջապես, եթե verify-ն անցած է)։
    - բացվում է ճիշտ էկրանը՝ backend-ով լուծված service context-ի հիման վրա։

- **Fallback**
  - Հեռացրեք հավելվածը և սեղմեք նույն հղումները։
  - Հաստատեք, որ օգտատերը ճիշտ է ուղղորդվում YesEm-ի և բրաուզերի միջոցով դեպի Service Portal հոսք։

