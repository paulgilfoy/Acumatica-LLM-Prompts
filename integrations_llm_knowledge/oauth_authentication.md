# Acumatica ERP OAuth 2.0 Authentication - LLM Guide

This document provides comprehensive OAuth 2.0 authentication information for Acumatica ERP API integration, optimized for Large Language Model consumption.

## OAuth 2.0 Overview

Acumatica ERP supports OAuth 2.0 authentication standard for secure API access. This is the recommended authentication method over basic authentication for production environments.

## Supported OAuth Flows

- **Authorization Code Flow**: Primary flow for web applications
- **Resource Owner Password Credentials**: For trusted applications
- **Client Credentials**: For server-to-server communication

## Setting Up OAuth Application

### Step 1: Register OAuth Application in Acumatica

1. Navigate to **System** → **Integration** → **Connected Applications** (SM303010)
2. Click **Add Row** to create new application
3. Configure the following fields:
   - **Client Name**: Your application name
   - **OAuth 2.0 Flow**: Select "Authorization Code"
   - **Plugin**: Leave as "No Plug-In"

### Step 2: Generate Client Secret

1. In the **Secrets** tab, click **Add Shared Secret**
2. Fill in:
   - **Description**: Custom description for the secret
   - **Expires On (UTC)**: Optional expiration date
   - **Value**: Auto-generated secret key (copy and save this)
3. Click **OK** to save

### Step 3: Configure Redirect URIs

1. In the **Redirect URIs** tab, click **Add Row**
2. Enter redirect URI format: `https://yourapp.com/callback`
3. URI must be absolute and not contain fragment (#) parts
4. Save the application to generate **Client ID**

## OAuth Endpoints

### Authorization Endpoint
```
GET https://{acumatica-instance}/identity/connect/authorize
```

### Token Endpoint
```
POST https://{acumatica-instance}/identity/connect/token
```

### Userinfo Endpoint
```
GET https://{acumatica-instance}/identity/connect/userinfo
```

## Authorization Code Flow Implementation

### Step 1: Authorization Request
```http
GET https://{acumatica-instance}/identity/connect/authorize?
    response_type=code&
    client_id={client_id}&
    redirect_uri={redirect_uri}&
    scope=api offline_access&
    state={state}
```

**Parameters:**
- `response_type`: Must be "code"
- `client_id`: Your application's client ID
- `redirect_uri`: Must match registered redirect URI
- `scope`: "api" for API access, "offline_access" for refresh tokens
- `state`: Random string for security (recommended)

### Step 2: Exchange Code for Token
```http
POST https://{acumatica-instance}/identity/connect/token
Content-Type: application/x-www-form-urlencoded

grant_type=authorization_code&
code={authorization_code}&
redirect_uri={redirect_uri}&
client_id={client_id}&
client_secret={client_secret}
```

**Response:**
```json
{
  "access_token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIs...",
  "expires_in": 3600,
  "token_type": "Bearer",
  "refresh_token": "CfDJ8M5t...",
  "scope": "api offline_access"
}
```

## Using Access Tokens

### API Request with Bearer Token
```http
GET https://{acumatica-instance}/entity/Default/24.200.001/Customer
Authorization: Bearer {access_token}
Content-Type: application/json
```

### Token Refresh
```http
POST https://{acumatica-instance}/identity/connect/token
Content-Type: application/x-www-form-urlencoded

grant_type=refresh_token&
refresh_token={refresh_token}&
client_id={client_id}&
client_secret={client_secret}
```

## Connection Properties for OAuth

When using OAuth with integration tools:

- **OAuthClientId**: Your client ID from Acumatica
- **OAuthClientSecret**: Your client secret from Acumatica
- **InitiateOAuth**: Set to "GETANDREFRESH" for automatic token management
- **CallbackURL**: Your registered redirect URI
- **OAuthGrantType**: "CODE" for authorization code flow

## Security Best Practices

1. **Store secrets securely**: Never expose client secrets in client-side code
2. **Use HTTPS**: All OAuth flows must use secure connections
3. **Validate state parameter**: Prevent CSRF attacks
4. **Token storage**: Store tokens securely, preferably encrypted
5. **Token expiration**: Implement proper token refresh mechanisms
6. **Scope limitation**: Request only necessary scopes

## Common OAuth Errors

### Invalid Client
```json
{
  "error": "invalid_client",
  "error_description": "Client authentication failed"
}
```
**Solution**: Verify client ID and secret are correct

### Invalid Grant
```json
{
  "error": "invalid_grant",
  "error_description": "The provided authorization grant is invalid"
}
```
**Solution**: Authorization code may be expired or already used

### Invalid Redirect URI
```json
{
  "error": "invalid_request",
  "error_description": "Invalid redirect_uri"
}
```
**Solution**: Ensure redirect URI exactly matches registered URI

## OAuth vs Basic Authentication

| Feature | OAuth 2.0 | Basic Auth |
|---------|-----------|------------|
| Security | High | Medium |
| Token Expiration | Yes | No |
| Refresh Capability | Yes | No |
| Recommended for Production | Yes | No |
| Setup Complexity | Higher | Lower |

## Example Implementation (JavaScript)

```javascript
// Step 1: Redirect to authorization endpoint
const authUrl = `https://myacumatica.com/identity/connect/authorize?` +
  `response_type=code&` +
  `client_id=${clientId}&` +
  `redirect_uri=${encodeURIComponent(redirectUri)}&` +
  `scope=api offline_access&` +
  `state=${state}`;

window.location.href = authUrl;

// Step 2: Exchange code for token (server-side)
const tokenResponse = await fetch('https://myacumatica.com/identity/connect/token', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/x-www-form-urlencoded'
  },
  body: new URLSearchParams({
    grant_type: 'authorization_code',
    code: authorizationCode,
    redirect_uri: redirectUri,
    client_id: clientId,
    client_secret: clientSecret
  })
});

const tokens = await tokenResponse.json();

// Step 3: Use access token for API calls
const apiResponse = await fetch('https://myacumatica.com/entity/Default/24.200.001/Customer', {
  headers: {
    'Authorization': `Bearer ${tokens.access_token}`,
    'Content-Type': 'application/json'
  }
});
```

This OAuth 2.0 implementation provides secure, scalable authentication for Acumatica ERP API integrations.