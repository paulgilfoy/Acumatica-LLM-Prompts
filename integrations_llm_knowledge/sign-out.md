# Acumatica ERP REST Service: Sign Out

To sign out from the Acumatica ERP contract-based REST service, use the **POST** HTTP method with the specified URL.

## HTTP Method and URL

- **Method:** `POST`
- **URL:** `http://<Acumatica ERP instance URL>/entity/auth/logout`

Replace `<Acumatica ERP instance URL>` with your Acumatica ERP instance's URL.

**Example:**
For a local instance named `AcumaticaDB`, the URL is: `http://localhost/AcumaticaDB/entity/auth/logout`

## Parameters

No parameters are required for sign-out requests.

## Request Headers

No headers are required in the request.

## Response Status Codes

| Code | Description |
|---|---|
| `204` | Request completed successfully. |
| `400` | Invalid data in the request. |
| `429` | Request limit exceeded (license restriction). |
| `500` | Internal server error. |

## Example Request

POST /entity/auth/logout HTTP/1.1
Host: [\<Acumatica ERP instance URL\>]
