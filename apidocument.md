# ACPLegal API Reference

Base URL (development): https://acp-legal-backend-production.up.railway.app

Purpose: concise, junior-friendly API reference for frontend developers. Each endpoint includes: HTTP method, full path, authentication requirements, headers, request body schema with types and required fields, example request JSON/cURL, expected responses (status codes + example JSON), and quick notes.

- Authentication requirement: Frontend MUST include `Authorization: Bearer <jwt>` header for all API requests except the explicitly public endpoints listed below (login, signup, refresh, init-superadmin, health).
Notes:
- Authentication requirement: Frontend MUST include `Authorization: Bearer <jwt>` header for all API requests except the explicitly public endpoints listed below (login, signup, refresh, init-superadmin, health).
- All JSON uses camelCase.
- Dates: use ISO 8601 (`YYYY-MM-DD` for dates, `YYYY-MM-DDTHH:mm:ssZ` for datetimes) unless otherwise stated.
- GUIDs use the standard 36-character format.

## Authentication

### POST /auth/login
- Method: POST
- Full path: /auth/login
- Auth: Public (no Authorization header required)
- Headers: `Content-Type: application/json`
- Request body (JSON):
  - `email` (string, required)
  - `password` (string, required)
  - `deviceInfo` (object, optional) — optional device metadata
  - `trustDevice` (boolean, optional, default false)
- Example request:
```json
{
  "email": "staff@acplegal.co.uk",
  "password": "Secret123"
}
```
- Success (200):
```json
{
  "access_token": "<jwt>",
  "refresh_token": "<refresh-token>",
  "user": { /* session object */ }
}
```
- Errors:
  - 400 Bad Request — missing or invalid fields
  - 401 Unauthorized — invalid credentials

Example cURL:
```bash
curl -X POST https://localhost:5001/auth/login \
  -H "Content-Type: application/json" \
  -d '{"email":"staff@acplegal.co.uk","password":"Secret123"}'
```

### POST /auth/signup
- Method: POST
- Path: /auth/signup
- Auth: Public (no Authorization header required)
- Headers: `Content-Type: application/json`
- Request body:
  - `email` (string, required)
  - `password` (string, required)
  - `name` (string, required)
  - `role` (string, optional, default `staff`)
  - `officeLocation` (string, optional, default `head-office`)
- Example request:
```json
{ "email":"new@acplegal.co.uk", "password":"Secret123", "name":"New User" }
```
- Success (200):
```json
{ "message": "Staff account created successfully", "user": { /* user object */ } }
```

### GET /auth/session
- Method: GET
- Path: /auth/session
- Auth: Bearer token required
- Headers: `Authorization: Bearer <token>`
- Success (200):
```json
{ "user": { /* SessionDto */ } }
```

### POST /auth/logout
- Method: POST
- Path: /auth/logout
- Auth: Bearer token required
- Headers: `Content-Type: application/json`, `Authorization: Bearer <token>`
- Request body (optional):
  - `refreshToken` (string, optional)
- Success (200): `{ "message": "Logged out successfully" }`

### POST /auth/refresh
- Method: POST
- Path: /auth/refresh
- Auth: Public (no Authorization header required)
- Headers: `Content-Type: application/json`
- Request body:
  - `refreshToken` (string, required)
- Success (200): `{ "access_token": "...", "refresh_token": "...", "user": { ... } }`

### POST /auth/init-superadmin
- Method: POST
- Path: /auth/init-superadmin
- Auth: requires header `X-Master-Key: <master-key>` (or `masterKey` header)
- Headers: `Content-Type: application/json`, `X-Master-Key: <key>`
- Request body:
  - `email` (string, required)
  - `password` (string, required)
  - `name` (string, required)
- Success (200): `{ "message": "Super Admin initialized successfully", "user": { ... } }`

## Bookings (Routes under /api/bookings)

Common DTO: `BookingDto` (fields used by requests/responses)
- `id` (GUID)
- `bookingReference` (string)
- `customerName` (string)
- `customerEmail` (string)
- `customerPhone` (string)
- `serviceType` (string)
- `preferredDate` (string, ISO `YYYY-MM-DD`)
- `preferredTime` (string, time `HH:mm:ss`)
- `durationMinutes` (integer)
- `price` (number)
- `currency` (string)
- `status` (string)

