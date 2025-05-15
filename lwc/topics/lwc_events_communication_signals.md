# LWC Events, Communication, and Signals

Lightning Web Components communicate using standard DOM Events, Lightning Message Service (LMS) for cross-DOM communication, and an experimental Signals feature for state management.

## Custom DOM Events (Child-to-Parent/Grandparent)

Components dispatch standard DOM `CustomEvent` objects to communicate with components in their containment hierarchy (parents or listening grandparents).

1.  **Dispatching an Event (Child):**

    - Create a `CustomEvent` with a name (lowercase, no spaces, use underscores if needed).
    - Optionally, include a `detail` property to pass data.
    - Use `this.dispatchEvent(myEvent)` to fire it.
    - **Non-Bubbling (Default):** The event must be handled directly by the immediate parent.
      ```javascript
      // contactListItem.js (Child)
      import { LightningElement, api } from "lwc";
      export default class ContactListItem extends LightningElement {
        @api contact;
        handleClick(event) {
          event.preventDefault();
          const selectEvent = new CustomEvent("select", {
            // Event name: 'select'
            detail: this.contact.Id, // Data payload
          });
          this.dispatchEvent(selectEvent);
        }
      }
      ```
    - **Bubbling:** Set `bubbles: true` in `CustomEvent` options. This allows an ancestor component (grandparent, etc.) to handle the event, not just the immediate parent. The `composed: true` option can also be used if the event needs to cross Shadow DOM boundaries (use with caution).
      ```javascript
      // contactListItemBubbling.js (Child)
      import { LightningElement, api } from "lwc";
      export default class ContactListItemBubbling extends LightningElement {
        @api contact;
        handleSelect(event) {
          event.preventDefault();
          const selectEvent = new CustomEvent("contactselect", {
            bubbles: true, // Event will bubble up
            // composed: true // If needing to cross shadow boundaries
            detail: { contactId: this.contact.Id }, // Passing the whole contact object, as seen in eventBubbling.js example
          });
          this.dispatchEvent(selectEvent);
        }
      }
      ```

2.  **Handling an Event (Parent/Ancestor):**
    - **Declarative (HTML):** Add an event listener in the parent's template using `on<eventname>`.
      ```html
      <!-- eventWithData.html (Parent of c-contact-list-item) -->
      <template>
          <!-- ... -->
          <c-contact-list-item
              contact={contact}
              onselect={handleSelect} <!-- Listener for 'select' event -->
          ></c-contact-list-item>
          <!-- ... -->
      </template>
      ```
      ```javascript
      // eventWithData.js (Parent)
      import { LightningElement, wire } from "lwc";
      // ...
      export default class EventWithData extends LightningElement {
        // ...
        handleSelect(event) {
          const contactId = event.detail; // Access data from event.detail
          // ...
        }
      }
      ```
    - **Bubbling Event Handler (Ancestor):** If the event bubbles, a higher-level ancestor can listen.
      ```html
      <!-- eventBubbling.html (Grandparent of c-contact-list-item-bubbling) -->
      <template>
        <lightning-layout-item oncontactselect="{handleContactSelect}">
          <!-- Listener on a containing element -->
          <template for:each="{contacts.data}" for:item="contact">
            <c-contact-list-item-bubbling
              key="{contact.Id}"
              contact="{contact}"
            ></c-contact-list-item-bubbling>
          </template>
        </lightning-layout-item>
        <!-- ... -->
      </template>
      ```
      ```javascript
      // eventBubbling.js (Grandparent)
      import { LightningElement, wire } from "lwc";
      // ...
      export default class EventBubbling extends LightningElement {
        selectedContact;
        // ...
        handleContactSelect(event) {
          // For bubbling events, event.target refers to the component that dispatched the event.
          // The detail payload might be nested if the event was retargeted.
          // In LWC Recipes 'eventBubbling' example, it directly accesses event.target.contact
          // implying 'contactselect' was directly handled on 'c-contact-list-item-bubbling'
          // or the data was structured simply in event.detail.
          // For `contactListItemBubbling.js` example, the data is `event.detail.contactId`.
          // If `contactListItemBubbling` passed the whole contact in detail: `event.detail.contact`
          this.selectedContact = event.detail.contactId
            ? this.contacts.data.find((c) => c.Id === event.detail.contactId)
            : event.target.contact; // Assuming event.target.contact from example
        }
      }
      ```

