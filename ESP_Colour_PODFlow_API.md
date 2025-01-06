# ESP Colour PODFlow API Documentation

## About

When communicating with the API directly, `JSON` will be returned in all responses, including errors. If you are using one of our SDKs, the SDK will transform the JSON responses into language-specific Site Flow objects.

Any off-the-shelf HTTP client written in any language should be able to access the API easily.

---

## API Documentation

### Error Handling

The API returns a JSON error along with one of the following error codes:

| Error Code | Meaning |
|------------|---------|
| **400**    | Service Unavailable – We’re temporarily offline for maintenance. Please try again later. |
| **401**    | Unauthorized – Your API key is wrong. |
| **403**    | Forbidden – Your API key doesn’t allow access to this resource. |
| **404**    | Not Found – The specified resource could not be found. |
| **500**    | Internal Server Error – We had a problem with our server. Try again later. |
| **503**    | Service Unavailable – We’re temporarily offline for maintenance. Please try again later. |

---

## Authentication

To connect to the Submission API, all requests must be authenticated. Follow the guide below to authenticate against our API:

### TLS

Connecting to Site Flow RESTful APIs will require at least **TLS 1.2** for all HTTPS connections.

### Authorization Header

PODFlow RESTful API uses an HTTP `Authorization` header to pass authorization information. The format is:

```
x-oneflow-authorization: Token:Signature
```

- **Token**: Access key ID used to compute the signature.
- **Signature**: HMAC SHA256 of selected elements from the request.

### Required Headers

| Header                  | Description                             |
|-------------------------|-----------------------------------------|
| `x-oneflow-authorization` | Authentication token and signature.   |
| `x-oneflow-date`        | Timestamp used in the signature.        |
| `x-oneflow-algorithm`   | Hash algorithm used (e.g., `SHA256`).   |

### Example Headers

```plaintext
x-oneflow-authorization: 124213431243214:431c0baaac21060fbba3a8c35c74ff565ec0113f6031586b99d978ffb6686e5b
x-oneflow-date: 2022-03-10T17:16:18Z
x-oneflow-algorithm: SHA256
```

---

## Generating the Authorization Request Header

Below are code examples to generate the `x-oneflow-authorization` header.

### JavaScript

```javascript
const crypto = require('crypto');
const timestamp = new Date().toISOString();

const stringToSign = `${method} ${decodeURIComponent(path)} ${timestamp}`;
const hmac = crypto.createHmac('SHA256', secret);
hmac.update(stringToSign);
const signature = hmac.digest('hex');
const authHeader = `${token}:${signature}`;
```

### C#

```csharp
using System;
using System.Security.Cryptography;

string stringToSign = method + " " + Uri.UnescapeDataString(path) + " " + timestamp;
HMACSHA256 hmac = new HMACSHA256(Encoding.UTF8.GetBytes(secret));
byte[] signatureBytes = hmac.ComputeHash(Encoding.UTF8.GetBytes(stringToSign));
string signature = BitConverter.ToString(signatureBytes).Replace("-", "").ToLower();
string authHeader = token + ":" + signature;
```

### PHP

```php
<?php

$stringToSign = strtoupper($method) . ' ' . urldecode($path) . ' ' . $timestamp;
$signature = hash_hmac('sha256', $stringToSign, $secret);
$authHeader = $token . ':' . $signature;
?>
```

---

## Order Submission

### Endpoint

The order submission endpoint is an **asynchronous endpoint**:

| HTTP Method | URL                                      |
|-------------|------------------------------------------|
| `POST`      | `https://podgateway.handelgroup.co.uk/podreceiver` |

### Response

A response will be returned immediately after the order request is stored, regardless of its validity:

```json
{
  "id": "966bcc5e-8a95-43cc-a999-0e28484d8032",
  "orderReferenceId": "TestSubmission01",
  "createdAt": "2024-08-08T10:54:54Z"
}
```

#### Response Fields

| Field               | Description                                 |
|---------------------|---------------------------------------------|
| **id**              | The unique order identifier.               |
| **orderReferenceId**| Customer-provided order reference.         |
| **createdAt**       | Date and time of order submission request. |

---


## Order Structure

Below is a basic JSON order structure for submitting an order:

```json
{
  "destination": {},
  "orderData": {
    "items": [
      {
        "components": []
      }
    ],
    "shipments": [
      {
        "shipTo": {},
        "returnAddress": {},
        "carrier": {}
      }
    ]
  }
}
```

### Key Objects

| Object       | Description |
|--------------|-------------|
| **destination** | Always set to `hp.espcolour`. |
| **orderData** | Contains the main order data. |
| **items** | Array of order items. |
| **components** | Array of components (e.g., text or cover). |
| **shipments** | Array of shipments (destinations). |

---

## Cancel Order

### Endpoint

To cancel an order, use the following endpoint:

| HTTP Method | URL                          |
|-------------|------------------------------|
| `GET`       | `/order/cancel/{poRef}`      |

### Request

| Parameter | Description                                   |
|-----------|-----------------------------------------------|
| `poRef`   | Purchase order reference for the order to cancel. |

### Response

| Status Code | Description           |
|-------------|-----------------------|
| **200 OK**  | Success               |
| **400 Bad Request** | Error message. |

---

## Update Order

### Endpoint

To update the shipping information for an order, use the following endpoint:

| HTTP Method | URL                          |
|-------------|------------------------------|
| `POST`      | `/order/update`              |

### Request

| Header           | Value                     |
|------------------|---------------------------|
| Content-Type     | application/json          |

| Parameter | Description                                   |
|-----------|-----------------------------------------------|
| JSON Body | JSON object with order details to update.    |

#### Example Payload

```json
{
  "shipments": [
    {
      "carrier": {
        "alias": "ukhermesPackesdsdtsITD"
      },
      "cost": {
        "currency": "GBP",
        "value": 4.99
      },
      "shipmentIndex": 0,
      "shipTo": {
        "address1": "sd",
        "address2": "here test",
        "address3": "dfd",
        "companyName": "",
        "country": "United Kingdom",
        "isoCountry": "ss",
        "name": "sd",
        "phone": "",
        "postcode": "ssd",
        "state": "",
        "town": "where test sdancelled"
      }
    }
  ]
}
```

### Response

| Status Code | Description           |
|-------------|-----------------------|
| **200 OK**  | Success               |
| **400 Bad Request** | Error message. |

---


## Full Documentation

For detailed field definitions, advanced order structures, and additional features, refer to the extended documentation provided in the SDK or API guide.
