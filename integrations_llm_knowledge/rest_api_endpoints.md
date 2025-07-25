# Acumatica ERP REST API Endpoints - LLM Guide

This document provides comprehensive information about Acumatica ERP REST API endpoints, optimized for Large Language Model consumption.

## Base URL Structure

**Contract-Based REST API URL Format:**
```
https://{acumatica-instance}/entity/{endpoint-name}/{endpoint-version}/{entity}
```

**Components:**
- `{acumatica-instance}`: Your Acumatica ERP instance URL (e.g., `mycompany.acumatica.com`)
- `{endpoint-name}`: API endpoint name (typically "Default")
- `{endpoint-version}`: Version number (e.g., "24.200.001")
- `{entity}`: Entity name (e.g., "Customer", "SalesOrder")

**Example:**
```
https://mycompany.acumatica.com/entity/Default/24.200.001/Customer
```

## Finding Endpoint Information

### Locating Endpoint Version and Name

1. Log into Acumatica web interface
2. Navigate to **System** → **Integration** → **Web Service Endpoints** (SM207060)
3. Find your endpoint properties:
   - **Endpoint Name**: Usually "Default"
   - **Endpoint Version**: Format like "24.200.001"

### Schema Discovery

**Get Available Entities:**
```http
GET https://{acumatica-instance}/entity/{endpoint-name}/{endpoint-version}
```

**Get Entity Schema:**
```http
GET https://{acumatica-instance}/entity/{endpoint-name}/{endpoint-version}/{entity}/$schema
```

## Core Business Entities

### Financial Management
- **Account**: Chart of accounts
- **Customer**: Customer records
- **Vendor**: Vendor records
- **Invoice**: Accounts receivable invoices
- **Bill**: Accounts payable bills
- **Payment**: Customer payments
- **JournalTransaction**: General ledger entries

### Inventory Management
- **StockItem**: Inventory items
- **ItemWarehouse**: Item warehouse settings
- **InventoryIssue**: Inventory issues
- **InventoryReceipt**: Inventory receipts
- **InventoryQuantityAvailable**: Available quantities

### Sales Management
- **SalesOrder**: Sales orders
- **SalesInvoice**: Sales invoices
- **Shipment**: Shipments
- **Opportunity**: CRM opportunities
- **Lead**: Sales leads

### Purchasing
- **PurchaseOrder**: Purchase orders
- **PurchaseReceipt**: Purchase receipts

### Project Management
- **Project**: Projects
- **ProjectTask**: Project tasks
- **ProjectBudget**: Project budgets
- **TimeEntry**: Time entries

### Manufacturing
- **BillOfMaterial**: Bills of material
- **ProductionOrder**: Production orders
- **ProductionOrderDetail**: Production order details

### Payroll
- **Employee**: Employee records
- **PayrollBatch**: Payroll batches
- **EmployeePayrollSettings**: Employee payroll settings

## HTTP Methods and Operations

### GET - Retrieve Records

**Get All Records:**
```http
GET https://{acumatica-instance}/entity/Default/24.200.001/Customer
```

**Get Specific Record:**
```http
GET https://{acumatica-instance}/entity/Default/24.200.001/Customer/JOHNDOE
```

**Get with Expansion:**
```http
GET https://{acumatica-instance}/entity/Default/24.200.001/Customer/JOHNDOE?$expand=Contacts,Addresses
```

### POST - Create Records

```http
POST https://{acumatica-instance}/entity/Default/24.200.001/Customer
Content-Type: application/json

{
  "CustomerID": {"value": "NEWCUST"},
  "CustomerName": {"value": "New Customer Inc"},
  "CustomerClass": {"value": "RETAIL"}
}
```

### PUT - Update Records

```http
PUT https://{acumatica-instance}/entity/Default/24.200.001/Customer/JOHNDOE
Content-Type: application/json

{
  "CustomerName": {"value": "Updated Customer Name"},
  "Status": {"value": "Active"}
}
```

### DELETE - Remove Records

```http
DELETE https://{acumatica-instance}/entity/Default/24.200.001/Customer/JOHNDOE
```

## Query Parameters

### $filter - Filter Records
```http
GET https://{acumatica-instance}/entity/Default/24.200.001/Customer?$filter=Status eq 'Active'
```