## Lightning Message Service (LMS)

LMS allows communication between components that are **not** in the same DOM hierarchy, such as components across different regions of a Lightning page, or even between LWC, Aura, and Visualforce pages.

1.  **Define a Message Channel (`.messageChannel-meta.xml`):**
    This XML file defines the "contract" of the message.

    ```xml
    <!-- Record_Selected.messageChannel-meta.xml -->
    <?xml version="1.0" encoding="UTF-8"?>
    <LightningMessageChannel xmlns="http://soap.sforce.com/2006/04/metadata">
        <masterLabel>RecordSelected</masterLabel>
        <isExposed>true</isExposed>
        <description>Message Channel to pass a record Id</description>
        <lightningMessageFields>
            <fieldName>recordId</fieldName>
            <description>This is the record Id that changed</description>
        </lightningMessageFields>
    </LightningMessageChannel>
    ```

2.  **Publishing a Message (Publisher LWC):**

    - Import `publish`, `MessageContext` from `lightning/messageService`.
    - Import the message channel.
    - Wire `MessageContext`.
    - Call `publish(this.messageContext, MESSAGE_CHANNEL, payload)`.

    ```javascript
    // lmsPublisherWebComponent.js
    import { LightningElement, wire } from "lwc";
    import { publish, MessageContext } from "lightning/messageService";
    import RECORD_SELECTED_CHANNEL from "@salesforce/messageChannel/Record_Selected__c";
    // ...
    export default class LmsPublisherWebComponent extends LightningElement {
      // ...
      @wire(MessageContext)
      messageContext;

      handleContactSelect(event) {
        const payload = { recordId: event.target.contact.Id }; // Or event.detail.contactId
        publish(this.messageContext, RECORD_SELECTED_CHANNEL, payload);
      }
    }
    ```

    - Aura components and Visualforce pages can also publish to LMS.

3.  **Subscribing to a Message (Subscriber LWC):**

    - Import `subscribe`, `unsubscribe`, `MessageContext`, `APPLICATION_SCOPE` from `lightning/messageService`.
    - Import the message channel.
    - Wire `MessageContext`.
    - In `connectedCallback`, call `subscribe(this.messageContext, MESSAGE_CHANNEL, (message) => this.handleMessage(message), { scope: APPLICATION_SCOPE })`. Store the subscription object.
    - In `disconnectedCallback`, call `unsubscribe(this.subscription)`.

    ```javascript
    // lmsSubscriberWebComponent.js
    import { LightningElement, wire } from "lwc";
    import {
      subscribe,
      unsubscribe,
      MessageContext,
      APPLICATION_SCOPE,
    } from "lightning/messageService";
    import RECORD_SELECTED_CHANNEL from "@salesforce/messageChannel/Record_Selected__c";
    // ...
    export default class LmsSubscriberWebComponent extends LightningElement {
      subscription = null;
      recordId;
      // ...
      @wire(MessageContext)
      messageContext;

      subscribeToMessageChannel() {
        if (!this.subscription) {
          this.subscription = subscribe(
            this.messageContext,
            RECORD_SELECTED_CHANNEL,
            (message) => this.handleMessage(message),
            { scope: APPLICATION_SCOPE } // Optional: APPLICATION_SCOPE for messages across apps
          );
        }
      }

      unsubscribeToMessageChannel() {
        unsubscribe(this.subscription);
        this.subscription = null;
      }

      handleMessage(message) {
        this.recordId = message.recordId;
        // Fetch data based on recordId if needed
      }

      connectedCallback() {
        this.subscribeToMessageChannel();
      }

      disconnectedCallback() {
        this.unsubscribeToMessageChannel();
      }
    }
    ```

    - Aura components and Visualforce pages can also subscribe.
    - `APPLICATION_SCOPE` makes the subscription active regardless of which Lightning App the publisher/subscriber are in, as long as they are on the same Lightning page. If omitted, scope defaults to the active Lightning App.

## Signals (Experimental)

Signals provide a reactive primitive for managing state that can be shared across different parts of an application, potentially outside the direct component hierarchy. It's an alternative or complementary approach to state management compared to events or LMS for certain scenarios.

