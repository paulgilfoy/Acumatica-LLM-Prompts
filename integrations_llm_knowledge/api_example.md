# Acumatica ERP Integration API - Example Calls

This document provides example API calls and their corresponding responses for training an LLM.

## 1.  Sign-in

*   **API Call:**

    ```
    POST http://localhost/AcumaticaDB/entity/auth/login
    ```

    *   **Request Body (JSON):**

    ```json
    {
      "name": "admin",
      "password": "123",
      "tenant": "MyStore",
      "branch": "MYSTORE",
      "locale": "EN-US"
    }
    ```

*   **Response:**
    *   **Code:** 204
    *   **Headers:**  Includes cookies (details not shown here).

## 2. Create a Stock Item

*   **API Call:**

    ```
    PUT http://localhost/AcumaticaDB/entity/Default/24.200.001/StockItem
    ```

    *   **Request Body (JSON):**

    ```json
    {
      "CustomerID" : { "value" : "JOHNGOOD" },
      "CustomerName" : { "value" : "John Good" }
    }
    ```

*   **Response:**
    *   **Code:** 200
    *   **Body (JSON):** (Example Response - Specific fields will vary)

    ```json
    {
        "CustomerID": {
            "value": "JOHNGOOD"
        },
        "CustomerName": {
            "value": "John Good"
        }
    }
    ```

## 3. Update a Customer (Example: Customer Class)

*   **API Call:**

    ```
    PUT ?$filter=MainContact/Email%20eq%20'demo@gmail.com'&$select=CustomerID,CustomerClass,MainContact/Email&$expand=MainContact /entity/Default/24.200.001/Customer
    ```

    *   **Request Body (JSON):**

    ```json
    {
      "CustomerClass" : { "value" : "ECCUSTOMER"}
    }
    ```

*   **Response:**
    *   **Code:** 200
    *   **Body (JSON):** (Example Response - Fields will vary)

    ```json
    {
        "CustomerID": {
            "value": "JOHNGOOD"
        },
        "CustomerClass": {
            "value": "ECCUSTOMER"
        }
    }
    ```

## 4. Retrieve a Record by Key Fields (Sales Order)

*   **API Call:**

    ```
    GET http://localhost/AcumaticaDB/entity/Default/24.200.001/SalesOrder/SO/000123
    ```

*   **Response:**
    *   **Code:** 200
    *   **Body (JSON):** (Example Response - Fields will vary)

    ```json
    {
        ...  // Sales order details
    }
    ```

## 5. Retrieve Records by Conditions (Stock Items)

*   **API Call:**

    ```
    GET http://localhost/AcumaticaDB/entity/Default/24.200.001/StockItem?$filter=ItemStatus eq 'Active' and LastModified gt datetimeoffset'2024-07-15T10%3A31%3A28.402%2B03%3A00'
    ```

*   **Response:**
    *   **Code:** 200
    *   **Body (JSON):** (Example Response -  List of stock items meeting the criteria)

    ```json
    [
        {
            ... // Stock Item 1
        },
        {
            ... // Stock Item 2
        }
    ]
    ```

## 6. Execute an Action (Confirm Shipment)

*   **API Call:**

    ```
    POST /entity/Default/24.200.001/SalesOrder/ConfirmShipment
    ```

    *   **Request Body (JSON):**

    ```json
    {
      "entity":{
        "id": "42bb9a17-a402-e911-b818-00155d408001"
      },
      "parameters" : {}
    }
    ```

*   **Response:**
    *   **Code:** 202
    *   **Headers:**  `Location` header (URL to check status).

## 7. Create an RMA Order

*   **API Call:**

    ```
    PUT /entity/Default/24.200.001/SalesOrder
    ```

    *   **Request Body (JSON):**

    ```json
    {
        "CustomerID": { "value": "FRUITICO" },
        "OrderType": { "value": "RM" },
        "Details": [
            {
                "Branch": { "value": "HEADOFFICE" },
                "InvoiceNbr": { "value": "000126" },
                "Operation": { "value": "Receipt" },
                "InventoryID": { "value": "SWB-32OZ-GBT" },
                "OrderQty": { "value": -1 },
                "UOM": { "value": "EA" },
                "WarehouseID": { "value": "WHOLESALE" },
                "AutoCreateIssue": { "value": false }
            },
            {
                "Branch": { "value": "HEADOFFICE" },
                "Operation": { "value": "Issue" },
                "InventoryID": { "value": "PEARJAM96" },
                "OrderQty": { "value": 1 },
                "UOM": { "value": "PIECE" },
                "WarehouseID": { "value": "RETAIL" },
                "AutoCreateIssue": { "value": false }
            }
        ]
    }
    ```

