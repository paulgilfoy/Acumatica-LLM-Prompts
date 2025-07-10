## Acumatica ERP REST Service: Sign-In

To interact with the Acumatica ERP contract-based REST service, an application must first sign in. This involves a `POST` request to a specific URL with credentials in the request body.

---

### Two-Factor Authentication (2FA) Bypass

If 2FA is enabled for a user account used for REST API access, the sign-in method bypasses it. To explicitly disable 2FA for such accounts, define a specific user type. Refer to "To Limit the Number of API Connections of Integrated Applications" for details.

---

### Alternative Authorization

Instead of direct sign-in, applications can use **OAuth 2.0** or **OpenID Connect (OIDC)** for authorization. See "Authorizing Client Applications to Work with Acumatica ERP" for more.

---

### Sign-In Endpoint

* **HTTP Method:** `POST`
* **URL:** `http://<Acumatica ERP instance URL>/entity/auth/login`
    * Replace `<Acumatica ERP instance URL>` with your specific Acumatica ERP instance URL (e.g., `http://localhost/AcumaticaDB`).

---

### Request Details

#### Parameters

* None required.

#### Request Headers

| Header       | Description                                                 | Content-Type Value(s)       |
| :----------- | :---------------------------------------------------------- | :-------------------------- |
| `Content-Type` | Format of the request body. | `application/json`, `application/x-www-form-urlencoded` |

#### Request Body (JSON Example)

```json
{
  "name": "admin",
  "password": "123",
  "tenant": "MyStore",
  "branch": "MYSTORE",
  "locale": "EN-US"
}
```

* **`name`**: Username (e.g., `"admin"`).
* **`password`**: Password for the username (e.g., `"123"`).
* **`tenant`**: Name of the tenant (e.g., `"MyStore"`, found in `Tenants` SM203520 form, `Login Name` box).
* **`branch`**: ID of the branch (e.g., `"MYSTORE"`, found in `Branches` CS102000 form, `Branch ID` box).
* **`locale`**: Locale in `System.Globalization.CultureInfo` format (e.g., `"EN-US"`). This parameter is for future use; setting its value is not currently required.

---

### Response Status Codes

| Code | Description                                                               |
| :--- | :------------------------------------------------------------------------ |
| `204`  | Success. Response headers contain authorization cookies for subsequent requests. |
| `400`  | Invalid data in the request.                                              |
| `429`  | Request limit exceeded due to license restrictions.                       |
| `500`  | Internal server error.                                                    |

---

### Sign-In Request Example

```http
POST /entity/auth/login HTTP/1.1
Host: [<Acumatica ERP instance URL>]
Accept: application/json
Content-Type: application/json

{
  "name": "admin",
  "password": "123",
  "tenant": "MyStore",
  "branch": "MYSTORE"
}
```