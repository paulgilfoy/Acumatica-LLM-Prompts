# Acumatica ERP Batch Operations - LLM Guide

This document provides comprehensive information about Acumatica ERP batch operations for efficient bulk data processing, optimized for Large Language Model consumption.

## Batch Operations Overview

Batch operations in Acumatica ERP allow processing multiple records in a single API request, significantly improving performance and reducing network overhead. This is essential for bulk data imports, exports, and synchronization scenarios.

## Benefits of Batch Operations

| Aspect | Single Operations | Batch Operations |
|--------|------------------|------------------|
| Network Calls | One per record | One per batch |
| Performance | Slow for bulk data | Fast for bulk data |
| Resource Usage | High overhead | Low overhead |
| Transaction Management | Individual | Grouped |
| Error Handling | Per record | Per batch |
| Rate Limiting Impact | High | Low |

## Batch Request Structure

### Basic Batch Request Format

```json
[
  {
    "CustomerID": {"value": "CUST001"},
    "CustomerName": {"value": "Customer One"},
    "CustomerClass": {"value": "RETAIL"}
  },
  {
    "CustomerID": {"value": "CUST002"},
    "CustomerName": {"value": "Customer Two"},
    "CustomerClass": {"value": "WHOLESALE"}
  },
  {
    "CustomerID": {"value": "CUST003"},
    "CustomerName": {"value": "Customer Three"},
    "CustomerClass": {"value": "RETAIL"}
  }
]
```

### HTTP Request Example

```http
POST https://mycompany.acumatica.com/entity/Default/24.200.001/Customer
Content-Type: application/json
Authorization: Bearer {access_token}

[
  {
    "CustomerID": {"value": "BATCH001"},
    "CustomerName": {"value": "Batch Customer 1"},
    "CustomerClass": {"value": "RETAIL"},
    "Status": {"value": "Active"}
  },
  {
    "CustomerID": {"value": "BATCH002"},
    "CustomerName": {"value": "Batch Customer 2"},
    "CustomerClass": {"value": "WHOLESALE"},
    "Status": {"value": "Active"}
  }
]
```

## Batch Response Handling

### Successful Batch Response

```json
[
  {
    "id": "BATCH001",
    "rowNumber": 1,
    "note": "Customer created successfully",
    "CustomerID": {"value": "BATCH001"},
    "CustomerName": {"value": "Batch Customer 1"},
    "Status": {"value": "Active"}
  },
  {
    "id": "BATCH002", 
    "rowNumber": 2,
    "note": "Customer created successfully",
    "CustomerID": {"value": "BATCH002"},
    "CustomerName": {"value": "Batch Customer 2"},
    "Status": {"value": "Active"}
  }
]
```

### Partial Failure Response

```json
[
  {
    "id": "BATCH001",
    "rowNumber": 1,
    "note": "Customer created successfully",
    "CustomerID": {"value": "BATCH001"},
    "Status": {"value": "Active"}
  },
  {
    "rowNumber": 2,
    "error": {
      "message": "Customer class 'INVALID' cannot be found in the system",
      "exceptionType": "ValidationException"
    }
  }
]
```

## Batch Size Considerations

### Optimal Batch Sizes

**Recommended batch sizes by entity type:**

- **Customers/Vendors**: 50-100 records per batch
- **Stock Items**: 25-50 records per batch
- **Sales Orders**: 10-25 records per batch (depending on detail lines)
- **Journal Transactions**: 20-40 records per batch
- **Simple lookups**: 100-200 records per batch

### Size Limitations

**System Limits:**
- Maximum request size: 25MB (configurable)
- Maximum processing time: 10 minutes per batch
- Memory constraints: Varies by server configuration

### Dynamic Batch Sizing

```javascript
function calculateOptimalBatchSize(entityType, recordComplexity) {
  const baseSizes = {
    'Customer': 100,
    'Vendor': 100,
    'StockItem': 50,
    'SalesOrder': 25,
    'JournalTransaction': 40
  };
  
  let batchSize = baseSizes[entityType] || 50;
  
  // Adjust for record complexity
  if (recordComplexity === 'high') {
    batchSize = Math.floor(batchSize * 0.5);
  } else if (recordComplexity === 'low') {
    batchSize = Math.floor(batchSize * 1.5);
  }
  
  return Math.min(batchSize, 200); // Cap at 200
}
```

## Batch Processing Patterns

### Sequential Batch Processing

```javascript
async function processDataInBatches(data, batchSize, processingFunction) {
  const results = [];
  
  for (let i = 0; i < data.length; i += batchSize) {
    const batch = data.slice(i, i + batchSize);
    
    try {
      const batchResult = await processingFunction(batch);
      results.push(...batchResult);
      
      // Optional delay between batches
      await delay(100);
      
    } catch (error) {
      console.error(`Batch ${i / batchSize + 1} failed:`, error);
      
      // Optionally retry individual records
      const individualResults = await processIndividually(batch);
      results.push(...individualResults);
    }
  }
  
  return results;
}
```