*   **Response:**
    *   **Code:** 200
    *   **Body (JSON):** (The created Sales Order)

## 8. Create a Credit Memo (Returning Multiple Items and Applying Payment)

*   **API Call:**

    ```
    PUT ?$expand=Details HTTP/1.1
    Host: [<Acumatica ERP instance URL>]/entity/Default/24.200.001/SalesInvoice
    ```

    *   **Request Body (JSON):**

    ```json
    {
        "CustomerID": { "value": "FRUITICO" },
        "Type": {"value": "Credit Memo"},
        "Hold": {"value": false},
        "Details":[
        {
        "OrigInvNbr": { "value": "000137" },
        "OrigInvType": { "value": "Invoice" },
        "OrigInvLineNbr": { "value": 1 },
        "Qty": { "value": 1 }
        },
        {
        "OrigInvNbr": { "value": "000030" },
        "OrigInvType": { "value": "Invoice" },
        "OrigInvLineNbr": { "value": 1 },
        "Qty": { "value": 1 }
        },
        {
        "Branch": {"value": "HEADOFFICE"},
        "InventoryID": {"value": "APJAM96"},
        "WarehouseID": {"value": "WHOLESALE"},
        "Location": {"value": "MAIN"},
        "Qty": {"value": 1},
        "UOM": {"value": "PIECE"}
        }
        ],
        "ApplicationsCreditMemo":[
        {
        "DocType": {"value": "Payment"},
        "AdjustingDocReferenceNbr": {"value": "000076"},
        "AmountPaid": {"value": 8.32}
        }
        ]
    }
    ```
    *   **Response:**
        *   **Code:** 200
        *   **Body:** (JSON - newly created invoice data)

## 9. Create a Sales Invoice with Tax Parameters Overridden

*   **API Call:**

    ```
    PUT ?$expand=Details, TaxDetails HTTP/1.1
    Host: [<Acumatica ERP instance URL>]/entity/Default/24.200.001/SalesInvoice
    ```

    *   **Request Body (JSON):**

    ```json
    {
    "CustomerID": { "value": "GOODFOOD" },
    "Type": {"value": "Invoice"},
    "Hold": {"value": false},
        "Details": [
    {
        "Branch": {"value": "HEADOFFICE"},
        "InventoryID": {"value": "APJAM96"},
        "OrderQty": {"value": 2},
        "UOM": {"value": "BOX"},
        "WarehouseID": {"value": "WHOLESALE"}
    }
    ],
    "TaxDetails": [
    {
      "TaxID": { "value": "NYSTATETAX" },
      "TaxAmount": { "value": 0.5 }
    }
    ]
    }
    ```

    *   **Response:**
        *   **Code:** 200
        *   **Body:** (JSON - invoice data)

## 10. Retrieve a List of Vendors with the CashAccount Field Matching 10200TG

*   **API Call:**

    ```
    GET ?$filter=CashAccount%20eq%20'10200TG' HTTP/1.1
    Host: [<Acumatica ERP instance URL>]/entity/Default/24.200.001/Vendor
    ```

    *   **Response:**
        *   **Code:** 200
        *   **Body (JSON):** (List of vendors - fields will vary, but include Vendor data. May return an empty list)
        ```json
        [
            {
                ... // Vendor 1 Data
            },
            {
                ... // Vendor 2 Data
            }
        ]
        ```

## 11. Create a Sales Order with Unit of Measure Specified

*   **API Call:**

    ```
    PUT ?$expand=Details&$select=CustomerID, Details/Branch, Details/InventoryID, Details/OrderQty, OrderNbr HTTP/1.1
    Host: [<Acumatica ERP instance URL>]/entity/Default/24.200.001/SalesOrder
    ```

    *   **Request Body (JSON):**

    ```json
    {
      "CustomerID": { "value": "GOODFOOD" },
      "Details": [
        {
          "Branch": { "value": "HEADOFFICE" },
          "InventoryID": { "value": "APJAM08" },
          "OrderQty": { "value": 2 },
          "UOM": { "value": "PIECE" },
          "WarehouseID": { "value": "WHOLESALE" }
        }
      ]
    }
    ```

    *   **Response:**
        *   **Code:** 200
        *   **Body (JSON):** (Sales Order data)

## 12. Create a Payment Method

