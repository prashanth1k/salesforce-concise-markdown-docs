# LWC Base Components Guide

Lightning Web Components provide a comprehensive suite of pre-built UI elements known as Base Lightning Components. These components accelerate development by offering ready-to-use, SLDS-styled (Salesforce Lightning Design System) building blocks for common user interface patterns.

This guide details several key base components. For a complete list and detailed documentation, refer to the [Component Library](https://developer.salesforce.com/docs/component-library/overview/components).

## Form Elements

These components are essential for building forms to create, view, or edit data.

- **`lightning-input`**:

  - **Description:** Renders various input types like text, number, email, password, date, checkbox, toggle, search, etc.
  - **Key Attributes:** `label`, `value`, `type`, `onchange`, `checked` (for checkbox/toggle), `min`, `max` (for number/date).
  - **Example (`helloBinding.html`):**
    ```html
    <lightning-input
      label="Name"
      value="{greeting}"
      onchange="{handleChange}"
    ></lightning-input>
    ```

- **`lightning-combobox`**:

  - **Description:** A dropdown list for single selection from a list of options.
  - **Key Attributes:** `label`, `value`, `options`, `onchange`, `placeholder`.
  - **Example (`miscSharedJavaScript.html`):**
    ```html
    <lightning-combobox
      label="Term"
      value="{term}"
      onchange="{termChange}"
      options="{termOptions}"
    >
    </lightning-combobox>
    ```

- **`lightning-record-form`**:

  - **Description:** Displays and/or edits fields of a Salesforce record. Automatically handles field layout, FLS, and data type formatting.
  - **Key Attributes:** `record-id`, `object-api-name`, `fields` (array of field API names), `columns`, `mode` (`view`, `edit`, `readonly`).
  - **Example (`recordFormDynamicContact.html`):**
    ```html
    <lightning-record-form
      object-api-name="{objectApiName}"
      record-id="{recordId}"
      fields="{fields}"
    ></lightning-record-form>
    ```

- **`lightning-record-edit-form`**:

  - **Description:** Provides a customizable form to create or edit a Salesforce record. Requires `lightning-input-field` for displaying fields and `lightning-button` for submission. Includes `lightning-messages` for error display.
  - **Key Attributes:** `record-id`, `object-api-name`, `onsuccess`, `onerror`, `onsubmit`.
  - **Example (`recordEditFormDynamicContact.html`):**
    ```html
    <lightning-record-edit-form
      object-api-name="{objectApiName}"
      record-id="{recordId}"
    >
      <lightning-messages></lightning-messages>
      <lightning-input-field field-name="AccountId"></lightning-input-field>
      <lightning-input-field field-name="Name"></lightning-input-field>
      <!-- ... more fields ... -->
      <lightning-button
        variant="brand"
        type="submit"
        label="Save"
      ></lightning-button>
    </lightning-record-edit-form>
    ```

- **`lightning-record-view-form`**:

  - **Description:** Provides a customizable read-only display of a Salesforce record. Requires `lightning-output-field` for displaying fields.
  - **Key Attributes:** `record-id`, `object-api-name`.
  - **Example (`recordViewFormDynamicContact.html`):**
    ```html
    <lightning-record-view-form
      object-api-name="{objectApiName}"
      record-id="{recordId}"
    >
      <lightning-output-field field-name="AccountId"></lightning-output-field>
      <lightning-output-field field-name="Name"></lightning-output-field>
      <!-- ... more fields ... -->
    </lightning-record-view-form>
    ```

- **`lightning-input-field`**:

  - **Description:** Used within `lightning-record-edit-form` to display an editable field for a record.
  - **Key Attributes:** `field-name`.

- **`lightning-output-field`**:
  - **Description:** Used within `lightning-record-view-form` to display a read-only field of a record.
  - **Key Attributes:** `field-name`.

## Layout and Structure

Components for organizing UI elements.

- **`lightning-card`**:

  - **Description:** A container to group related information, with an optional title, icon, and footer.
  - **Key Attributes:** `title`, `icon-name`. Can contain slots like `footer` and `actions`.
  - **Example (`hello.html`):**
    ```html
    <lightning-card title="Hello" icon-name="custom:custom14">
      <!-- Card content -->
      <c-view-source source="lwc/hello" slot="footer"></c-view-source>
    </lightning-card>
    ```

- **`lightning-layout`**:

  - **Description:** A flexible grid system for arranging components horizontally or vertically. Used with `lightning-layout-item`.
  - **Key Attributes:** `vertical-align`, `horizontal-align`, `pull-to-boundary`.
  - **Example (`contactTile.html`):**
    ```html
    <lightning-layout vertical-align="center">
      <lightning-layout-item>...</lightning-layout-item>
      <lightning-layout-item padding="around-small">...</lightning-layout-item>
    </lightning-layout>
    ```

- **`lightning-layout-item`**:
  - **Description:** Represents a section within a `lightning-layout`.
  - **Key Attributes:** `padding`, `size`, `small-device-size`, `medium-device-size`, `large-device-size`, `flexibility`.

## Data Display

Components for presenting data in various formats.

- **`lightning-datatable`**:

  - **Description:** Displays tabular data with features like column resizing, sorting, row selection, inline editing, and custom data types.
  - **Key Attributes:** `key-field`, `data`, `columns`, `onsave` (for inline edit), `hide-checkbox-column`.
  - **Example (`datatableInlineEditWithApex.html`):**
    ```html
    <lightning-datatable
      key-field="Id"
      data="{contacts.data}"
      columns="{columns}"
      onsave="{handleSave}"
      draft-values="{draftValues}"
      hide-checkbox-column
    >
    </lightning-datatable>
    ```

- **`lightning-formatted-text`**: Displays formatted text content.
- **`lightning-formatted-number`**: Displays numbers formatted according to locale, including currency and percentages.
  - **Example (`miscSharedJavaScript.html`):**
    ```html
    <lightning-formatted-number
      format-style="currency"
      currency-code="USD"
      value="{monthlyPayment}"
    ></lightning-formatted-number>
    ```
- **`lightning-formatted-email`**: Displays an email address as a `mailto:` link.
- **`lightning-formatted-phone`**: Displays a phone number as a `tel:` link.
- **`lightning-formatted-date-time`**: Displays date and/or time formatted according to locale.
  - **Example (`clock.html`):**
    ```html
    <lightning-formatted-date-time
      value="{timestamp}"
      year="numeric"
      month="numeric"
      day="numeric"
      hour="2-digit"
      minute="2-digit"
      second="2-digit"
    ></lightning-formatted-date-time>
    ```
- **`lightning-pill-container`**:
  - **Description:** A container for displaying a list of `lightning-pill` components.
  - **Key Attributes:** `items`, `onitemremove`.
  - **Example (`recordPickerMultiValue.html`):**
    ```html
    <lightning-pill-container
      items="{pillItems}"
      onitemremove="{handlePillRemove}"
    ></lightning-pill-container>
    ```

## Navigation and Interaction

- **`lightning-button`**:

  - **Description:** A standard button element.
  - **Key Attributes:** `label`, `variant` (`neutral`, `brand`, `outline-brand`, `destructive`, `success`), `icon-name`, `icon-position`, `onclick`, `type` (`submit`, `reset`, `button`).
  - **Example (`helloBinding.html`):**
    ```html
    <lightning-button
      label="Add"
      onclick="{increaseCounter}"
    ></lightning-button>
    ```

- **`lightning-button-icon`**:

  - **Description:** An icon-only button.
  - **Key Attributes:** `icon-name`, `variant`, `alternative-text`, `size`, `onclick`.
  - **Example (`ldsDeleteRecord.html`):**
    ```html
    <lightning-button-icon
      icon-name="utility:delete"
      onclick="{deleteAccount}"
      data-recordid="{account.Id}"
    ></lightning-button-icon>
    ```

- **`lightning-record-picker`**:
  - **Description:** Provides a UI for searching and selecting a single Salesforce record.
  - **Key Attributes:** `label`, `object-api-name`, `placeholder`, `filter`, `display-info`, `matching-info`, `onchange`.
  - **Example (`recordPickerHello.html`):**
    ```html
    <lightning-record-picker
      object-api-name="Contact"
      placeholder="Search..."
      label="Select a record"
      onchange="{handleChange}"
    >
    </lightning-record-picker>
    ```

## Notifications and Modals

- **`lightning-spinner`**:
  - **Description:** Displays an animated spinner to indicate loading.
  - **Key Attributes:** `alternative-text`, `size`, `variant`.
  - **Example (`miscRestApiCall.html`):**
    ```html
    <lightning-spinner
      lwc:if="{isLoading}"
      alternative-text="Loading records"
    ></lightning-spinner>
    ```
- **`LightningAlert` (from `lightning/alert`)**:
  - **Description:** A modal dialog that displays an alert message. Replaces `window.alert()`.
  - **Usage (JS):**
    ```javascript
    import LightningAlert from 'lightning/alert';
    // ...
    async handleAlertClick() {
        await LightningAlert.open({
            message: 'This is an alert message',
            theme: 'info', // e.g., info, success, warning, error
            label: 'Alert!'
        });
    }
    ```
- **`LightningConfirm` (from `lightning/confirm`)**:
  - **Description:** A modal dialog that asks for user confirmation. Replaces `window.confirm()`.
  - **Usage (JS):**
    ```javascript
    import LightningConfirm from 'lightning/confirm';
    // ...
    async handleConfirmClick() {
        const result = await LightningConfirm.open({
            message: 'this is the prompt message',
            variant: 'headerless', // Or default with header
            label: 'this is the aria-label value'
        });
        // result is true if OK was clicked, false if Cancel was clicked
    }
    ```
- **`LightningPrompt` (from `lightning/prompt`)**:
  - **Description:** A modal dialog that prompts the user for input. Replaces `window.prompt()`.
  - **Usage (JS):**
    ```javascript
    import LightningPrompt from 'lightning/prompt';
    // ...
    async handlePromptClick() {
        const result = await LightningPrompt.open({
            message: 'Please enter a value',
            label: 'Please Respond', // Aria label for the input field
            defaultValue: 'initial value',
            theme: 'shade' // e.g., default, shade, warning, error, success
        });
        // result is the entered text if OK was clicked, or null if Cancel was clicked
    }
    ```
- **`lightning-modal` (Base Component for Custom Modals)**:
  - Create custom modals by extending `LightningModal`. Includes `lightning-modal-header`, `lightning-modal-body`, `lightning-modal-footer`.
  - **Example (`myModal.html`):**
    ```html
    <template>
      <lightning-modal-header label="{header}"></lightning-modal-header>
      <lightning-modal-body>Content: {content}</lightning-modal-body>
      <lightning-modal-footer>
        <lightning-button
          label="Close"
          onclick="{handleClose}"
        ></lightning-button>
      </lightning-modal-footer>
    </template>
    ```
  - **Open programmatically (from another LWC):**
    ```javascript
    import MyModal from 'c/myModal';
    // ...
    async handleShowModal() {
        const result = await MyModal.open({
            size: 'small',
            description: 'Accessible description of the modal',
            header: this.header,
            content: this.content
        });
        // result is the value passed to this.close() in MyModal
    }
    ```

## Utility Components

- **`lightning-icon`**:

  - **Description:** Displays an SLDS icon.
  - **Key Attributes:** `icon-name` (e.g., `utility:user`, `standard:account`), `alternative-text`, `size`, `variant`.
  - **Example (`contactTile.html`):**
    ```html
    <lightning-icon
      icon-name="standard:avatar"
      alternative-text="Missing profile photo"
      size="medium"
    ></lightning-icon>
    ```

- **`lightning-messages`**:

  - **Description:** Displays messages, typically errors, from `lightning-record-edit-form` or `lightning-record-form`.
  - **Example (`recordEditFormDynamicContact.html`):**
    ```html
    <lightning-messages></lightning-messages>
    ```

- **`lightning-tree`**:

  - **Description:** Displays a hierarchical list of items.
  - **Key Attributes:** `items`, `header`.
  - **Example (`wireGetPicklistValuesByRecordType.html`):**
    ```html
    <lightning-tree
      items="{treeModel}"
      header="Account Picklists"
    ></lightning-tree>
    ```

- **`lightning-quick-action-panel`**:
  - **Description:** Provides the standard styling for the body of a screen quick action that launches an LWC.
  - **Key Attributes:** `header`.
  - **Example (`editRecordScreenAction.html`):**
    ```html
    <template>
      <lightning-quick-action-panel header="Edit contact name">
        <!-- Form fields -->
        <div slot="footer">
          <lightning-button
            label="Cancel"
            onclick="{handleCancel}"
          ></lightning-button>
          <lightning-button
            variant="brand"
            label="Save"
            onclick="{handleSave}"
          ></lightning-button>
        </div>
      </lightning-quick-action-panel>
    </template>
    ```