### Parallel Batch Processing

```javascript
async function processDataInParallelBatches(data, batchSize, maxConcurrency = 3) {
  const batches = [];
  
  // Split data into batches
  for (let i = 0; i < data.length; i += batchSize) {
    batches.push(data.slice(i, i + batchSize));
  }
  
  // Process batches with concurrency limit
  const results = [];
  for (let i = 0; i < batches.length; i += maxConcurrency) {
    const concurrentBatches = batches.slice(i, i + maxConcurrency);
    
    const batchPromises = concurrentBatches.map(batch => 
      processBatch(batch).catch(error => ({ error, batch }))
    );
    
    const batchResults = await Promise.all(batchPromises);
    results.push(...batchResults);
  }
  
  return results;
}
```

## Error Handling in Batch Operations

### Batch-Level Error Handling

```javascript
async function processBatchWithErrorHandling(batch, retryCount = 3) {
  for (let attempt = 1; attempt <= retryCount; attempt++) {
    try {
      const response = await fetch(apiUrl, {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json',
          'Authorization': `Bearer ${accessToken}`
        },
        body: JSON.stringify(batch)
      });
      
      if (response.ok) {
        return await response.json();
      }
      
      if (response.status === 413) { // Request too large
        // Split batch and retry
        return await processSmallerBatches(batch);
      }
      
      if (attempt === retryCount) {
        throw new Error(`Batch failed after ${retryCount} attempts`);
      }
      
    } catch (error) {
      if (attempt === retryCount) {
        throw error;
      }
      
      // Exponential backoff
      await delay(Math.pow(2, attempt) * 1000);
    }
  }
}
```

### Individual Record Error Recovery

```javascript
async function processWithFallback(batch) {
  try {
    // Try batch processing first
    return await processBatch(batch);
  } catch (error) {
    console.log('Batch failed, falling back to individual processing');
    
    // Fall back to individual record processing
    const results = [];
    for (const record of batch) {
      try {
        const result = await processIndividualRecord(record);
        results.push(result);
      } catch (recordError) {
        results.push({
          record: record,
          error: recordError.message,
          status: 'failed'
        });
      }
    }
    
    return results;
  }
}
```

## Transaction Management

### Batch Transactions

```javascript
class BatchTransactionManager {
  constructor(apiClient) {
    this.apiClient = apiClient;
    this.transactionId = null;
  }
  
  async startTransaction() {
    const response = await this.apiClient.post('/transaction/begin');
    this.transactionId = response.transactionId;
    return this.transactionId;
  }
  
  async processBatch(batch) {
    if (!this.transactionId) {
      throw new Error('Transaction not started');
    }
    
    return await this.apiClient.post('/batch', batch, {
      headers: {
        'X-Transaction-ID': this.transactionId
      }
    });
  }
  
  async commitTransaction() {
    if (!this.transactionId) {
      throw new Error('No active transaction');
    }
    
    const result = await this.apiClient.post('/transaction/commit', {
      transactionId: this.transactionId
    });
    
    this.transactionId = null;
    return result;
  }
  
  async rollbackTransaction() {
    if (!this.transactionId) {
      return;
    }
    
    await this.apiClient.post('/transaction/rollback', {
      transactionId: this.transactionId
    });
    
    this.transactionId = null;
  }
}
```

## Performance Optimization

### Memory-Efficient Batch Processing

```javascript
class StreamingBatchProcessor {
  constructor(batchSize = 50) {
    this.batchSize = batchSize;
    this.currentBatch = [];
    this.results = [];
  }
  
  async processRecord(record) {
    this.currentBatch.push(record);
    
    if (this.currentBatch.length >= this.batchSize) {
      await this.flushBatch();
    }
  }
  
  async flushBatch() {
    if (this.currentBatch.length === 0) {
      return;
    }
    
    try {
      const batchResult = await this.processBatch(this.currentBatch);
      this.results.push(...batchResult);
    } catch (error) {
      console.error('Batch processing failed:', error);
    }
    
    this.currentBatch = [];
  }
  
  async finalize() {
    await this.flushBatch();
    return this.results;
  }
}
```

### Connection Pooling for Batch Operations

```javascript
class BatchApiClient {
  constructor(baseUrl, poolSize = 5) {
    this.baseUrl = baseUrl;
    this.connectionPool = [];
    this.poolSize = poolSize;
    this.activeConnections = 0;
  }
  
  async getConnection() {
    if (this.connectionPool.length > 0) {
      return this.connectionPool.pop();
    }
    
    if (this.activeConnections < this.poolSize) {
      this.activeConnections++;
      return this.createConnection();
    }
    
    // Wait for available connection
    return new Promise((resolve) => {
      const checkForConnection = () => {
        if (this.connectionPool.length > 0) {
          resolve(this.connectionPool.pop());
        } else {
          setTimeout(checkForConnection, 100);
        }
      };
      checkForConnection();
    });
  }
  
  async releaseConnection(connection) {
    this.connectionPool.push(connection);
  }
  
  async processBatch(batch) {
    const connection = await this.getConnection();
    
    try {
      const result = await connection.post('/batch', batch);
      return result;
    } finally {
      await this.releaseConnection(connection);
    }
  }
}
```