1.  **Creating a Signal:**

    - Use the `signal()` function (from a signals utility or potentially a future LWC module) to create a signal, initializing it with a value.

    ```javascript
    // In a shared JavaScript module or a component
    // import { signal } from 'some/signals'; // Hypothetical import
    // For this example, let's assume a simple custom signal implementation
    class MySignal {
      _value;
      _subscribers = new Set();
      constructor(initialValue) {
        this._value = initialValue;
      }
      get value() {
        return this._value;
      }
      set value(newValue) {
        if (this._value !== newValue) {
          this._value = newValue;
          this._subscribers.forEach((cb) => cb());
        }
      }
      subscribe(callback) {
        this._subscribers.add(callback);
        return () => this._subscribers.delete(callback);
      }
    }
    const countSignal = new MySignal(0); // Creating a signal
    ```

2.  **Updating a Signal's Value:**

    - Access and modify the `.value` property of the signal.

    ```javascript
    // In a component method
    increment() {
        countSignal.value++; // Modifying the signal's value
    }
    ```

3.  **Using a Signal in a Component (Consuming):**

    - Import the signal.
    - In the component's JavaScript, assign the signal to a property.
    - In the template, bind to the signal's `.value` property.
    - LWC's reactivity system (with appropriate signal integration) will automatically re-render the component when the signal's value changes.

    ```javascript
    // exampleComponent.js
    import { LightningElement } from "lwc";
    // import { countSignal, incrementCount } from './mySignalsStore.js'; // Assuming signal is in a store
    // For this example, let's use the locally defined signal for simplicity
    // (In real apps, signals for sharing are usually in separate modules)

    // Assume countSignal is defined as above and imported or accessible
    // For testing/example, let's re-define a local version if not truly shared
    class MySignal {
      /* ... as above ... */
    }
    const countSignal = new MySignal(0);

    export default class ExampleComponent extends LightningElement {
      count = countSignal; // Assign the signal itself

      // If signal updates are external, component needs to re-render
      // This often requires a subscription mechanism or framework support
      _unsubscribe;
      connectedCallback() {
        this._unsubscribe = this.count.subscribe(() => this.requestUpdate());
      }
      disconnectedCallback() {
        if (this._unsubscribe) this._unsubscribe();
      }
      requestUpdate() {
        // A simple way to trigger re-render, more robust solutions exist
        // eslint-disable-next-line no-self-assign
        this.count = this.count;
      }

      increment() {
        this.count.value++; // Update the signal
      }
    }
    ```

    ```html
    <!-- exampleComponent.html -->
    <template>
      <button onclick="{increment}">Increment</button>
      <p>{count.value}</p>
      <!-- Access .value in template -->
    </template>
    ```

    _Note: The LWC Recipes app does not currently have a "signals" example. The code above is illustrative based on common signal patterns and the LWC documentation for the experimental feature. True reactivity with external signals in LWC typically requires the signal implementation to integrate with LWC's rendering lifecycle or for the component to subscribe and manually trigger re-renders._

## Dynamic Interactions (App Builder Configuration)

- **Description:** Allows admins or developers to configure component interactions directly in the Lightning App Builder without code.
- **Mechanism:**
  - **Source Component:** Exposes an event in its `.js-meta.xml` with a defined schema for its payload.
    ```xml
    <!-- contactSelector.js-meta.xml -->
    <targetConfigs>
        <targetConfig targets="lightning__AppPage">
            <event
                name="select"
                label="Record Selected"
                description="Fires when a record is selected."
            >
                <schema>
                    {
                        "type": "object",
                        "properties": {
                            "recordId": {
                                "type": "string",
                                "title": "Record Id"
                            }
                       }
                    }
                </schema>
            </event>
        </targetConfig>
    </targetConfigs>
    ```
  - **Target Component:** Exposes a public `@api` property that can receive data from the source component's event.
    ```javascript
    // contactInfo.js
    import { LightningElement, api, wire } from "lwc";
    // ...
    export default class ContactInfo extends LightningElement {
      @api recordId; // This property can be targeted by the event
      // ...
    }
    ```
  - **Configuration:** In App Builder, an interaction is configured by mapping the source event payload (e.g., `recordId`) to the target component's `@api` property.
- When the source component dispatches the configured event, the Lightning App Builder runtime automatically updates the target component's property.
