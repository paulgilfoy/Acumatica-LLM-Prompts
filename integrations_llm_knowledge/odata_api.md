# Acumatica ERP OData API - LLM Guide

This document provides comprehensive information about Acumatica ERP OData API, optimized for Large Language Model consumption.

## OData Overview

Acumatica ERP provides OData (Open Data Protocol) endpoints for accessing Generic Inquiries and specific data views. OData is particularly useful for reporting, analytics, and read-only integrations.

## OData vs REST API Comparison

| Feature | OData API | REST API |
|---------|-----------|----------|
| Primary Use | Read-only queries, reporting | Full CRUD operations |
| Data Source | Generic Inquiries | Business entities |
| Query Capabilities | Advanced OData queries | Basic filtering |
| Performance | Optimized for analytics | Optimized for transactions |
| Complexity | Lower learning curve | Higher learning curve |

## OData URL Structure

**Base OData URL Format:**
```
https://{acumatica-instance}/odatav4/{company}/{inquiry-name}
```

**Components:**
- `{acumatica-instance}`: Your Acumatica ERP instance URL
- `{company}`: Company database name (required for OData)
- `{inquiry-name}`: Name of the Generic Inquiry

**Example:**
```
https://mycompany.acumatica.com/odatav4/MyCompany/CustomerInquiry
```

## Authentication for OData

### Basic Authentication
```http
GET https://mycompany.acumatica.com/odatav4/MyCompany/CustomerInquiry
Authorization: Basic {base64-encoded-credentials}
```

### OAuth 2.0 Authentication
```http
GET https://mycompany.acumatica.com/odatav4/MyCompany/CustomerInquiry
Authorization: Bearer {access_token}
```

## Connection Properties for OData

When configuring OData connections:

- **URL**: Base Acumatica instance URL
- **Schema**: Set to "OData" 
- **Company**: Company/tenant name (required)
- **User**: Acumatica username
- **Password**: Acumatica password

## Available Generic Inquiries

### Standard Inquiry Tables (Contract 3 API version 17.200.001)

- **AccountByPeriodInquiry**: Account balances by period
- **AccountBySubaccountInquiry**: Account balances by subaccount
- **AccountDetailsInquiry**: Detailed account information
- **AccountSummaryInquiry**: Account summary data
- **InventoryAllocationInquiry**: Inventory allocation details
- **InventorySummaryInquiry**: Inventory summary information
- **InvoicedItemsInquiry**: Invoiced items data
- **SalesPricesInquiry**: Sales price information
- **VendorPricesInquiry**: Vendor price information

### Discovering Available Inquiries

**List all available inquiries:**
```http
GET https://{acumatica-instance}/odatav4/{company}/$metadata
```

**Get inquiry schema:**
```http
GET https://{acumatica-instance}/odatav4/{company}/{inquiry-name}/$metadata
```

## OData Query Operations

### Basic Data Retrieval

**Get all records:**
```http
GET https://{acumatica-instance}/odatav4/MyCompany/CustomerInquiry
```

**Response format:**
```json
{
  "@odata.context": "https://mycompany.acumatica.com/odatav4/MyCompany/$metadata#CustomerInquiry",
  "value": [
    {
      "CustomerID": "CUST001",
      "CustomerName": "Customer One",
      "Status": "Active"
    },
    {
      "CustomerID": "CUST002", 
      "CustomerName": "Customer Two",
      "Status": "Active"
    }
  ]
}
```

### $filter - Filtering Data

**Basic filtering:**
```http
GET https://{acumatica-instance}/odatav4/MyCompany/CustomerInquiry?$filter=Status eq 'Active'
```

**Multiple conditions:**
```http
GET https://{acumatica-instance}/odatav4/MyCompany/CustomerInquiry?$filter=Status eq 'Active' and CustomerClass eq 'RETAIL'
```

**Date filtering:**
```http
GET https://{acumatica-instance}/odatav4/MyCompany/SalesInquiry?$filter=OrderDate ge 2024-01-01T00:00:00Z
```

**String functions:**
```http
GET https://{acumatica-instance}/odatav4/MyCompany/CustomerInquiry?$filter=contains(CustomerName,'Inc')
```

### $select - Field Selection

**Select specific fields:**
```http
GET https://{acumatica-instance}/odatav4/MyCompany/CustomerInquiry?$select=CustomerID,CustomerName,Status
```

### $orderby - Sorting

**Sort ascending:**
```http
GET https://{acumatica-instance}/odatav4/MyCompany/CustomerInquiry?$orderby=CustomerName
```

**Sort descending:**
```http
GET https://{acumatica-instance}/odatav4/MyCompany/CustomerInquiry?$orderby=CustomerName desc
```

**Multiple sort fields:**
```http
GET https://{acumatica-instance}/odatav4/MyCompany/CustomerInquiry?$orderby=Status,CustomerName desc
```

### $top and $skip - Pagination

