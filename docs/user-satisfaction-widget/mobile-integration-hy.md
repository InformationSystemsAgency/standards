## NPS Satisfaction API – Մոբայլ ինտեգրման ուղեցույց

Այս փաստաթուղթը նկարագրում է, թե ինչպես ինտեգրել մոբայլ հավելվածները (iOS/Android) բավարարվածության ծառայության հետ (NPS)։ 

- **Վիջեթի URL** (`Widget URL`): `https://nps.services.catalog.isaa.cloud/?serviceId=00000000-0000-0000-0000-000000000000`
- **Ծառայության ID** (`Service ID`): `00000000-0000-0000-0000-000000000000`

Վիջեթը հետադարձ կապը (feedback) ուղարկում է հետեւյալ հասցեին․

- **Հիմնական URL** (`Base URL`): `https://api.nps.services.catalog.isaa.cloud`
- **Endpoint**: `POST /v1/satisfaction`
- **Ձեւաչափ** (`Format`): JSON request/response, `Content-Type: application/json`

---

## Հարցման (request) տվյալների մոդել

**Ընդհանուր դաշտեր (օգտագործվում են բոլոր տարբերակների դեպքում)**

- **`serviceId`** (`string`, պարտադիր): գնահատվող ծառայության ID-ն։
  Օրինակ․ `"00000000-0000-0000-0000-000000000000"`։
- **`rating`** (`number`, պարտադիր): ամբողջ թիվ 1–5 միջակայքից։
  - 1 – “Շատ վատ” (Very bad)
  - 2 – “Վատ” (Bad)
  - 3 – “Բավարար” (Satisfactory)
  - 4 – “Լավ” (Good)
  - 5 – “Գերազանց” (Excellent)
- **`comment`** (`string`, ոչ պարտադիր): օգտատիրոջ թողած ազատ տեքստային մեկնաբանությունը (կարող է լինել նույնիսկ դատարկ տող)։
- **`commentSubmitted`** (`boolean`, պարտադիր):
  - `false`, երբ վարկանիշն ուղարկվում է, սակայն վերջնական մեկնաբանություն դեռ չկա։
  - `true`, երբ ուղարկվում է վերջնական հետադարձ կապը (վարկանիշ + վերջնական մեկնաբանություն)։
- **`pageLanguage`** (`string`, պարտադիր): UI-ում օգտագործվող լեզվի կոդը, օրինակ՝ `"hy"`։
- **`userAgent`** (`string`, պարտադիր):
  - Օրինակ (Android)․ `"MyApp/1.0 (Android; Pixel 8; Android 15)"`
  - Օրինակ (iOS)․ `"MyApp/1.0 (iOS; iPhone15,2; iOS 18.1)"`

**Լրացուցիչ դաշտ՝ թարմացումների համար**

- **`id`** (`string`, պարտադիր միայն արդեն գոյություն ունեցող գրառման թարմացման դեպքում)․
  արդեն ստեղծված բավարարվածության գրառման ID։ Օրինակ․ `"648e06b3-1e7e-417c-86a3-e7fd53c11c14"`։

---

## Տարբերակ A – Երկու-քայլանոց տարբերակ

նախ ուղարկում եք վարկանիշը, ապա (ըստ անհրաժեշտության) թարմացնում եք գրառումը մեկնաբանությամբ։

### Քայլ A1 – Վարկանիշի ուղարկում (առանց մեկնաբանության)

Թրիգեր․ օգտատերը հավելվածում ընտրում է 1–5 վարկանիշներից մեկը։

- **Մեթոդ** (`Method`): `POST`
- **URL**: `https://api.nps.services.catalog.isaa.cloud/v1/satisfaction`
- **Մարմնի օրինակ** (`Body example`)․

```json
{
  "rating": 5,
  "comment": "",
  "commentSubmitted": false,
  "pageLanguage": "hy",
  "serviceId": "00000000-0000-0000-0000-000000000000",
  "userAgent": "MyApp/1.0 (Android; Pixel 8; Android 15)"
}
```

**Սպասվող վարքագիծ**

- Backend-ը **ստեղծում է** բավարարվածության նոր գրառում։
- Պատասխանն իր մեջ առնվազն ներառում է․
  - **`id`** – ստեղծված գրառման գեներացված ID։
  - Հիմնական դաշտերի կրկնումը (`rating`, `comment` և այլն)։
