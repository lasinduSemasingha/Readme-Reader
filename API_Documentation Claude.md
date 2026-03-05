# API Documentation

> **Version:** 1.0  
> **Format:** REST / JSON  
> **Base URL:** `/api`

---

## Table of Contents

1. [Overview](#overview)
2. [Response Format](#response-format)
3. [Bank](#bank)
4. [Bank Transfer](#bank-transfer)
5. [Batch](#batch)
6. [Brand](#brand)
7. [Sub Brand](#sub-brand)
8. [Category](#category)
9. [Cheque Payment](#cheque-payment)
10. [Customer](#customer)
11. [Damage Stock](#damage-stock)
12. [Invoice](#invoice)
13. [Invoice Item](#invoice-item)
14. [Item](#item)
15. [Item Type](#item-type)
16. [Payment](#payment)
17. [Role](#role)
18. [Service](#service)
19. [Supplier](#supplier)
20. [User](#user)
21. [Data Models](#data-models)

---

## Overview

This API follows RESTful conventions. All request and response bodies use JSON. Every endpoint that returns data wraps it in a standard `ServiceResponse` object.

---

## Response Format

All endpoints return a consistent response envelope:

```json
{
  "flag": true,
  "message": "Operation successful",
  "data": { }
}
```

| Field     | Type    | Description                                      |
|-----------|---------|--------------------------------------------------|
| `flag`    | boolean | `true` if the operation succeeded, `false` if not |
| `message` | string  | Human-readable status or error message           |
| `data`    | any     | The returned payload (object, array, or null)    |

---

## Bank

### Create Bank

**`POST`** `/api/Bank/create-bank`

Creates a new bank record.

**Request Body**

```json
{
  "bankName": "string",
  "bankCode": "string"
}
```

| Field      | Type   | Required | Description           |
|------------|--------|----------|-----------------------|
| `bankName` | string | No       | Full name of the bank |
| `bankCode` | string | No       | Short code identifier |

**Response** — `ServiceResponse`

---

### Get All Banks

**`GET`** `/api/Bank/get-all-banks`

Retrieves a list of all registered banks.

**Response** — `ServiceResponse` with array of bank records in `data`.

---

## Bank Transfer

### Create Bank Transfer

**`POST`** `/api/BankTransfer/create-bank-transfer`

Records a bank transfer linked to a payment.

**Request Body**

```json
{
  "paymentId": 0,
  "bankId": 0,
  "referenceNumber": "string",
  "transferAmount": 0.0,
  "transferDate": "2024-01-01T00:00:00Z"
}
```

| Field             | Type     | Required | Description                        |
|-------------------|----------|----------|------------------------------------|
| `paymentId`       | integer  | No       | ID of the associated payment       |
| `bankId`          | integer  | No       | ID of the bank involved            |
| `referenceNumber` | string   | No       | Bank reference or transaction code |
| `transferAmount`  | double   | No       | Amount transferred                 |
| `transferDate`    | datetime | No       | Date and time of the transfer      |

**Response** — `ServiceResponse`

---

## Batch

### Register Batch

**`POST`** `/api/Batch/register-batch`

Registers a new stock batch for an item from a supplier.

**Request Body**

```json
{
  "batchCode": "string",
  "itemId": 0,
  "supplierId": 0,
  "quantity": 0.0,
  "purchasePrice": 0.0,
  "retailPrice": 0.0,
  "wholesalePrice": 0.0,
  "expiryDate": "2024-01-01T00:00:00Z"
}
```

| Field            | Type     | Required | Description                         |
|------------------|----------|----------|-------------------------------------|
| `batchCode`      | string   | No       | Unique code for this batch          |
| `itemId`         | integer  | No       | ID of the item being stocked        |
| `supplierId`     | integer  | No       | ID of the supplying company         |
| `quantity`       | double   | No       | Number of units in the batch        |
| `purchasePrice`  | double   | No       | Cost price per unit                 |
| `retailPrice`    | double   | No       | Retail selling price per unit       |
| `wholesalePrice` | double   | No       | Wholesale selling price per unit    |
| `expiryDate`     | datetime | No       | Expiry date of the batch (optional) |

**Response** — `200 OK` (no body)

---

## Brand

### Register Brand

**`POST`** `/api/Brand/register-brand`

Creates a new brand.

**Request Body**

```json
{
  "brandName": "string",
  "brandDescription": "string"
}
```

| Field              | Type   | Required | Description              |
|--------------------|--------|----------|--------------------------|
| `brandName`        | string | No       | Name of the brand        |
| `brandDescription` | string | No       | Description of the brand |

**Response** — `ServiceResponse`

---

### Get All Brands

**`GET`** `/api/Brand/get-all-brand-details`

Retrieves a list of all brands.

**Response** — `ServiceResponse` with array of brand records in `data`.

---

### Update Brand

**`PUT`** `/api/Brand/update-brand/{id}`

Updates an existing brand.

**Path Parameters**

| Parameter | Type    | Required | Description         |
|-----------|---------|----------|---------------------|
| `id`      | integer | Yes      | ID of the brand     |

**Request Body**

```json
{
  "brandName": "string",
  "brandDescription": "string"
}
```

**Response** — `ServiceResponse`

---

### Delete Brand

**`DELETE`** `/api/Brand/delete-brand/{id}`

Deletes a brand by ID.

**Path Parameters**

| Parameter | Type    | Required | Description     |
|-----------|---------|----------|-----------------|
| `id`      | integer | Yes      | ID of the brand |

**Response** — `ServiceResponse`

---

## Sub Brand

### Register Sub Brand

**`POST`** `/api/SubBrand/register-sub-brand`

Creates a new sub-brand under a parent brand.

**Request Body**

```json
{
  "brandId": 0,
  "subBrandName": "string",
  "subBrandDescription": "string"
}
```

| Field                | Type    | Required | Description                    |
|----------------------|---------|----------|--------------------------------|
| `brandId`            | integer | No       | ID of the parent brand         |
| `subBrandName`       | string  | No       | Name of the sub-brand          |
| `subBrandDescription`| string  | No       | Description of the sub-brand   |

**Response** — `ServiceResponse`

---

### Get All Sub Brands

**`GET`** `/api/SubBrand/get-all-sub-brand-details`

Retrieves all sub-brands.

**Response** — `ServiceResponse` with array of sub-brand records in `data`.

---

### Update Sub Brand

**`PUT`** `/api/SubBrand/update-sub-brand/{id}`

Updates an existing sub-brand.

**Path Parameters**

| Parameter | Type    | Required | Description         |
|-----------|---------|----------|---------------------|
| `id`      | integer | Yes      | ID of the sub-brand |

**Request Body**

```json
{
  "brandId": 0,
  "subBrandName": "string",
  "subBrandDescription": "string"
}
```

**Response** — `ServiceResponse`

---

### Delete Sub Brand

**`DELETE`** `/api/SubBrand/delete-sub-brand/{id}`

Deletes a sub-brand by ID.

**Path Parameters**

| Parameter | Type    | Required | Description         |
|-----------|---------|----------|---------------------|
| `id`      | integer | Yes      | ID of the sub-brand |

**Response** — `ServiceResponse`

---

## Category

### Register Category

**`POST`** `/api/Category/register-category`

Creates a new product category.

**Request Body**

```json
{
  "categoryName": "string",
  "description": "string",
  "status": "string"
}
```

| Field          | Type   | Required | Description                     |
|----------------|--------|----------|---------------------------------|
| `categoryName` | string | No       | Name of the category            |
| `description`  | string | No       | Description of the category     |
| `status`       | string | No       | Active/inactive status          |

**Response** — `ServiceResponse`

---

### Get All Categories

**`GET`** `/api/Category/get-all-categories`

Retrieves all categories.

**Response** — `ServiceResponse` with array of category records in `data`.

---

### Get Category by ID

**`GET`** `/api/Category/get-category/{id}`

Retrieves a single category by its ID.

**Path Parameters**

| Parameter | Type    | Required | Description         |
|-----------|---------|----------|---------------------|
| `id`      | integer | Yes      | ID of the category  |

**Response** — `ServiceResponse` with category details in `data`.

---

### Update Category

**`PUT`** `/api/Category/update-category/{id}`

Updates an existing category.

**Path Parameters**

| Parameter | Type    | Required | Description        |
|-----------|---------|-----------|--------------------|
| `id`      | integer | Yes       | ID of the category |

**Request Body**

```json
{
  "categoryName": "string",
  "description": "string",
  "status": "string"
}
```

**Response** — `ServiceResponse`

---

### Delete Category

**`DELETE`** `/api/Category/delete-category/{id}`

Deletes a category by ID.

**Path Parameters**

| Parameter | Type    | Required | Description        |
|-----------|---------|----------|--------------------|
| `id`      | integer | Yes      | ID of the category |

**Response** — `ServiceResponse`

---

## Cheque Payment

### Create Cheque Payment

**`POST`** `/api/ChequePayment/create-cheque-payment`

Records a cheque payment linked to a payment record.

**Request Body**

```json
{
  "paymentId": 0,
  "chequeNumber": "string",
  "chequeAmount": 0.0,
  "brankId": 0,
  "chequeDate": "2024-01-01T00:00:00Z"
}
```

| Field          | Type     | Required | Description                      |
|----------------|----------|----------|----------------------------------|
| `paymentId`    | integer  | No       | ID of the linked payment         |
| `chequeNumber` | string   | No       | Cheque number                    |
| `chequeAmount` | double   | No       | Amount written on the cheque     |
| `brankId`      | integer  | No       | ID of the issuing bank           |
| `chequeDate`   | datetime | No       | Date of the cheque               |

**Response** — `ServiceResponse`

---

### Get Cheque Details

**`GET`** `/api/ChequePayment/get-cheque-details`

Retrieves all cheque payment records.

**Response** — `ServiceResponse` with array of cheque records in `data`.

---

### Update Cheque Status

**`PATCH`** `/api/ChequePayment/update-cheque-status/{chequeId}/{status}`

Updates the status of a specific cheque (e.g., cleared, bounced, pending).

**Path Parameters**

| Parameter   | Type    | Required | Description                    |
|-------------|---------|----------|--------------------------------|
| `chequeId`  | integer | Yes      | ID of the cheque               |
| `status`    | string  | Yes      | New status value for the cheque|

**Response** — `ServiceResponse`

---

## Customer

### Register Customer

**`POST`** `/api/Customer/service-register`

Registers a new customer.

**Request Body**

```json
{
  "customerName": "string",
  "phone": "string",
  "nic": "string",
  "dateOfBirth": "2000-01-01T00:00:00Z",
  "address": "string",
  "email": "string",
  "creditLimit": 0.0
}
```

| Field          | Type     | Required | Description                         |
|----------------|----------|----------|-------------------------------------|
| `customerName` | string   | No       | Full name of the customer           |
| `phone`        | string   | No       | Contact phone number                |
| `nic`          | string   | No       | National Identity Card number       |
| `dateOfBirth`  | datetime | No       | Customer's date of birth            |
| `address`      | string   | No       | Physical address                    |
| `email`        | string   | No       | Email address                       |
| `creditLimit`  | double   | No       | Maximum credit allowed for customer |

**Response** — `ServiceResponse`

---

### Get All Customers Summary

**`GET`** `/api/Customer/get-all-customers-summary`

Returns a summary list of all customers.

**Response** — `ServiceResponse` with array of customer summaries in `data`.

---

### Update Customer

**`PUT`** `/api/Customer/{id}`

Updates an existing customer's details.

**Path Parameters**

| Parameter | Type    | Required | Description          |
|-----------|---------|----------|----------------------|
| `id`      | integer | Yes      | ID of the customer   |

**Request Body**

```json
{
  "customerName": "string",
  "phone": "string",
  "nic": "string",
  "address": "string",
  "email": "string",
  "creditLimit": 0.0
}
```

**Response** — `200 OK` (no body)

---

### Delete Customer

**`DELETE`** `/api/Customer/{id}`

Deletes a customer by ID.

**Path Parameters**

| Parameter | Type    | Required | Description        |
|-----------|---------|----------|--------------------|
| `id`      | integer | Yes      | ID of the customer |

**Response** — `200 OK` (no body)

---

### Get Credit Customers Count

**`GET`** `/api/Customer/credit-customers-count`

Returns the count of customers with active credit accounts.

**Response** — `ServiceResponse` with count value in `data`.

---

### Get Credit Customer Details

**`GET`** `/api/Customer/credit-customer-details`

Returns detailed information about all credit customers.

**Response** — `ServiceResponse` with array of credit customer records in `data`.

---

### Get Customer Balance

**`GET`** `/api/Customer/{customerId}/balance`

Retrieves the current balance for a specific customer.

**Path Parameters**

| Parameter    | Type    | Required | Description        |
|--------------|---------|----------|--------------------|
| `customerId` | integer | Yes      | ID of the customer |

**Response** — `ServiceResponse` with balance value in `data`.

---

### Search Customers

**`GET`** `/api/Customer/search`

Searches customers by a keyword.

**Query Parameters**

| Parameter | Type   | Required | Description               |
|-----------|--------|----------|---------------------------|
| `keyword` | string | No       | Search term (name, phone, etc.) |

**Response** — `200 OK`

---

## Damage Stock

### Create Damage Stock Record

**`POST`** `/api/DamageStock/create`

Records a new damage stock entry with one or more affected items.

**Request Body**

```json
{
  "referenceNo": "string",
  "date": "2024-01-01T00:00:00Z",
  "remarks": "string",
  "items": [
    {
      "itemId": 0,
      "quantity": 0,
      "reason": "string"
    }
  ]
}
```

| Field         | Type     | Required | Description                         |
|---------------|----------|----------|-------------------------------------|
| `referenceNo` | string   | No       | Reference number for the record     |
| `date`        | datetime | No       | Date the damage was recorded        |
| `remarks`     | string   | No       | Additional notes                    |
| `items`       | array    | No       | List of damaged items (see below)   |

**Damage Stock Item Object**

| Field      | Type    | Required | Description              |
|------------|---------|----------|--------------------------|
| `itemId`   | integer | No       | ID of the damaged item   |
| `quantity` | integer | No       | Quantity that was damaged|
| `reason`   | string  | No       | Reason for damage        |

**Response** — `ServiceResponse`

---

### Get All Damage Stock Records

**`GET`** `/api/DamageStock/all`

Retrieves all damage stock records.

**Response** — `ServiceResponse` with array of records in `data`.

---

### Get Today's Damage Stats

**`GET`** `/api/DamageStock/stats/today`

Returns damage stock statistics for the current day.

**Response** — `ServiceResponse` with today's damage summary in `data`.

---

### Get Damage Record by ID

**`GET`** `/api/DamageStock/{id}`

Retrieves a specific damage record by its ID.

**Path Parameters**

| Parameter | Type    | Required | Description                |
|-----------|---------|----------|----------------------------|
| `id`      | integer | Yes      | ID of the damage stock record |

**Response** — `ServiceResponse` with record details in `data`.

---

## Invoice

### Register Invoice

**`POST`** `/api/Invoice/register-invoice`

Creates a new sales invoice.

**Request Body**

```json
{
  "saleTypeId": 0,
  "customerId": 0,
  "invoiceDate": "2024-01-01T00:00:00Z",
  "userId": 0,
  "totalPrice": 0.0,
  "totalDiscount": 0.0,
  "status": "string"
}
```

| Field           | Type     | Required | Description                          |
|-----------------|----------|----------|--------------------------------------|
| `saleTypeId`    | integer  | No       | Type of sale (retail, wholesale, etc.)|
| `customerId`    | integer  | No       | ID of the customer                   |
| `invoiceDate`   | datetime | No       | Date and time of the invoice         |
| `userId`        | integer  | No       | ID of the user creating the invoice  |
| `totalPrice`    | double   | No       | Total amount of the invoice          |
| `totalDiscount` | double   | No       | Total discount applied               |
| `status`        | string   | No       | Invoice status (e.g., pending, paid) |

**Response** — `ServiceResponse`

---

### Get Latest Invoice ID

**`GET`** `/api/Invoice/get-latest-invoice-id`

Returns the ID of the most recently created invoice.

**Response** — `ServiceResponse` with latest invoice ID in `data`.

---

### Get Total Sales Today

**`GET`** `/api/Invoice/get-total-sales-today`

Returns the total sales amount for the current day.

**Response** — `ServiceResponse` with total sales figure in `data`.

---

### Get Pending Invoice Count

**`GET`** `/api/Invoice/get-pending-invoice-count`

Returns the number of invoices with a pending status.

**Response** — `ServiceResponse` with count in `data`.

---

### Get Total Items Sold Today

**`GET`** `/api/Invoice/get-total-items-sold-today`

Returns the total number of items sold today.

**Response** — `ServiceResponse` with item count in `data`.

---

### Get Today's Collected Cash

**`GET`** `/api/Invoice/get-today-collected-cash`

Returns the total cash collected through invoices today.

**Response** — `ServiceResponse` with cash total in `data`.

---

### Get Today's Cash Transaction Details

**`GET`** `/api/Invoice/today-cash-transaction-details`

Returns a detailed breakdown of today's cash transactions.

**Response** — `ServiceResponse` with transaction details in `data`.

---

### Get Invoice Details

**`GET`** `/api/Invoice/get-invoice-details`

Retrieves all invoice records.

**Response** — `ServiceResponse` with array of invoices in `data`.

---

### Get Invoice by Code

**`GET`** `/api/Invoice/get-invoice-by-id/{invoiceCode}`

Retrieves a specific invoice by its code.

**Path Parameters**

| Parameter     | Type   | Required | Description               |
|---------------|--------|----------|---------------------------|
| `invoiceCode` | string | Yes      | Unique code of the invoice|

**Response** — `ServiceResponse` with invoice details in `data`.

---

### Get Hourly Sales

**`GET`** `/api/Invoice/get-hourly-sales`

Returns sales data broken down by hour for the current day.

**Response** — `ServiceResponse` with hourly sales array in `data`.

---

### Get Today's Report Details

**`GET`** `/api/Invoice/get-today-report-details`

Returns a comprehensive sales report for the current day.

**Response** — `ServiceResponse` with report data in `data`.

---

## Invoice Item

### Create Invoice Item

**`POST`** `/api/InvoiceItem/create-invoice-item`

Adds a line item to an existing invoice.

**Request Body**

```json
{
  "invoiceId": 0,
  "batchId": 0,
  "itemId": 0,
  "mixId": 0,
  "discountPercent": 0.0,
  "discountValue": 0.0,
  "quantity": 0,
  "price": 0.0,
  "totalPrice": 0.0
}
```

| Field             | Type    | Required | Description                         |
|-------------------|---------|----------|-------------------------------------|
| `invoiceId`       | integer | No       | ID of the parent invoice            |
| `batchId`         | integer | No       | Batch ID (optional)                 |
| `itemId`          | integer | No       | ID of the item being sold           |
| `mixId`           | integer | No       | Mix/bundle ID (optional)            |
| `discountPercent` | double  | No       | Discount percentage applied         |
| `discountValue`   | double  | No       | Discount amount in currency         |
| `quantity`        | integer | No       | Quantity sold                       |
| `price`           | double  | No       | Unit price                          |
| `totalPrice`      | double  | No       | Total price after discount          |

**Response** — `ServiceResponse`

---

## Item

### Register Item(s)

**`POST`** `/api/Item/register-item`

Registers one or more new items. Accepts an array of item objects.

**Request Body**

```json
[
  {
    "name": "string",
    "unit": "string",
    "bulkCode": "string",
    "categoryId": 0,
    "brandId": 0,
    "subBrandId": 0,
    "itemTypeId": 0,
    "availableQty": 0,
    "reorderLevel": 0,
    "retailPrice": 0.0,
    "wholeSalePrice": 0.0,
    "cost": 0.0,
    "make": "string",
    "warrrenty": "string",
    "damageStatus": "string",
    "status": "string",
    "expirationDate": "2024-01-01T00:00:00Z",
    "barcode": "string"
  }
]
```

| Field            | Type     | Required | Description                          |
|------------------|----------|----------|--------------------------------------|
| `name`           | string   | No       | Item name                            |
| `unit`           | string   | No       | Unit of measurement (e.g., pcs, kg)  |
| `bulkCode`       | string   | No       | Bulk packaging code                  |
| `categoryId`     | integer  | No       | Associated category ID               |
| `brandId`        | integer  | No       | Associated brand ID                  |
| `subBrandId`     | integer  | No       | Associated sub-brand ID              |
| `itemTypeId`     | integer  | No       | Associated item type ID              |
| `availableQty`   | integer  | No       | Current quantity in stock            |
| `reorderLevel`   | integer  | No       | Quantity threshold for reorder alert |
| `retailPrice`    | double   | No       | Retail selling price                 |
| `wholeSalePrice` | double   | No       | Wholesale selling price              |
| `cost`           | double   | No       | Purchase/cost price                  |
| `make`           | string   | No       | Make or model info                   |
| `warrrenty`      | string   | No       | Warranty information                 |
| `damageStatus`   | string   | No       | Damage status indicator              |
| `status`         | string   | No       | Active/inactive status               |
| `expirationDate` | datetime | No       | Expiry date (if applicable)          |
| `barcode`        | string   | No       | Barcode value                        |

**Response** — `ServiceResponse`

---

### Get All Items

**`GET`** `/api/Item/get-all-items`

Retrieves all registered items.

**Response** — `ServiceResponse` with array of items in `data`.

---

### Search Items

**`GET`** `/api/Item/search-items`

Searches for items by barcode, item code, or name.

**Query Parameters**

| Parameter  | Type   | Required | Description           |
|------------|--------|----------|-----------------------|
| `barcode`  | string | No       | Item barcode          |
| `itemCode` | string | No       | Item code             |
| `itemName` | string | No       | Full or partial name  |

**Response** — `ServiceResponse` with matching items in `data`.

---

### Get New Bulk Code

**`GET`** `/api/Item/get-new-bulk-code`

Generates a new bulk code for an item based on a given prefix.

**Query Parameters**

| Parameter        | Type   | Required | Description                |
|------------------|--------|----------|----------------------------|
| `bulkCodePrefix` | string | No       | Prefix for the bulk code   |

**Response** — `ServiceResponse` with generated code in `data`.

---

## Item Type

### Register Item Type

**`POST`** `/api/ItemType/register-item-type`

Creates a new item type linked to a sub-brand.

**Request Body**

```json
{
  "subBrandId": 0,
  "itemTypeName": "string",
  "itemTypeDescription": "string"
}
```

| Field                 | Type    | Required | Description                    |
|-----------------------|---------|----------|--------------------------------|
| `subBrandId`          | integer | No       | ID of the associated sub-brand |
| `itemTypeName`        | string  | No       | Name of the item type          |
| `itemTypeDescription` | string  | No       | Description of the item type   |

**Response** — `ServiceResponse`

---

### Get All Item Types

**`GET`** `/api/ItemType/get-all-item-type-details`

Retrieves all item types.

**Response** — `ServiceResponse` with array of item types in `data`.

---

### Delete Item Type

**`DELETE`** `/api/ItemType/delete-item-type/{itemTypeId}`

Deletes an item type by ID.

**Path Parameters**

| Parameter    | Type    | Required | Description          |
|--------------|---------|----------|----------------------|
| `itemTypeId` | integer | Yes      | ID of the item type  |

**Response** — `ServiceResponse`

---

## Payment

### Create Payment

**`POST`** `/api/Payment`

Records a payment against an invoice.

**Request Body**

```json
{
  "invoiceId": 0,
  "paymentMethodId": 0,
  "bankId": 0,
  "customerId": 0,
  "userId": 0,
  "paymentType": "string",
  "amount": 0.0,
  "balance": 0.0,
  "description": "string",
  "status": "string"
}
```

| Field             | Type    | Required | Description                           |
|-------------------|---------|----------|---------------------------------------|
| `invoiceId`       | integer | No       | ID of the invoice being paid          |
| `paymentMethodId` | integer | No       | ID of the payment method used         |
| `bankId`          | integer | No       | ID of the bank (if applicable)        |
| `customerId`      | integer | No       | ID of the paying customer             |
| `userId`          | integer | No       | ID of the user recording the payment  |
| `paymentType`     | string  | No       | Type of payment (cash, card, etc.)    |
| `amount`          | double  | No       | Amount paid                           |
| `balance`         | double  | No       | Remaining balance after payment       |
| `description`     | string  | No       | Optional notes about the payment      |
| `status`          | string  | No       | Payment status                        |

**Response** — `ServiceResponse`

---

### Get Payment Methods

**`GET`** `/api/Payment/payment-methods`

Retrieves all available payment methods.

**Response** — `ServiceResponse` with payment methods list in `data`.

---

### Get Customer Payments

**`GET`** `/api/Payment/customer/{customerId}/payments`

Retrieves all payments made by a specific customer.

**Path Parameters**

| Parameter    | Type    | Required | Description        |
|--------------|---------|----------|--------------------|
| `customerId` | integer | Yes      | ID of the customer |

**Response** — `ServiceResponse` with array of payment records in `data`.

---

## Role

### Add Role

**`POST`** `/api/Role/add-role`

Creates a new user role.

**Request Body**

```json
{
  "roleName": "string",
  "roleDescription": "string"
}
```

| Field             | Type   | Required | Description              |
|-------------------|--------|----------|--------------------------|
| `roleName`        | string | No       | Name of the role         |
| `roleDescription` | string | No       | Description of the role  |

**Response** — `ServiceResponse`

---

### Get All Roles

**`GET`** `/api/Role/get-all-roles`

Retrieves all user roles.

**Response** — `ServiceResponse` with array of roles in `data`.

---

## Service

### Register Service

**`POST`** `/api/Service/service-register`

Registers a new billable service.

**Request Body**

```json
{
  "serviceName": "string",
  "serviceDescription": "string",
  "amount": 0.0,
  "status": "string"
}
```

| Field                | Type   | Required | Description                   |
|----------------------|--------|----------|-------------------------------|
| `serviceName`        | string | No       | Name of the service           |
| `serviceDescription` | string | No       | Description of the service    |
| `amount`             | double | No       | Price/cost of the service     |
| `status`             | string | No       | Active/inactive status        |

**Response** — `ServiceResponse`

---

## Supplier

### Register Supplier

**`POST`** `/api/Supplier/supplier-register`

Registers a new supplier.

**Request Body**

```json
{
  "company": "string",
  "address": "string",
  "contactPerson": "string",
  "phone": "string",
  "email": "string",
  "description": "string",
  "status": "string"
}
```

| Field           | Type   | Required | Description                     |
|-----------------|--------|----------|---------------------------------|
| `company`       | string | No       | Company/supplier name           |
| `address`       | string | No       | Physical address                |
| `contactPerson` | string | No       | Name of the primary contact     |
| `phone`         | string | No       | Contact phone number            |
| `email`         | string | No       | Email address                   |
| `description`   | string | No       | Additional notes                |
| `status`        | string | No       | Active/inactive status          |

**Response** — `ServiceResponse`

---

### Get All Suppliers Summary

**`GET`** `/api/Supplier/get-all-suppliers-summary`

Returns a summary list of all suppliers.

**Response** — `ServiceResponse` with array of supplier summaries in `data`.

---

### Get Supplier by ID

**`GET`** `/api/Supplier/{id}`

Retrieves a specific supplier's details.

**Path Parameters**

| Parameter | Type    | Required | Description         |
|-----------|---------|----------|---------------------|
| `id`      | integer | Yes      | ID of the supplier  |

**Response** — `ServiceResponse` with supplier details in `data`.

---

### Update Supplier

**`PUT`** `/api/Supplier/{id}`

Updates a supplier's information.

**Path Parameters**

| Parameter | Type    | Required | Description        |
|-----------|---------|----------|--------------------|
| `id`      | integer | Yes      | ID of the supplier |

**Request Body**

```json
{
  "company": "string",
  "contactPerson": "string",
  "phone": "string",
  "address": "string",
  "email": "string",
  "description": "string",
  "status": "string"
}
```

**Response** — `200 OK` (no body)

---

### Delete Supplier

**`DELETE`** `/api/Supplier/{id}`

Deletes a supplier by ID.

**Path Parameters**

| Parameter | Type    | Required | Description        |
|-----------|---------|----------|--------------------|
| `id`      | integer | Yes      | ID of the supplier |

**Response** — `200 OK` (no body)

---

### Search Suppliers

**`GET`** `/api/Supplier/search`

Searches suppliers by a keyword.

**Query Parameters**

| Parameter | Type   | Required | Description        |
|-----------|--------|----------|--------------------|
| `keyword` | string | No       | Search term        |

**Response** — `200 OK`

---

## User

### Register User

**`POST`** `/api/User/user-registration`

Creates a new system user.

**Request Body**

```json
{
  "username": "string",
  "password": "string",
  "roleId": 0,
  "status": "string"
}
```

| Field      | Type    | Required | Description                |
|------------|---------|----------|----------------------------|
| `username` | string  | No       | Unique username            |
| `password` | string  | No       | User password              |
| `roleId`   | integer | No       | ID of the assigned role    |
| `status`   | string  | No       | Active/inactive status     |

**Response** — `ServiceResponse`

---

### Get All Users

**`GET`** `/api/User/get-all-users`

Retrieves all registered users.

**Response** — `ServiceResponse` with array of users in `data`.

---

### Update User

**`PUT`** `/api/User/{id}`

Updates an existing user's information.

**Path Parameters**

| Parameter | Type    | Required | Description    |
|-----------|---------|----------|----------------|
| `id`      | integer | Yes      | ID of the user |

**Request Body**

```json
{
  "username": "string",
  "roleId": 0,
  "password": "string",
  "status": "string"
}
```

**Response** — `200 OK` (no body)

---

### Delete User

**`DELETE`** `/api/User/{id}`

Deletes a user by ID.

**Path Parameters**

| Parameter | Type    | Required | Description    |
|-----------|---------|----------|----------------|
| `id`      | integer | Yes      | ID of the user |

**Response** — `200 OK` (no body)

---

### Update User Status

**`PUT`** `/api/User/status/{id}`

Updates only the status of a user (active/inactive).

**Path Parameters**

| Parameter | Type    | Required | Description    |
|-----------|---------|----------|----------------|
| `id`      | integer | Yes      | ID of the user |

**Request Body**

```json
{
  "status": "string"
}
```

**Response** — `200 OK` (no body)

---

### User Login

**`POST`** `/api/User/login`

Authenticates a user with username and password.

**Request Body**

```json
{
  "username": "string",
  "password": "string"
}
```

| Field      | Type   | Required | Description    |
|------------|--------|----------|----------------|
| `username` | string | No       | The username   |
| `password` | string | No       | The password   |

**Response** — `200 OK`

---

## Data Models

### ServiceResponse

The standard wrapper returned by most endpoints.

| Field     | Type    | Description                              |
|-----------|---------|------------------------------------------|
| `flag`    | boolean | `true` = success, `false` = failure      |
| `message` | string  | Status or error message                  |
| `data`    | any     | Returned payload (null on failure)       |

---

### CreateBankDto

| Field      | Type   | Description           |
|------------|--------|-----------------------|
| `bankName` | string | Full name of the bank |
| `bankCode` | string | Short identifier code |

---

### CreateBankTransferDto

| Field             | Type     | Description                  |
|-------------------|----------|------------------------------|
| `paymentId`       | integer  | Linked payment ID            |
| `bankId`          | integer  | Bank ID                      |
| `referenceNumber` | string   | Transfer reference number    |
| `transferAmount`  | double   | Amount transferred           |
| `transferDate`    | datetime | Date of transfer             |

---

### BatchRegistrationRequestDto

| Field            | Type     | Description             |
|------------------|----------|-------------------------|
| `batchCode`      | string   | Unique batch code       |
| `itemId`         | integer  | Linked item ID          |
| `supplierId`     | integer  | Linked supplier ID      |
| `quantity`       | double   | Quantity in batch       |
| `purchasePrice`  | double   | Cost price per unit     |
| `retailPrice`    | double   | Retail price per unit   |
| `wholesalePrice` | double   | Wholesale price per unit|
| `expiryDate`     | datetime | Expiry date (optional)  |

---

### BrandRegistrationRequest / BrandUpdateRequest

| Field              | Type   | Description              |
|--------------------|--------|--------------------------|
| `brandName`        | string | Brand name               |
| `brandDescription` | string | Brand description        |

---

### SubBrandRegistrationRequest / SubBrandUpdateRequest

| Field                 | Type    | Description               |
|-----------------------|---------|---------------------------|
| `brandId`             | integer | Parent brand ID           |
| `subBrandName`        | string  | Sub-brand name            |
| `subBrandDescription` | string  | Sub-brand description     |

---

### CategoryRegistrationRequest / CategoryUpdateRequest

| Field          | Type   | Description          |
|----------------|--------|----------------------|
| `categoryName` | string | Category name        |
| `description`  | string | Category description |
| `status`       | string | Status value         |

---

### CreateChequePaymentDto

| Field          | Type     | Description            |
|----------------|----------|------------------------|
| `paymentId`    | integer  | Linked payment ID      |
| `chequeNumber` | string   | Cheque number          |
| `chequeAmount` | double   | Cheque amount          |
| `brankId`      | integer  | Issuing bank ID        |
| `chequeDate`   | datetime | Cheque date            |

---

### CustomerRegistrationRequestDto

| Field          | Type     | Description          |
|----------------|----------|----------------------|
| `customerName` | string   | Full name            |
| `phone`        | string   | Phone number         |
| `nic`          | string   | NIC number           |
| `dateOfBirth`  | datetime | Date of birth        |
| `address`      | string   | Physical address     |
| `email`        | string   | Email address        |
| `creditLimit`  | double   | Credit limit amount  |

---

### CustomerUpdateDto

| Field          | Type   | Description         |
|----------------|--------|---------------------|
| `customerName` | string | Full name           |
| `phone`        | string | Phone number        |
| `nic`          | string | NIC number          |
| `address`      | string | Physical address    |
| `email`        | string | Email address       |
| `creditLimit`  | double | Credit limit amount |

---

### CreateDamageStockRequest

| Field         | Type     | Description           |
|---------------|----------|-----------------------|
| `referenceNo` | string   | Reference number      |
| `date`        | datetime | Date of damage        |
| `remarks`     | string   | Notes                 |
| `items`       | array    | Array of damaged items|

### DamageStockItemRequest

| Field      | Type    | Description         |
|------------|---------|---------------------|
| `itemId`   | integer | Item ID             |
| `quantity` | integer | Damaged quantity    |
| `reason`   | string  | Reason for damage   |

---

### InvoiceCreateDto

| Field           | Type     | Description                |
|-----------------|----------|----------------------------|
| `saleTypeId`    | integer  | Sale type ID               |
| `customerId`    | integer  | Customer ID                |
| `invoiceDate`   | datetime | Invoice date               |
| `userId`        | integer  | Creating user's ID         |
| `totalPrice`    | double   | Invoice total              |
| `totalDiscount` | double   | Total discount applied     |
| `status`        | string   | Invoice status             |

---

### InvoiceItemCreateDto

| Field             | Type    | Description             |
|-------------------|---------|-------------------------|
| `invoiceId`       | integer | Parent invoice ID       |
| `batchId`         | integer | Batch ID (optional)     |
| `itemId`          | integer | Item ID                 |
| `mixId`           | integer | Mix ID (optional)       |
| `discountPercent` | double  | Discount percentage     |
| `discountValue`   | double  | Discount amount         |
| `quantity`        | integer | Quantity sold           |
| `price`           | double  | Unit price              |
| `totalPrice`      | double  | Total line price        |

---

### ItemRegistrationRequest

| Field            | Type     | Description               |
|------------------|----------|---------------------------|
| `name`           | string   | Item name                 |
| `unit`           | string   | Unit of measure           |
| `bulkCode`       | string   | Bulk code                 |
| `categoryId`     | integer  | Category ID               |
| `brandId`        | integer  | Brand ID                  |
| `subBrandId`     | integer  | Sub-brand ID              |
| `itemTypeId`     | integer  | Item type ID              |
| `availableQty`   | integer  | Stock quantity            |
| `reorderLevel`   | integer  | Reorder threshold         |
| `retailPrice`    | double   | Retail price              |
| `wholeSalePrice` | double   | Wholesale price           |
| `cost`           | double   | Cost price                |
| `make`           | string   | Make/model                |
| `warrrenty`      | string   | Warranty info             |
| `damageStatus`   | string   | Damage status             |
| `status`         | string   | Item status               |
| `expirationDate` | datetime | Expiry date (optional)    |
| `barcode`        | string   | Barcode                   |

---

### ItemTypeRegistrationRequest

| Field                 | Type    | Description             |
|-----------------------|---------|-------------------------|
| `subBrandId`          | integer | Sub-brand ID            |
| `itemTypeName`        | string  | Item type name          |
| `itemTypeDescription` | string  | Item type description   |

---

### PaymentCreateDto

| Field             | Type    | Description              |
|-------------------|---------|--------------------------|
| `invoiceId`       | integer | Invoice ID               |
| `paymentMethodId` | integer | Payment method ID        |
| `bankId`          | integer | Bank ID                  |
| `customerId`      | integer | Customer ID              |
| `userId`          | integer | User ID                  |
| `paymentType`     | string  | Payment type             |
| `amount`          | double  | Amount paid              |
| `balance`         | double  | Remaining balance        |
| `description`     | string  | Notes                    |
| `status`          | string  | Payment status           |

---

### ServiceRegistrationRequest

| Field                | Type   | Description          |
|----------------------|--------|----------------------|
| `serviceName`        | string | Service name         |
| `serviceDescription` | string | Service description  |
| `amount`             | double | Service amount       |
| `status`             | string | Status value         |

---

### SupplierRegistrationRequest / SupplierUpdateDto

| Field           | Type   | Description         |
|-----------------|--------|---------------------|
| `company`       | string | Company name        |
| `address`       | string | Address             |
| `contactPerson` | string | Contact person name |
| `phone`         | string | Phone number        |
| `email`         | string | Email address       |
| `description`   | string | Additional notes    |
| `status`        | string | Status value        |

---

### UserRegistrationRequest / UserUpdateDto

| Field      | Type    | Description       |
|------------|---------|-------------------|
| `username` | string  | Username          |
| `password` | string  | Password          |
| `roleId`   | integer | Assigned role ID  |
| `status`   | string  | User status       |

---

### UserStatusUpdateDto

| Field    | Type   | Description        |
|----------|--------|--------------------|
| `status` | string | New status value   |

---

### UserLoginRequestDto

| Field      | Type   | Description |
|------------|--------|-------------|
| `username` | string | Username    |
| `password` | string | Password    |

---

### AddRoleRequest

| Field             | Type   | Description      |
|-------------------|--------|------------------|
| `roleName`        | string | Role name        |
| `roleDescription` | string | Role description |

---

*End of API Documentation*