**Limit results:**
```http
GET https://{acumatica-instance}/odatav4/MyCompany/CustomerInquiry?$top=10
```

**Skip records (pagination):**
```http
GET https://{acumatica-instance}/odatav4/MyCompany/CustomerInquiry?$skip=20&$top=10
```

### $count - Record Counting

**Get total count:**
```http
GET https://{acumatica-instance}/odatav4/MyCompany/CustomerInquiry?$count=true
```

**Get count only:**
```http
GET https://{acumatica-instance}/odatav4/MyCompany/CustomerInquiry/$count
```

## Advanced OData Queries

### Combining Multiple Parameters

```http
GET https://{acumatica-instance}/odatav4/MyCompany/CustomerInquiry?
$filter=Status eq 'Active'&
$select=CustomerID,CustomerName&
$orderby=CustomerName&
$top=50&
$count=true
```

### Date and Time Functions

**Filter by date range:**
```http
GET https://{acumatica-instance}/odatav4/MyCompany/SalesInquiry?
$filter=OrderDate ge 2024-01-01T00:00:00Z and OrderDate le 2024-12-31T23:59:59Z
```

**Use date functions:**
```http
GET https://{acumatica-instance}/odatav4/MyCompany/SalesInquiry?
$filter=year(OrderDate) eq 2024
```

### String Functions

**Contains:**
```http
$filter=contains(CustomerName,'Corporation')
```

**Starts with:**
```http
$filter=startswith(CustomerID,'CUST')
```

**Ends with:**
```http
$filter=endswith(CustomerName,'Inc')
```

**Length:**
```http
$filter=length(CustomerID) eq 8
```

### Arithmetic Functions

**Mathematical operations:**
```http
$filter=TotalAmount gt 1000.00
```

**Round function:**
```http
$filter=round(TotalAmount) eq 1500
```

## Working with Generic Inquiries

### Creating Custom Generic Inquiries

1. Navigate to **System** → **Customization** → **Generic Inquiry** (SM208000)
2. Create new inquiry with desired tables and fields
3. Set inquiry as "Exposed via OData" 
4. Save and publish the inquiry

### Inquiry Design Best Practices

- **Performance**: Limit result sets with appropriate filters
- **Indexing**: Ensure underlying tables have proper indexes
- **Field Selection**: Only include necessary fields
- **Joins**: Minimize complex joins for better performance

## Error Handling

### Common OData Errors

**Invalid filter syntax:**
```json
{
  "error": {
    "code": "BadRequest",
    "message": "Invalid filter expression"
  }
}
```

**Authentication failure:**
```json
{
  "error": {
    "code": "Unauthorized",
    "message": "Authentication required"
  }
}
```

**Inquiry not found:**
```json
{
  "error": {
    "code": "NotFound", 
    "message": "The requested resource was not found"
  }
}
```

## Performance Optimization

### Query Optimization Tips

1. **Use $select**: Only request needed fields
2. **Apply $filter**: Filter data at server level
3. **Limit results**: Use $top for large datasets
4. **Avoid complex joins**: Design inquiries with performance in mind
5. **Use indexes**: Ensure proper database indexing

### Caching Strategies

**Client-side caching:**
```http
GET https://{acumatica-instance}/odatav4/MyCompany/CustomerInquiry
Cache-Control: max-age=3600
```

## Integration Examples

### Power BI Integration

**Power BI OData connection string:**
```
https://mycompany.acumatica.com/odatav4/MyCompany/SalesAnalysisInquiry
```

### Excel Integration

1. Data → Get Data → From Other Sources → From OData Feed
2. Enter OData URL
3. Configure authentication
4. Select desired inquiry

### Custom Applications

**JavaScript example:**
```javascript
const odataUrl = 'https://mycompany.acumatica.com/odatav4/MyCompany/CustomerInquiry';
const query = '?$filter=Status eq \'Active\'&$select=CustomerID,CustomerName&$top=100';

fetch(odataUrl + query, {
  headers: {
    'Authorization': 'Bearer ' + accessToken,
    'Accept': 'application/json'
  }
})
.then(response => response.json())
.then(data => {
  console.log(data.value);
});
```

## Metadata and Schema Discovery

### Service Document

**Get service capabilities:**
```http
GET https://{acumatica-instance}/odatav4/{company}
```

### Metadata Document

**Get full metadata:**
```http
GET https://{acumatica-instance}/odatav4/{company}/$metadata
```

**Response includes:**
- Entity types and properties
- Navigation properties  
- Function and action definitions
- Service capabilities

## Security Considerations

### Access Control

- OData respects Acumatica security roles
- Users can only access data they have permissions for
- Generic Inquiries inherit security from underlying tables

### Best Practices

1. **Use HTTPS**: Always use secure connections
2. **Limit exposure**: Only expose necessary inquiries via OData
3. **Monitor usage**: Track OData endpoint usage
4. **Regular audits**: Review exposed inquiries regularly

This comprehensive guide covers OData API usage for effective Acumatica ERP data access and integration.