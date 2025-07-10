***

# LLM Guide: Generating Acumatica REST API Calls

This guide provides the core principles and patterns for interacting with the Acumatica ERP REST API. It is based on the official Integration Development Guide examples.

## Core Principles

1.  **Authentication is Required**: All requests, except for the initial login, must be authenticated. The API uses cookie-based authentication.
2.  **Content-Type is JSON**: All request bodies are `application/json`. The `Accept` header should also be `application/json`.
3.  **Data Structure**:
    * When sending data (e.g., in a `PUT` or `POST` body), field values are typically wrapped in a `{"value": ...}` object. Example: `"CustomerID": {"value": "CUST01"}`.
    * The API uses OData conventions for querying.
4.  **Endpoints**: API calls are made to specific endpoints that expose Acumatica entities. The URL structure is generally:
    `{instance_url}/entity/{endpoint_name}/{version}/{entity_name}`
5.  **Actions**: Specific operations beyond simple CRUD are exposed as "actions." These are invoked with a `POST` request to the action's URL.

---

## 1. Authentication

You must perform a `login` request to authenticate and receive session cookies. These cookies are automatically used by the browser/client for subsequent requests. Always `logout` to terminate the session.

### Sign In

* **Method**: `POST`
* **Endpoint**: `{instance_url}/entity/auth/login`
* **Body**:

    ```json
    {
        "name": "{username}",
        "password": "{password}",
        "company": "{tenant_name}",
        "branch": "{branch_name}"
    }
    ```

### Sign Out

* **Method**: `POST`
* **Endpoint**: `{instance_url}/entity/auth/logout`

---

## 2. CRUD Operations

Standard Create, Retrieve, Update, and Delete operations form the basis of most interactions.

### a. Retrieve (GET)

#### Retrieve Multiple Records

Use `GET` on the entity's root. Use OData query parameters to control the response.

* **Method**: `GET`
* **Endpoint**: `{endpoint_url}/{entity_name}`
* **Key OData Parameters**:
    * `$select`: Specifies which fields to return.
        *Example: `$select=OrderNbr,OrderType,OrderTotal`
    * `$filter`: Filters the results based on field values.
        *Example: `$filter=ItemStatus eq 'Active' and LastModified gt datetimeoffset'2019-08-18T23:59:59Z'`
    * `$expand`: Includes related child entities in the response.
        *Example: `$expand=Details,Shipments`
    * `$top`: Limits the number of records returned.
        *Example: `$top=5`
    * `$skip`: Skips a specified number of records, used for paging.
        *Example: `$skip=5`

* **Example: Get the first 5 sales orders, expanding details.**

    ```http
    GET {endpoint_url}/SalesOrder?$top=5&$expand=Details
    ```

#### Retrieve a Single Record by ID

* **Method**: `GET`
* **Endpoint**: `{endpoint_url}/{entity_name}/{entity_id}`
* **Example: Get a sales order by its GUID ID.**

    ```http
    GET {endpoint_url}/SalesOrder/a6295b33-c7f6-e811-b817-00155d408001
    ```

#### Retrieve a Single Record by Key Fields

* **Method**: `GET`
* **Endpoint**: `{endpoint_url}/{entity_name}/{key1}/{key2}`
* **Example: Get a sales order by its type and number.**

    ```http
    GET {endpoint_url}/SalesOrder/SO/000001
    ```

### b. Create (PUT)

Use `PUT` on the entity's root endpoint. The body contains the full data for the new record. Acumatica will generate the key fields if `<NEW>` is not used.

* **Method**: `PUT`
* **Endpoint**: `{endpoint_url}/{entity_name}`
* **Body**: A JSON object representing the new entity.
* **Example: Create a new customer.**

    ```json
    {
      "CustomerID" : {"value" : "JOHNGOOD"},
      "CustomerName" : {"value" : "John Good"},
      "CustomerClass" : {"value" : "DEFAULT"}
    }
    ```

### c. Update (PUT or PATCH)

To update a record, use `PUT` or `PATCH`. `PUT` is used for a full entity update, while `PATCH` is ideal for updating only specific fields, especially in entities with details.

