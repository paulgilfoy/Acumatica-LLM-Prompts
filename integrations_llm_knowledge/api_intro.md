# Acumatica ERP Integration API - Training for LLMs

This document provides key information for training a Large Language Model (LLM) to interact with the Acumatica ERP Integration API.

## 1. Base URL Structure

*   **Contract-Based API:**  `http(s)://<Acumatica ERP instance URL>/entity/<Endpoint name>/<Endpoint version>/`
    *   `<Acumatica ERP instance URL>`: The URL of your Acumatica ERP instance (e.g., `https://my.acumatica.com`).
    *   `<Endpoint name>`:  Name of the API endpoint (e.g., `Default`).
    *   `<Endpoint version>`: Version of the endpoint (e.g., `24.200.001`).
*   **Example:** `http://localhost/AcumaticaDB/entity/Default/24.200.001/StockItem` (Accessing Stock Items)

## 2. Authentication

*   **Sign-in:** Requires a `POST` request to: `http://<Acumatica ERP instance URL>/entity/auth/login`
    *   **Request Body (JSON):**

    ```json
    {
      "name": "admin",  // Your Acumatica username
      "password": "123",  // Your Acumatica password
      "tenant": "MyStore",  // Your tenant name
      "branch": "MYSTORE",  // Your branch ID
      "locale": "EN-US" // Your locale (optional)
    }
    ```

    *   **Success (204):**  The response headers contain the cookies needed for subsequent requests.
    *   **Subsequent Requests:** Include the cookies received during sign-in in all subsequent requests.
*   **Sign-out:** Requires a `POST` request to: `http://<Acumatica ERP instance URL>/entity/auth/logout`

*   **Alternative (Recommended):** Implement OAuth 2.0 or OpenID Connect (OIDC) for authentication.

## 3. Common API Endpoints

*   **General:** `/entity`: Provides information about the API's structure and available endpoints.
*   **Examples:**

    *   `/entity/Default/24.200.001/StockItem` (Accessing and manipulating stock items).
    *   `/entity/Default/24.200.001/SalesOrder` (Accessing and manipulating sales orders).

## 4. Request and Response Body Data Structures (JSON)

*   **General Field:**  `<Field name> : { "value" : <Value>}`
    *   **Example:** `"CustomerID" : { "value" : "JOHNGOOD" }`
*   **Linked Entities:**

    ```json
    <Field name> :
    {
      <List of fields of the linked entity with values>
    }
    ```

    *   **Example:**

    ```json
    "MainContact" :
    {
      "Email" : {"value" : "demo@gmail.com" },
      "Address" :
      {
        "AddressLine1" : {"value" : "4030 Lake Washington Blvd NE" },
        "AddressLine2" : {"value": "Suite 100" },
        "City" : {"value": "Kirkland" },
        "State": {"value": "WA" },
        "PostalCode" : {"value": "98033" }
      }
    }
    ```
*   **Detail Entities:**

    ```json
    <Field name> :
    [
      {
        <List of fields of the detail entity with the values>
      },
      {
        <List of fields of the detail entity with the values>
      }
    ]
    ```

    *   **Example:**

    ```json
    "Details" :
    [
      {
        "InventoryID" : {"value": "AALEGO500"},
        "Quantity" : {"value": 10},
        "UOM" : {"value": "PIECE"}
      },
      {
        "InventoryID" : {"value": "CONGRILL"},
        "Quantity" : {"value":1},
        "UOM" : {"value": "PIECE"}
      }
    ]
    ```
*   **Custom Fields:**

    ```json
    "custom" :
    {
      <View name> :
      {
        <Field name> :
        {
          "type" : <value>,
          "value" : <value>
        }
      }
    }
    ```

    *   **Example:** (If you added a custom field called `UsrPersonalID` to the `DefContact` view of the contact entity)

    ```json
    "CustomerID" : {value: "JOHNGOOD" }
    "MainContact" :
    {
      "custom" :
      {
        "DefContact" :
        {
          "UsrPersonalID" :
          {
            "type" : "CustomStringField",
            "value" : "AB123456"
          }
        }
      }
    }
    ```

## 5. Parameters for Retrieving Records

