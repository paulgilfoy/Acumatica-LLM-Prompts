# Acumatica ERP Webhooks - LLM Guide

This document provides comprehensive information about Acumatica ERP webhooks for real-time integration, optimized for Large Language Model consumption.

## Webhooks Overview

Acumatica ERP supports webhooks for real-time notifications when data changes occur in the system. Webhooks enable bidirectional synchronization and event-driven integrations by automatically sending HTTP POST requests to external endpoints when specific events occur.

## Webhook vs Polling Comparison

| Feature | Webhooks | Polling |
|---------|----------|---------|
| Real-time Updates | Yes | Delayed |
| Resource Efficiency | High | Low |
| Network Traffic | Minimal | High |
| Implementation Complexity | Medium | Low |
| Reliability | Requires retry logic | More predictable |
| Server Load | Low | High |

## Webhook Configuration

### Setting Up Webhooks in Acumatica

1. Navigate to **System** → **Integration** → **Generic Inquiries** (SM208000)
2. Create or modify a Generic Inquiry
3. In the **Parameters** tab, configure webhook settings:
   - **Enable Webhooks**: Check to enable
   - **Webhook URL**: Target endpoint URL
   - **Authentication**: Configure authentication method
   - **Events**: Select trigger events

### Webhook URL Format

**Target Endpoint URL:**
```
https://your-application.com/webhooks/acumatica
```

**Requirements:**
- Must be publicly accessible HTTPS endpoint
- Should respond with HTTP 200 status for successful receipt
- Must handle POST requests with JSON payload

## Webhook Events

### Available Event Types

- **Insert**: Triggered when new records are created
- **Update**: Triggered when existing records are modified
- **Delete**: Triggered when records are deleted
- **Status Change**: Triggered when document status changes
- **Custom Events**: Triggered by custom business logic

### Entity-Specific Events

**Customer Events:**
- Customer.Inserted
- Customer.Updated
- Customer.StatusChanged

**Sales Order Events:**
- SalesOrder.Inserted
- SalesOrder.Updated
- SalesOrder.Released
- SalesOrder.Cancelled

**Inventory Events:**
- StockItem.Inserted
- StockItem.Updated
- StockItem.Deleted

## Webhook Payload Structure

### Standard Webhook Payload

```json
{
  "eventType": "Customer.Updated",
  "timestamp": "2024-01-15T10:30:00Z",
  "entityType": "Customer",
  "entityId": "CUST001",
  "tenantId": "MyCompany",
  "userId": "admin",
  "data": {
    "CustomerID": {"value": "CUST001"},
    "CustomerName": {"value": "Updated Customer Name"},
    "Status": {"value": "Active"}
  },
  "changes": [
    {
      "field": "CustomerName",
      "oldValue": "Old Customer Name",
      "newValue": "Updated Customer Name"
    }
  ],
  "metadata": {
    "source": "WebUI",
    "correlationId": "12345-67890-abcdef"
  }
}
```

### Webhook Headers

```http
POST /webhooks/acumatica HTTP/1.1
Host: your-application.com
Content-Type: application/json
X-Acumatica-Event: Customer.Updated
X-Acumatica-Signature: sha256=abc123...
X-Acumatica-Delivery: 12345-67890-abcdef
User-Agent: Acumatica-Webhook/1.0
```

## Webhook Security

### Signature Verification

Acumatica signs webhook payloads using HMAC-SHA256:

```javascript
function verifyWebhookSignature(payload, signature, secret) {
  const crypto = require('crypto');
  const expectedSignature = crypto
    .createHmac('sha256', secret)
    .update(payload, 'utf8')
    .digest('hex');
  
  const actualSignature = signature.replace('sha256=', '');
  
  return crypto.timingSafeEqual(
    Buffer.from(expectedSignature, 'hex'),
    Buffer.from(actualSignature, 'hex')
  );
}
```

### IP Whitelisting

Configure firewall rules to only accept webhooks from Acumatica server IPs:

```javascript
const allowedIPs = [
  '192.168.1.100',  // Acumatica server IP
  '10.0.0.50'       // Backup server IP
];

function isAllowedIP(clientIP) {
  return allowedIPs.includes(clientIP);
}
```

### Authentication Methods

**API Key Authentication:**
```http
Authorization: Bearer your-api-key
```

**Basic Authentication:**
```http
Authorization: Basic base64(username:password)
```

