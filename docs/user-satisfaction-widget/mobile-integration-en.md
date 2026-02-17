## NPS Satisfaction API – Mobile Integration Guide

This document explains how to integrate mobile apps (iOS/Android) with the NPS satisfaction service.

- **Widget URL**: `https://nps.services.catalog.isaa.cloud/?serviceId=00000000-0000-0000-0000-000000000000`
- **Service ID**: `00000000-0000-0000-0000-000000000000`

The widget sends feedback to:

- **Base URL**: `https://api.nps.services.catalog.isaa.cloud`
- **Endpoint**: `POST /v1/satisfaction`
- **Format**: JSON request/response, `Content-Type: application/json`

---

## Request data model

**Common fields (used in all flows)**

- **`serviceId`** (`string`, required): ID of the service being rated.
  Example: `"00000000-0000-0000-0000-000000000000"`.
- **`rating`** (`number`, required): integer from 1–5.
  - 1 – “Շատ վատ” (Very bad)
  - 2 – “Վատ” (Bad)
  - 3 – “Բավարար” (Satisfactory)
  - 4 – “Լավ” (Good)
  - 5 – “Գերազանց” (Excellent)
- **`comment`** (`string`, optional): free text comment from the user (can be empty string).
- **`commentSubmitted`** (`boolean`, required):
  - `false` when rating is sent without a final comment yet.
  - `true` when feedback (rating + final comment) is submitted.
- **`pageLanguage`** (`string`, required): language code used in the UI, e.g. `"hy"`.
- **`userAgent`** (`string`, required): description of the client.
  - Example (Android): `"MyApp/1.0 (Android; Pixel 8; Android 15)"`
  - Example (iOS): `"MyApp/1.0 (iOS; iPhone15,2; iOS 18.1)"`

**Additional field for updates**

- **`id`** (`string`, required only when updating an existing record):
  Identifier of an already created satisfaction entry. Example: `"648e06b3-1e7e-417c-86a3-e7fd53c11c14"`.

---

## Flow A – Two-step flow

### Step A1 – Send rating (no comment yet)

Trigger: user selects a rating (1–5) in the app.

- **Method**: `POST`
- **URL**: `https://api.nps.services.catalog.isaa.cloud/v1/satisfaction`
- **Body example**:

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

**Expected behavior**

- Backend **creates** a new satisfaction record.
- Response includes at least:
  - **`id`** – generated identifier of the created entry.
  - Echo of main fields (`rating`, `comment`, etc.).
- The mobile app should **store `id` in memory** for the rest of this feedback flow.

If the user never writes a comment, this single call is enough and no further action is required.

### Step A2 – Update with comment (optional)

Trigger: user decides to leave a comment and taps “Submit” (or similar).

- **Method**: `POST`
- **URL**: `https://api.nps.services.catalog.isaa.cloud/v1/satisfaction`
- **Body example**:

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

**Key points**

- **`id`** must be the value returned from Step A1.
- `commentSubmitted` must be **`true`**.
- Send the **final rating and comment**.

The backend treats this as an **update** of the existing record.

---

## Flow B – One-step flow (rating + comment in a single call)

You can also perform **single API call**. 

- `POST https://api.nps.services.catalog.isaa.cloud/v1/satisfaction`

### One-shot request example

Trigger: user selects a rating and optionally enters a comment, then taps “Submit feedback”.

```json
{
  "rating": 5,
  "comment": "One-step feedback",
  "commentSubmitted": true,
  "pageLanguage": "hy",
  "serviceId": "00000000-0000-0000-0000-000000000000",
  "userAgent": "MyApp/1.0 (Android; Pixel 8; Android 15)"
}
```

**Behavior**

- Request **succeeds without sending a previous call** (i.e. no `id` is required in the request).
- Backend **creates** a record and returns an **`id`** in the response just like in Flow A.
- No second call is needed.

This is the **recommended flow for mobile apps** when:

- You display rating and comment in a single UI.
- You don’t need to persist an intermediate “rating only” state.

---

## Recommended client-side state handling

You can model the client as a simple state machine:

- **State 1 – Idle**
  - User has not interacted with the feedback component yet.

- **State 2 – Rating chosen**
  - **Flow A**:
    - On rating tap → call Step A1.
    - Store `id` and rating; show optional “leave a comment” prompt.
  - **Flow B**:
    - Just store the selected rating locally, do not call the API yet.

- **State 3 – Comment handling**
  - **User skips comment**:
    - Flow A: nothing more to do; rating-only data already persisted.
    - Flow B: you may still send a one-shot call with `comment: ""` and `commentSubmitted: true` when the user explicitly confirms.
  - **User writes comment and submits**:
    - Flow A: call Step A2 with `id` and `commentSubmitted: true`.
    - Flow B: send the one-shot call with rating + comment + `commentSubmitted: true`.

---

## Error handling guidelines

- **Network / timeout errors**
  - **Behavior**: show a generic “Could not send feedback, please try again later” message.
  - Optionally offer a **retry** button.

- **4xx errors (client-side issues)**
  - Likely caused by invalid payload (missing fields, invalid types).
  - Log error details (without exposing internal messages to the user).
  - Show a safe message such as “Could not send feedback due to an unexpected error.”

- **5xx errors (server issues)**
  - Treat as temporary failures.
  - Optionally allow retry; otherwise, fail silently with a short message.

---

## Platform-agnostic pseudo-code

The following pseudo-code shows how to implement both flows.

### Flow A – Two-step (rating then comment)

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

### Flow B – One-step

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
    // Optional: store response.json.id for analytics or debugging
    log("Submitted NPS feedback id = " + response.json.id)
```

---

## Summary

- **Endpoint**: `POST https://api.nps.services.catalog.isaa.cloud/v1/satisfaction`
- **Two-step flow**: first send rating (`commentSubmitted: false`), then optionally update with `id` and `commentSubmitted: true`.
- **One-step flow**: send rating + comment once with `commentSubmitted: true`; the server creates a record and returns `id` without requiring a prior call.