### GET /api/bookings
- Method: GET
- Path: /api/bookings
- Auth: Authorization: Bearer <token> required
- Query: none (currently)
- Success (200):
```json
{ "data": [ /* BookingDto */ ], "count": 123 }
```

### GET /api/bookings/{id}
- Method: GET
- Path: /api/bookings/{id}
- Path params:
  - `id` (GUID, required)
- Success (200): `BookingDto`
- Errors: 404 Not Found

### GET /api/bookings/by-reference/{reference}
- Method: GET
- Path: /api/bookings/by-reference/{reference}
- Path params:
  - `reference` (string, required)
- Success (200): `BookingDto`, 404 if not found

### POST /api/bookings
- Method: POST
- Path: /api/bookings
- Headers: `Content-Type: application/json`
- Request body (JSON) — required fields marked:
  - `customerName` (string, required)
  - `customerEmail` (string, required)
  - `customerPhone` (string, required)
  - `serviceType` (string, required)
  - `preferredDate` (string, required) — `YYYY-MM-DD`
  - `preferredTime` (string, required) — `HH:mm:ss`
  - `durationMinutes` (integer, required)
  - `price` (number, required)
  - `currency` (string, optional, default `GBP`)
  - `bookingReference` (string, optional — server may generate)
  - `status` (string, optional)
- Example request:
```json
{
  "customerName":"Jane Doe",
  "customerEmail":"jane@example.com",
  "customerPhone":"07123456789",
  "serviceType":"Consultation",
  "preferredDate":"2026-02-01",
  "preferredTime":"14:30:00",
  "durationMinutes":30,
  "price":50.00,
  "currency":"GBP"
}
```
- Success (201): Created resource. Response body echoes created `BookingDto`. Response includes `Location` header `/api/bookings/{id}`.
- Errors:
  - 400 Bad Request — validation errors (e.g. missing required fields, invalid time format)
  - 500 Internal Server Error

### PUT /api/bookings/{id}
- Method: PUT
- Path: /api/bookings/{id}
- Purpose: full update — client MUST send the complete `BookingDto` in the request body; the path `id` must match the `id` field in the body.
- Headers: `Content-Type: application/json`
- Success: 204 No Content
- Errors:
  - 400 Bad Request — id mismatch or invalid payload
  - 404 Not Found — booking does not exist

### DELETE /api/bookings/{id}
- Method: DELETE
- Path: /api/bookings/{id}
- Success: 204 No Content
- Errors: 404 Not Found

### POST /api/bookings/{id}/confirm
- Method: POST
- Path: /api/bookings/{id}/confirm
- Headers: `Content-Type: application/json`
- Request body: `BookingConfirmDto`
  - `assignedLawyer` (string, optional)
  - `notes` (string, optional)
- Success (200):
```json
{ "message": "Booking confirmed and email sent successfully", "bookingId": "{id}", "emailSent": true }
```
- Errors: 404 if booking not found

### PATCH /api/bookings/{id}/payment-status
- Method: PATCH
- Path: /api/bookings/{id}/payment-status
- Headers: `Content-Type: application/json`
- Request body:
  - `paymentStatus` (string, required) — e.g. `paid`, `pending`, `failed`
  - `paymentMethod` (string, optional)
  - `paymentNotes` (string, optional)
- Example:
```json
{ "paymentStatus": "paid", "paymentMethod": "stripe", "paymentNotes": "Txn 12345" }
```
- Success (200): `{ "message": "Payment status updated successfully", "bookingId": "{id}" }`

### PUT /api/bookings/{id}/invoice
- Method: PUT
- Path: /api/bookings/{id}/invoice
- Request body (InvoiceDto):
  - `invoiceNumber` (string, required)
  - `bookingId` (GUID, required)
  - `subtotal` (number, required)
  - `totalAmount` (number, required)
  - `issueDate` (ISO datetime, required)
  - `dueDate` (ISO datetime, required)
  - `status` (string, optional)
- Success (200): `{ "message": "Invoice number updated successfully", "bookingId": "{id}" }`

### PUT /api/bookings/{id}/case-details
- Method: PUT
- Path: /api/bookings/{id}/case-details
- Request body (CaseDetailsDto)
  - `lawyerAssessment` (string, optional)
  - `caseComplexity` (string, optional)
  - `estimatedHours` (number, optional)
- Success (200): `{ "message": "Case details updated successfully", "bookingId": "{id}" }`

