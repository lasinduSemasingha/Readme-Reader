# Damage / Waste Item API (Frontend expectations)

## Purpose

This document describes the API contract the frontend expects for "damage / waste" (stock adjustment) operations. The frontend's `DamageWasteEntry` component collects a batch of items and submits them as a single stock-adjustment request. The backend should expose endpoints that match the shapes and conventions used across the project (see `src/services/*` for examples).

## Conventions

- All endpoints return the project's ServiceResponse wrapper when applicable: `{ flag: boolean, message?: string, data?: any }`.
- Authorization: `Authorization: Bearer <token>` sent by the frontend via `src/services/api.ts` wrapper.
- Content-Type: `application/json` for JSON bodies.

## Key concepts frontend sends

- A single stock-adjustment (batch) representing multiple damaged/wasted items.
- Each item includes an identifier (either `itemId` or `itemCode`), quantity, unit cost (used to compute loss), and an optional reason.
- The frontend expects the backend to accept the batch, persist a record (id/reference) and apply inventory changes atomically.

## Types / Schemas

StockAdjustment (request body)

```json
{
  "id": "optional-client-id-or-empty",
  "date": "YYYY-MM-DD",
  "time": "HH:mm:ss", 
  "reference": "optional-reference-or-server-generated",
  "type": "damage" | "waste" | "expire" | "other",
  "reason": "Overall reason for the batch",
  "items": [
    {
      "itemId": 123,          // preferred when available
      "itemCode": "ITM001", // alternative if itemId is not present
      "name": "Sample Item", // optional, server can ignore
      "qty": 2,
      "unitCost": 100.0,      // cost per unit (used to compute loss)
      "unitPrice": 150.0,     // optional retail price
      "reason": "Broken in transit" // optional per-item reason
    }
  ]
}
```

Validation rules (expected):
- `items` must be a non-empty array.
- Each item must include `qty` > 0 and either `itemId` or `itemCode`.
- `unitCost` should be >= 0.

Response wrapper (success):

```json
{
  "flag": true,
  "message": "Stock adjustment recorded",
  "data": {
    "adjustmentId": "BATCH-2026-0001",
    "appliedAt": "2026-02-16T12:34:56Z"
  }
}
```

Error response example (conventional):

```json
{
  "flag": false,
  "message": "Validation failed",
  "data": {
    "errors": ["items cannot be empty", "item 0: qty must be > 0"]
  }
}
```

## Recommended Endpoints

1) Create (submit) a damage/waste batch

- Method: `POST`
- URL: `/api/StockAdjustment/register-adjustment`
- Description: Accepts a batch (StockAdjustment). The server persists the adjustment, atomically updates inventory quantities, and returns an `adjustmentId`.

Request example:

```json
POST /api/StockAdjustment/register-adjustment
Content-Type: application/json
Authorization: Bearer <token>

{ "date":"2026-02-16","time":"12:00:00","type":"damage","reason":"Broken during restock","items":[{"itemId":101,"qty":3,"unitCost":50.0,"reason":"Dropped"}]} 
```

Response example: (see Response wrapper above)

2) List / search adjustments

- Method: `GET`
- URL: `/api/StockAdjustment/get-all?from=YYYY-MM-DD&to=YYYY-MM-DD&type=damage&page=1&pageSize=50`
- Description: Returns list of adjustments matching filters. Returns ServiceResponse with `data` array.

3) Get adjustment by id

- Method: `GET`
- URL: `/api/StockAdjustment/get-by-id?id={adjustmentId}`

4) Cancel / reverse an adjustment

- Method: `POST`
- URL: `/api/StockAdjustment/cancel-adjustment`
- Body: `{ "adjustmentId": "BATCH-...", "reason": "Mistaken entry" }`
- Description: Reverses the inventory change if allowed by business rules and marks original record as canceled. Returns `{ flag, message, data }`.

5) Preview impact (optional)

- Method: `POST`
- URL: `/api/StockAdjustment/preview-impact`
- Body: same as register but does not persist — returns projection of resulting stock levels and total loss so frontend can show a confirmation.

## Frontend usage example (TypeScript)

Use the existing `apiFetch` wrapper in `src/services/api.ts`:

```ts
import { apiFetch } from '../services/api';

export async function submitDamageBatch(payload: any) {
  const endpoint = '/api/StockAdjustment/register-adjustment';
  const res = await apiFetch<any>(endpoint, { method: 'POST', body: JSON.stringify(payload) });
  if (!res || !res.flag) {
    const err: any = new Error(res?.message || 'Failed to submit damage batch');
    err.data = res;
    throw err;
  }
  return res.data; // { adjustmentId, appliedAt }
}
```

## Suggested server-side behaviors (for compatibility)

- The endpoint should accept both `itemId` and `itemCode`. If only `itemCode` is provided, resolve to `itemId`.
- The server should return a stable `adjustmentId` (e.g. `DMG-YYYYMMDD-XXXX` or `BATCH-...`) and the applied timestamp.
- Inventory updates should be atomic for the batch; if any item fails validation, the whole request should fail and return specific errors.
- When possible, return `data` even on non-2xx HTTP status if the server includes helpful payload (frontend's `apiFetch` tries to handle this pattern).

## Quick validation checklist frontend expects from API

- 200/201 with `{ flag: true, data: { adjustmentId } }` on success
- 4xx with `{ flag:false, message, data:{ errors: [...] } }` on validation failures
- Authorization failures should return 401/403
- Idempotency: if frontend re-submits same payload (e.g. retry), server may detect duplicate `reference` or client `id` and return the existing `adjustmentId` instead of duplicating.

---

File location: [src/guidelines/DamageItemAPI.md](src/guidelines/DamageItemAPI.md)

If you want, I can also scaffold a small `src/services/stockAdjustmentService.ts` wrapper using the `apiFetch` helper that the frontend can import directly. Want me to add that next?
