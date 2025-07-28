# Acumatica ERP API Error Handling - LLM Guide

This document provides comprehensive error handling information for Acumatica ERP API integration, optimized for Large Language Model consumption.

## HTTP Status Codes

### Success Codes
- **200 OK**: Successful GET request, data returned
- **201 Created**: Successful POST request, resource created
- **204 No Content**: Successful PUT/DELETE request, no content returned

### Client Error Codes
- **400 Bad Request**: Invalid request syntax or parameters
- **401 Unauthorized**: Authentication required or invalid credentials
- **403 Forbidden**: Access denied, insufficient permissions
- **404 Not Found**: Resource or endpoint not found
- **405 Method Not Allowed**: HTTP method not supported for endpoint
- **406 Not Acceptable**: Request format not acceptable
- **409 Conflict**: Resource conflict (e.g., duplicate key)
- **413 Request Entity Too Large**: Request payload exceeds size limits
- **422 Unprocessable Entity**: Request valid but contains semantic errors
- **429 Too Many Requests**: Rate limit exceeded

### Server Error Codes
- **500 Internal Server Error**: Unexpected server error
- **502 Bad Gateway**: Invalid response from upstream server
- **503 Service Unavailable**: Server temporarily unavailable
- **504 Gateway Timeout**: Upstream server timeout

## Common Error Response Format

### Standard Error Response Structure
```json
{
  "message": "Error description",
  "exceptionMessage": "Detailed exception information",
  "exceptionType": "System.Exception",
  "stackTrace": "Stack trace details",
  "innerException": {
    "message": "Inner exception details",
    "exceptionType": "InnerExceptionType"
  }
}
```

### Validation Error Response
```json
{
  "message": "Validation failed",
  "modelState": {
    "CustomerID": ["Customer ID is required"],
    "CustomerName": ["Customer Name cannot exceed 60 characters"]
  }
}
```

## Authentication Errors

### Invalid Credentials (401)
```json
{
  "error": "invalid_grant",
  "error_description": "The user name or password is incorrect."
}
```

**Causes:**
- Incorrect username or password
- Account locked or disabled
- Invalid tenant/company name

**Solutions:**
- Verify credentials are correct
- Check account status in Acumatica
- Confirm tenant/company name

### Token Expired (401)
```json
{
  "error": "invalid_token",
  "error_description": "The access token has expired"
}
```

**Solutions:**
- Refresh the access token using refresh token
- Re-authenticate to obtain new tokens
- Implement automatic token refresh

### Insufficient Permissions (403)
```json
{
  "message": "Access denied",
  "exceptionMessage": "User does not have permission to access this resource"
}
```

**Solutions:**
- Check user role assignments in Acumatica
- Verify screen access rights
- Ensure proper field-level permissions

## Validation Errors

### Required Field Missing (400)
```json
{
  "message": "Validation error",
  "exceptionMessage": "'CustomerID' cannot be empty."
}
```

**Common Required Fields:**
- CustomerID, VendorID for business partners
- InventoryID for inventory items
- Type, Date for transactions

### Invalid Field Value (400)
```json
{
  "message": "Invalid field value",
  "exceptionMessage": "Value 'INVALID' is not valid for field 'Status'"
}
```

**Solutions:**
- Check field constraints and valid values
- Verify data types match expected format
- Consult entity schema for valid options

### Field Length Exceeded (400)
```json
{
  "message": "Field length exceeded",
  "exceptionMessage": "CustomerName cannot exceed 60 characters"
}
```

**Solutions:**
- Truncate data to fit field limits
- Check entity schema for field lengths
- Implement client-side validation

## Business Logic Errors

### Duplicate Key (409)
```json
{
  "message": "Duplicate key error",
  "exceptionMessage": "Customer 'DUPLICATE' already exists"
}
```

**Solutions:**
- Check for existing records before creating
- Use unique identifiers
- Implement upsert logic (update if exists, create if not)

### Reference Not Found (400)
```json
{
  "message": "Reference error",
  "exceptionMessage": "Customer 'NONEXISTENT' cannot be found in the system"
}
```

**Solutions:**
- Verify referenced entities exist
- Create dependent records first
- Check for typos in reference IDs

### Document Status Error (400)
```json
{
  "message": "Document status error",
  "exceptionMessage": "Cannot modify released document"
}
```

**Solutions:**
- Check document status before operations
- Use appropriate actions (e.g., Correct for released documents)
- Implement status-based logic

## System Errors

### Request Too Large (413)
```json
{
  "message": "Request entity too large",
  "exceptionMessage": "Request size exceeds maximum allowed"
}
```

**Solutions:**
- Break large requests into smaller batches
- Reduce number of detail lines per request
- Implement pagination for large datasets

### Timeout Error (504)
```json
{
  "message": "Gateway timeout",
  "exceptionMessage": "Request timeout exceeded"
}
```

**Solutions:**
- Reduce request complexity
- Implement retry logic with exponential backoff
- Optimize queries and filters

### Rate Limit Exceeded (429)
```json
{
  "message": "Too many requests",
  "exceptionMessage": "API rate limit exceeded"
}
```

**Solutions:**
- Implement request throttling
- Add delays between requests
- Use bulk operations when possible

## Error Handling Best Practices

### Retry Logic Implementation

