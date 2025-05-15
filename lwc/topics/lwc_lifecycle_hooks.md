# LWC Lifecycle Hooks

Lightning Web Components have a lifecycle managed by the framework. Developers can tap into this lifecycle by using callback methods, often referred to as lifecycle hooks, to execute logic at specific phases of a component's existence.

## Core Lifecycle Hooks

These are the most commonly used lifecycle hooks, executed in a predictable order:

1.  **`constructor()`**

    - **Purpose:** Called when an instance of the component is created.
    - **Usage:**
      - The first statement **must** be `super()` to call the `LightningElement` constructor.
      - Initialize local properties.
      - **Do not** access the component's DOM elements (e.g., using `this.template.querySelector`) because they haven't been rendered yet.
      - **Do not** access public properties (`@api`) set by a parent, as they are not available at this stage.
    - **Example:**

      ```javascript
      import { LightningElement } from "lwc";

      export default class MyComponent extends LightningElement {
        myProperty;

        constructor() {
          super();
          this.myProperty = "Initialized in constructor";
          console.log("Constructor called");
        }
      }
      ```

2.  **`connectedCallback()`**

    - **Purpose:** Called when the component is inserted into the DOM.
    - **Usage:**
      - Perform initialization tasks that require DOM access (though `renderedCallback` is often preferred for DOM manipulation after rendering).
      - Fetch data.
      - Subscribe to message channels (like Lightning Message Service).
      - Set up event listeners (e.g., on `window` or `document`).
    - This hook flows from parent to child.
    - **Example (`lmsSubscriberWebComponent.js`):**

      ```javascript
      import { LightningElement, wire } from "lwc";
      import { subscribe, MessageContext } from "lightning/messageService";
      import RECORD_SELECTED_CHANNEL from "@salesforce/messageChannel/Record_Selected__c";

      export default class LmsSubscriberWebComponent extends LightningElement {
        subscription = null;
        @wire(MessageContext) messageContext;

        connectedCallback() {
          console.log("ConnectedCallback called");
          this.subscribeToMessageChannel();
        }

        subscribeToMessageChannel() {
          this.subscription = subscribe(
            this.messageContext,
            RECORD_SELECTED_CHANNEL,
            (message) => this.handleMessage(message)
          );
        }
        // ...
      }
      ```

3.  **`render()`**

    - **Purpose:** Overrides the standard rendering functionality to conditionally render a different template.
    - **Usage:**
      - Return an imported HTML template.
      - Use this hook if your component needs to switch between entirely different HTML structures based on some condition.
    - **Example (`miscMultipleTemplates.js`):**

      ```javascript
      import { LightningElement } from "lwc";
      import templateOne from "./templateOne.html";
      import templateTwo from "./templateTwo.html";

      export default class MiscMultipleTemplates extends LightningElement {
        showTemplateOne = true;

        render() {
          console.log("Render called");
          return this.showTemplateOne ? templateOne : templateTwo;
        }

        switchTemplate() {
          this.showTemplateOne = !this.showTemplateOne;
        }
      }
      ```

4.  **`renderedCallback()`**

    - **Purpose:** Called after the component's template has been rendered (and re-rendered).
    - **Usage:**
      - Perform logic after the DOM is updated.
      - Ideal for interacting with the component's rendered DOM elements or integrating third-party libraries that manipulate the DOM.
      - **Caution:** This hook can be called multiple times during a component's lifecycle. To avoid infinite loops if you modify reactive properties within it, use a boolean flag to ensure logic runs only once or under specific conditions.
    - This hook flows from child to parent.
    - **Example (`libsChartjs.js`):**

      ```javascript
      import { LightningElement } from "lwc";
      import chartjs from "@salesforce/resourceUrl/chartJs";
      import { loadScript } from "lightning/platformResourceLoader";

      export default class LibsChartjs extends LightningElement {
        chartjsInitialized = false; // Flag to run initialization once

        async renderedCallback() {
          console.log("RenderedCallback called");
          if (this.chartjsInitialized) {
            return;
          }
          this.chartjsInitialized = true;

          try {
            await loadScript(this, chartjs);
            // ... initialize chart ...
          } catch (error) {
            // ... handle error ...
          }
        }
      }
      ```

5.  **`disconnectedCallback()`**

    - **Purpose:** Called when the component is removed from the DOM.
    - **Usage:**
      - Clean up work done in `connectedCallback`.
      - Unsubscribe from message channels.
      - Remove event listeners that were added to global objects like `window` or `document`.
    - This hook flows from parent to child.
    - **Example (`lmsSubscriberWebComponent.js`):**

      ```javascript
      // (Continuing from the LmsSubscriberWebComponent example above)
      // ...
      import { unsubscribe } from "lightning/messageService";

      export default class LmsSubscriberWebComponent extends LightningElement {
        // ...
        disconnectedCallback() {
          console.log("DisconnectedCallback called");
          this.unsubscribeToMessageChannel();
        }

        unsubscribeToMessageChannel() {
          unsubscribe(this.subscription);
          this.subscription = null;
        }
        // ...
      }
      ```

## Error Handling Lifecycle Hook

- **`errorCallback(error, stack)`**

  - **Purpose:** Called when a descendant component throws an error during one of its lifecycle hooks or when rendering. It acts as an error boundary.
  - **Usage:**
    - Capture errors from child components.
    - Log errors or display a user-friendly error message.
    - The `error` argument is a JavaScript native error object.
    - The `stack` argument is a string detailing the component stack.
  - This hook is specific to LWC and helps in creating robust applications by gracefully handling errors in the component tree.
  - **Note:** An `errorCallback` in a parent component does _not_ catch errors from the parent component itself, only from its children.
  - **Example (Conceptual):**

    ```javascript
    import { LightningElement } from "lwc";

    export default class ErrorBoundaryParent extends LightningElement {
      errorMessage;
      errorStack;

      errorCallback(error, stack) {
        console.error("Error caught in errorCallback:", error.message);
        console.error("Stack:", stack);
        this.errorMessage =
          "An error occurred in a child component: " + error.message;
        this.errorStack = stack;
      }
    }
    ```

    ```html
    <!-- errorBoundaryParent.html -->
    <template>
      <template lwc:if="{errorMessage}">
        <div>
          <p>{errorMessage}</p>
          <pre>{errorStack}</pre>
        </div>
      </template>
      <template lwc:else>
        <c-child-component-that-might-throw-error></c-child-component-that-might-throw-error>
      </template>
    </template>
    ```

Understanding and utilizing these lifecycle hooks is crucial for managing component initialization, data fetching, DOM interaction, cleanup, and error handling effectively in LWC.