*   **API Call:**

    ```
    PUT / HTTP/1.1
    Host: [<Acumatica ERP instance URL>]/entity/Default/24.200.001/PaymentMethod
    ```

    *   **Request Body (JSON):**

    ```json
    {
        "Active": { "value": false },
        "Description": { "value": "Test Method" },
        "IntegratedProcessing": { "value": false },
        "MeansOfPayment": { "value": "Cash/Check" },
        "PaymentMethodID": { "value": "TST" },
        "SettingsForPR": {
        "PRProcessing": { "value": "Print Checks" },
        "Report": { "value": "PR641010" },
        "UseInPR": { "value": true },
        "UseInAP": { "value": false },
        "UseInAR": { "value": false },
        "RequireRemittanceInformationforCashAccount": { "value": false },
        "AllowedCashAccounts": 
        [{
                "CashAccount": { "value": "10200WH" },
                "Description": { "value": "Company Checking Account" }
        }]
        }
    }```
*   **Response:**
    *   **Code:** 200
    *   **Body:** (Data for the payment method)

## 13. Create a Return for Credit for Items with Lot or Serial Numbers

*   **API Call:**

    ```
    PUT ?$expand=Details HTTP/1.1
    Host: [<Acumatica ERP instance URL>]/entity/Default/24.200.001/SalesOrder
    ```

    *   **Request Body (JSON):**

    ```json
    {
    "CustomerID": { "value": "FRUITICO" },
    "Description": { "value": "Return for Credit for Items with Lot or Serial Numbers" },
    "Details": [
        {
            "InvoiceType": { "value": "Invoice" },
            "InvoiceNbr": { "value": "AR013584" },
            "InvoiceLineNbr": { "value": "2" },
            "OrderQty": { "value": 20 },
            "UOM": { "value": "EA" }
        },
        {
            "InvoiceType": { "value": "Invoice" },
            "InvoiceNbr": { "value": "AR013584" },
            "InvoiceLineNbr": { "value": "3" },
            "OrderQty": { "value": 6900 },
            "UOM": { "value": "EA" }
        }
    ],
    "LocationID": { "value": "MAIN" },
    "OrderType": { "value": "RC" }
    }


*   **Response:**
    *   **Code:** 200
    *   **Body** (JSON): (Return sales order data)

## 14. Retrieve the Status of the Release Operation

*   **API Call:**  (Replace with the *actual* URL obtained from the `Location` header of a previous `POST` call)

    ```
    GET /entity/<Endpoint name>/<Endpoint version>/SalesInvoice/ReleaseSalesInvoice/status/<UUID>
    ```

    *   **Response:**
        *   **Code:** 200 or 204 (if complete)
        *   **Code:** 202 (if in progress)
        *   **Body:** (Empty for 204) (May contain information to determine the state when in progress)

## 15. Create a Sales Order with an External Credit Card Payment

*   **API Call:**

    ```
    PUT /entity/Default/24.200.001/SalesOrder
    ```

    *   **Request Body (JSON):**

    ```json
    {
        "CustomerID": {"value": "WIDGETCC"},
        "Date": {"value": "2023-05-10T00:00:00"},
        "Description": {"value": "External CC payment"},
        "Details": [
          {
            "Branch": {"value": "PRODWHOLE"},
            "DiscountAmount": {"value": 10.00},
            "ExtendedPrice": {"value": 500.00},
            "FreeItem": {"value": false},
            "InventoryID": {"value": "AACOMPUT01"},
            "OrderQty": {"value": 1.00},
            "UnitPrice": {"value": 500.00},
            "UOM": {"value": "EA"},
            "WarehouseID": {"value": "WHOLESALE"}
          },
          {
            "Branch": {"value": "PRODWHOLE"},
            "DiscountAmount": {"value": 10.00},
            "ExtendedPrice": {"value": 500.00},
            "FreeItem": {"value": false},
            "InventoryID": {"value": "AALEGO500"},
            "OrderQty": {"value": 50.00},
            "UnitPrice": {"value": 50.00},
            "UOM": {"value": "EA"},
            "WarehouseID": {"value": "WHOLESALE"}
          }
        ],
        "Hold": {"value": false},
        "LocationID": {"value": "MAIN"},
        "OrderType": {"value": "SO"},
        "Payments": [
          {
            "ApplicationDate": {"value": "2023-08-11T00:00:00+03:00"},
            "AppliedToOrder": {"value": 480.00},
            "CashAccount": {"value": "10600"},
            "CreditCardTransactionInfo": [
              {
                "TranNbr": {"value": "40050474170"},
                "TranType": {"value": "AUT"}
              }
            ],
            "PaymentAmount": {"value": 980.00},
            "PaymentMethod": {"value": "ACUPAYCC"}
          }
        ],
        "PaymentMethod": {"value": "ACUPAYCC"},
        "RequestedOn": {"value": "2023-05-10T00:00:00"}
    }
    ```

    *   **Response:**
        *   **Code:** 200
        *   **Body (JSON):**  (Sales Order data). The Payment is in pending processing.