- Մոբայլ հավելվածը պետք է **հիշողության մեջ պահի `id`-ը** այս feedback հոսքի մնացած մասի համար։

Եթե օգտատերը երբեք չի գրում մեկնաբանություն, այս մեկ հարցումը բավարար է, և այլ գործողություններ անհրաժեշտ չեն։

### Քայլ A2 – Թարմացում մեկնաբանությամբ (ըստ ցանկության)

Թրիգեր․ օգտատերը որոշում է թողնել մեկնաբանություն և սեղմում է «Ուղարկել» (կամ նմանատիպ) կոճակը։

- **Մեթոդ**: `POST`
- **URL**: `https://api.nps.services.catalog.isaa.cloud/v1/satisfaction`
- **Մարմնի օրինակ** (`Body example`)․

```json
{
  "id": "648e06b3-1e7e-417c-86a3-e7fd53c11c14",
  "rating": 5,
  "comment": "Great service, very fast.",
  "commentSubmitted": true,
  "pageLanguage": "hy",
  "serviceId": "00000000-0000-0000-0000-000000000000",
  "userAgent": "MyApp/1.0 (Android; Pixel 8; Android 15)"
}
```

**Կարևոր կետեր**

- **`id`** դաշտում պետք է ուղարկվի A1 քայլից ստացած ID֊ն։
- `commentSubmitted` դաշտը պետք է լինի **`true`**։
- Պետք է ուղարկել **վերջնական վարկանիշն ու մեկնաբանությունը**։

Backend-ը սա դիտարկում է որպես արդեն առկա գրառման **թարմացում**։

---

## Տարբերակ B – Մեկ-քայլանոց տարբերակ (վարկանիշ + մեկնաբանություն մեկ հարցմամբ)


- `POST https://api.nps.services.catalog.isaa.cloud/v1/satisfaction`

### Մեկ հարցման (one-shot) օրինակ

Թրիգեր․ օգտատերը ընտրում է վարկանիշը և ըստ ցանկության մուտքագրում մեկնաբանությունը, հետո սեղմում է «Ուղարկել կարծիքը» կոճակը։

```json
{
  "rating": 5,
  "comment": "One-step feedback from mobile",
  "commentSubmitted": true,
  "pageLanguage": "hy",
  "serviceId": "00000000-0000-0000-0000-000000000000",
  "userAgent": "MyApp/1.0 (Android; Pixel 8; Android 15)"
}
```

**Վարքագիծ**

- Հարցումը (request-ում `id` դաշտը պարտադիր չէ)։
- Backend-ը **ստեղծում է** գրառում և պատասխանում վերադարձնում է **`id`**, ինչպես տարբերակ A-ի դեպքում։
- Երկրորդ հարցում անելու անհրաժեշտություն չկա։


---

## Խորհուրդ տրվող client-side վիճակի կառավարում

Client-ի տրամաբանությունը կարելի է պատկերացնել որպես պարզ վիճակների մեքենա․

- **Վիճակ 1 – Idle (ոչ մի գործողություն)**
  - Օգտատերը դեռ չի փոխգործակցել feedback կոմպոնենտի հետ։

- **Վիճակ 2 – Վարկանիշն ընտրված է**
  - **Տարբերակ A**․
    - Վարկանիշի սեղմման պահին → կանչել Քայլ A1-ը։
    - Պահպանել `id`-ն ու rating-ը, ցուցադրել «թողնել մեկնաբանություն» առաջարկը (ըստ ցանկության)։
  - **Տարբերակ B**․
    - Պահպանել միայն ընտրված վարկանիշը լոկալ, բայց դեռ API հարցում չանել։

- **Վիճակ 3 – Մեկնաբանության մշակում**
  - **Օգտատերը բաց է թողնում մեկնաբանությունը**․
    - Տարբերակ A․ այլ գործողություն չի պահանջվում, միայն վարկանիշը արդեն պահպանված է։
    - Տարբերակ B․ կարելի է մեկ հարցմամբ ուղարկել request՝ `comment: ""` եւ `commentSubmitted: true`, երբ օգտատերը հստակ հաստատում է։
  - **Օգտատերը գրում եւ ուղարկում է մեկնաբանությունը**․
    - Տարբերակ A․ կանչել Քայլ A2-ը՝ `id`-ով եւ `commentSubmitted: true` դաշտով։
    - Տարբերակ B․ ուղարկել մեկ հարցումով request՝ rating + comment + `commentSubmitted: true`։