*   `$filter`: Filter records.  Uses OData URI conventions (e.g., `$filter=ItemStatus eq 'Active'`).  Supports `AND`, `OR`, functions like `substringof`, `startswith`, and `endswith`.
*   `$top`:  Limits the number of records returned.
*   `$skip`: Skips a number of records from the beginning.
*   `$expand`: Specifies linked and detail entities to expand. (e.g., `$expand=WarehouseDetails`).
*   `$select`: Specifies the fields to be returned.  (e.g., `$select=OrderType, OrderNbr`).
*   `$custom`:  Specifies custom fields to be returned.  (e.g., `$custom=ItemSettings.UsrRepairItemType`).

## 6. Rate Limits and Error Handling

*   **Acumatica ERP License Limits:**  Each license includes limits on the number of web services API users,
    concurrent API requests, and API requests per minute.
*   **Error Handling:** The system returns HTTP status codes and error messages in the response body (e.g., code 400 for invalid data, code 429 for exceeding the rate limit).
*   **Example Error Body:**

    ```json
    "CustomerID": {
      "value": "ABARTENDE1",
      "error": "'Customer' cannot be found in the system."
    }
    ```

## 7. Practical API Examples (Focus on Key Functions)

*   **Create a Stock Item:**  `PUT /entity/<Endpoint name>/<Endpoint version>/StockItem`
    *   **Request Body:** (Example - Creating a customer record with the CustomerID and CustomerName)

    ```json
    {
      "CustomerID" : { "value" : "JOHNGOOD" },
      "CustomerName" : { "value" : "John Good" }
    }
    ```

    *   **If-None-Match Header:**  When creating a record, use the `If-None-Match: *` header to prevent duplicate record creation.
*   **Update a Stock Item:** `PUT /entity/<Endpoint name>/<Endpoint version>/StockItem? $filter=...`
    *   **Example:** (Update the customer's customer class)

    ```json
    PUT ?$filter=MainContact/Email%20eq%20'demo@gmail.com'
    $select=CustomerID, CustomerClass, MainContact/Email&$expand=MainContact /entity/<Endpoint name>/<Endpoint version>/Customer
    {
      "CustomerClass" : { "value" : "ECCUSTOMER"}
    }
    ```
*   **Retrieve a Record by Key Fields:**  `GET /entity/<Endpoint name>/<Endpoint version>/SalesOrder/SO/000123`
*   **Retrieve Records by Conditions:**  `GET /entity/<Endpoint name>/<Endpoint version>/StockItem?$filter=ItemStatus eq 'Active' and LastModified gt datetimeoffset'2024-07-15T10%3A31%3A28.402%2B03%3A00'`
*   **Execute an Action:** `POST /entity/<Endpoint name>/<Endpoint version>/SalesOrder/ConfirmShipment` (example)
    *   **Request Body:**

    ```json
    {
      "entity":{
        "ShipmentNbr": {"value": "000070"},
        "ShipmentType": {"value": "Shipment"}
      },
      "parameters" : {}
    }
    ```
*   **Retrieving the Schema of Custom Fields:**  `GET /entity/<Endpoint name>/<Endpoint version>/StockItem/$adHocSchema`

## 8. Important Notes

*   **Customization Impact:** Be aware that UI changes in Acumatica ERP (e.g., field name changes) can break existing API integrations. Use the screen-based API wrapper (Helper.GetSchema()) to mitigate these risks.
*   **Session Management:**  Sign in to Acumatica ERP before each API call and sign out after completing your operations.
*   **Performance:**  Use the `$select` parameter to retrieve only the necessary fields and the `$expand` parameter to limit the data included.
*   **Idempotency (PUT):** Use the `If-None-Match: *` header when creating records to prevent accidental duplicate creation.
*   **Date/Time Formatting:**  Use the correct OData format for date/time values (e.g., datetimeoffset).
*   **Testing:** Thoroughly test your API integrations in a test environment before deploying to production.

## 9. References

*   *Always* refer to the official Acumatica ERP documentation for the most up-to-date information.
*   See the example requests in the file `IntegrationDevelopmentGuide.postman_collection.json` in the `Help-and-Training-Examples` repository on GitHub.
*   API documentation is linked in the Acumatica ERP instance via the Web Service Endpoints (SM207060) form or the following URL:
    *   `http://<Acumatica ERP instance URL>/entity/swagger.json`