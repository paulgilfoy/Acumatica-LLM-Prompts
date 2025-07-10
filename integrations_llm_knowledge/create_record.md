### Creating Records via Acumatica Contract-Based REST API

To create a record, use the `PUT` HTTP method with the appropriate URL and pass the record's JSON representation in the request body.

---
### HTTP Method and URL

**Method:** `PUT`

**URL Format:** `http://<Base endpoint URL>/<Top-level entity>`

**URL Components:**
* `<Base endpoint URL>`: `http://<Acumatica ERP instance URL>/entity/<Endpoint name>/<Endpoint version>/`
* `<Top-level entity>`: The name of the entity for which you're creating a record.

**Example URL:**
To create a stock item in a local Acumatica ERP instance named `AcumaticaDB` using the `Default` endpoint (version 24.200.001):
`http://localhost/AcumaticaDB/entity/Default/24.200.001/StockItem`

---
### Parameters

The following parameters can be used:

* **`$expand`**: Specifies linked and detail entities to be expanded in the response. All detail and related entities expected in the response body must be listed.
* **`$select`**: Specifies the fields of the entity to be returned.
* **`$custom`**: Specifies fields not defined in the contract to be returned.

---
### Request Headers

| Header           | Description                                                                                                                                                                             |
| :--------------- | :-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `Accept`         | Specifies the format of the response body, which should be `application/json`.                                                                                                          |
| `Content-Type`   | Specifies the format of the request body, which should be `application/json`.                                                                                                           |
| `If-None-Match`  | (Optional) Use with value `*` (asterisk) if you only want to create a new record and prevent updating an existing one.                                                                  |

---
### Request Body

The record is passed in JSON format.

**Example Customer Record JSON:**

```json
{
  "CustomerID" : {value : "JOHNGOOD"},
  "CustomerName" : {value : "John Good"},
  "CustomerClass" : {value : "DEFAULT"},
  "MainContact" :
    {
      "Email" : {value : "demo@gmail.com"},
      "Address" :
        {
          "AddressLine1" : {"value" : "4030 Lake Washington Blvd NE"},
          "AddressLine2" : {"value" : "Suite 100"},
          "City" : {"value" : "Kirkland"},
          "State" : {"value" : "WA" },
          "PostalCode" : {"value" : "98033"}
        }
    }
}
```

---
### Response Status Codes

| Code | Description                                                                                                                                                                                                    |
| :--- | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `200`| Request completed successfully. The response body contains the created record in JSON format, including specified or returned fields.                                                                         |
| `400`| Invalid data in the request.                                                                                                                                                                                   |
| `401`| User not signed in.                                                                                                                                                                                            |
| `403`| Insufficient user rights to access the corresponding Acumatica ERP form.                                                                                                                                       |
| `412`| `If-None-Match` header with `*` was used, but the record already exists.                                                                                                                                       |
| `422`| Invalid data in the request; validation errors are returned in the `error` fields of the response body. Example: `"CustomerID": {"value": "ABARTENDE1", "error": "'Customer' cannot be found in the system."}` |
| `429`| Number of requests exceeded license limit.                                                                                                                                                                     |
| `500`| Internal server error.                                                                                                                                                                                         |

---
### Example Request

Here's an example of creating a `Customer` record:

```http
PUT / HTTP/1.1
Host: [<Acumatica ERP instance URL>]/entity/Default/24.200.001/Customer
Accept: application/json
Content-Type: application/json

{
  "CustomerID" : {"value" : "JOHNGOOD"},
  "CustomerName" : {"value" : "John Good"},
  "CustomerClass" : {"value" : "DEFAULT"},
  "MainContact" :
    {
      "Email" : {"value" : "demo@gmail.com"},
      "Address" :
        {
          "AddressLine1" : {"value" : "4030 Lake Washington Blvd NE"},
          "AddressLine2" : {"value" : "Suite 100"},
          "City" : {"value" : "Kirkland"},
          "State" : {"value" : "WA" },
          "PostalCode" : {"value" : "98033"}
        }
    }
}
```
* `<Acumatica ERP instance URL>` is the URL of your Acumatica ERP instance (e.g., `https://my.acumatica.com/MyInstance`). If the instance is in the root of the website, you can omit the instance name (e.g., `https://my.acumatica.com`).

---
### Usage Notes

#### Multiple Detail Lines
You can create a record with multiple detail lines using a single `PUT` request.

#### Records with a Large Number of Detail Lines
For records with many detail lines, consider these approaches:

* **Single Request:** Sending all data in one request may lead to timeouts.
* **Many Small Requests:** Sending each detail line in a separate request is time-consuming.
* **Balanced Approach:** The best compromise is multiple requests, each containing a subset of detail lines. For example, for 10,000 detail lines, send 500 lines per request.

#### Values for a Drop-Down List
You can only specify a value for a drop-down list if it's already defined for that control. An `error` message will be returned if the value is not present. For multiselect drop-down lists, only internal values can be specified.