### GET /api/bookings/availability/{office}/{date}
- Method: GET
- Path: /api/bookings/availability/{office}/{date}
- Path params:
  - `office` (string, required) — office slug
  - `date` (string, required) — `YYYY-MM-DD`
- Success (200): `AvailabilityDto`:
```json
{
  "office":"leicester",
  "date":"2026-02-01",
  "availableSlots": ["09:00","09:30"],
  "bookedSlots": ["10:00"]
}
```

## Clients (Routes under /api/clients)

### GET /api/clients
- Method: GET
- Path: /api/clients
- Success (200): `{ "data": [ /* ClientDto */ ], "count": N }`

### GET /api/clients/{id}
- Method: GET
- Path: /api/clients/{id}
- Success: `ClientDto` or 404

### GET /api/clients/by-user/{userId}
- Method: GET
- Path: /api/clients/by-user/{userId}
- Success: `ClientDto` or 404

### POST /api/clients
- Method: POST
- Path: /api/clients
- Request body:
  - `userId` (GUID, required)
  - `fullName` (string, required)
  - `email` (string, required)
  - `phone` (string, optional)
  - `clientReference` (string, optional)
- Success (201): created `ClientDto` (Location header `/api/clients/{id}`)

### PUT /api/clients/{id}
- Method: PUT
- Path: /api/clients/{id}
- Request body: full `ClientDto` (id in path must match id in body)
- Success: 204 No Content

### DELETE /api/clients/{id}
- Method: DELETE
- Path: /api/clients/{id}
- Success: 204 No Content

## Cases (Routes under /api/cases)

### GET /api/cases
- Method: GET
- Path: /api/cases
- Success (200): list of `CaseDto`

### GET /api/cases/{id}
- Method: GET
- Path: /api/cases/{id}

### POST /api/cases
- Method: POST
- Path: /api/cases
- Request body: `CaseDto` (fields defined in backend DTO)
- Success (201): created case payload returned

### PUT /api/cases/{id}
- Method: PUT
- Path: /api/cases/{id}
- Request body: full `CaseDto`
- Success: 204 No Content

### DELETE /api/cases/{id}
- Method: DELETE
- Path: /api/cases/{id}
- Success: 204 No Content

## Inquiries (Routes under /inquiries)

### POST /inquiries
- Method: POST
- Path: /inquiries
- Request body:
  - `callerName` (string, required)
  - `callerPhone` (string, required)
  - `callerEmail` (string, optional)
  - `inquiryDetails` (string, required)
- Success (200): `{ "message": "Inquiry saved successfully", "inquiry": { /* dto */ } }`

### GET /inquiries
- Method: GET
- Path: /inquiries
- Success (200): `{ "inquiries": [ /* InquiryDto */ ], "count": N }`

### GET /inquiries/{id}
- Method: GET
- Path: /inquiries/{id}
- Success: `{ "inquiry": { /* InquiryDto */ } }`

### PUT /inquiries/{id}
- Method: PUT
- Path: /inquiries/{id}
- Purpose: full replace — include full `InquiryDto` with `id` matching path
- Success (200): `{ "message": "Inquiry updated successfully", "inquiry": { /* dto */ } }`

### PATCH /inquiries/{id}
- Method: PATCH
- Path: /inquiries/{id}
- Purpose: partial update — only include fields to change. Supported fields:
  - `status` (string)
  - `inquiryDetails` (string)
  - `callerName` (string)
- Example request:
```json
{ "status": "closed", "inquiryDetails": "Resolved via phone" }
```
- Success (200): `{ "message": "Inquiry updated successfully", "inquiry": { /* updated dto */ } }`

### DELETE /inquiries/{id}
- Method: DELETE
- Path: /inquiries/{id}
- Success: 200 `{ "message": "Inquiry deleted successfully" }`

## Email (Routes under /email)

### POST /email/send
- Method: POST
- Path: /email/send
- Request body:
  - `recipientEmail` (string, required)
  - `recipientName` (string, optional)
  - `subject` (string, required)
  - `htmlBody` (string, optional)
  - `textBody` (string, optional)
  - `emailType` (string, optional)
  - `relatedEntityType` (string, optional)
  - `relatedEntityId` (GUID, optional)
- Success (200): `{ "message": "..." }`
- Error (500): `{ "error": "..." }`

### GET /email/logs
- Method: GET
- Path: /email/logs
- Success: 501 Not Implemented — the endpoint currently returns a 501 with a message explaining logs listing is not implemented.