#### Simple Update (PUT)

Identify the record to update using a `$filter` query parameter.

* **Method**: `PUT`
* **Endpoint**: `{endpoint_url}/{entity_name}?$filter={field} eq '{value}'`
* **Body**: A JSON object with the fields to update.
* **Example: Update the customer class for a customer identified by email.**

    ```http
    PUT {endpoint_url}/Customer?$filter=MainContact/Email eq 'demo@gmail.com'
    ```

    ```json
    {
      "CustomerClass" : {"value" : "ECCUSTOMER"}
    }
    ```

#### Partial Update (PATCH)

This is used for updating specific fields within a record, including lines in a detail section. You must provide the `id` of the detail line to be changed.

* **Method**: `PATCH`
* **Endpoint**: `{endpoint_url}/{entity_name}`
* **Body**: A JSON object specifying the parent record's keys and a `Details` array with the fields to update, identified by their `id`.
* **Example: Update the quantity and discount on a specific sales order line.**

    ```json
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

### d. Delete (DELETE)

* **Method**: `DELETE`
* **Endpoint**: `{endpoint_url}/{entity_name}/{key1}/{key2}` OR `{endpoint_url}/{entity_name}/{id}`
* **Example: Delete a stock item by its key field.**

    ```http
    DELETE {endpoint_url}/StockItem/CGFEEDER
    ```

---

## 3. Executing Actions

Actions perform business logic beyond simple CRUD (e.g., releasing a document, reopening an order). They are invoked via a `POST` request.

* **Method**: `POST`
* **Endpoint**: `{endpoint_url}/{entity_name}/{ActionName}`
* **Body**: The body specifies the `entity` to act upon. It can be identified by its keys or its `id`. Some actions may require additional `parameters`.

* **Example: Reopen a sales order.**

    ```http
    POST {endpoint_url}/SalesOrder/ReopenSalesOrder
    ```

    ```json
    {
      "entity": {
        "OrderType": {"value": "SO"},
        "OrderNbr": {"value": "000001"}
      }
    }
    ```

* **Example: Release an invoice.**

    ```http
    POST {endpoint_url}/Invoice/ReleaseInvoice
    ```

    ```json
    {
      "entity": {
        "Type": {"value": "Invoice"},
        "ReferenceNbr": {"value": "INV000046"}
      }
    }
    ```

---

## 4. Advanced Operations

### Handling Custom Fields

* **Retrieve Schema**: To discover available custom fields, query the `$adHocSchema` endpoint.
    * `GET {endpoint_url}/StockItem/$adHocSchema`
* **Retrieve Data**: Use the `$custom` query parameter.
    * `GET {endpoint_url}/SalesOrder/SO/000087?$custom=Document.AttributePRODUCT`
* **Update/Create Data**: Include a `"custom"` object in the request body.

    ```json
    {
        "CustomerID": {"value": "GOODFOOD"},
        "OrderType": {"value": "SO"},
        "custom": {
            "Document": {
                "AttributePRODUCT": {
                    "type": "CustomStringField",
                    "value": "Apple jam 8 oz."
                }
            }
        }
    }
    ```

### File Attachments

Attaching files is a two-step process.

1.  **Get the upload URL**: `GET` the entity record and find the `files:put` link in the `_links` section of the response.
2.  **Upload the file**: Send a `PUT` request to the URL obtained in step 1.
    * **Method**: `PUT`
    * **URL**: The URL from the `files:put` link.
    * **Headers**: `Content-Type: application/octet-stream`
    * **Body**: The raw binary data of the file.

### Using Inquiry and Processing Screens

#### Inquiry Screens

Inquiry screens often require parameters to be passed in the body of a `PUT` request to get results.

* **Method**: `PUT`
* **Endpoint**: `{endpoint_url}/{InquiryScreenName}?$expand=Results`
* **Example: Get an inventory summary.**

    ```http
    PUT {endpoint_url}/InventorySummaryInquiry?$expand=Results
    ```

    ```json
    {
      "InventoryID": {"value": "SIMCARD"},
      "WarehouseID": {"value": "YOGI"}
    }
    ```

#### Processing Screens

This is a multi-step process for long-running operations.

1.  **Filter Records (Optional but Recommended)**: `PUT` filter values to the processing screen to narrow down the records to be processed. The response will contain a new record `id` representing the filtered list.
2.  **Execute Action**: `POST` to the action endpoint (e.g., `ProcessEmailProcessing`). The body contains the `Entity` object with the `id` from the previous step.
3.  **Check Status**: The action call will return a `202 Accepted` status with a `Location` header. `GET` this URL repeatedly until a `204 No Content` status is returned, indicating completion.

***

## 5. Entity-Specific Examples & Workflows

This section provides concrete examples for common Acumatica entities. These illustrate the patterns described in the previous sections.

### a. Customer

Used for managing customer records.

#### **Goal: Create a customer with a main contact and address.**

* **Method**: `PUT`
* **Endpoint**: `{endpoint_url}/Customer`
* **Body**:

    ```json
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
              "City" : {"value" : "Kirkland"},
              "State" : {"value" : "WA" },
              "PostalCode" : {"value" : "98033"}
            }
        }
    }
    ```

#### **Goal: Retrieve customers with their main contact's address.**

* **Method**: `GET`
* **Endpoint**: `{endpoint_url}/Customer?$expand=MainContact/Address`

---

### b. SalesOrder and Shipment (Workflow)

A common workflow is to create a sales order, create a shipment from that order, confirm the shipment, and then prepare the invoice.

#### **Step 1: Create a Sales Order**

* **Goal**: Create a new sales order for a customer with specific items.
* **Method**: `PUT`
* **Endpoint**: `{endpoint_url}/SalesOrder`
* **Body**:

    ```json
    {
        "OrderType": {"value": "SO"},
        "CustomerID": { "value": "GOODFOOD" },
        "Details": [
            {
                "InventoryID": { "value": "APJAM08" },
                "OrderQty": { "value": 2 },
                "UOM": { "value": "PIECE" }
            }
        ]
    }
    ```

#### **Step 2: Create a Shipment from the Sales Order**

This is an **action**. You must provide the ID of the sales order to be shipped.

* **Goal**: Initiate the shipment process for the created sales order.
* **Method**: `POST`
* **Endpoint**: `{endpoint_url}/SalesOrder/SalesOrderCreateShipment`
* **Body**:

    ```json
    {
      "entity":{
          "id":"{sales_order_guid}"
      },
      "parameters": {
            "ShipmentDate": { "value": "2025-08-20T00:00:00" },
            "WarehouseID": { "value": "RETAIL" }
      }
    }
    ```

#### **Step 3: Update Shipment with Packages and Tracking**

Once the shipment is created, you can add package details and tracking numbers.

* **Goal**: Add a package with a tracking number to an existing shipment.
* **Method**: `PUT`
* **Endpoint**: `{endpoint_url}/Shipment`
* **Body**:

    ```json
    {
        "id": "{shipment_guid}",
        "Packages": [
            {
                "id": "{package_guid}",
                "TrackingNbr": { "value": "398305336619" }
            }
        ]
    }
    ```

#### **Step 4: Confirm the Shipment**

This is an **action** that finalizes the shipment.

* **Goal**: Confirm the shipment is ready to leave the warehouse.
* **Method**: `POST`
* **Endpoint**: `{endpoint_url}/Shipment/ConfirmShipment`
* **Body**:

    ```json
    {
      "entity":{
        "ShipmentNbr": {"value": "{shipment_number}"},
        "Type":  {"value": "Shipment"}
      }
    }
    ```

---

### c. Bill and Purchase Order (Workflow)

This workflow shows creating a bill directly from existing purchase order lines.

#### **Step 1: Create a Bill from Purchase Order Lines**

The `Details` array links to existing POs. The system pulls in the line details automatically.

* **Goal**: Create a vendor bill for specific items received on a purchase order.
* **Method**: `PUT`
* **Endpoint**: `{endpoint_url}/Bill?$expand=Details`
* **Body**:

    ```json
    {
    	"Vendor": {
            "value": "PRINTICO"
        },
        "Description": {
            "value": "Bill for particular lines of a purchase order"
        },
        "Details": [
            {
                // Link to an existing Purchase Order line
                "POOrderType": { "value": "Normal" },
                "POOrderNbr": { "value": "000001" },
                 "POLine": { "value": 1 },
                "Qty":{ "value": 5 }
            }
        ]
    }
    ```

#### **Step 2: Release the Bill**

Releasing a bill makes it available for payment. This is an **action**.

* **Goal**: Make the bill active in the Accounts Payable module.
* **Method**: `POST`
* **Endpoint**: `{endpoint_url}/Bill/Release`
* **Body**:

    ```json
    {
        "entity": {
            "Type": {"value": "Bill"},
            "ReferenceNbr": {"value": "{bill_reference_nbr}"}
        }
    }
    ```

---

### d. Sales Invoice and Payment (Workflow)

This workflow shows creating a payment and applying it to an invoice.

#### **Step 1: Create an Invoice**

This example creates a standard invoice. In a real workflow, this might be generated from a shipment.

* **Goal**: Create a new sales invoice for a customer.
* **Method**: `PUT`
* **Endpoint**: `{endpoint_url}/SalesInvoice`
* **Body**:

    ```json
    {
        "CustomerID": { "value": "AACUSTOMER" },
        "Date": { "value": "2025-01-23T00:00:00" },
        "Description": { "value": "Invoice for services" },
        "Hold": {"value": false},
        "Details": [
            {
                "InventoryID": { "value": "CONSULTING" },
                "Qty": { "value": 10 },
                "UnitPrice": { "value": 100 }
            }
        ]
    }
    ```

#### **Step 2: Create and Release a Payment**

This example creates a payment document. The `Hold` is set to false, and it's released in the same step.

* **Goal**: Register a customer payment.
* **Method**: `PUT`
* **Endpoint**: `{endpoint_url}/Payment`
* **Body**:

    ```json
    {
        "CustomerID": { "value": "AACUSTOMER" },
        "PaymentMethod": { "value": "CHECK" },
        "CashAccount": { "value": "10200WH" },
        "PaymentAmount": { "value": 1000.00 },
        "Hold": {"value": false},
        "Type": { "value": "Payment" }
    }
    ```

#### **Step 3: Apply the Payment to the Invoice**

Update the Payment record to apply it to one or more documents.

* **Goal**: Apply the previously created payment to the invoice.
* **Method**: `PUT`
* **Endpoint**: `{endpoint_url}/Payment`
* **Body**:

    ```json
    {
        "ReferenceNbr": {"value": "{payment_reference_nbr}"},
        "Type": {"value": "Payment"},
        "DocumentsToApply": [
            {
                "DocType": { "value": "Invoice" },
                "ReferenceNbr": { "value": "{invoice_reference_nbr}" },
                "AmountPaid": { "value": 1000.00 }
            }
        ]
    }
    ```

#### **Step 4: Release the Payment**

The final step is to execute the `Release` action on the payment.

* **Goal**: Finalize the payment application.
* **Method**: `POST`
* **Endpoint**: `{endpoint_url}/Payment/Release`
* **Body**:

    ```json
    {
        "entity": {
            "Type": {"value": "Payment"},
            "ReferenceNbr": {"value": "{payment_reference_nbr}"}
        }
    }
    ```
***

## 6. Advanced Workflows & Entities

This section covers multi-step processes and more complex entity interactions, providing deeper context for creating robust integrations.

### a. Advanced Inventory Management

#### **Goal: Create a Stock Item with Attributes and UOM Conversions**

This shows how to create a new stock item, defining its characteristics (`Attributes`) and its packaging units (`UOMConversions`) in a single call.

* **Method**: `PUT`
* **Endpoint**: `{endpoint_url}/StockItem`
* **Body**:

    ```json
    {
        "InventoryID": {"value": "BASESERV1"},
        "Description": {"value": "Baseline level of performance"},
        "ItemClass": {"value": "STOCKITEM"},
        "Attributes": [
            {
                "AttributeID": {"value": "Operation System"},
                "Value": {"value": "Windows"}
            },
            {
                "AttributeID": {"value": "SOFTVER"},
                "Value": {"value": "Server 2012 R2"}
            }
        ],
        "UOMConversions": [
            {
                "FromUOM": {"value": "BOX"},
                "ToUOM": {"value": "PIECE"},
                "ConversionFactor": {"value": 12}
            }
        ]
    }
    ```

#### **Goal: Create and Release an Inventory Issue**

This two-step workflow is used to remove items from stock for internal use.

* **Step 1: Create the Inventory Issue document.**
    * **Method**: `PUT`
    * **Endpoint**: `{endpoint_url}/InventoryIssue?$expand=Details`
    * **Body**:

        ```json
        {
            "Date": { "value": "2025-12-02T00:00:00" },
            "Description": { "value": "Internal use for marketing" },
            "Details": [
                {
                    "InventoryID": { "value": "APJAM08" },
                    "Warehouse": { "value": "RETAIL" },
                    "Location": { "value": "MAIN" },
                    "Qty": { "value": 1 },
                    "UOM": { "value": "PIECE" }
                }
            ]
        }
        ```

* **Step 2: Release the Inventory Issue document (Action).**
    * **Method**: `POST`
    * **Endpoint**: `{endpoint_url}/InventoryIssue/ReleaseInventoryIssue`
    * **Body**:

        ```json
        {
            "entity": {
                "ReferenceNbr": { "value": "{issue_reference_nbr}" }
            }
        }
        ```

### b. Advanced Sales Order - Returns and Taxes

#### **Goal: Create a Return for Credit (RC) from a previous Invoice**

This is used when a customer returns goods for credit, referencing the original sale.

* **Method**: `PUT`
* **Endpoint**: `{endpoint_url}/SalesOrder?$expand=Details`
* **Body**:

    ```json
    {
        "OrderType": { "value": "RC" }, // RC indicates Return for Credit
        "CustomerID": { "value": "GOODFOOD" },
        "Description": { "value": "Return for Credit for Two Invoices" },
        "Details": [
            {
                // Links the return line to the original invoice line
                "InvoiceType": { "value": "Invoice" },
                "InvoiceNbr": { "value": "000062" },
                "InventoryID": { "value": "INSTALL" }
            },
            {
                "InvoiceType": { "value": "Invoice" },
                "InvoiceNbr": { "value": "000030" },
                "InventoryID": { "value": "ORANGES" },
                "OrderQty": { "value": "50" } // Quantity being returned
            }
        ]
    }
    ```

#### **Goal: Create a Sales Order with Overridden Tax Calculations**

This is for situations where taxes must be manually specified, overriding the system's automatic calculation.

* **Method**: `PUT`
* **Endpoint**: `{endpoint_url}/SalesOrder?$expand=TaxDetails`
* **Body**:

    ```json
    {
        "CustomerID": { "value": "GOODFOOD" },
        "IsTaxValid": { "value": true }, // Flag to tell Acumatica the manual tax data is valid
        "Details": [
            {
                "InventoryID": { "value": "APJAM08" },
                "OrderQty": { "value": 2 }
            }
        ],
        "TaxDetails": [
            {
                "TaxID": { "value": "NYSTATETAX" },
                "TaxAmount": { "value": 0.50 } // Manually specified tax amount
            }
        ]
    }
    ```

### c. Project Accounting Workflow (Pro Forma Invoice)

This detailed workflow covers the lifecycle from project creation to billing.

#### **Step 1: Create a Project from a Template**

* **Goal**: Initiate a new project using predefined settings from a template.
* **Method**: `PUT`
* **Endpoint**: `{endpoint_url}/Project`
* **Body**:

    ```json
    {
        "ProjectID" : {"value" : "TESTPR3"},
        "ProjectTemplateID" : {"value" : "PROGRESS"}, // Using a template
        "Customer" : {"value" : "COFFEESHOP"},
        "BillingAndAllocationSettings" :
        {
            "BillingRule" : {"value" : "PROGRESS"},
            "BillingPeriod" : {"value" : "Month"}
        }
    }
    ```

#### **Step 2: Activate the Project and a Project Task**

* **Goal**: Make the project and its first task active. This requires two separate API calls.
* **Call 1: Activate Project**
    * **Method**: `PUT`
    * **Endpoint**: `{endpoint_url}/Project`
    * **Body**: `{ "ProjectID": {"value": "TESTPR3"}, "Hold": {"value": false} }`
* **Call 2: Activate Project Task (Action)**
    * **Method**: `POST`
    * **Endpoint**: `{endpoint_url}/ProjectTask/Activate`
    * **Body**:

        ```json
        {
            "entity":
            {
                "ProjectID": { "value": "TESTPR3" },
                "ProjectTaskID": { "value": "PHASE1" }
            }
        }
        ```

#### **Step 3: Update Task Progress and Pending Invoice Amount**

* **Goal**: Record the percentage of completion for a task, which drives progress billing.
* **Method**: `PUT`
* **Endpoint**: `{endpoint_url}/ProjectBudget`
* **Body**:

    ```json
    {
      "ProjectTaskID" : {"value" : "PHASE1"},
      "ProjectID" : {"value" : "TESTPR3"},
      "Completed" : {"value" : 25}, // Setting progress to 25%
      "PendingInvoiceAmount" : {"value" : 725}
    }
    ```

#### **Step 4: Run Project Billing (Action)**

* **Goal**: Trigger the system to generate pro forma invoices based on the progress recorded. This is a long-running process.
* **Method**: `POST`
* **Endpoint**: `{endpoint_url}/Project/RunProjectBilling`
* **Body**:

    ```json
    {
      "entity" : { "ProjectID": { "value": "TESTPR3" } }
    }
    ```

    *Note: This will return a `202 Accepted` status. You must monitor the `Location` header URL until it returns `204 No Content`.*

#### **Step 5: Retrieve and Email the Pro Forma Invoice**

* **Goal**: Get the generated pro forma invoice and email it.
* **Call 1: Retrieve Pro Forma Invoice Details**
    * **Method**: `GET`
    * **Endpoint**: `{endpoint_url}/Project/TESTPR3?$expand=Invoices`
* **Call 2: Email the Invoice (Action)**
    * **Method**: `POST`
    * **Endpoint**: `{endpoint_url}/ProFormaInvoice/EmailProFormaInvoice`
    * **Body**:

        ```json
        {
          "entity" : {
            "RefNbr": { "value": "{pro_forma_reference_nbr}" }
          }
        }
        ```

***

## 7. Manufacturing Workflows

This section covers entities and workflows specific to the Acumatica Manufacturing Edition.

### a. Bill of Material (BOM)

#### **Goal: Create a Bill of Material with Operations and Materials**

This example shows the creation of a complex, hierarchical BOM record.

* **Method**: `PUT`
* **Endpoint**: `{manufacturing_endpoint_url}/BillOfMaterial?$expand=Operations/Material`
* **Body**:

    ```json
    {
        "BOMID": { "value": "<NEW>" },
        "Revision": { "value": "A" },
        "Description": {"value": "Test BOM for Cabinet"},
        "InventoryID": {"value": "CABINET"},
        "Operations": [
            {
                "OperationNbr": {"value": "0010"},
                "WorkCenter": {"value": "WC10"},
                "Material" :[
                    {
                        "InventoryID": {"value": "HINGE"},
                        "UOM":{"value": "EA"}
                    }
                ]
            }
        ]
    }
    ```

### b. Production Order Management

#### **Goal: Update Dates on a Production Order**

This shows how to update the start and end dates for a specific production order and its associated operations.

* **Method**: `PUT`
* **Endpoint**: `{manufacturing_endpoint_url}/ProductionOrderDetail?$expand=Operations`
* **Body**:

    ```json
    {
        "OrderType": { "value": "RO" },
        "ProductionNbr": { "value": "AM000037" },
        "SchedulingMethod": { "value": "User Dates" },
        "StartDate": { "value": "2025-01-08T00:00:00" },
        "EndDate": { "value": "2025-01-30T00:00:00" },
        "Operations": [
            {
                "id": "{operation_guid_1}",
                "StartDate": { "value": "2025-01-08T00:00:00" },
                "EndDate": { "value": "2025-01-30T00:00:00" }
            },
            {
                "id": "{operation_guid_2}",
                "StartDate": { "value": "2025-01-08T00:00:00" },
                "EndDate": { "value": "2025-01-30T00:00:00" }
            }
        ]
    }
    ```

---

## 8. Payroll & Employee Setup

This section covers creating the necessary records for employee payroll.

### a. Supporting Payroll Records

#### **Goal: Create a Deduction Code**

* **Method**: `PUT`
* **Endpoint**: `{endpoint_url}/DeductionBenefitCode`
* **Body**:

    ```json
    {
        "DeductionBenefitCodeID": { "value": "TST" },
        "Description": { "value": "Test deduction" },
        "ContributionType": { "value": "DED" },
        "AssociatedWith": { "value": "Employee Settings" },
        "EmployeeDeduction": {
            "CalculationMethod": { "value": "GRS" },
            "Percent": { "value": 20 }
        },
        "GLAccounts": {
            "DeductionLiabilityAccount": { "value": "20000" }
        }
    }
    ```

#### **Goal: Create a Work Location**

* **Method**: `PUT`
* **Endpoint**: `{endpoint_url}/WorkLocation?$expand=AddressInfo`
* **Body**:

    ```json
    {
        "Active": { "value": true },
        "WorkLocationID": { "value": "DELLEVUE" },
        "WorkLocationName": { "value": "New address location" },
        "AddressInfo": {
            "AddressLine1": { "value": "1st Browny St." },
            "City": { "value": "Bellevue" },
            "Country": { "value": "US" },
            "PostalCode": { "value": "98004" },
            "State": { "value": "WA" }
        }
    }
    ```

### b. Employee Payroll Settings Workflow

#### **Goal: Configure Payroll Settings for an Existing Employee**

This workflow updates an employee's record with essential payroll information.

* **Method**: `PUT`
* **Endpoint**: `{endpoint_url}/EmployeePayrollSettings?$expand=WorkLocations/WorkLocationDetails,EmploymentRecords`
* **Body**:

    ```json
    {
        "EmployeeID": { "value": "EP00000004" },
        "ClassID": { "value": "HOURLY" },
        "PaymentMethod": { "value": "CHECK" },
        "CashAccount": { "value": "10200WH" },
        "WorkLocations": {
            "WorkLocationClassDefaults": { "value": false },
            "WorkLocationDetails": [
                {
                    "LocationID": { "value": "BELLEVUE" },
                    "DefaultWorkLocation": { "value": true }
                }
            ]
        },
        "EmploymentRecords": [
            {
                "StartDate": { "value": "2021-05-13" },
                "StartReason": { "value": "REH" },
                "Active": { "value": true },
                "Position": { "value": "ACCOUNTANT" }
            }
        ]
    }
    ```

---

## 9. Other Advanced API Patterns

#### **Goal: Filter Records using a Custom Field**

The syntax `cf.Type(f='{FieldName}')` is used to filter by custom fields.

* **Method**: `GET`
* **Endpoint**: `{endpoint_url}/SalesInvoice?$filter=cf.Decimal(f='Document.CuryBalanceWOTotal') eq 0M and cf.DateTime(f='Document.DiscDate') gt datetimeoffset'2024-02-18T23:59:59Z'`

#### **Goal: Update a Nested Allocation in a Sales Order**

This shows how to pinpoint a single allocation line deep inside a sales order for modification. You must provide the `id` or keys for each level of the hierarchy.

* **Method**: `PUT`
* **Endpoint**: `{endpoint_url}/SalesOrder?$expand=Details,Details/Allocations`
* **Body**:

    ```json
    {
        "OrderNbr": { "value": "{order_number}" },
        "OrderType": { "value": "RC" },
        "Details": [
            {
                "id": "{detail_line_guid}", // ID of the sales order detail line
                "Allocations": [
                    {
                        "id": "{allocation_line_guid}", // ID of the specific allocation to update
                        "LotSerialNbr": { "value": "LREX15098" } // New value
                    }
                ]
            }
        ]
    }
    ```