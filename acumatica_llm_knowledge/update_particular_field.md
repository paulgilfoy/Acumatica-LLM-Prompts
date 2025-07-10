### Updating Record Fields with REST API

This document outlines how to update specific fields of an existing record using the contract-based REST API with the `PATCH` HTTP method.

### HTTP Method and URL

To update particular fields of a record in Acumatica ERP, use the `PATCH` HTTP method with the following URL structure:

`PATCH http://<Base endpoint URL>/<Top-level entity>`

* **`<Base endpoint URL>`**: `http://<Acumatica ERP instance URL>/entity/<Endpoint name>/<Endpoint version>/`
* **`<Top-level entity>`**: The name of the entity whose record you are updating.

**Example URL:**
`http://localhost/AcumaticaDB/entity/Default/24.200.001/StockItem`

---
### Parameters

The following parameters can be used:

* `$filter`: Specifies conditions to identify the record to be updated.
* `$expand`: Specifies linked and detail entities to be expanded in the response. All detail and related entities in the response body must be listed here.
* `$select`: Specifies the fields of the entity to be returned.
* `$custom`: Specifies non-contract fields to be returned.

---
### HTTP Headers

| Header       | Description                                              |
| :----------- | :------------------------------------------------------- |
| `Accept`     | Specifies `application/json` for the response body.      |
| `Content-Type` | Specifies `application/json` for the request body.       |

---
### HTTP Body

The request body should contain the record in JSON format.

To identify the record for update, specify one of the following:

* Values of key fields in the JSON record representation.
* The `ID` property value in the JSON record representation.
* Filtering conditions in the `$filter` parameter.

To delete a detail line during an update, set `"delete": true` for the corresponding detail entity. Identify the detail line to be deleted by its `ID` property value or key field values.

---
### Response Status Codes

| Code | Description                                                                                                                                                                                                                                                                          |
| :--- | :----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 200  | Success. Response body contains the updated record in JSON.                                                                                                                                                                                                                          |
| 400  | Invalid data in the request.                                                                                                                                                                                                                                                         |
| 401  | User not signed in.                                                                                                                                                                                                                                                                  |
| 403  | Insufficient user rights to access the corresponding Acumatica ERP form.                                                                                                                                                                                                             |
| 404  | Record not found.                                                                                                                                                                                                                                                                    |
| 412  | `If-None-Match` header with `*` was used (indicating insertion), but `PATCH` cannot insert records.                                                                                                                                                                                  |
| 422  | Invalid data, with validation errors returned in `error` fields (e.g., `"CustomerID": { "value": "ABARTENDE1", "error": "'Customer' cannot be found in the system." }`).                                                                                                            |
| 429  | Request limit exceeded due to license restrictions.                                                                                                                                                                                                                                  |
| 500  | Internal server error.                                                                                                                                                                                                                                                               |

---
### Example Request

Update order quantity and discount amount for a sales order detail line:

```http
PATCH /entity/Default/24.200.001/SalesOrder?
  $expand=Details&
  $select=Details/DiscountAmount,Details/OrderQty,OrderNbr,OrderType HTTP/1.1
Host: [<Acumatica ERP instance URL>]
Accept: application/json
Content-Type: application/json

{
    "OrderType": {"value": "SO"},
    "OrderNbr": { "value": "000029" },
    "Details": [
        {
             "id": "a1d07920-a402-e911-b818-00155d408001",
              "OrderQty": { "value": 2 },
              "DiscountAmount": { "value": 5 }  
        }
    ]
}
```

---
### Usage Notes

* **`PATCH` vs. `PUT`**: `PATCH` updates only specified fields, respecting graph logic. Use it when synchronizing specific changes from external systems or when `PUT` requests cannot update particular fields.
* **Drop-Down List Values**: Values for drop-down lists (including multi-select) must be predefined. An error occurs if an undefined value is used. For multi-select, only internal values can be specified.