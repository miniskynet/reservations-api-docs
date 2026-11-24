# Get started with the Reservations API

> **Portfolio sample.** The Reservations API is fictional. This guide was written by Desira Wijesundara to show how she writes task-based developer documentation.

In this guide you'll make your first reservation through the API: find a product and a customer, reserve a table, confirm it, and keep your own system in sync. It takes about ten minutes.

**You'll need:**

- A sandbox API key. Find it in the dashboard under **Developers > API keys**. Sandbox keys start with `rk_test_`.
- A terminal with `curl`. The examples use Bash syntax; on Windows, use Git Bash or WSL.

Everything here runs in the sandbox, so nothing you create affects real bookings. Sandbox data resets every Sunday at 00:00 UTC.

## 1. Save your API key

Store your key in an environment variable so it doesn't end up in your shell history or your code:

```bash
export RES_API_KEY="rk_test_8d2f0c6a9b1e4f7a"
```

Replace the value with your own key. The rest of this guide reads it from `$RES_API_KEY`.

## 2. Check the connection

List the products you can reserve. This confirms your key works:

```bash
curl https://sandbox.api.example.com/v1/products \
  -H "Authorization: Bearer $RES_API_KEY"
```

You should see the sandbox products:

```json
{
  "data": [
    { "id": "prd_14", "name": "Private dining room", "capacity_per_slot": 1, "slot_minutes": 180 },
    { "id": "prd_20", "name": "Tasting menu ticket", "capacity_per_slot": 24, "slot_minutes": 150 },
    { "id": "prd_12", "name": "Window table for two", "capacity_per_slot": 6, "slot_minutes": 90 }
  ],
  "meta": { "page": 1, "size": 25, "total": 3 }
}
```

Note the ID of the product you want. This guide uses `prd_12`, the window table.

> **Got `401 Unauthorized`?** Check that the key starts with `rk_test_` and that the URL starts with `https://sandbox.`. Live keys don't work in the sandbox, and test keys don't work in production.

## 3. Find the customer

Every reservation belongs to a customer. Look one up by email address:

```bash
curl "https://sandbox.api.example.com/v1/customers?email=jordan.lee@example.com" \
  -H "Authorization: Bearer $RES_API_KEY"
```

```json
{
  "data": [
    {
      "id": "cus_310",
      "name": "Jordan Lee",
      "email": "jordan.lee@example.com",
      "phone": "+44 20 7946 0958",
      "created_at": "2030-01-14T08:30:00Z"
    }
  ],
  "meta": { "page": 1, "size": 25, "total": 1 }
}
```

The customer ID is `cus_310`. If `data` is empty, no customer has that email address. Customers are added in the dashboard, not through the API.

## 4. Create the reservation

Reserve the window table for two people at 10:00 UTC on 1 May 2030:

```bash
curl https://sandbox.api.example.com/v1/reservations \
  -H "Authorization: Bearer $RES_API_KEY" \
  -H "Content-Type: application/json" \
  -H "Idempotency-Key: $(uuidgen)" \
  -d '{
    "customer_id": "cus_310",
    "start": "2030-05-01T10:00:00Z",
    "items": [{ "product_id": "prd_12", "quantity": 2 }],
    "notes": "Window table requested"
  }'
```

The API returns `201 Created` with the new reservation:

```json
{
  "data": {
    "id": "res_1043",
    "status": "pending",
    "customer_id": "cus_310",
    "start": "2030-05-01T10:00:00Z",
    "items": [{ "product_id": "prd_12", "quantity": 2 }],
    "notes": "Window table requested",
    "created_at": "2030-04-20T09:15:00Z",
    "updated_at": "2030-04-20T09:15:00Z",
    "cancelled_at": null,
    "cancellation_reason": null
  }
}
```

Two things to notice:

- **The status is `pending`.** The table is held for you for 30 minutes. If you don't confirm it in that time, the places are released for other customers.
- **The request sent an `Idempotency-Key`.** If your connection drops and you're not sure the request worked, send it again with the *same* key. You'll get the original reservation back instead of a duplicate. Generate a new key for each new reservation.

## 5. Confirm the reservation

When the customer has committed, for example by paying, confirm the reservation:

```bash
curl -X PATCH https://sandbox.api.example.com/v1/reservations/res_1043 \
  -H "Authorization: Bearer $RES_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{ "status": "confirmed" }'
```

The response is the full reservation, now with `"status": "confirmed"`. A confirmed reservation holds its places until it's cancelled.

> **Already know the customer will commit?** Send `"status": "confirmed"` in step 4 and skip this step.

## 6. Handle the errors you're most likely to hit

Every error returns the same shape, so one handler covers them all:

```json
{
  "error": {
    "code": "insufficient_capacity",
    "message": "prd_12 has 1 place left at 2030-05-01T10:00:00Z; 2 were requested.",
    "field": "items[0].quantity",
    "request_id": "req_77b0aa"
  }
}
```

Branch on `code`, not `message`. Messages are written for people and can change.

| If you get | It means | Do this |
|---|---|---|
| `400 validation_error` | A field is missing or invalid. `field` names it. | Fix that field and send the request again. |
| `404 not_found` | An ID in your request doesn't exist. | Check the customer and product IDs. |
| `409 insufficient_capacity` | Not enough places at that time. Nothing was reserved. | Offer the customer another time, or a smaller quantity. |
| `429 rate_limited` | More than 600 requests in a minute. | Wait for the seconds in the `Retry-After` header, then retry. |

## 7. Keep your system in sync

If you mirror reservations in your own database, don't fetch every reservation on each run. Ask only for what changed since your last sync, including cancellations:

```bash
curl "https://sandbox.api.example.com/v1/reservations?updated_from=2030-04-30T00:00:00Z&include=items&size=100" \
  -H "Authorization: Bearer $RES_API_KEY"
```

Save the time you start each sync, and use it as `updated_from` next time. If `meta.total` is more than `size`, request the next `page` until you have them all.

## Next steps

- Browse every endpoint, field and error in the [API reference](https://miniskynet.github.io/reservations-api-docs/).
- Read [Cancel a reservation](https://miniskynet.github.io/reservations-api-docs/#tag/Reservations/operation/cancelReservation) before you build cancellations: cancelling can't be undone.
- When you're ready for production, swap in an `rk_live_` key and the base URL `https://api.example.com/v1`. Nothing else changes.