## Security (Routes under /security)

### GET /security/blocked-ips
- Method: GET
- Path: /security/blocked-ips
- Success (200):
```json
{ "blockedIPs": [ { "ip": "1.2.3.4", "reason": "brute force", "blockedAt": "2026-01-10T12:34:56Z", "blockedBy": "system" } ] }
```

### POST /security/block-ip
- Method: POST
- Path: /security/block-ip
- Headers: `Content-Type: application/json`
- Request body:
  - `ip` (string, required) — IPv4 dotted-decimal (validation enforced)
  - `reason` (string, required)
- Example:
```json
{ "ip": "203.0.113.12", "reason": "excessive failed logins" }
```
- Success (200):
```json
{ "message": "IP address blocked successfully", "blockedIP": { "ip":"203.0.113.12", "reason":"excessive failed logins", "blockedAt":"2026-01-10T...Z" } }
```

## Branches

### GET /branches
- Method: GET
- Path: /branches
- Query parameters:
  - `activeOnly` (boolean, optional, default true)
- Success (200): `{ "data": [ /* branches */ ], "count": N }`

### GET /branches/slug/{slug}
- Method: GET
- Path: /branches/slug/{slug}

### POST /branches/seed
- Method: POST
- Path: /branches/seed
- Purpose: seed demo branches (idempotent); intended for dev/test environments
- Success (200): `{ "message": "Seed completed.", "added": ["leicester","newcastle"] }`

## System

### GET /system/health
- Method: GET
- Path: /system/health
- Success (200): `{ "status":"ok", "timestamp":"2026-01-10T...Z" }`

### GET /system/settings
- Method: GET
- Path: /system/settings
- Success (200): returns an array of `SystemSettingDto` objects

## Users

### GET /users
- Method: GET
- Path: /users
- Note: currently not implemented — returns 501

## Headers & Authentication
- `Authorization: Bearer <jwt>` — include for protected endpoints.
- `X-Master-Key: <master-key>` — required for `POST /auth/init-superadmin`.
- `Content-Type: application/json` — include for JSON POST/PUT/PATCH requests.

## PUT vs PATCH guidance
- `PUT` = full resource replacement. Send the full DTO; path `id` must match body `id`.
- `PATCH` = partial update. Send only fields you want to change.

## Field naming & types
- JSON: camelCase.
- Dates: `YYYY-MM-DD`.
- DateTimes: RFC3339/ISO 8601 `YYYY-MM-DDTHH:mm:ssZ`.

## DTO source-of-truth
- The definitive field lists live in `src/Acplegal.Application/DTOs` in the repository — consult these DTO files when in doubt.

## Quick Examples

Login:
```bash
curl -X POST https://localhost:5001/auth/login -H "Content-Type: application/json" -d '{"email":"staff@acplegal.co.uk","password":"Secret123"}'
```

Create booking (curl):
```bash
curl -X POST https://localhost:5001/api/bookings -H "Content-Type: application/json" -d '{"customerName":"Jane Doe","customerEmail":"jane@example.com","customerPhone":"07123456789","serviceType":"Consultation","preferredDate":"2026-02-01","preferredTime":"14:30:00","durationMinutes":30,"price":50.00,"currency":"GBP"}'
```

---

File: [apidocument.md](apidocument.md)

- **POST /auth/refresh**
  - Body: `RefreshRequestDto` { RefreshToken }
  - Returns new tokens: 200 { access_token, refresh_token, user }

- **POST /auth/init-superadmin**
  - Body: `InitSuperAdminDto` { Email, Password, Name }
  - Requires `X-Master-Key` header for authorization

## Bookings

- Base route: `/api/bookings`

- **GET /api/bookings**
  - List bookings: 200 { data:[], count }

- **GET /api/bookings/{id}**
  - Returns `BookingDto` or 404

- **GET /api/bookings/by-reference/{reference}**
  - Returns `BookingDto` or 404

