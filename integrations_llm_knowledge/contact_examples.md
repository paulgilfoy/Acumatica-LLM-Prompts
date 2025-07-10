### Create Contact with Attributes

**Purpose:** Integrate Acumatica ERP with an external system via contract-based REST API to **create contacts with attributes**.

### Request Example

This example creates the "Brent Edds" lead via the contract-based REST API.


```http
PUT / HTTP/1.1
Host: [<Acumatica ERP instance URL>]/entity/Default/24.200.001/Contact
Accept: application/json
Content-Type: application/json

{
    "FirstName": {"value": "Brent"},
    "LastName": {"value": "Edds"},
    "Email": {"value": "brent.edds.test27@avantehs.com"},
    "ContactClass": {"value": "ENDCUST"},
    "Attributes": [
        {
            "AttributeID": {"value": "INTEREST"},
            "Value": {"value": "Jam,Maint"}
        }
    ]
}
```

### Usage Notes on Attributes

Attributes of top-level entities are exposed in **AttributeValue** entities.

#### AttributeValue Entity Fields:

* **AttributeID:**
    * **Set:** Both internal and external values accepted.
    * **Retrieve:** Only internal value returned.
* **AttributeDescription:** (Read-only) External value of the attribute identifier.
* **Value:**
    * **Retrieve:** Internal value returned.
    * **Set:** Internal value and external value (for most control types) can be used.
        * **Multiselect Combo Boxes:** External value accepted only if it contains a single value without commas.
        * **Check Boxes:**
            * **Returned:** `0` (not selected), `1` (selected).
            * **Set:** `0`, `1`, `false` (case-insensitive), `true` (case-insensitive).
        * **Multiselect Combo Boxes (Internal Value):** Values separated by commas (e.g., `Value1,Value2,Value3`).
        * **Selectors:** Set and retrieved like text boxes.
        * **Date Edit Boxes:** Internal values must be parsable using `System.Globalization.CultureInfo.InvariantCulture`.
        * **Error Handling:** For combo boxes, date edit boxes, and check boxes, an unsupported value attempt writes an error to the `error` field.
* **ValueDescription:** (Read-only) External value of the attribute, based on control type:
    * **Text boxes, Date edit boxes:** Same as internal value.
    * **Check boxes:** `True` or `False`.
    * **Combo boxes:** Label used.
    * **Multiselect Combo Boxes:** Labels separated by commas and spaces (e.g., `Label1, Label2, Label3`).
* **Required:** (Read-only) Indicates if the attribute is mandatory.
* **RefNoteID:** (Read-only) Value of the `NoteID` field of the referred object.

---

### Deactivate a Contact
When integrating Acumatica ERP with an external system via the **contract-based REST API**, contacts can be **activated or deactivated** by setting the `Active` field of a `Contact` object to `true` or `false`.

### Request Example
The following request deactivates the "Brent Edds" contact. (For contact creation, refer to "Create a Contact with Attributes".)

```
PUT $filter=FirstName%20eq%20'Brent'%20and%20LastName%20eq%20'Edds' HTTP/1.1
Host: [<Acumatica ERP instance URL>]/entity/Default/24.200.001/Contacts
Accept: application/json
Content-Type: application/json

{
    "Active": {"value": false}
}
```

---
### Retrieve the List of Contacts

When integrating Acumatica ERP with an external system via the contract-based REST API, you can retrieve a list of available contacts and their data from Acumatica ERP.

## Request Example

To retrieve detailed information for contact ID `100073` using the contract-based REST API, use the following request:

```http
GET ?$expand=Activities,Address,Campaigns,Cases,Notifications,Opportunities&
    $filter=ContactID%20eq%20100073 HTTP/1.1
Host: [<Acumatica ERP instance URL>]/entity/Default/24.200.001/Contact
Accept: application/json
Content-Type: application/json
```

---

### Linking Multiple Contacts to a Customer via Acumatica ERP REST API

When integrating an external system with Acumatica ERP using the contract-based REST API, it's possible to associate multiple contacts with a single customer.

### Request Example

To link an additional contact (e.g., ContactID **100196**) to an existing business account (e.g., **ABAKERY**), use a **PUT** request similar to the following:

```
PUT / HTTP/1.1
Host: [<Acumatica ERP instance URL>]/entity/Default/24.200.001/Contact
Accept: application/json
Content-Type: application/json

{
    "BusinessAccount": {"value": "ABAKERY"},
    "ContactID": {"value": 100196},
    "LastName": {"value": "Doe"}
}
```

After this request, the **ABAKERY** customer will be linked to the specified contact, in addition to any previously associated contacts.