```javascript
async function apiCallWithRetry(url, options, maxRetries = 3) {
  for (let attempt = 1; attempt <= maxRetries; attempt++) {
    try {
      const response = await fetch(url, options);
      
      if (response.ok) {
        return await response.json();
      }
      
      // Don't retry client errors (4xx)
      if (response.status >= 400 && response.status < 500) {
        throw new Error(`Client error: ${response.status}`);
      }
      
      // Retry server errors (5xx)
      if (attempt === maxRetries) {
        throw new Error(`Server error after ${maxRetries} attempts`);
      }
      
      // Exponential backoff
      await delay(Math.pow(2, attempt) * 1000);
      
    } catch (error) {
      if (attempt === maxRetries) {
        throw error;
      }
      await delay(Math.pow(2, attempt) * 1000);
    }
  }
}

function delay(ms) {
  return new Promise(resolve => setTimeout(resolve, ms));
}
```

### Comprehensive Error Handler

```javascript
function handleAcumaticaError(error, context) {
  const errorInfo = {
    timestamp: new Date().toISOString(),
    context: context,
    status: error.status,
    message: error.message
  };
  
  switch (error.status) {
    case 400:
      return handleValidationError(error, errorInfo);
    case 401:
      return handleAuthenticationError(error, errorInfo);
    case 403:
      return handleAuthorizationError(error, errorInfo);
    case 404:
      return handleNotFoundError(error, errorInfo);
    case 409:
      return handleConflictError(error, errorInfo);
    case 413:
      return handleRequestTooLargeError(error, errorInfo);
    case 429:
      return handleRateLimitError(error, errorInfo);
    case 500:
    case 502:
    case 503:
    case 504:
      return handleServerError(error, errorInfo);
    default:
      return handleUnknownError(error, errorInfo);
  }
}
```

### Logging and Monitoring

```javascript
class AcumaticaErrorLogger {
  static logError(error, context, additionalData = {}) {
    const logEntry = {
      timestamp: new Date().toISOString(),
      level: 'ERROR',
      context: context,
      httpStatus: error.status,
      message: error.message,
      exceptionType: error.exceptionType,
      stackTrace: error.stackTrace,
      additionalData: additionalData
    };
    
    // Send to logging service
    console.error('Acumatica API Error:', logEntry);
    
    // Send to monitoring service if available
    if (window.errorReporting) {
      window.errorReporting.captureException(error, logEntry);
    }
  }
}
```

## Specific Entity Error Patterns

### Customer Creation Errors

**Missing Customer Class:**
```json
{
  "message": "'Customer Class' cannot be empty.",
  "field": "CustomerClass"
}
```

**Invalid Terms:**
```json
{
  "message": "Terms 'INVALID' cannot be found in the system.",
  "field": "Terms"
}
```

### Sales Order Errors

**Invalid Inventory Item:**
```json
{
  "message": "Inventory ID 'NONEXISTENT' cannot be found in the system.",
  "field": "Details[0].InventoryID"
}
```

**Insufficient Stock:**
```json
{
  "message": "Insufficient quantity available for item 'ITEM001'",
  "field": "Details[0].Qty"
}
```

### Purchase Order Errors

**Invalid Vendor:**
```json
{
  "message": "Vendor 'BADVENDOR' cannot be found in the system.",
  "field": "VendorID"
}
```

**Missing Required Approval:**
```json
{
  "message": "Purchase order requires approval before release",
  "field": "Hold"
}
```

## Debugging Techniques

### Enable Detailed Error Messages

Add to request headers:
```http
X-PX-Detailed-Errors: true
```

### Trace Request/Response

```javascript
function traceApiCall(url, options) {
  console.log('API Request:', {
    url: url,
    method: options.method,
    headers: options.headers,
    body: options.body
  });
  
  return fetch(url, options)
    .then(response => {
      console.log('API Response:', {
        status: response.status,
        statusText: response.statusText,
        headers: Object.fromEntries(response.headers.entries())
      });
      return response;
    });
}
```

### Schema Validation

```javascript
function validateEntityData(entityData, schema) {
  const errors = [];
  
  // Check required fields
  schema.required?.forEach(field => {
    if (!entityData[field] || !entityData[field].value) {
      errors.push(`Required field '${field}' is missing`);
    }
  });
  
  // Check field lengths
  Object.keys(entityData).forEach(field => {
    const fieldSchema = schema.properties[field];
    if (fieldSchema?.maxLength) {
      const value = entityData[field]?.value;
      if (value && value.length > fieldSchema.maxLength) {
        errors.push(`Field '${field}' exceeds maximum length of ${fieldSchema.maxLength}`);
      }
    }
  });
  
  return errors;
}
```

## Error Prevention Strategies

### Data Validation Before API Calls

1. **Client-side validation**: Validate data format and constraints
2. **Reference validation**: Verify referenced entities exist
3. **Business rule validation**: Check business logic constraints
4. **Schema compliance**: Ensure data matches entity schema

### Robust Integration Design

1. **Idempotent operations**: Design operations to be safely retryable
2. **Transactional integrity**: Group related operations appropriately
3. **Graceful degradation**: Handle partial failures gracefully
4. **Circuit breaker pattern**: Prevent cascading failures

### Monitoring and Alerting

1. **Error rate monitoring**: Track API error rates over time
2. **Performance monitoring**: Monitor response times and timeouts
3. **Business metric monitoring**: Track successful transaction rates
4. **Automated alerting**: Set up alerts for critical error patterns

This comprehensive error handling guide provides the foundation for building robust Acumatica ERP API integrations with proper error management and recovery mechanisms.