## Monitoring and Logging

### Batch Processing Metrics

```javascript
class BatchMetrics {
  constructor() {
    this.metrics = {
      totalBatches: 0,
      successfulBatches: 0,
      failedBatches: 0,
      totalRecords: 0,
      successfulRecords: 0,
      failedRecords: 0,
      averageBatchSize: 0,
      averageProcessingTime: 0,
      processingTimes: []
    };
  }
  
  recordBatchStart(batchSize) {
    this.currentBatchStart = Date.now();
    this.currentBatchSize = batchSize;
    this.metrics.totalBatches++;
    this.metrics.totalRecords += batchSize;
  }
  
  recordBatchSuccess(successCount) {
    const processingTime = Date.now() - this.currentBatchStart;
    this.metrics.processingTimes.push(processingTime);
    this.metrics.successfulBatches++;
    this.metrics.successfulRecords += successCount;
    this.updateAverages();
  }
  
  recordBatchFailure(failureCount) {
    this.metrics.failedBatches++;
    this.metrics.failedRecords += failureCount;
  }
  
  updateAverages() {
    this.metrics.averageBatchSize = 
      this.metrics.totalRecords / this.metrics.totalBatches;
    
    this.metrics.averageProcessingTime = 
      this.metrics.processingTimes.reduce((a, b) => a + b, 0) / 
      this.metrics.processingTimes.length;
  }
  
  getReport() {
    return {
      ...this.metrics,
      successRate: (this.metrics.successfulRecords / this.metrics.totalRecords) * 100,
      batchSuccessRate: (this.metrics.successfulBatches / this.metrics.totalBatches) * 100
    };
  }
}
```

## Advanced Batch Patterns

### Batch Validation Before Processing

```javascript
class BatchValidator {
  constructor(schema) {
    this.schema = schema;
  }
  
  validateBatch(batch) {
    const validRecords = [];
    const invalidRecords = [];
    
    batch.forEach((record, index) => {
      const validation = this.validateRecord(record);
      
      if (validation.isValid) {
        validRecords.push(record);
      } else {
        invalidRecords.push({
          index: index,
          record: record,
          errors: validation.errors
        });
      }
    });
    
    return { validRecords, invalidRecords };
  }
  
  validateRecord(record) {
    const errors = [];
    
    // Check required fields
    this.schema.required?.forEach(field => {
      if (!record[field] || !record[field].value) {
        errors.push(`Required field '${field}' is missing`);
      }
    });
    
    // Check field lengths
    Object.keys(record).forEach(field => {
      const fieldSchema = this.schema.properties[field];
      if (fieldSchema?.maxLength) {
        const value = record[field]?.value;
        if (value && value.length > fieldSchema.maxLength) {
          errors.push(`Field '${field}' exceeds maximum length`);
        }
      }
    });
    
    return {
      isValid: errors.length === 0,
      errors: errors
    };
  }
}
```

### Batch Progress Tracking

```javascript
class BatchProgressTracker {
  constructor(totalRecords, onProgress) {
    this.totalRecords = totalRecords;
    this.processedRecords = 0;
    this.onProgress = onProgress;
    this.startTime = Date.now();
  }
  
  updateProgress(recordsProcessed) {
    this.processedRecords += recordsProcessed;
    const progress = (this.processedRecords / this.totalRecords) * 100;
    const elapsed = Date.now() - this.startTime;
    const estimatedTotal = (elapsed / this.processedRecords) * this.totalRecords;
    const remaining = estimatedTotal - elapsed;
    
    this.onProgress({
      progress: progress,
      processedRecords: this.processedRecords,
      totalRecords: this.totalRecords,
      elapsedTime: elapsed,
      estimatedTimeRemaining: remaining,
      recordsPerSecond: this.processedRecords / (elapsed / 1000)
    });
  }
}
```

## Best Practices

### Batch Operation Guidelines

1. **Size Management**: Use appropriate batch sizes for entity types
2. **Error Recovery**: Implement fallback to individual processing
3. **Progress Tracking**: Provide progress updates for long operations
4. **Memory Management**: Process large datasets in streams
5. **Transaction Control**: Use transactions for related operations
6. **Monitoring**: Track success rates and performance metrics

### Performance Tips

1. **Connection Reuse**: Use connection pooling
2. **Parallel Processing**: Process multiple batches concurrently
3. **Data Validation**: Validate before API calls
4. **Compression**: Use compression for large payloads
5. **Caching**: Cache reference data lookups
6. **Retry Logic**: Implement intelligent retry mechanisms

This comprehensive batch operations guide provides the foundation for efficient bulk data processing with Acumatica ERP APIs.