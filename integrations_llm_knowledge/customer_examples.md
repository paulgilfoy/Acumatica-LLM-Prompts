
### Creating a Customer via Acumatica ERP REST API

This document outlines how an external system can create a customer in Acumatica ERP using the contract-based REST API.

---
### Request Details

To create a customer, use the following request example.

**URL Structure:**

The base URL for your Acumatica ERP instance, like `https://my.acumatica.com/MyInstance`, should replace `<Acumatica ERP instance URL>`. If your instance is in the website's root, you can omit the instance name (e.g., `https://my.acumatica.com`).

**HTTP Method:** `PUT`

**Endpoint:** `/entity/Default/24.200.001/Customer`

**Headers:**

* `Host`: `[<Acumatica ERP instance URL>]`
* `Accept`: `application/json`
* `Content-Type`: `application/json`

**Request Body Example:**

```json
{
    "CustomerID": {"value": "Cust01"},
    "CustomerName": {"value": "Customer 01"},
    "CustomerClass": {"value": "DEFAULT"}
}
```

---

### Retrieve Customers with Contacts via REST API

**Method:** GET
**Endpoint:** `/entity/Default/24.200.001/Customer`
**Mapped Form:** `Customers (AR303000)`

**Purpose:** List customer records including main contact and address information.

**Sample Request:**

```

GET /entity/Default/24.200.001/Customer?
$expand=MainContact,MainContact/Address&
$select=CustomerID,CustomerName,CustomerClass,MainContact/Email,
MainContact/Phone1,MainContact/Address/AddressLine1,
MainContact/Address/AddressLine2,MainContact/Address/City,
MainContact/Address/State,MainContact/Address/PostalCode HTTP/1.1
Host: [\<Acumatica ERP instance URL\>]
Accept: application/json

```

**Notes:**
* `<Acumatica ERP instance URL>` is the base URL (e.g., `https://my.acumatica.com/MyInstance`). Instance name can be omitted if in the root.
* `$select` parameter: Specifies desired fields for performance.
* `$expand` parameter: Specifies nested entities to retrieve (e.g., `MainContact`, `MainContact/Address`).
```

---

### Update a Customer Record in Acumatica ERP via REST API

To **update an existing customer record** in Acumatica ERP using its contract-based REST API, you utilize the **`PUT` HTTP method**.

The record is identified using the **`$filter` parameter**, specifically by the customer's **email address**.

### Request Structure

**Method:** `PUT`
**Endpoint:** `/entity/Default/24.200.001/Customer`

**Query Parameters:**
* `$expand`: `MainContact`, `BillingContact`
* `$select`: `CustomerID`, `CustomerClass`, `BillingContact/Email`
* `$filter`: `MainContact/Email eq 'info@jevy-comp.con'` (URL-encoded: `MainContact/Email%20eq%20'info@jevy-comp.con'`)

**Headers:**
* `Host`: `[<Acumatica ERP instance URL>]`
* `Accept`: `application/json`
* `Content-Type`: `application/json`

**Request Body (JSON Example):**
```json
{
    "CustomerClass": {"value": "INTL"},
    "BillingContactOverride": {"value": true},
    "BillingContact": {
        "Email": {"value": "green@jevy-comp.con"},
        "Attention": {"value": "Mr. Jack Green"},
        "JobTitle": {"value": ""}
    }
}
```

**Purpose:** This example demonstrates updating a customer with the email `info@jevy-comp.con`. The update modifies the **customer class** and the **billing contact details**.

---

###Retrieve Customer Shipping Contact via Acumatica ERP REST API

**Purpose:** An external system can retrieve a customer's shipping contact using the contract-based REST API for Acumatica ERP integration.

**Example Request:**

To retrieve the shipping contact for customer 'ABAKERY':

```

GET ?$expand=ShippingContact&$filter=CustomerID%20eq%20'ABAKERY' HTTP/1.1
Host: [\<Acumatica ERP instance URL\>]/entity/Default/24.200.001/Customer
Accept: application/json
Content-Type: application/json

```

---