**Filter Operators:**
- `eq`: Equal to
- `ne`: Not equal to
- `gt`: Greater than
- `ge`: Greater than or equal
- `lt`: Less than
- `le`: Less than or equal
- `and`: Logical AND
- `or`: Logical OR

### $expand - Include Related Data
```http
GET https://{acumatica-instance}/entity/Default/24.200.001/Customer?$expand=Contacts,Addresses
```

### $select - Specify Fields
```http
GET https://{acumatica-instance}/entity/Default/24.200.001/Customer?$select=CustomerID,CustomerName,Status
```

### $top - Limit Results
```http
GET https://{acumatica-instance}/entity/Default/24.200.001/Customer?$top=10
```

### $skip - Skip Records
```http
GET https://{acumatica-instance}/entity/Default/24.200.001/Customer?$skip=20
```

### $orderby - Sort Results
```http
GET https://{acumatica-instance}/entity/Default/24.200.001/Customer?$orderby=CustomerName asc
```

## Request/Response Format

### Request Headers
```http
Authorization: Bearer {access_token}
Content-Type: application/json
Accept: application/json
```

### JSON Field Structure
```json
{
  "FieldName": {
    "value": "FieldValue"
  }
}
```

### Nested Objects
```json
{
  "MainContact": {
    "Email": {"value": "contact@company.com"},
    "Phone1": {"value": "555-0123"}
  }
}
```

### Arrays/Collections
```json
{
  "Details": [
    {
      "InventoryID": {"value": "ITEM001"},
      "Qty": {"value": 10}
    },
    {
      "InventoryID": {"value": "ITEM002"},
      "Qty": {"value": 5}
    }
  ]
}
```

## Actions and Operations

### Entity Actions
Actions are operations that can be performed on entities beyond basic CRUD.

**Invoke Action:**
```http
POST https://{acumatica-instance}/entity/Default/24.200.001/SalesOrder/SO001234/ReleaseFromHold
```

**Common Actions:**
- `ReleaseFromHold`: Release document from hold
- `PutOnHold`: Put document on hold
- `Release`: Release document
- `Correct`: Correct released document

## File Attachments

### Upload File
```http
PUT https://{acumatica-instance}/entity/Default/24.200.001/Customer/JOHNDOE/files/document.pdf
Content-Type: application/pdf

[Binary file content]
```

### Download File
```http
GET https://{acumatica-instance}/entity/Default/24.200.001/Customer/JOHNDOE/files/document.pdf
```

### List Files
```http
GET https://{acumatica-instance}/entity/Default/24.200.001/Customer/JOHNDOE/files
```

## Error Handling

### Common HTTP Status Codes
- **200 OK**: Successful GET request
- **201 Created**: Successful POST request
- **204 No Content**: Successful PUT/DELETE request
- **400 Bad Request**: Invalid request format
- **401 Unauthorized**: Authentication required
- **403 Forbidden**: Access denied
- **404 Not Found**: Resource not found
- **500 Internal Server Error**: Server error

### Error Response Format
```json
{
  "message": "Error description",
  "exceptionMessage": "Detailed exception message",
  "exceptionType": "Exception type",
  "stackTrace": "Stack trace information"
}
```

## Performance Optimization

### Batch Operations
Process multiple records in a single request:
```json
[
  {
    "CustomerID": {"value": "CUST001"},
    "CustomerName": {"value": "Customer 1"}
  },
  {
    "CustomerID": {"value": "CUST002"},
    "CustomerName": {"value": "Customer 2"}
  }
]
```

### Field Selection
Only request needed fields:
```http
GET https://{acumatica-instance}/entity/Default/24.200.001/Customer?$select=CustomerID,CustomerName
```

### Pagination
Use $top and $skip for large datasets:
```http
GET https://{acumatica-instance}/entity/Default/24.200.001/Customer?$top=100&$skip=200
```

## Custom Fields

Access custom fields using their field names:
```json
{
  "UsrCustomField": {"value": "Custom Value"},
  "UsrAnotherField": {"value": "Another Value"}
}
```

## Generic Inquiries via OData

Access Generic Inquiries through OData endpoints:
```http
GET https://{acumatica-instance}/odatav4/{company}/{inquiry-name}
```

**Example:**
```http
GET https://{acumatica-instance}/odatav4/MyCompany/CustomerInquiry
```

This comprehensive guide covers the essential aspects of Acumatica ERP REST API endpoints for effective integration development.