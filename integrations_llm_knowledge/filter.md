***
# Acumatica REST API: Filtering Records

When retrieving records from Acumatica ERP via the contract-based REST API, the **$filter** parameter is used to specify conditions for record selection. This parameter adheres to **OData URI conventions**.

## Filter Capabilities

* **Multiple Conditions:** Combine conditions for the same or different fields using **AND** and **OR** operators.
* **Supported Functions (OData):**
    * `substringof`
    * `startswith`
    * `endswith`
* **Custom Fields:** Use the custom function `cf.<Type name>(f='<View name>.<Field name>')` for filtering.
    * `<Type name>`: Type of the custom element.
    * `<View name>`: Data view name containing the element.
    * `<Field name>`: Element name.

---

## Examples

### Simple Condition

**Objective:** Get stock items with `Active` status.
**Filter:** `$filter=ItemStatus eq ’Active’`

### Condition on a Linked Entity

**Objective:** Get customer with email `demo@gmail.com`.
**Filter:** `$filter=MainContact/Email eq ’demo@gmail.com’`
*(Here, `Email` is in the `MainContact` linked entity.)*

### Multiple Conditions

**Objective:** Get active stock items modified after July 15, 2024.
**Filter:** `$filter=ItemStatus eq ’Active’ and LastModified gt datetimeoffset’2024-07-15T10%3A31%3A28.402%2B03%3A00’`

**Note on Date/Time:** Encode date and time values in URL format.
*Example:* `WebUtility.UrlEncode(new DateTimeOffset(DateTime.Now).ToString("yyyy-MM-ddTHH:mm:ss.fffK"))`

### Condition on a Custom Field

**Scenario:** An `RepairItemType` custom field (corresponding to `ItemSettings.UsrRepairItemType`) added to the `StockItem` entity.
**Objective:** Get stock items where `Repair Item Type` is `Battery`.
**Filter:** `$filter=cf.String(f=’ItemSettings.UsrRepairItemType’) eq ’Battery’`

*For custom element and data view names, refer to "Custom Fields" documentation.*

---

## Important Limitation

* **Detail Records:** The REST API **does not support filtering on detail records**. Filtering on them will lead to unpredictable results.