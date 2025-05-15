# LWC Data Services

Lightning Web Components provide several ways to interact with Salesforce data, primarily through Lightning Data Service (LDS) and Apex. LDS offers a declarative and reactive way to perform CRUD (Create, Read, Update, Delete) operations on single records without writing Apex code.

## Lightning Data Service (LDS) Overview

LDS is built on top of the User Interface API (UI API). It handles data caching, change tracking, and data synchronization across components that use the same record data. This significantly simplifies data management and improves performance.

Key benefits:

- **No Apex Required (for basic CRUD):** Perform common data operations declaratively.
- **Caching:** LDS caches record data client-side, reducing server roundtrips.
- **Reactivity:** Components using LDS automatically re-render when record data changes.
- **FLS and Sharing:** UI API respects Field-Level Security and sharing rules.

## Wire Adapters for LDS

The `@wire` decorator is used to provision data from LDS wire adapters.

- **`getRecord`**:

  - **Description:** Fetches a single record's data.
  - **Usage:** Specify `recordId` and `fields` (or `layoutTypes`/`optionalFields`).
  - **Example (`wireGetRecordStaticContact.js`):**

    ```javascript
    import { LightningElement, api, wire } from "lwc";
    import { getRecord, getFieldValue } from "lightning/uiRecordApi";
    import NAME_FIELD from "@salesforce/schema/Contact.Name";
    // ... other field imports

    const fields = [NAME_FIELD /* ... other fields */];

    export default class WireGetRecordStaticContact extends LightningElement {
      @api recordId;

      @wire(getRecord, { recordId: "$recordId", fields })
      contact; // contact.data or contact.error will be populated

      get name() {
        return getFieldValue(this.contact.data, NAME_FIELD);
      }
      // ... other getters
    }
    ```

    - `$recordId` makes the wire reactive to changes in the `recordId` property.
    - `getFieldValue` is a utility to extract field values from the wired data.

- **`getRecords`**:

  - **Description:** Fetches data for multiple records of the same or different SObject types.
  - **Usage:** Takes an array of record objects, each specifying `recordIds` and `fields` (or `layoutTypes`/`optionalFields`).
  - **Example (`wireGetRecords.js`):**

    ```javascript
    import { LightningElement, wire } from "lwc";
    import { getRecords } from "lightning/uiRecordApi";
    import NAME_FIELD from "@salesforce/schema/Contact.Name";
    // ... other imports ...
    import getContactList from "@salesforce/apex/ContactController.getContactList"; // To get record IDs

    export default class WireGetRecords extends LightningElement {
      records; // Will hold the configuration for getRecords

      @wire(getContactList) // First, get some record IDs (e.g., from Apex)
      wiredContacts({ error, data }) {
        if (data) {
          this.records = [
            {
              recordIds: [data[0].Id, data[1].Id],
              fields: [NAME_FIELD],
              // optionalFields: [EMAIL_FIELD]
            },
          ];
        }
        // ... error handling ...
      }

      // Then, wire getRecords with the reactive 'records' property
      @wire(getRecords, { records: "$records" })
      recordResults; // recordResults.data or recordResults.error

      get recordStr() {
        return this.recordResults
          ? JSON.stringify(this.recordResults.data, null, 2)
          : "";
      }
    }
    ```

    - The `wireGetRecordsDifferentTypes.js` example shows fetching records of different SObject types in a single call.

- **`getListUi`**:

  - **Description:** Fetches records for a specific list view.
  - **Usage:** Specify `objectApiName` and `listViewApiName`. Can also include `sortBy`, `pageSize`, etc.
  - **Example (`wireListView.js`):**

    ```javascript
    import { LightningElement, wire } from "lwc";
    import { getListUi } from "lightning/uiListApi";
    import CONTACT_OBJECT from "@salesforce/schema/Contact";
    import NAME_FIELD from "@salesforce/schema/Contact.Name";

    export default class WireListView extends LightningElement {
      @wire(getListUi, {
        objectApiName: CONTACT_OBJECT,
        listViewApiName: "All_Recipes_Contacts", // Developer name of the list view
        sortBy: NAME_FIELD,
        pageSize: 10,
      })
      listView; // listView.data or listView.error

      get contacts() {
        return this.listView.data.records.records;
      }
    }
    ```

## Imperative Data Operations (UI API Functions)

LDS also provides JavaScript functions for imperative DML operations.

- **`createRecord(recordInput)`**:

  - **Description:** Creates a new record.
  - **Usage:** Pass a `recordInput` object containing `apiName` (SObject API name) and `fields` (an object of field API names to values).
  - **Example (`ldsCreateRecord.js`):**

    ```javascript
    import { LightningElement } from "lwc";
    import { createRecord } from "lightning/uiRecordApi";
    import ACCOUNT_OBJECT from "@salesforce/schema/Account";
    import NAME_FIELD from "@salesforce/schema/Account.Name";
    // ... other imports for toast, error handling

    export default class LdsCreateRecord extends LightningElement {
      accountId;
      name = "";
      // ...
      async createAccount() {
        const fields = {};
        fields[NAME_FIELD.fieldApiName] = this.name;
        const recordInput = { apiName: ACCOUNT_OBJECT.objectApiName, fields };
        try {
          const account = await createRecord(recordInput);
          this.accountId = account.id;
          // ... show success toast
        } catch (error) {
          // ... show error toast
        }
      }
    }
    ```