- **POST /api/bookings**
- **POST /auth/login**
  - Method: POST
  - Path: `/auth/login`
  - Auth: Public (no Authorization header required)
  - Headers: `Content-Type: application/json`
  - Request body schema (JSON):
    - `email` (string, required)
    - `password` (string, required)
    - `deviceInfo` (object, optional)
    - `trustDevice` (boolean, optional, default false)
  - Example request:
    ```json
    { "email": "staff@acplegal.co.uk", "password": "Secret123" }
    ```
  - Success response (200):
    ```json
    {
      "access_token": "<jwt>",
      "refresh_token": "<refresh-token>",
      "user": { /* SessionDto / StaffProfileDto depending on role */ }
    }
    ```
  - Errors:
    - 400 Bad Request: validation error or missing fields
    - 401 Unauthorized: invalid credentials
  - Body: `PaymentStatusDto` { PaymentStatus, PaymentMethod?, PaymentNotes? }

- **PUT /api/bookings/{id}/invoice**
  - Body: `InvoiceDto` { InvoiceNumber, BookingId, Subtotal, TotalAmount, IssueDate, DueDate, Status? }

- **PUT /api/bookings/{id}/case-details**
  - Body: `CaseDetailsDto` { LawyerAssessment?, CaseComplexity?, EstimatedHours? }

- **GET /api/bookings/availability/{office}/{date}**
  - `date` is parseable by server (ISO). Returns `AvailabilityDto`.

## Clients

- Base: `/api/clients`
- **GET /api/clients** List clients: 200 { data, count }
- **GET /api/clients/{id}** Get client by id
- **GET /api/clients/by-user/{userId}** Get client linked to user
- **POST /api/clients** Create `ClientDto` { Id, UserId, FullName, Email, Phone?, ClientReference }
- **PUT /api/clients/{id}** Update client
- **DELETE /api/clients/{id}** Delete client

## Cases

- Base: `/api/cases`
- **GET /api/cases** List
- **GET /api/cases/{id}** Get
- **POST /api/cases** Create `CaseDto`
- **PUT /api/cases/{id}** Update
- **DELETE /api/cases/{id}** Delete

## Inquiries

- Base: `/inquiries`
- **POST /inquiries** Create `InquiryDto` { CallerName, CallerPhone, CallerEmail?, InquiryDetails }
- **GET /inquiries** List
- **GET /inquiries/{id}** Get
- **PUT /inquiries/{id}** Update (full)
- **PATCH /inquiries/{id}** Partial update (status, details, caller name)
- **DELETE /inquiries/{id}** Delete

## Email

- Base: `/email`
- **POST /email/send**
  - Body: `SendEmailRequestDto` { RecipientEmail, RecipientName?, Subject, HtmlBody?, TextBody?, EmailType?, RelatedEntityType?, RelatedEntityId? }
  - Returns 200 { message } on success, 500 on failure.
- **GET /email/logs**: Not implemented (501)

## Security

- Base: `/security`
- **GET /security/blocked-ips**
  - Returns list of `BlockedIpDto` { Ip, Reason, BlockedAt, BlockedBy? }
- **POST /security/block-ip**
  - Body: `BlockIpRequest` { Ip (IPv4), Reason }
  - Returns 200 { message, blockedIP }

## Branches

- **GET /branches**
  - Query: `activeOnly` (default true)
  - Returns list of branches
- **GET /branches/slug/{slug}** Get by slug
- **POST /branches/seed** Seed demo branches (idempotent)

## System

- **GET /system/health** — 200 { status: "ok", timestamp }
- **GET /system/settings** — Returns public settings (via `ISystemService`)

## Users

- **GET /users** — Not implemented (501)

## Authentication & Headers
- Requirement: include `Authorization: Bearer <jwt>` header on ALL API requests except these public endpoints:
  - `POST /auth/login`
  - `POST /auth/signup`
  - `POST /auth/refresh`
  - `POST /auth/init-superadmin`
  - `GET /system/health`
- `X-Master-Key` header is required for `POST /auth/init-superadmin` in addition to being public.

## Validation Notes
- Booking `PreferredTime` expects a time string, e.g. `"14:30:00"`.
- `BlockIpRequest.Ip` validates IPv4 using regex; use dotted-decimal IPv4.

## Examples

- Login example request body:
  ```json
  { "email": "staff@acplegal.co.uk", "password": "Secret123" }
  ```

- Create booking example:
  ```json
  {
    "customerName":"Jane Doe",
    "customerEmail":"jane@example.com",
    "customerPhone":"07123456789",
    "serviceType":"Consultation",
    "preferredDate":"2026-02-01",
    "preferredTime":"14:30:00",
    "durationMinutes":30,
    "price":50.00,
    "currency":"GBP"
  }
  ```

---

File: [apidocument.md](apidocument.md)
