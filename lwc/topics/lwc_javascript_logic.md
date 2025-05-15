# LWC JavaScript Logic

The JavaScript file is the heart of a Lightning Web Component, defining its behavior, managing its state, and interacting with the Salesforce platform.

## Core Concepts

- **ES Modules:** LWC JavaScript files are ES6 modules. Use `import` to bring in functionality from other modules (like `LightningElement` from `lwc`, or Apex methods) and `export default class` to define the component.
- **`LightningElement`:** All LWCs must extend the `LightningElement` base class.
- **Reactivity:**
  - Properties declared in the class are reactive by default for primitive types. If their value changes, the component re-renders if that property is used in the template.
  - For objects and arrays, reactivity applies to re-assignment of the variable. For mutations _within_ an object or array to trigger re-renders, either re-assign the variable with a new copy (e.g., using spread syntax) or use the `@track` decorator (less common in modern LWC).
- **Methods:** Define methods within the class to handle events, perform calculations, or manage component logic.

## Importing Resources

- **LWC Modules:**
  ```javascript
  import { LightningElement, api, wire, track } from "lwc"; // Core LWC features
  import { ShowToastEvent } from "lightning/platformShowToastEvent"; // Platform events
  import { NavigationMixin } from "lightning/navigation"; // Navigation
  import {
    getRecord,
    createRecord,
    updateRecord,
    deleteRecord,
  } from "lightning/uiRecordApi"; // UI API
  import { MessageContext, publish, subscribe } from "lightning/messageService"; // LMS
  ```
- **Apex Methods:**
  ```javascript
  import getContactList from "@salesforce/apex/ContactController.getContactList";
  import updateContacts from "@salesforce/apex/ContactController.updateContacts";
  ```
- **Schema (SObjects and Fields):**
  ```javascript
  import ACCOUNT_OBJECT from "@salesforce/schema/Account";
  import NAME_FIELD from "@salesforce/schema/Contact.Name";
  import EMAIL_FIELD from "@salesforce/schema/Contact.Email";
  ```
- **Static Resources:**
  ```javascript
  import trailheadLogo from "@salesforce/resourceUrl/trailhead_logo";
  import D3 from "@salesforce/resourceUrl/d3"; // For libraries like D3.js
  ```
- **Content Assets:**
  ```javascript
  import recipesLogo from "@salesforce/contentAssetUrl/recipes_sq_logo";
  ```
- **Custom Labels:**
  ```javascript
  // import myLabel from '@salesforce/label/c.My_Custom_Label'; // Not directly in recipes, but common
  ```
- **User Information:**
  ```javascript
  import Id from "@salesforce/user/Id";
  import isGuest from "@salesforce/user/isGuest"; // Not directly in recipes
  ```
- **Permissions:**
  ```javascript
  import hasAccessRestrictedUI from "@salesforce/customPermission/accessRestrictedUIPermission";
  // import hasStandardPermission from '@salesforce/userPermission/ViewSetup'; // Not directly in recipes
  ```
- **Internationalization (i18n):**
  ```javascript
  import USER_LOCALE from "@salesforce/i18n/locale";
  import USER_CURRENCY from "@salesforce/i18n/currency";
  ```
- **Other JavaScript Modules (Shared Code):**
  ```javascript
  import { getTermOptions, calculateMonthlyPayment } from "c/mortgage"; // Importing from another LWC 'mortgage'
  import { reduceErrors } from "c/ldsUtils"; // Utility functions
  ```

## Properties and Fields

- Declare properties directly within the class.
  ```javascript
  export default class MyComponent extends LightningElement {
    greeting = "Hello"; // Reactive property
    count = 0;
    contact = { name: "Salesforce", type: "CRM" };
    isLoading = false;
  }
  ```
- **Getters:** Compute derived values. They are reactive if their dependent properties are reactive.
  ```javascript
  // helloExpressions.js
  get uppercasedFullName() {
      return `${this.firstName} ${this.lastName}`.trim().toUpperCase();
  }
  ```

## Event Handlers

- Define methods to respond to DOM events or custom events.
- The method receives an `event` object. Access event data via `event.target` (for DOM properties like `value`, `checked`) or `event.detail` (for custom event payloads).

  ```javascript
  // helloBinding.js
  handleChange(event) {
      this.greeting = event.target.value;
  }

  // eventWithData.js (Parent handling custom event from child)
  handleSelect(event) {
      const contactId = event.detail; // Accessing payload from custom event
      // ...
  }
  ```

## Asynchronous Operations

- Use `async/await` for cleaner handling of Promises, especially with Apex calls or UI API functions.
- **Example (Imperative Apex Call):**
  ```javascript
  // apexImperativeMethod.js
  import getContactList from '@salesforce/apex/ContactController.getContactList';
  // ...
  async handleLoad() {
      try {
          this.contacts = await getContactList();
          this.error = undefined;
      } catch (error) {
          this.contacts = undefined;
          this.error = error;
      }
  }
  ```
- **Example (UI API `createRecord`):**
  ```javascript
  // ldsCreateRecord.js
  import { createRecord } from 'lightning/uiRecordApi';
  // ...
  async createAccount() {
      // ... build recordInput ...
      try {
          const account = await createRecord(recordInput);
          // ... handle success ...
      } catch (error) {
          // ... handle error ...
      }
  }
  ```

## Interacting with DOM

- `this.template.querySelector(selector)`: Selects the first matching element within the component's shadow DOM.
- `this.template.querySelectorAll(selector)`: Selects all matching elements.
- For Light DOM components, use `this.querySelector()` and `this.querySelectorAll()`.

  ```javascript
  // miscDomQuery.js
  handleCheckboxChange() {
      const checkedElements = Array.from(
          this.template.querySelectorAll('lightning-input')
      ).filter(element => element.checked);
      // ...
  }
  ```

## Calling Public Methods on Child Components

- Use `this.template.querySelector('c-child-component-name').publicMethodName();`

  ```javascript
  // apiMethod.js
  handleRefresh() {
      this.template.querySelector('c-clock').refresh();
  }
  ```

## Utilities and Shared Code

- Create separate LWC JavaScript modules (e.g., `ldsUtils.js`, `mortgage.js`) for utility functions.
- Import these functions into other components. This promotes code reuse and organization.

  ```javascript
  // ldsUtils.js
  export function reduceErrors(errors) {
    /* ... */
  }

  // anotherComponent.js
  import { reduceErrors } from "c/ldsUtils";
  // ... use reduceErrors() ...
  ```

## Debouncing

- For event handlers that trigger expensive operations (like Apex calls on input change), use debouncing to limit the frequency of calls. This involves using `setTimeout` and `clearTimeout`.

  ```javascript
  // apexWireMethodWithParams.js
  const DELAY = 300;
  // ...
  handleKeyChange(event) {
      window.clearTimeout(this.delayTimeout);
      const searchKey = event.target.value;
      this.delayTimeout = setTimeout(() => {
          this.searchKey = searchKey; // This reactive property change triggers the @wire
      }, DELAY);
  }
  ```

## Logging

- `console.log()`: For basic client-side logging during development.
- `lightning/logger`: For logging to Event Monitoring in production, providing better observability.
  ```javascript
  // miscLogger.js
  import { log } from 'lightning/logger';
  // ...
  logMessageEventMonitoring() {
      let msg = { type: 'click', action: 'Log' };
      log(msg); // Logs to Event Monitoring (if enabled) and console
  }
  ```