**Custom Headers:**
```http
X-API-Key: your-api-key
X-Client-ID: your-client-id
```

## Webhook Implementation

### Node.js Express Example

```javascript
const express = require('express');
const crypto = require('crypto');
const app = express();

app.use(express.json());

// Webhook endpoint
app.post('/webhooks/acumatica', (req, res) => {
  try {
    // Verify signature
    const signature = req.headers['x-acumatica-signature'];
    const payload = JSON.stringify(req.body);
    
    if (!verifyWebhookSignature(payload, signature, process.env.WEBHOOK_SECRET)) {
      return res.status(401).json({ error: 'Invalid signature' });
    }
    
    // Process webhook
    processWebhook(req.body);
    
    // Respond with success
    res.status(200).json({ received: true });
    
  } catch (error) {
    console.error('Webhook processing error:', error);
    res.status(500).json({ error: 'Processing failed' });
  }
});

function processWebhook(webhookData) {
  const { eventType, entityType, entityId, data } = webhookData;
  
  switch (eventType) {
    case 'Customer.Updated':
      handleCustomerUpdate(entityId, data);
      break;
    case 'SalesOrder.Released':
      handleSalesOrderRelease(entityId, data);
      break;
    default:
      console.log(`Unhandled event type: ${eventType}`);
  }
}

app.listen(3000, () => {
  console.log('Webhook server listening on port 3000');
});
```

### Python Flask Example

```python
from flask import Flask, request, jsonify
import hmac
import hashlib
import json

app = Flask(__name__)

@app.route('/webhooks/acumatica', methods=['POST'])
def handle_webhook():
    try:
        # Verify signature
        signature = request.headers.get('X-Acumatica-Signature')
        payload = request.get_data()
        
        if not verify_signature(payload, signature):
            return jsonify({'error': 'Invalid signature'}), 401
        
        # Process webhook
        webhook_data = request.json
        process_webhook(webhook_data)
        
        return jsonify({'received': True}), 200
        
    except Exception as e:
        print(f'Webhook processing error: {e}')
        return jsonify({'error': 'Processing failed'}), 500

def verify_signature(payload, signature):
    secret = os.environ.get('WEBHOOK_SECRET')
    expected_signature = hmac.new(
        secret.encode('utf-8'),
        payload,
        hashlib.sha256
    ).hexdigest()
    
    actual_signature = signature.replace('sha256=', '')
    return hmac.compare_digest(expected_signature, actual_signature)

def process_webhook(webhook_data):
    event_type = webhook_data.get('eventType')
    entity_id = webhook_data.get('entityId')
    data = webhook_data.get('data')
    
    if event_type == 'Customer.Updated':
        handle_customer_update(entity_id, data)
    elif event_type == 'SalesOrder.Released':
        handle_sales_order_release(entity_id, data)

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=3000)
```

## Webhook Reliability

### Retry Logic

Acumatica implements automatic retry logic for failed webhook deliveries:

- **Initial Retry**: Immediate retry after failure
- **Exponential Backoff**: Delays increase exponentially (1s, 2s, 4s, 8s, 16s)
- **Maximum Retries**: Typically 5 attempts
- **Timeout**: 30-second timeout per attempt

### Idempotency Handling

```javascript
const processedWebhooks = new Set();

function processWebhook(webhookData) {
  const deliveryId = webhookData.metadata?.correlationId;
  
  // Check for duplicate delivery
  if (processedWebhooks.has(deliveryId)) {
    console.log(`Duplicate webhook ignored: ${deliveryId}`);
    return;
  }
  
  try {
    // Process webhook logic here
    handleWebhookEvent(webhookData);
    
    // Mark as processed
    processedWebhooks.add(deliveryId);
    
  } catch (error) {
    // Don't mark as processed on error
    throw error;
  }
}
```

### Dead Letter Queue

Implement dead letter queue for failed webhooks:

```javascript
const Queue = require('bull');
const webhookQueue = new Queue('webhook processing');
const deadLetterQueue = new Queue('failed webhooks');

webhookQueue.process(async (job) => {
  const webhookData = job.data;
  
  try {
    await processWebhookEvent(webhookData);
  } catch (error) {
    // Move to dead letter queue after max retries
    if (job.attemptsMade >= 3) {
      await deadLetterQueue.add('failed-webhook', {
        originalData: webhookData,
        error: error.message,
        attempts: job.attemptsMade
      });
    }
    throw error;
  }
});
```

