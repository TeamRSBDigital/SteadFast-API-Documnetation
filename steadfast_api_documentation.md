# SteadFast Courier API Documentation

Official REST API documentation for integrating **SteadFast Courier** (`Packzy`) shipping, order booking, tracking, return requests, pickup scheduling, and payment reconciliation into your application.

- **Base URL:** `https://portal.packzy.com/api/v1`
- **Data Format:** `JSON`
- **Authentication:** Custom Header Keys (`api-key` & `secret-key`)

---

## Table of Contents

1. [Introduction & Authentication](#1-introduction--authentication)
2. [Endpoint Overview](#2-endpoint-overview)
3. [Orders](#3-orders)
   - [Create Order](#31-create-order)
   - [Create Bulk Order](#32-create-bulk-order)
4. [Returns & Pickups](#4-returns--pickups)
   - [Create Return Request](#41-create-return-request)
   - [Send Pickup Request](#42-send-pickup-request)
   - [Single Return Request View](#43-single-return-request-view)
   - [Get Return Requests](#44-get-return-requests)
5. [Payments & Balance](#5-payments--balance)
   - [Get Payments](#51-get-payments)
   - [Get Payments Single](#52-get-payments-single)
   - [Check Balance](#53-check-balance)
6. [Tracking & Status](#6-tracking--status)
   - [Check Status by CID (Consignment ID)](#61-check-status-by-cid-consignment-id)
   - [Check Status by Invoice / Tracking Code](#62-check-status-by-invoice--tracking-code)
   - [Get Trackings By Invoice](#63-get-trackings-by-invoice)
7. [Reference & Utilities](#7-reference--utilities)
   - [Get Police Station](#71-get-police-station)
   - [Fraud Check](#72-fraud-check)
8. [Delivery Status Reference](#8-delivery-status-reference)
9. [Webhooks & Error Handling](#9-webhooks--error-handling)

---

## 1. Introduction & Authentication

Every API request requires your merchant `api-key` and `secret-key` passed in the HTTP request headers. You can obtain these credentials from your SteadFast Merchant Dashboard or by contacting SteadFast support.

### Required Headers

| Header | Type | Required | Description |
| :--- | :--- | :--- | :--- |
| `api-key` | `string` | **Yes** | Your merchant API Key |
| `secret-key` | `string` | **Yes** | Your merchant Secret Key |
| `Content-Type` | `string` | **Yes** | Must be `application/json` |

---

## 2. Endpoint Overview

| Method | Endpoint | Description |
| :--- | :--- | :--- |
| `POST` | `/create_order` | Create a single parcel consignment |
| `POST` | `/create_order/bulk-order` | Create up to 500 parcel consignments in one request |
| `POST` | `/create_return_request` | Submit a return request for a consignment |
| `POST` | `/create_pickup_request` | Request a rider pickup from your merchant address |
| `GET` | `/get_return_request/{id}` | Retrieve details of a single return request |
| `GET` | `/get_return_requests` | List all return requests |
| `GET` | `/payments` | List all merchant payment settlements |
| `GET` | `/payments/{payment_id}` | Retrieve single payment details with linked consignments |
| `GET` | `/get_balance` | Check current merchant account balance |
| `GET` | `/status_by_cid/{consignment_id}` | Check delivery status using Consignment ID |
| `GET` | `/status_by_invoice/{invoice}` | Check delivery status using Merchant Invoice ID |
| `GET` | `/status_by_trackingcode/{code}` | Check delivery status using Tracking Code |
| `GET` | `/trackings_by_invoice/{invoice}` | Retrieve full chronological tracking logs by Invoice ID |
| `GET` | `/police_stations` | Retrieve all supported districts and police stations (thanas) |
| `GET` | `/fraud_check/{phone}` | Check customer delivery and cancellation history |

---

## 3. Orders

### 3.1 Create Order

Creates a single delivery order/consignment.

- **Method:** `POST`
- **URL:** `https://portal.packzy.com/api/v1/create_order`

#### Request Body Parameters

| Field | Type | Required | Validation / Notes |
| :--- | :--- | :--- | :--- |
| `invoice` | `string` | **Yes** | Unique merchant invoice ID (alphanumeric, hyphens, underscores) |
| `recipient_name` | `string` | **Yes** | Recipient full name (max 100 characters) |
| `recipient_phone` | `string` | **Yes** | Recipient mobile number (11 digits, e.g., `01309055966`) |
| `recipient_address` | `string` | **Yes** | Full delivery address (max 250 characters) |
| `cod_amount` | `numeric` | **Yes** | Cash on Delivery amount in BDT (`>= 0`) |
| `alternative_phone` | `string` | No | Secondary 11-digit contact number |
| `recipient_email` | `string` | No | Valid email address |
| `note` | `string` | No | Special delivery instructions for the rider |
| `item_description` | `string` | No | Description of parcel contents |
| `total_lot` | `integer` | No | Number of parcels/lots (default: `1`) |
| `delivery_type` | `integer` | No | `0` = Home Delivery (default), `1` = Hub Pickup |

#### Example Request

```bash
curl -X POST "https://portal.packzy.com/api/v1/create_order" \
  -H "api-key: YOUR_API_KEY" \
  -H "secret-key: YOUR_SECRET_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "invoice": "INV-1001",
    "recipient_name": "Badrul Hasan Sajib",
    "recipient_phone": "01309055966",
    "recipient_address": "Dhanmondi, Zigatola, Dhaka",
    "cod_amount": 3010,
    "note": "Call before delivery."
  }'
```

#### Example Response (`200 OK`)

```json
{
  "status": 200,
  "message": "Consignment has been created successfully.",
  "consignment": {
    "consignment_id": 131822154,
    "invoice": "INV-1001",
    "tracking_code": "7DB724A65D4",
    "recipient_name": "Badrul Hasan Sajib",
    "recipient_phone": "01309055966",
    "recipient_address": "Dhanmondi, Zigatola, Dhaka",
    "cod_amount": 3010,
    "status": "in_review",
    "note": "Call before delivery.",
    "created_at": "2026-03-21T07:05:31.000000Z",
    "updated_at": "2026-03-21T07:05:31.000000Z"
  }
}
```

---

### 3.2 Create Bulk Order

Books multiple orders in a single batch (maximum **500 orders** per request).

- **Method:** `POST`
- **URL:** `https://portal.packzy.com/api/v1/create_order/bulk-order`

#### Request Body

An array of order objects following the same schema as [Create Order](#31-create-order).

```bash
curl -X POST "https://portal.packzy.com/api/v1/create_order/bulk-order" \
  -H "api-key: YOUR_API_KEY" \
  -H "secret-key: YOUR_SECRET_KEY" \
  -H "Content-Type: application/json" \
  -d '[
    {
      "invoice": "INV-1001",
      "recipient_name": "Badrul Hasan Sajib",
      "recipient_phone": "01309055966",
      "recipient_address": "Dhanmondi, Zigatola, Dhaka",
      "cod_amount": 3010
    },
    {
      "invoice": "INV-1002",
      "recipient_name": "Jane Smith",
      "recipient_phone": "01711111111",
      "recipient_address": "House 44, Road 2/A, Dhanmondi, Dhaka 1209",
      "cod_amount": 1500
    }
  ]'
```

#### Example Response (`200 OK`)

> **Note:** A bulk request returns `200 OK` even if individual items fail validation. Always inspect the `status` field (`"success"` or `"error"`) for each item in `data`.

```json
{
  "status": 200,
  "message": "We have a response for you.",
  "data": [
    {
      "invoice": "INV-1001",
      "recipient_name": "Badrul Hasan Sajib",
      "recipient_phone": "01309055966",
      "recipient_address": "Dhanmondi, Zigatola, Dhaka",
      "cod_amount": "3010",
      "consignment_id": 11543968,
      "tracking_code": "B025A038",
      "status": "success"
    }
  ]
}
```

---

## 4. Returns & Pickups

### 4.1 Create Return Request

Requests a parcel to be returned to the merchant.

- **Method:** `POST`
- **URL:** `https://portal.packzy.com/api/v1/create_return_request`

#### Request Body Parameters

| Field | Type | Required | Description |
| :--- | :--- | :--- | :--- |
| `consignment_id` | `integer` | Conditional | Provide at least one of `consignment_id`, `invoice`, or `tracking_code` |
| `invoice` | `string` | Conditional | Merchant invoice reference |
| `tracking_code` | `string` | Conditional | SteadFast tracking code |
| `reason` | `string` | No | Reason for requesting the return |

#### Example Request

```bash
curl -X POST "https://portal.packzy.com/api/v1/create_return_request" \
  -H "api-key: YOUR_API_KEY" \
  -H "secret-key: YOUR_SECRET_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "consignment_id": 160757328,
    "reason": "Customer requested return"
  }'
```

---

### 4.2 Send Pickup Request

Requests a SteadFast rider to pick up parcels from your location.

- **Method:** `POST`
- **URL:** `https://portal.packzy.com/api/v1/create_pickup_request`

#### Request Body Parameters

| Field | Type | Required | Description |
| :--- | :--- | :--- | :--- |
| `address_id` | `integer` | Optional | Saved merchant pickup address ID |
| `police_station_id` | `integer` | **Yes** | ID from [Get Police Station](#71-get-police-station) |
| `address` | `string` | **Yes** | Pickup street address |
| `contact_number` | `string` | **Yes** | Contact phone number for the pickup rider |
| `estim_qty` | `integer` | No | Estimated number of parcels |
| `note` | `string` | No | Additional pickup instructions |

#### Example Request

```bash
curl -X POST "https://portal.packzy.com/api/v1/create_pickup_request" \
  -H "api-key: YOUR_API_KEY" \
  -H "secret-key: YOUR_SECRET_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "address_id": 5,
    "police_station_id": 1,
    "address": "Dhanmondi, Zigatola 2/A",
    "contact_number": "01722743076",
    "estim_qty": 12,
    "note": "Parcels ready after 3 PM"
  }'
```

---

### 4.3 Single Return Request View

Retrieves details of a specific return request by its ID.

- **Method:** `GET`
- **URL:** `https://portal.packzy.com/api/v1/get_return_request/{id}`

```bash
curl -X GET "https://portal.packzy.com/api/v1/get_return_request/1" \
  -H "api-key: YOUR_API_KEY" \
  -H "secret-key: YOUR_SECRET_KEY"
```

---

### 4.4 Get Return Requests

Retrieves a list of all return requests associated with your merchant account.

- **Method:** `GET`
- **URL:** `https://portal.packzy.com/api/v1/get_return_requests`

```bash
curl -X GET "https://portal.packzy.com/api/v1/get_return_requests" \
  -H "api-key: YOUR_API_KEY" \
  -H "secret-key: YOUR_SECRET_KEY"
```

---

## 5. Payments & Balance

### 5.1 Get Payments

Retrieves a list of all payment settlements made to your merchant account.

- **Method:** `GET`
- **URL:** `https://portal.packzy.com/api/v1/payments`

```bash
curl -X GET "https://portal.packzy.com/api/v1/payments" \
  -H "api-key: YOUR_API_KEY" \
  -H "secret-key: YOUR_SECRET_KEY"
```

---

### 5.2 Get Payments Single

Retrieves detailed settlement information for a single payment, including every consignment included in that payout.

- **Method:** `GET`
- **URL:** `https://portal.packzy.com/api/v1/payments/{payment_id}`

#### Path Parameters

| Parameter | Type | Description |
| :--- | :--- | :--- |
| `payment_id` | `string / int` | Payment identifier (e.g., `SFC-23618228` or numeric ID) |

#### Example Request

```bash
curl -X GET "https://portal.packzy.com/api/v1/payments/SFC-23618228" \
  -H "api-key: YOUR_API_KEY" \
  -H "secret-key: YOUR_SECRET_KEY"
```

#### Example Response (`200 OK`)

```json
{
  "status": 1,
  "alertClass": "success",
  "message": "Fetched successfully!",
  "payment": {
    "payment_id": "SFC-23618228",
    "amount": 1,
    "method": "Cash",
    "due_bills": 0,
    "paid_bills": 0,
    "charges": 0,
    "total": 1,
    "status_label": "ready",
    "created_at": "2025-02-27 16:08:24",
    "ready_at": "2025-02-27 16:27:20",
    "paid_at": null,
    "consignments": [
      {
        "consignment_id": 100544401,
        "invoice": "FAM00001",
        "tracking_code": "5FE2F9127",
        "recipient_name": "Badrul Hasan Sajib",
        "recipient_phone": "01309055966",
        "recipient_address": "Gangachara, Rangpur",
        "recipient_email": null,
        "alternative_phone": null,
        "item_description": null,
        "total_lot": 1,
        "cod_amount": 1,
        "status": "delivered",
        "note": "",
        "created_at": "2024-09-28T09:09:38.000000Z",
        "updated_at": "2025-02-27T10:08:24.000000Z"
      },
      {
        "consignment_id": 98339475,
        "invoice": "240914-20",
        "tracking_code": "5DC8A93D5BB",
        "recipient_name": "Test Test",
        "recipient_phone": "01309055966",
        "recipient_address": "Test,Test-",
        "recipient_email": null,
        "alternative_phone": null,
        "item_description": null,
        "total_lot": 1,
        "cod_amount": 0,
        "status": "partial_delivered",
        "note": "",
        "created_at": "2024-09-14T10:00:46.000000Z",
        "updated_at": "2025-02-27T10:08:24.000000Z"
      }
    ]
  }
}
```

---

### 5.3 Check Balance

Retrieves the current available balance in your SteadFast merchant account.

- **Method:** `GET`
- **URL:** `https://portal.packzy.com/api/v1/get_balance`

#### Example Request

```bash
curl -X GET "https://portal.packzy.com/api/v1/get_balance" \
  -H "api-key: YOUR_API_KEY" \
  -H "secret-key: YOUR_SECRET_KEY"
```

#### Example Response (`200 OK`)

```json
{
  "status": 200,
  "current_balance": -10
}
```

---

## 6. Tracking & Status

### 6.1 Check Status by CID (Consignment ID)

Retrieves the current delivery status of a parcel using its SteadFast `consignment_id`.

- **Method:** `GET`
- **URL:** `https://portal.packzy.com/api/v1/status_by_cid/{consignment_id}`

#### Example Request

```bash
curl -X GET "https://portal.packzy.com/api/v1/status_by_cid/98339475" \
  -H "api-key: YOUR_API_KEY" \
  -H "secret-key: YOUR_SECRET_KEY"
```

#### Example Response (`200 OK`)

```json
{
  "status": 200,
  "delivery_status": "partial_delivered"
}
```

---

### 6.2 Check Status by Invoice / Tracking Code

You can also query the current delivery status using your own `invoice` ID or SteadFast's `tracking_code`:

- **By Invoice:** `GET https://portal.packzy.com/api/v1/status_by_invoice/{invoice}`
- **By Tracking Code:** `GET https://portal.packzy.com/api/v1/status_by_trackingcode/{tracking_code}`

Both endpoints return the same JSON structure as `/status_by_cid/{consignment_id}`.

---

### 6.3 Get Trackings By Invoice

Retrieves the complete chronological audit and tracking log for a consignment using the merchant `invoice` ID.

- **Method:** `GET`
- **URL:** `https://portal.packzy.com/api/v1/trackings_by_invoice/{invoice}`

#### Example Request

```bash
curl -X GET "https://portal.packzy.com/api/v1/trackings_by_invoice/INV0EEBA2A" \
  -H "api-key: YOUR_API_KEY" \
  -H "secret-key: YOUR_SECRET_KEY"
```

#### Example Response (`200 OK`)

```json
{
  "status": 200,
  "tracking": [
    {
      "id": 1138890547,
      "consignment_id": 170577767,
      "sender_id": null,
      "tracking_type": 1,
      "icon": null,
      "text": "Consignment created by Sender(API).",
      "user_type": null,
      "user_id": 37722,
      "deliveryman_id": null,
      "admin_id": null,
      "updated_by": null,
      "confirm_status": 0,
      "status": 1,
      "hide_sender": 0,
      "created_at": "2025-09-04T09:46:19.000000Z",
      "updated_at": "2025-09-04T09:46:19.000000Z"
    },
    {
      "id": 1139149020,
      "consignment_id": 170577767,
      "sender_id": 37722,
      "tracking_type": 10,
      "icon": null,
      "text": "Consignment status has been updated as Pending",
      "user_type": null,
      "user_id": null,
      "deliveryman_id": null,
      "admin_id": 1438,
      "updated_by": null,
      "confirm_status": 0,
      "status": 1,
      "hide_sender": 0,
      "created_at": "2025-09-04T11:51:07.000000Z",
      "updated_at": "2025-09-04T11:51:07.000000Z"
    },
    {
      "id": 1938322443,
      "consignment_id": 170577767,
      "sender_id": 37722,
      "tracking_type": 9,
      "icon": null,
      "text": "Consignment status has been updated as Cancelled",
      "user_type": null,
      "user_id": null,
      "deliveryman_id": null,
      "admin_id": 1438,
      "updated_by": null,
      "confirm_status": 0,
      "status": 1,
      "hide_sender": 0,
      "created_at": "2026-04-13T11:39:13.000000Z",
      "updated_at": "2026-04-13T11:39:13.000000Z"
    }
  ]
}
```

---

## 7. Reference & Utilities

### 7.1 Get Police Station

Retrieves all supported districts across Bangladesh along with their nested `policestations` (thanas/upazilas), hub mappings, and `big_parcel` eligibility flags.

- **Method:** `GET`
- **URL:** `https://portal.packzy.com/api/v1/police_stations`

#### Example Request

```bash
curl -X GET "https://portal.packzy.com/api/v1/police_stations" \
  -H "api-key: YOUR_API_KEY" \
  -H "secret-key: YOUR_SECRET_KEY"
```

#### Example Response (`200 OK`)

```json
{
  "status": "success",
  "data": [
    {
      "id": 1,
      "name": "Dhaka City",
      "policestations": [
        {
          "id": 1,
          "name": "Dhanmondi",
          "hub_id": 1,
          "district_id": 1,
          "ps_type": 1,
          "big_parcel": 1,
          "post_code": null,
          "address": null,
          "search_tags": "Dhanmondi, ধানমন্ডি",
          "phone": null,
          "admin_id": null,
          "updated_by": 65,
          "status": 1,
          "created_at": "2020-06-01T15:15:52.000000Z",
          "updated_at": "2023-02-02T09:36:34.000000Z"
        },
        {
          "id": 44,
          "name": "Uttara",
          "hub_id": 10,
          "district_id": 1,
          "ps_type": 1,
          "big_parcel": 0,
          "post_code": null,
          "address": null,
          "search_tags": "Uttara, uttora, উত্তরা",
          "phone": null,
          "admin_id": 1,
          "updated_by": 65,
          "status": 1,
          "created_at": "2020-06-02T00:17:20.000000Z",
          "updated_at": "2023-01-26T23:52:23.000000Z"
        }
      ]
    },
    {
      "id": 2,
      "name": "Chittagong",
      "policestations": [
        {
          "id": 53,
          "name": "Akbar Shah",
          "hub_id": 151,
          "district_id": 2,
          "ps_type": 1,
          "big_parcel": 1,
          "post_code": null,
          "address": null,
          "search_tags": "Akbar Shah, আকবার শাহ, আকবর শাহ,",
          "phone": null,
          "admin_id": 1,
          "updated_by": 65,
          "status": 1,
          "created_at": "2020-06-02T23:29:20.000000Z",
          "updated_at": "2023-02-02T09:55:12.000000Z"
        }
      ]
    }
  ]
}
```

---

### 7.2 Fraud Check

Retrieves historical delivery success and cancellation metrics for a recipient phone number across the SteadFast network.

- **Method:** `GET`
- **URL:** `https://portal.packzy.com/api/v1/fraud_check/{phone}`

#### Path Parameters

| Parameter | Type | Description |
| :--- | :--- | :--- |
| `phone` | `string` | Customer 11-digit phone number (e.g., `01309055966`) |

#### Example Request

```bash
curl -X GET "https://portal.packzy.com/api/v1/fraud_check/01309055966" \
  -H "api-key: YOUR_API_KEY" \
  -H "secret-key: YOUR_SECRET_KEY"
```

#### Example Response (`200 OK`)

```json
{
  "total_parcels": 24,
  "total_delivered": 9,
  "total_cancelled": 15,
  "total_fraud_reports": []
}
```

---

## 8. Delivery Status Reference

| Status Value | State Category | Description |
| :--- | :--- | :--- |
| `in_review` | In Progress | Order placed and awaiting initial review |
| `pending` | In Progress | Parcel is in transit / not yet delivered or cancelled |
| `hold` | In Progress | Delivery temporarily on hold |
| `delivered_approval_pending` | Awaiting Approval | Delivered by rider, awaiting admin verification |
| `partial_delivered_approval_pending` | Awaiting Approval | Partially delivered, awaiting admin verification |
| `cancelled_approval_pending` | Awaiting Approval | Marked cancelled, awaiting admin verification |
| `unknown_approval_pending` | Awaiting Approval | Unclassified state awaiting support review |
| `delivered` | **Final** | Delivered and merchant balance updated |
| `partial_delivered` | **Final** | Partially delivered and merchant balance updated |
| `cancelled` | **Final** | Cancelled and merchant balance updated |
| `unknown` | In Progress | Unknown status; contact SteadFast support |

---

## 9. Webhooks & Error Handling

### Webhooks

SteadFast can push real-time delivery status updates to a callback URL registered in your merchant dashboard.

- **Authentication:** SteadFast includes an `Authorization: Bearer <YOUR_WEBHOOK_TOKEN>` header on webhook POST requests.
- **Idempotency:** Duplicate webhook events may occasionally be delivered. Deduplicate incoming events using `consignment_id` and `updated_at`.

### Common HTTP Status Codes

| HTTP Code | Meaning | Resolution |
| :--- | :--- | :--- |
| `200` | **OK** | Request succeeded; inspect JSON `status` field for details |
| `401` | **Unauthorized** | Verify your `api-key` and `secret-key` headers |
| `422` | **Unprocessable Entity** | Validation error (e.g., duplicate `invoice` or invalid 11-digit phone) |
| `429` | **Too Many Requests** | Rate limit exceeded; implement exponential backoff or caching |
| `500` / `503` | **Server Error** | Temporary SteadFast server issue; verify status before retrying writes |