- **`updateRecord(recordInput)`**:

  - **Description:** Updates an existing record.
  - **Usage:** Pass a `recordInput` object where `fields` must include the `Id` of the record to update.
  - **Example (`datatableInlineEditWithUiApi.js` - simplified for focus):**
    ```javascript
    import { updateRecord } from 'lightning/uiRecordApi';
    // ...
    async handleSave(event) {
        const recordsToUpdate = event.detail.draftValues.map(draft => ({ fields: { Id: draft.Id, ...draft } }));
        try {
            const promises = recordsToUpdate.map(record => updateRecord(record));
            await Promise.all(promises);
            // ... show success toast, refresh data
        } catch (error) {
            // ... show error toast
        }
    }
    ```

- **`deleteRecord(recordId)`**:

  - **Description:** Deletes a record.
  - **Usage:** Pass the `recordId` of the record to delete.
  - **Example (`ldsDeleteRecord.js`):**

    ```javascript
    import { LightningElement } from "lwc";
    import { deleteRecord } from "lightning/uiRecordApi";
    // ... other imports for toast, refreshApex, error handling

    export default class LdsDeleteRecord extends LightningElement {
      // ...
      async deleteAccount(event) {
        const recordId = event.target.dataset.recordid;
        try {
          await deleteRecord(recordId);
          // ... show success toast, refresh data
        } catch (error) {
          // ... show error toast
        }
      }
    }
    ```

## Preparing Record Inputs (`generateRecordInputForCreate`)

- **`getRecordCreateDefaults(config)`**:
  - Wires to get default values for a new record based on object and record type info.
- **`generateRecordInputForCreate(record, objectInfo)`**:

  - Takes the raw record from `getRecordCreateDefaults` and the `objectInfo` to prepare a `recordInput` object suitable for `createRecord`. It filters out non-createable fields and includes default values.
  - **Example (`ldsGenerateRecordInputForCreate.js`):**

    ```javascript
    import {
      getRecordCreateDefaults,
      generateRecordInputForCreate,
      createRecord,
    } from "lightning/uiRecordApi";
    import ACCOUNT_OBJECT from "@salesforce/schema/Account";
    // ...
    export default class LdsGenerateRecordInput extends LightningElement {
      recordInput;
      // ...
      @wire(getRecordCreateDefaults, { objectApiName: ACCOUNT_OBJECT })
      loadDefaults({ data, error }) {
        if (data) {
          this.recordInput = generateRecordInputForCreate(
            data.record,
            data.objectInfos[ACCOUNT_OBJECT.objectApiName]
          );
          // ...
        }
        // ...
      }

      async createAccount() {
        // this.recordInput has been updated by user input via handleFieldChange
        await createRecord(this.recordInput);
        // ...
      }
    }
    ```

## Notifying LDS of External Data Changes (`notifyRecordUpdateAvailable`)

When data is modified outside of LDS (e.g., via an Apex imperative call that updates a record), you need to inform LDS so it can update its cache and refresh wired data.

- **`notifyRecordUpdateAvailable(recordIds)`**:
  - **Description:** Informs LDS that one or more records have been updated.
  - **Usage:** Pass an array of objects, each containing a `recordId`.
  - **Example (`ldsNotifyRecordUpdateAvailable.js`):**
    ```javascript
    import { notifyRecordUpdateAvailable } from "lightning/uiRecordApi";
    import updateContact from "@salesforce/apex/ContactController.updateContact"; // Imperative Apex
    // ...
    export default class LdsNotifyRecordUpdateAvailable extends LightningElement {
      @api recordId;
      // ...
      async handleContactUpdate() {
        try {
          await updateContact({ recordId: this.recordId /* ...fields */ });
          // ... show success
          notifyRecordUpdateAvailable([{ recordId: this.recordId }]); // Notify LDS
        } catch (error) {
          // ... show error
        }
      }
    }
    ```

## Refreshing Wired Data (`refreshApex`)

- If you wire an Apex method and need to refresh its data (e.g., after an imperative DML operation or an external change), use `refreshApex`.
- You must save the provisioned value from the wire to a property to pass to `refreshApex`.

  ```javascript
  // ldsDeleteRecord.js - (Illustrative, also uses notifyRecordUpdateAvailable)
  import { refreshApex } from "@salesforce/apex";
  import getAccountList from "@salesforce/apex/AccountController.getAccountList";
  // ...
  export default class LdsDeleteRecord extends LightningElement {
    wiredAccountsResult; // Property to hold the provisioned value

    @wire(getAccountList)
    wiredAccounts(result) {
      this.wiredAccountsResult = result; // Save the provisioned value
      if (result.data) {
        /* ... */
      }
    }

    async deleteAccount(event) {
      // ... deleteRecord call ...
      await refreshApex(this.wiredAccountsResult); // Refresh the wired Apex data
    }
  }
  ```

## Schema Imports

Import references to SObjects and Fields for type safety and to prevent issues with API name changes.

- **SObject:** `import ACCOUNT_OBJECT from '@salesforce/schema/Account';`
- **Field:** `import NAME_FIELD from '@salesforce/schema/Account.Name';`
- Use `FIELD_NAME.fieldApiName` to get the string API name if needed by an underlying API.

These are the primary ways LWC interacts with Salesforce data directly from the client-side, offering a powerful and efficient approach. For more complex logic or operations not covered by UI API, Apex is used.
