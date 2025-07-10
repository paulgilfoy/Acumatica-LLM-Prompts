### Updating Records with Acumatica ERP REST API

This document describes how to update existing records in Acumatica ERP using the contract-based REST API, focusing on the **PUT** HTTP method.

### HTTP Method and URL

To update a record, use the **PUT** HTTP method with the following URL structure:

`PUT http://<Base endpoint URL>/<Top-level entity>`

* **`<Base endpoint URL>`**: The URL of your Acumatica ERP instance and endpoint. Format: `http://<Acumatica ERP instance URL>/entity/<Endpoint name>/<Endpoint version>/`
* **`<Top-level entity>`**: The name of the entity you are updating (e.g., `StockItem`, `Customer`).

**Example URL:** `http://localhost/AcumaticaDB/entity/Default/24.200.001/StockItem`

### Parameters

You can use the following parameters with the PUT request:

* **`$filter`**: Specifies conditions to identify the record to be updated.
* **`$expand`**: Specifies linked and detail entities to be included in the response body. All detail and related entities expected in the response *must* be listed here.
* **`$select`**: Specifies the fields of the entity to be returned.
* **`$custom`**: Specifies non-contract fields to be returned.

### HTTP Headers

Include these headers in your request:

| Header        | Description                                                                                                                                                                                                                                           |
| :------------ | :---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `Accept`      | Specifies the response body format: `application/json`.                                                                                                                                                                                             |
| `Content-Type` | Specifies the request body format: `application/json`.                                                                                                                                                                                              |
| `If-Match`    | Optional. Use with value `*` (`"If-Match" : "*"`) to ensure the PUT request only updates an *existing* record, preventing creation if the record isn't found.                                                                                          |

### HTTP Body

The request body must contain the record representation in **JSON format**.

To identify the record for update, specify one of the following in the JSON body:

* Values of the **key fields**.
* The **`ID` property value**.

Alternatively, you can identify the record using **filtering conditions** in the `$filter` parameter.

To delete a detail line during an update, set `"delete" : true` for the corresponding detail entity in the JSON body. Identify the detail line to delete by its `ID` property or key field values.

### Response Status Codes

| Code | Description                                                                                                                                                                                                                                    |
| :--- | :--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `200` | **Success.** The request completed successfully, and the response body contains the updated record in JSON format.                                                                                                                               |
| `400` | **Bad Request.** Invalid data in the request.                                                                                                                                                                                                  |
| `401` | **Unauthorized.** User is not signed in.                                                                                                                                                                                                       |
| `403` | **Forbidden.** User lacks rights to access the corresponding Acumatica ERP form.                                                                                                                                                               |
| `412` | **Precondition Failed.** `If-Match` header with `*` was used, but the record does not exist.                                                                                                                                                     |
| `422` | **Unprocessable Entity.** Invalid data in the request; validation errors are in the `error` fields of the response body (e.g., `"error": "'Customer' cannot be found in the system."`).                                                        |
| `429` | **Too Many Requests.** License request limit exceeded.                                                                                                                                                                                         |
| `500` | **Internal Server Error.** An unexpected server error occurred.                                                                                                                                                                                |

### Example Request

```
PUT ?$filter=MainContact/Email%20eq%20'demo@gmail.com'&
   $select=CustomerID,CustomerClass,MainContact/Email&$expand=MainContact HTTP/1.1
Host: [<Acumatica ERP instance URL>]/entity/Default/24.200.001/Customer
Accept: application/json
Content-Type: application/json

{
  "CustomerClass" : {"value" : "ECCUSTOMER"}
}
```

This example updates a customer record where the `MainContact/Email` is `demo@gmail.com`, setting its `CustomerClass` to `ECCUSTOMER`.

### Usage Notes

---
#### PUT vs. PATCH Methods

The **PUT** method compares values in the request body with existing record values. It only updates fields where the value in the request body *differs* from the system's value. Be aware that other fields might change due to graph logic, and the system won't save values for fields already updated by this logic in the request body. If this behavior is problematic, consider using the **PATCH** method.

#### Drop-Down List Values

For drop-down lists (including multiselect), you can only specify values that are pre-defined for the control. Attempting to set an undefined value will result in an error in the response's `error` field. For multiselect lists, specify internal values, not displayed ones.

#### Records with Many Detail Lines

When creating or updating records with a large number of detail lines, avoid sending all information in a single request (risk of timeout) or many requests with single detail lines (slow overall). A balanced approach involves sending **multiple requests, each containing a reasonable batch of detail lines** (e.g., 500 lines for a 10,000-line sales order) to optimize performance.