---

## Սխալների մշակման ուղեցույց

- **Ցանցային / timeout սխալներ**
  - **Վարքագիծ**․ ցուցադրել ընդհանուր հաղորդագրություն՝ «Չհաջողվեց ուղարկել կարծիքը, խնդրում ենք փորձել քիչ ուշ»։
  - Ըստ ցանկության՝ տրամադրել **կրկին փորձել** (retry) կոճակ։

- **4xx սխալներ (client-side խնդիրներ)**
  - Ամենայն հավանականությամբ պայմանավորված են սխալ payload-ով (վերացվող կամ սխալ տիպի դաշտեր)։
  - Գրանցեք (log) սխալի մանրամասները՝ դրանք օգտատիրոջը չցուցադրելով։
  - Ցուցադրեք անվտանգ հաղորդագրություն, օրինակ՝ «Չհաջողվեց ուղարկել կարծիքը՝ անհայտ սխալի պատճառով»։

- **5xx սխալներ (server-side խնդիրներ)**
  - Դիտարկել որպես ժամանակավոր խափանումներ։
  - Ըստ ցանկության թույլ տալ կրկին փորձել, հակառակ դեպքում լռելյայն ձեւով ձախողել՝ ցուցադրելով կարճ հաղորդագրություն։

---

## Պլատֆորմից անկախ pseudo-code

Ստորեւ բերված pseudo-code-ը ցույց է տալիս, թե ինչպես կարելի է իրականացնել երկու տարբերակն էլ։

### Տարբերակ A – Երկու-քայլանոց (նախ վարկանիշ, հետո մեկնաբանություն)

```pseudo
const SERVICE_ID = "00000000-0000-0000-0000-000000000000"

state = {
  satisfactionId: null,
  rating: null
}

function buildUserAgent():
  return "<your app UA string>"

function sendRating(rating):
  state.rating = rating

  body = {
    rating: rating,
    comment: "",
    commentSubmitted: false,
    pageLanguage: currentLanguageCode(),
    serviceId: SERVICE_ID,
    userAgent: buildUserAgent()
  }

  response = http.post("https://api.nps.services.catalog.isaa.cloud/v1/satisfaction", json=body)

  if response.isSuccessful():
    state.satisfactionId = response.json.id

function submitComment(comment):
  if state.satisfactionId == null:
    return  // no rating was sent; optionally handle this case

  body = {
    id: state.satisfactionId,
    rating: state.rating,
    comment: comment,
    commentSubmitted: true,
    pageLanguage: currentLanguageCode(),
    serviceId: SERVICE_ID,
    userAgent: buildUserAgent()
  }

  http.post("https://api.nps.services.catalog.isaa.cloud/v1/satisfaction", json=body)
```

### Տարբերակ B – Մեկ-քայլանոց

```pseudo
const SERVICE_ID = "00000000-0000-0000-0000-000000000000"

function buildUserAgent():
  return "<your app UA string>"

function submitFeedback(rating, comment):
  body = {
    rating: rating,
    comment: comment ?? "",
    commentSubmitted: true,
    pageLanguage: currentLanguageCode(),
    serviceId: SERVICE_ID,
    userAgent: buildUserAgent()
  }

  response = http.post("https://api.nps.services.catalog.isaa.cloud/v1/satisfaction", json=body)

  if response.isSuccessful():
    // Ընտրովի (optional)․ կարելի է պահել response.json.id-ը անալիտիկայի կամ debugging-ի համար
    log("Submitted NPS feedback id = " + response.json.id)
```

---

## Ամփոփում

- **Endpoint**: `POST https://api.nps.services.catalog.isaa.cloud/v1/satisfaction`
- **Երկու-քայլանոց Տարբերակ**․ նախ ուղարկում եք վարկանիշը (`commentSubmitted: false`), ապա ըստ անհրաժեշտության թարմացնում եք գրառումը՝ `id` եւ `commentSubmitted: true` դաշտերով։
- **Մեկ-քայլանոց Տարբերակ**․ մեկ հարցմամբ ուղարկում եք վարկանիշ + մեկնաբանություն՝ `commentSubmitted: true` դաշտով, սերվերը ստեղծում է գրառում և վերադարձնում է `id` առանց նախորդ հարցման անհրաժեշտության։