## Advanced Webhook Patterns

### Webhook Filtering

Filter webhooks based on criteria:

```javascript
function shouldProcessWebhook(webhookData) {
  const { eventType, data } = webhookData;
  
  // Only process active customers
  if (eventType.startsWith('Customer.') && data.Status?.value !== 'Active') {
    return false;
  }
  
  // Only process orders above threshold
  if (eventType.startsWith('SalesOrder.') && 
      parseFloat(data.OrderTotal?.value || 0) < 1000) {
    return false;
  }
  
  return true;
}
```

### Webhook Aggregation

Aggregate multiple events before processing:

```javascript
const eventBuffer = new Map();
const BUFFER_TIMEOUT = 5000; // 5 seconds

function bufferWebhook(webhookData) {
  const key = `${webhookData.entityType}-${webhookData.entityId}`;
  
  // Cancel existing timeout
  if (eventBuffer.has(key)) {
    clearTimeout(eventBuffer.get(key).timeout);
  }
  
  // Buffer the event
  eventBuffer.set(key, {
    data: webhookData,
    timeout: setTimeout(() => {
      processBufferedEvent(key);
    }, BUFFER_TIMEOUT)
  });
}

function processBufferedEvent(key) {
  const bufferedEvent = eventBuffer.get(key);
  if (bufferedEvent) {
    processWebhookEvent(bufferedEvent.data);
    eventBuffer.delete(key);
  }
}
```

### Webhook Transformation

Transform webhook data for downstream systems:

```javascript
function transformWebhookData(webhookData) {
  const { eventType, data } = webhookData;
  
  // Transform for CRM system
  if (eventType.startsWith('Customer.')) {
    return {
      id: data.CustomerID?.value,
      name: data.CustomerName?.value,
      status: data.Status?.value,
      lastModified: webhookData.timestamp
    };
  }
  
  // Transform for inventory system
  if (eventType.startsWith('StockItem.')) {
    return {
      sku: data.InventoryID?.value,
      description: data.Description?.value,
      quantity: data.QtyOnHand?.value
    };
  }
  
  return data;
}
```

## Monitoring and Debugging

### Webhook Logging

```javascript
class WebhookLogger {
  static logWebhook(webhookData, status, processingTime) {
    const logEntry = {
      timestamp: new Date().toISOString(),
      eventType: webhookData.eventType,
      entityId: webhookData.entityId,
      status: status,
      processingTime: processingTime,
      deliveryId: webhookData.metadata?.correlationId
    };
    
    console.log('Webhook processed:', logEntry);
    
    // Send to monitoring service
    if (global.monitoring) {
      global.monitoring.recordWebhook(logEntry);
    }
  }
}
```

### Health Check Endpoint

```javascript
app.get('/webhooks/health', (req, res) => {
  const health = {
    status: 'healthy',
    timestamp: new Date().toISOString(),
    webhooksProcessed: getWebhookCount(),
    lastWebhook: getLastWebhookTime(),
    errors: getErrorCount()
  };
  
  res.json(health);
});
```

### Webhook Testing

```javascript
// Test webhook endpoint
app.post('/webhooks/test', (req, res) => {
  const testWebhook = {
    eventType: 'Test.Event',
    timestamp: new Date().toISOString(),
    entityType: 'Test',
    entityId: 'TEST001',
    data: { message: 'Test webhook' }
  };
  
  processWebhook(testWebhook);
  res.json({ message: 'Test webhook processed' });
});
```

## Best Practices

### Performance Optimization

1. **Asynchronous Processing**: Process webhooks asynchronously
2. **Database Connections**: Use connection pooling
3. **Caching**: Cache frequently accessed data
4. **Batch Operations**: Group related operations

### Error Handling

1. **Graceful Degradation**: Handle partial failures
2. **Circuit Breaker**: Prevent cascade failures
3. **Monitoring**: Track webhook success rates
4. **Alerting**: Set up alerts for critical failures

### Security

1. **HTTPS Only**: Always use secure connections
2. **Signature Verification**: Verify all webhook signatures
3. **IP Whitelisting**: Restrict access to known IPs
4. **Rate Limiting**: Implement rate limiting

This comprehensive webhooks guide provides the foundation for implementing real-time, event-driven integrations with Acumatica ERP.