# Sentry API — Complete Integration Guide

> **EVA Software (Electronic Vehicle Assessment) · API v1.2**  
> Base URL: `https://sentry.evasoftware.co.uk/api/`

---

## Table of Contents

1. [Overview](#1-overview)
2. [Authentication & JWT Tokens](#2-authentication--jwt-tokens)
   - 2.1 [Obtaining a Token](#21-obtaining-a-token)
   - 2.2 [Using the Token](#22-using-the-token)
   - 2.3 [Token Expiry & Refresh Strategy](#23-token-expiry--refresh-strategy)
3. [Writing Data to EVA](#3-writing-data-to-eva)
   - 3.1 [Instruct Claim](#31-instruct-claim)
   - 3.2 [Claim Location Update](#32-claim-location-update)
   - 3.3 [Authority Status Update](#33-authority-status-update)
   - 3.4 [Submit Note](#34-submit-note)
   - 3.5 [Claim Update](#35-claim-update)
   - 3.6 [Submit Report](#36-submit-report)
4. [Retrieving Data from EVA](#4-retrieving-data-from-eva)
   - 4.1 [Retrieve Available Report List](#41-retrieve-available-report-list)
   - 4.2 [Retrieve a Specific Report](#42-retrieve-a-specific-report)
5. [Obtaining Specific Cases](#5-obtaining-specific-cases)
6. [Batch Operations](#6-batch-operations)
7. [Standard Response Model](#7-standard-response-model)
8. [Quick Reference](#8-quick-reference)

---

## 1. Overview

The Sentry API is a secure REST API built by EVA Software that enables authorised external partners — including insurers, claims management companies, and repair networks — to exchange vehicle claim data efficiently.

### What the API supports

- Submitting claim instructions to vehicle assessors
- Updating claim locations and authority statuses
- Posting notes and supplementary information
- Submitting full engineer inspection reports
- Retrieving released assessment reports

### Data Format

All requests and responses use **JSON** (`application/json`), except the authentication endpoint which uses `application/x-www-form-urlencoded`.

> **Note:** Fields marked as *Conditionally Required* must be provided as part of a valid identifying combination — for example, `ClmNo + VehReg`. At least one valid combination must be present to identify the target claim.

---

## 2. Authentication & JWT Tokens

Every Sentry API endpoint — **without exception** — requires a valid JSON Web Token (JWT) passed in the HTTP `Authorization` header. Requests without a token, or with an expired or invalid token, will receive a `401 Unauthorized` response.

---

### 2.1 Obtaining a Token

Authenticate using your assigned **Client ID** and **Client Secret** credentials, provided to you by EVA Software during onboarding.

```
POST /Connect/token
```

> ⚠️ **Important:** This endpoint uses `application/x-www-form-urlencoded` encoding — **not JSON**. Ensure your HTTP client sets the `Content-Type` header accordingly.

#### Request Fields

| Field           | Type   | Required | Description                                      |
|-----------------|--------|----------|--------------------------------------------------|
| `Client_Id`     | string | Yes      | Unique identifier assigned to you by EVA Software. |
| `Client_Secret` | string | Yes      | Secret key assigned to you by EVA Software.       |

#### Example Request

```http
POST /api/Connect/token
Content-Type: application/x-www-form-urlencoded

Client_Id=your_client_id&Client_Secret=your_client_secret
```

#### Success Response — `200 OK`

```json
{
  "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "expires_in": 5
}
```

| Field          | Description                                                      |
|----------------|------------------------------------------------------------------|
| `access_token` | The JWT string to include in all subsequent API requests.        |
| `expires_in`   | Minutes until the token expires. Default is **5 minutes**.       |

#### Failure Response — `401 Unauthorized`

```json
{
  "error": "unauthorized_client",
  "error_description": "Invalid Client ID or Secret"
}
```

---

### 2.2 Using the Token

Once obtained, include the token in the `Authorization` header of **every** subsequent request:

```http
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
```

---

### 2.3 Token Expiry & Refresh Strategy

> ⏱ **Tokens expire after 5 minutes.** You must implement automated token refresh logic in your integration middleware or API client. Do not cache tokens long-term.

#### Recommended Refresh Pattern

1. Request a new token before making any API call.
2. Store the token in memory alongside its calculated expiry time (`current time + 5 minutes`).
3. Before each subsequent request, check whether the token has expired or will expire within 30 seconds.
4. If expired or near expiry, request a fresh token before proceeding.
5. Never store tokens on disk or in a shared cache without encryption.

#### Token Refresh Logic (Pseudocode)

```js
function getValidToken() {
  if (token == null || isExpiredOrNearExpiry(token)) {
    token = requestNewToken(CLIENT_ID, CLIENT_SECRET);
    token.expiresAt = now() + 270; // 4.5 mins — 30s safety buffer
  }
  return token.access_token;
}

function callSentryAPI(endpoint, body) {
  return httpPost(endpoint, body, {
    headers: { Authorization: 'Bearer ' + getValidToken() }
  });
}
```

---

## 3. Writing Data to EVA

The following endpoints submit and update data within the EVA system. All are `POST` requests and require a valid JWT bearer token.

---

### 3.1 Instruct Claim

Submit a new claim instruction to the vehicle assessor. This is typically the **first step** in a claim lifecycle. The more complete the information provided, the more efficiently assessors can action the claim.

```
POST /Instruction/Inspection
```

#### Key Request Fields

| Field               | Type              | Required | Description                                          |
|---------------------|-------------------|----------|------------------------------------------------------|
| `RequestFrom`       | string (max 40)   | Yes      | Contact code supplied by EVA with your credentials.  |
| `ExternalRef`       | string            |          | Your own external reference for the claim.           |
| `VehReg`            | string (max 20)   |          | Vehicle registration number.                         |
| `ClmNo`             | string (max 24)   |          | Claim number.                                        |
| `PolNo`             | string (max 24)   |          | Policy number.                                       |
| `InsName`           | string (max 60)   |          | Name of the insurer.                                 |
| `VehDesc`           | string (max 40)   |          | Vehicle description.                                 |
| `DtIncident`        | DateTime          |          | Date of the incident (ISO 8601).                     |
| `InspType`          | string (max 25)   |          | Type of inspection — see accepted values below.      |
| `VehDriveable`      | string (max 9)    |          | `Yes` / `No` / `Not Known`                           |
| `CoverType`         | string (max 5)    |          | `Comp` / `TBA` / `TP` / `TPFT` / `WOP`              |
| `Urgent`            | bool              |          | Flag instruction as urgent.                          |
| `NotesStr`          | string            |          | Additional free-text notes.                          |
| `Files`             | List              |          | File attachments (base64 encoded).                   |

#### Accepted Values — `InspType`

- `Vehicle Damage Inspection`
- `Inspect and Authorise`
- `Inspect Only`
- `WOP Inspection`
- `Rectification work`
- `Quality/Audit Inspection`
- `Post Repair`
- `Low Velocity Inspection`
- `Desktop`
- `Other`

#### File Attachment Model

| Field       | Type   | Description                          |
|-------------|--------|--------------------------------------|
| `Name`      | string | File name (without extension).       |
| `Extension` | string | File extension, e.g. `.jpg`, `.pdf`. |
| `Data`      | byte[] | Base64-encoded file content.         |

#### Example Request

```http
POST /api/Instruction/Inspection
Authorization: Bearer <token>
Content-Type: application/json

{
  "RequestFrom":  "PARTNER01",
  "ExternalRef":  "ACME-CLM-2025-00981",
  "VehReg":       "AB12CDE",
  "ClmNo":        "CLM20251022001",
  "PolNo":        "POL998877",
  "InsName":      "Acme Insurance Group",
  "VehDesc":      "2020 BMW 320d M Sport",
  "DtIncident":   "2025-10-15T14:30:00Z",
  "InspType":     "Vehicle Damage Inspection",
  "VehDriveable": "No",
  "CoverType":    "COMP",
  "Urgent":       false,
  "NotesStr":     "Customer requests inspection to be expedited.",
  "Files": [
    { "Name": "damage_photo_1", "Extension": ".jpg", "Data": "base64string==" }
  ]
}
```

#### Responses

| Code  | Description                                                           |
|-------|-----------------------------------------------------------------------|
| `200` | ✅ Success — instruction created. Response includes the assessor `Id`. |
| `400` | Bad Request — invalid or missing data.                                |
| `401` | Unauthorized — missing or invalid JWT token.                          |
| `409` | Conflict — duplicate or conflicting instruction.                      |
| `500` | Internal Server Error.                                                |

---

### 3.2 Claim Location Update

Update the location associated with an existing claim. Supports four location types: `REPAIRER`, `INSPECTION`, `INSURED`, and `THIRDPARTY`.

```
POST /Claim/LocationUpdate
```

> 🔑 **Claim Identification:** Provide one of the following combinations to identify the target claim: `ClmNo + VehReg` **or** `EvaRef + VehReg`. A `404` is returned if neither matches.

#### Key Request Fields

| Field              | Type             | Required    | Description                                           |
|--------------------|------------------|-------------|-------------------------------------------------------|
| `VehReg`           | string (max 20)  | Conditional | Vehicle registration — required with `ClmNo` or `EvaRef`. |
| `ClmNo`            | string (max 24)  | Conditional | Claim number — required with `VehReg`.                |
| `EVARef`           | string           | Conditional | EVA reference — alternative to `ClmNo`.               |
| `LocationName`     | string (max 40)  | Yes         | Descriptive name, e.g. `Claimant's home`.             |
| `Address`          | string (max 40)  | Yes         | Address line 1.                                       |
| `Postcode`         | string (max 10)  | Yes         | Location postcode.                                    |
| `LocationType`     | string           | Yes         | `REPAIRER` / `INSPECTION` / `INSURED` / `THIRDPARTY` |
| `ApprovedRepairer` | bool             | Conditional | Required only when `LocationType` is `REPAIRER`.      |

#### Example Request

```http
POST /api/Claim/LocationUpdate
Authorization: Bearer <token>
Content-Type: application/json

{
  "VehReg":           "AB12CDE",
  "ClmNo":            "CLM20251022001",
  "LocationName":     "Example Repairers",
  "Address":          "15 High Street",
  "Town":             "Watford",
  "City":             "London",
  "County":           "Hertfordshire",
  "Postcode":         "WD17 1AA",
  "Telephone":        "02012345678",
  "Email":            "info@example.com",
  "ContactName":      "Sarah Mills",
  "LocationType":     "REPAIRER",
  "ApprovedRepairer": true
}
```

#### Responses

| Code  | Description                                           |
|-------|-------------------------------------------------------|
| `200` | ✅ Success — location updated.                         |
| `400` | Bad Request — invalid data.                           |
| `404` | Not Found — claim could not be located.               |
| `500` | Internal Server Error.                                |

---

### 3.3 Authority Status Update

Communicate a repair authorisation decision to the assessor. Informs the system whether the repairer is permitted to begin repairs.

```
POST /Claim/AuthorityStatusUpdate
```

> 🔑 **Claim Identification:** `ClmNo + VehReg` **or** `EvaRef + VehReg`

#### Key Request Fields

| Field        | Type             | Required    | Description                                              |
|--------------|------------------|-------------|----------------------------------------------------------|
| `VehReg`     | string (max 20)  | Conditional | Vehicle registration — required with `ClmNo` or `EVARef`. |
| `ClmNo`      | string (max 24)  | Conditional | Claim number.                                            |
| `EVARef`     | string           | Conditional | EVA reference — alternative to `ClmNo`.                  |
| `AuthStatus` | string (max 50)  | Yes         | `Yes` / `No` / `Other` / `After Instruction`             |
| `Comment`    | string           |             | Optional notes regarding the authority decision.         |
| `Files`      | List             |             | Supporting documents (base64 encoded).                   |

#### Example Request

```http
POST /api/Claim/AuthorityStatusUpdate
Authorization: Bearer <token>
Content-Type: application/json

{
  "VehReg":     "AB12CDE",
  "ClmNo":      "CLM20251022001",
  "AuthStatus": "Yes",
  "Comment":    "Repair authorised following assessment review.",
  "Files": [
    { "Name": "authority_letter", "Extension": ".pdf", "Data": "base64string==" }
  ]
}
```

#### Responses

| Code  | Description                                     |
|-------|-------------------------------------------------|
| `200` | ✅ Success — authority status updated.           |
| `400` | Bad Request — invalid data.                     |
| `404` | Not Found — claim could not be located.         |
| `500` | Internal Server Error.                          |

---

### 3.4 Submit Note

Send a message, query, or supporting information to the assessing company relating to an existing claim. File attachments can be included to help assessors complete reports accurately.

```
POST /Note/SubmitNote
```

> 🔑 **Claim Identification:** `ClmNo + VehReg` **or** `EvaRef + VehReg`

#### Key Request Fields

| Field    | Type   | Required    | Description                                           |
|----------|--------|-------------|-------------------------------------------------------|
| `VehReg` | string | Conditional | Vehicle registration — required with `ClmNo` or `EvaRef`. |
| `ClmNo`  | string | Conditional | Claim number.                                         |
| `EvaRef` | string | Conditional | EVA reference — alternative to `ClmNo`.               |
| `Note`   | string | Yes         | The message text or details being submitted.          |
| `Files`  | List   |             | Optional file attachments (base64 encoded).           |

#### Example Request

```http
POST /api/Note/SubmitNote
Authorization: Bearer <token>
Content-Type: application/json

{
  "ClmNo":  "CLM20251022001",
  "VehReg": "AB12CDE",
  "Note":   "Please confirm if additional photos are required before authorisation.",
  "Files": [
    { "Name": "damage_closeup", "Extension": ".jpg", "Data": "base64string==" }
  ]
}
```

#### Responses

| Code  | Description                                        |
|-------|----------------------------------------------------|
| `200` | ✅ Success — note received.                         |
| `400` | Bad Request — missing or invalid fields.           |
| `404` | Not Found — no matching claim found.               |
| `409` | Conflict — duplicate submission or invalid state.  |
| `500` | Internal Server Error.                             |

---

### 3.5 Claim Update

Update specific fields on an existing claim. Currently supports updating the **Excess** amount and **Claimant VAT Status**. The model can be extended to include additional fields on request.

```
POST /Claim/Update
```

> 🔑 **Claim Identification:** `ClmNo + VehReg` **or** `FileRef + VehReg`

#### Key Request Fields

| Field        | Type            | Required    | Description                                              |
|--------------|-----------------|-------------|----------------------------------------------------------|
| `VehReg`     | string          | Conditional | Vehicle registration — required with `FileRef` or `ClmNo`. |
| `ClmNo`      | string          | Conditional | Claim number.                                            |
| `FileRef`    | string          | Conditional | Internal file reference — alternative to `ClmNo`.        |
| `Excess`     | string (max 10) |             | Updated excess value, e.g. `250` or `£350`.              |
| `ClmVatStat` | string (max 3)  |             | Claimant VAT status: `Yes` / `No` / `n%`                 |

#### Example Request

```http
POST /api/Claim/Update
Authorization: Bearer <token>
Content-Type: application/json

{
  "VehReg":     "AB12CDE",
  "ClmNo":      "CLM20251022001",
  "Excess":     "350",
  "ClmVatStat": "20%"
}
```

#### Responses

| Code  | Description                               |
|-------|-------------------------------------------|
| `200` | ✅ Success — claim updated.               |
| `400` | Bad Request — invalid or incomplete data. |
| `404` | Not Found — claim not found.              |
| `500` | Internal Server Error.                    |

---

### 3.6 Submit Report

Submit a comprehensive vehicle inspection report to the EVA system. Used by external engineers and assessors. The payload includes inspection metadata, vehicle details, valuations, up to three damage areas, repair and financial data, salvage information, a parts list, impact image, and file attachments.

```
POST /Report/SubmitReport
```

#### Core Report Fields

| Field                | Type             | Required    | Description                                               |
|----------------------|------------------|-------------|-----------------------------------------------------------|
| `InspectEngineer`    | string (max 12)  | Yes         | Name or code of the inspecting engineer.                  |
| `EvaRef`             | string           | Conditional | EVA reference — required if used alongside `ClmNo`.       |
| `VehReg`             | string           | Yes         | Vehicle registration.                                     |
| `ClmNo`              | string           | Yes         | Claim number.                                             |
| `InsuredName`        | string           |             | Name of the insured party.                                |
| `ClaimType`          | string           |             | `Cash-In-Lieu` / `Diminution` / `Other` / `Post Repair` / `Repair` / `T/Loss` / `Repudiation` |
| `IncidentDate`       | DateTime         |             | Date of the incident.                                     |
| `InspectionDate`     | DateTime         |             | Date the vehicle was inspected.                           |
| `InspectionType`     | string (max 25)  |             | Type of inspection performed.                             |
| `ReportType`         | string (max 27)  |             | `Full Report` / `Desktop Report` / `Cold Call Report` / etc. |
| `ReportText`         | string           |             | Free-text narrative content of the engineer's report.     |
| `IsSupplementary`    | bool             |             | `true` if this is a supplementary submission.             |

#### Valuation Fields

| Field             | Type    | Description                                              |
|-------------------|---------|----------------------------------------------------------|
| `ValMarket`       | decimal | Market value of the vehicle.                             |
| `ValRetail`       | decimal | Retail value.                                            |
| `ValTrade`        | decimal | Trade-in value.                                          |
| `ValMid`          | decimal | Mid-point valuation.                                     |
| `ValSalvage`      | decimal | Salvage value.                                           |
| `ValDisposal`     | decimal | Disposal value.                                          |
| `MileageAdjust`   | decimal | Adjustment for mileage.                                  |
| `ConditionAdjust` | decimal | Adjustment for condition.                                |
| `ValResearch`     | string  | `Internet` / `Magazines` / `Main Dealer` / `Other`       |

#### Damage Fields (up to 3 areas)

Suffix with `2` or `3` for additional damage areas (e.g. `DamageSeverity2`, `DamageType3`).

| Field            | Type   | Accepted Values                                                                 |
|------------------|--------|---------------------------------------------------------------------------------|
| `DamageSeverity` | string | `Very Heavy` / `Heavy` / `Moderate to Heavy` / `Moderate` / `Light to Moderate` / `Light` / `Very Light` |
| `DamageType`     | string | `Collision/Impact` / `Accidental Damage` / `Fire (Electrical)` / `Storm Damage` / `Vandalism` / `Water/Flood/Damp` / etc. |
| `DamageArea`     | string | `Front` / `Rear` / `LH Front` / `LH Rear` / `RH Front` / `RH Rear` / `LH Side` / `RH Side` |
| `DamageLocation` | string | `Bumper` / `Bonnet` / `Boot` / `Door` / `Roof` / `Wing` / `Interior` / `Wheel` / etc. |

#### Impact Image

The impact image uses 8 numbered points around a top-down vehicle outline. Submit as an array of `Start`/`End` pairs — each pair draws an arrow between the two points on the final report PDF.

```
Point layout (top-down view):
  8 ——— 7 ——— 6
  |           |
  1           5
  |           |
  2 ——— 3 ——— 4
```

```json
"ImpactImage": [
  { "Start": 1, "End": 7 },
  { "Start": 2, "End": 4 }
]
```

#### Parts List

```json
"Parts": [
  {
    "Description": "Rear Bumper",
    "Quantity":    1,
    "PartType":    "New",
    "LabourTime":  2.25,
    "PaintTime":   1.0,
    "MaterialCost": 50.0,
    "Price":       350.0
  }
]
```

| Field          | Type    | Accepted `PartType` Values                                              |
|----------------|---------|-------------------------------------------------------------------------|
| `Description`  | string  | Part name or description.                                               |
| `Quantity`     | int     | Number of units.                                                        |
| `PartType`     | string  | `Blend` / `New` / `Paint` / `R & R` / `Repair` / `Specialist` / `Unknown` / `Check` |
| `LabourTime`   | float   | Labour time in hours.                                                   |
| `PaintTime`    | float   | Paint time in hours.                                                    |
| `MaterialCost` | decimal | Cost of materials.                                                      |
| `Price`        | decimal | Total price of the part.                                                |

#### Responses

| Code  | Description                             |
|-------|-----------------------------------------|
| `200` | ✅ Success — report submitted.           |
| `400` | Bad Request — invalid or missing data.  |
| `401` | Unauthorized — missing or invalid JWT.  |
| `500` | Internal Server Error.                  |

---

## 4. Retrieving Data from EVA

Report retrieval follows a **two-step pattern**: first retrieve the list of available reports, then fetch an individual report by its ID.

---

### 4.1 Retrieve Available Report List

Returns all reports that have been released and are available for retrieval. Results are ordered by `releasedDate` descending (newest first). Returns `200` even when no reports are available.

```
GET /Report/GetAvailableReports
```

#### Example Request

```http
GET /api/Report/GetAvailableReports
Authorization: Bearer <token>
```

#### Example Response

```json
[
  {
    "id": 123,
    "registration": "AB12CDE",
    "releasedDate": "2026-05-06T10:30:00Z"
  },
  {
    "id": 124,
    "registration": "XY99ZZZ",
    "releasedDate": "2026-05-05T14:15:00Z"
  }
]
```

#### Response Model

| Field          | Type     | Description                                                   |
|----------------|----------|---------------------------------------------------------------|
| `id`           | int      | Unique report identifier. Use this to fetch the full report.  |
| `registration` | string   | Vehicle registration associated with the report.              |
| `releasedDate` | datetime | Date and time the report was released (ISO 8601).             |

#### Responses

| Code  | Description                                             |
|-------|---------------------------------------------------------|
| `200` | ✅ Success — list retrieved (empty array if none available). |
| `401` | Unauthorized — invalid or missing token.                |
| `500` | Internal Server Error.                                  |

---

### 4.2 Retrieve a Specific Report

Fetch the full data of a single released report using an `id` from the list endpoint.

```
GET /Report/GetReport?id={id}
```

#### Example Request

```http
GET /api/Report/GetReport?id=123
Authorization: Bearer <token>
```

#### Query Parameter

| Parameter | Type | Required | Description                                             |
|-----------|------|----------|---------------------------------------------------------|
| `id`      | int  | Yes      | The unique report ID from `GetAvailableReports`.        |

The response mirrors the Submit Report payload, returning all vehicle, inspection, valuation, damage, repair, salvage, and financial data, plus parts, impact image, and file attachments.

#### Additional Response-Only Fields

These fields appear in the `GetReport` response but are not part of the `SubmitReport` request:

| Field                | Type    | Description                                        |
|----------------------|---------|----------------------------------------------------|
| `GlassMonth`         | string  | Date of the vehicle valuation from Glass's Guide.  |
| `SupplementaryCount` | integer | Number of supplementary reports completed.         |
| `FeeNet`             | decimal | Last live invoice net value.                       |
| `FeeVat`             | decimal | Last live invoice VAT value.                       |
| `FeeGross`           | decimal | Last live invoice gross value.                     |

#### Responses

| Code  | Description                                                   |
|-------|---------------------------------------------------------------|
| `200` | ✅ Success — full report data returned.                        |
| `400` | Bad Request — invalid report ID.                              |
| `401` | Unauthorized — missing or invalid token.                      |
| `404` | Not Found — report does not exist or has not been released.   |
| `500` | Internal Server Error.                                        |

---

## 5. Obtaining Specific Cases

The Sentry API does not provide a dedicated search or filter endpoint. Specific cases are targeted using claim identifier combinations on write endpoints, and by filtering the available reports list client-side.

---

### 5.1 Claim Identifier Combinations

Every POST endpoint that operates on an existing claim uses one of the following combinations to locate it:

| Combination         | Applicable Endpoints                                                  |
|---------------------|-----------------------------------------------------------------------|
| `ClmNo + VehReg`    | LocationUpdate, AuthorityStatusUpdate, SubmitNote, Claim/Update       |
| `EvaRef + VehReg`   | LocationUpdate, AuthorityStatusUpdate, SubmitNote                     |
| `FileRef + VehReg`  | Claim/Update only                                                     |

> ⚠️ **Always provide `VehReg` alongside your primary identifier.** Both fields in the combination must match exactly — there is no partial matching. A `404` is returned if the claim cannot be located.

---

### 5.2 Fetching a Specific Report by Registration or Claim

```
Step 1 → GET /Report/GetAvailableReports   (retrieve full list)
Step 2 → Filter client-side by registration or date
Step 3 → GET /Report/GetReport?id={id}     (fetch the matched report)
```

#### Example — Filter by Registration

```js
// Step 1: Get all available reports
const reports = await fetch('/api/Report/GetAvailableReports', {
  headers: { Authorization: 'Bearer ' + token }
}).then(r => r.json());

// Step 2: Find the report for a specific vehicle
const target = reports.find(r => r.registration === 'AB12CDE');

// Step 3: Fetch the full report
const report = await fetch('/api/Report/GetReport?id=' + target.id, {
  headers: { Authorization: 'Bearer ' + token }
}).then(r => r.json());
```

#### Example — Most Recent Report for a Vehicle

```js
const vehicleReports = reports
  .filter(r => r.registration === 'AB12CDE')
  .sort((a, b) => new Date(b.releasedDate) - new Date(a.releasedDate));

if (vehicleReports.length > 0) {
  const latest = await getReport(vehicleReports[0].id);
}
```

#### Example — All Reports Released After a Specific Date

```js
const since = new Date('2026-05-01T00:00:00Z');

const recentReports = reports
  .filter(r => new Date(r.releasedDate) > since)
  .sort((a, b) => new Date(b.releasedDate) - new Date(a.releasedDate));
```

---

## 6. Batch Operations

> 📌 **The Sentry API v1.2 does not provide a native batch endpoint.** All endpoints accept a single claim per request. Batch processing must be implemented at the client or integration layer.

Despite this, efficient bulk operations are fully achievable with the patterns below.

---

### 6.1 Batch Submitting Instructions

Iterate over your claims array and submit each via `POST /Instruction/Inspection`. Reuse a valid token across calls, refreshing as needed.

```js
async function batchInstructClaims(claims) {
  const results = [];

  for (const claim of claims) {
    const token = await getValidToken();
    const response = await fetch('/api/Instruction/Inspection', {
      method: 'POST',
      headers: {
        'Authorization': 'Bearer ' + token,
        'Content-Type': 'application/json'
      },
      body: JSON.stringify(claim)
    });

    const result = await response.json();
    results.push({
      claim:  claim.ExternalRef,
      status: result.StatusCode,
      id:     result.Id
    });

    await sleep(100); // Small delay to avoid overwhelming the server
  }

  return results;
}
```

---

### 6.2 Batch Retrieving All Released Reports

`GetAvailableReports` returns all available reports in a single call, making it well-suited for bulk retrieval workflows.

#### Sequential Retrieval (safe, predictable)

```js
async function downloadAllReports() {
  const list = await fetch('/api/Report/GetAvailableReports', {
    headers: { Authorization: 'Bearer ' + await getValidToken() }
  }).then(r => r.json());

  const reports = [];
  for (const item of list) {
    const report = await fetch('/api/Report/GetReport?id=' + item.id, {
      headers: { Authorization: 'Bearer ' + await getValidToken() }
    }).then(r => r.json());

    reports.push(report);
    await sleep(100);
  }

  return reports;
}
```

#### Parallel Retrieval (faster — use with caution)

```js
async function downloadAllReportsParallel(list) {
  const CONCURRENCY = 5; // Limit concurrent requests
  const chunks = chunkArray(list, CONCURRENCY);
  const results = [];

  for (const chunk of chunks) {
    const token = await getValidToken();
    const fetched = await Promise.all(
      chunk.map(item =>
        fetch('/api/Report/GetReport?id=' + item.id, {
          headers: { Authorization: 'Bearer ' + token }
        }).then(r => r.json())
      )
    );
    results.push(...fetched);
  }

  return results;
}
```

---

### 6.3 Batch Note Submission

```js
const notes = [
  { ClmNo: 'CLM001', VehReg: 'AB12CDE', Note: 'Photos requested.' },
  { ClmNo: 'CLM002', VehReg: 'XY99ZZZ', Note: 'Awaiting estimate confirmation.' },
];

for (const note of notes) {
  await fetch('/api/Note/SubmitNote', {
    method: 'POST',
    headers: {
      'Authorization': 'Bearer ' + await getValidToken(),
      'Content-Type': 'application/json'
    },
    body: JSON.stringify(note)
  });
}
```

---

### 6.4 Batch Best Practices

- **Always refresh the JWT token** between batches or when within 30 seconds of the 5-minute expiry.
- **Add a short delay** (50–200 ms) between sequential requests to avoid overwhelming the server.
- **Limit parallel concurrency** to 5 or fewer simultaneous requests.
- **Log every response** — capture `StatusCode` and `Id` for audit and retry tracking.
- **Implement retry logic** with exponential backoff for `5xx` errors.
- **Schedule large batches** during off-peak hours where possible.

---

## 7. Standard Response Model

All write endpoints (`POST`) return the same standard response structure:

```json
{
  "StatusCode": 200,
  "Message":    "Instruction received successfully.",
  "Id":         "123"
}
```

| Field        | Type           | Description                                                              |
|--------------|----------------|--------------------------------------------------------------------------|
| `StatusCode` | HttpStatusCode | The HTTP status code returned (e.g. `200`, `400`, `404`).                |
| `Message`    | string         | A human-readable description or explanation of the result.               |
| `Id`         | string         | Populated on success for **Instruct Claim** only; not used on other endpoints. |

---

## 8. Quick Reference

### All Endpoints

| Method | Endpoint                           | Purpose                        | Auth                  |
|--------|------------------------------------|--------------------------------|-----------------------|
| `POST` | `/Connect/token`                   | Obtain JWT access token        | Client ID + Secret    |
| `POST` | `/Instruction/Inspection`          | Submit new claim instruction   | Bearer token          |
| `POST` | `/Claim/LocationUpdate`            | Update claim location          | Bearer token          |
| `POST` | `/Claim/AuthorityStatusUpdate`     | Update repair authority status | Bearer token          |
| `POST` | `/Note/SubmitNote`                 | Submit note to assessor        | Bearer token          |
| `POST` | `/Claim/Update`                    | Update excess / VAT status     | Bearer token          |
| `POST` | `/Report/SubmitReport`             | Submit inspection report       | Bearer token          |
| `GET`  | `/Report/GetAvailableReports`      | List all released reports      | Bearer token          |
| `GET`  | `/Report/GetReport?id={id}`        | Retrieve specific report by ID | Bearer token          |

### Claim Identifier Combinations

| Combination        | Endpoints                                                       |
|--------------------|-----------------------------------------------------------------|
| `ClmNo + VehReg`   | LocationUpdate, AuthorityStatusUpdate, SubmitNote, Claim/Update |
| `EvaRef + VehReg`  | LocationUpdate, AuthorityStatusUpdate, SubmitNote               |
| `FileRef + VehReg` | Claim/Update only                                               |

### HTTP Status Codes

| Code  | Meaning                                                    |
|-------|------------------------------------------------------------|
| `200` | ✅ Success                                                  |
| `400` | Bad Request — invalid or missing data                      |
| `401` | Unauthorized — missing or invalid JWT token                |
| `404` | Not Found — claim or report could not be located           |
| `409` | Conflict — duplicate or conflicting submission             |
| `500` | Internal Server Error                                      |

---

*Sentry API Documentation v1.2 — EVA Software (Electronic Vehicle Assessment)*
