````markdown
# LWC Decorators and Properties

Decorators in Lightning Web Components are special JavaScript features (currently at Stage 3 proposal for ECMAScript) that modify the behavior of properties or functions. LWC uses three core decorators: `@api`, `@track`, and `@wire`.

## Properties

Properties in an LWC JavaScript class are reactive. When a property's value changes, and that property is used in the component's template, the component re-renders to reflect the new value.

```javascript
// hello.js
import { LightningElement } from "lwc";

export default class Hello extends LightningElement {
  greeting = "World"; // This is a reactive property

  // Getter, also reactive if its dependent properties change
  get capitalizedGreeting() {
    return this.greeting.toUpperCase();
  }
}
```
````

## `@api` Decorator

- **Purpose:** Marks a property or method as public.
- **Public Properties:**
  - Can be set by a parent component in its HTML template.
  - Can be configured in tools like Lightning App Builder if the component's metadata (`.js-meta.xml`) defines them as configurable.
  - Are reactive. If a parent changes an `@api` property value, the child component re-renders.
  - **Example (`apiProperty.js` in parent, `chartBar.js` in child):**
    ```javascript
    // apiProperty.js (Parent)
    import { LightningElement } from "lwc";
    export default class ApiProperty extends LightningElement {
      percentage = 50;
      handlePercentageChange(event) {
        this.percentage = event.target.value;
      }
    }
    ```
    ```html
    <!-- apiProperty.html (Parent) -->
    <template>
      <lightning-input
        label="Percentage"
        type="number"
        value="{percentage}"
        onchange="{handlePercentageChange}"
      ></lightning-input>
      <!-- 'percentage' is passed to the child's @api property -->
      <c-chart-bar percentage="{percentage}"></c-chart-bar>
    </template>
    ```
    ```javascript
    // chartBar.js (Child)
    import { LightningElement, api } from "lwc";
    export default class ChartBar extends LightningElement {
      @api percentage; // Public property, set by parent
    }
    ```
- **Public Properties with Getters and Setters:**
  - Allows custom logic to run when a public property is set or retrieved.
  - Useful for transforming or validating incoming data.
  - **Example (`apiSetterGetter.js` parent, `todoList.js` child):**
    ```javascript
    // todoList.js (Child)
    import { LightningElement, api } from "lwc";
    export default class TodoList extends LightningElement {
      _todos = [];
      @api
      get todos() {
        return this._todos;
      }
      set todos(value) {
        // Setter allows custom logic
        this._todos = value;
        this.filterTodos(); // Example: run filtering logic when todos are set
      }
      // ... filterTodos() and other logic ...
    }
    ```
- **Public Methods:**
  - Allow parent components to call methods on child components.
  - **Example (`apiMethod.js` parent, `clock.js` child):**
    ```javascript
    // apiMethod.js (Parent)
    import { LightningElement } from "lwc";
    export default class ApiMethod extends LightningElement {
      handleRefresh() {
        // Call the public 'refresh' method on the child c-clock component
        this.template.querySelector("c-clock").refresh();
      }
    }
    ```
    ```html
    <!-- apiMethod.html (Parent) -->
    <template>
      <lightning-button
        label="Refresh Time"
        onclick="{handleRefresh}"
      ></lightning-button>
      <c-clock></c-clock>
    </template>
    ```
    ```javascript
    // clock.js (Child)
    import { LightningElement, api } from "lwc";
    export default class Clock extends LightningElement {
      timestamp = new Date();
      @api // Exposes the refresh method publicly
      refresh() {
        this.timestamp = new Date();
      }
    }
    ```

## `@wire` Decorator

- **Purpose:** Provides a reactive way to get Salesforce data or use other wire adapters.
- **Usage:**
  - Decorates a property or a function.
  - The wire adapter (e.g., `getRecord`, an Apex method) is called with specified parameters.
  - If any reactive parameter (prefixed with `$`) changes, the wire adapter is re-invoked.
  - The wired property or function receives an object with `data` and `error` properties.
- **Wiring to a Property:**

  ```javascript
  // apexWireMethodToProperty.js
  import { LightningElement, wire } from "lwc";
  import getContactList from "@salesforce/apex/ContactController.getContactList";

  export default class ApexWireMethodToProperty extends LightningElement {
    @wire(getContactList)
    contacts; // contacts.data or contacts.error will be populated
  }
  ```

- **Wiring to a Function:**

  ```javascript
  // apexWireMethodToFunction.js
  import { LightningElement, wire } from "lwc";
  import getContactList from "@salesforce/apex/ContactController.getContactList";

  export default class ApexWireMethodToFunction extends LightningElement {
    contacts;
    error;

    @wire(getContactList)
    wiredContacts({ error, data }) {
      // Function receives {data, error}
      if (data) {
        this.contacts = data;
        this.error = undefined;
      } else if (error) {
        this.error = error;
        this.contacts = undefined;
      }
    }
  }
  ```

- **Reactive Parameters:** Use `'$propertyName'` to make the wire adapter reactive to changes in `this.propertyName`.

  ```javascript
  // apexWireMethodWithParams.js
  import { LightningElement, wire } from "lwc";
  import findContacts from "@salesforce/apex/ContactController.findContacts";

  export default class ApexWireMethodWithParams extends LightningElement {
    searchKey = ""; // This will be reactive parameter

    @wire(findContacts, { searchKey: "$searchKey" }) // '$searchKey'
    contacts;

    handleKeyChange(event) {
      // ... (debouncing logic) ...
      this.searchKey = event.target.value; // Changing searchKey re-invokes the wire
    }
  }
  ```

- **Complex Reactive Parameters:** If the wire adapter parameter is an object, and its properties change, a getter function is needed for the parameter to ensure reactivity.

  ```javascript
  // apexWireMethodWithComplexParams.js
  import { LightningElement, wire } from "lwc";
  import checkApexTypes from "@salesforce/apex/ApexTypesController.checkApexTypes";

  export default class ApexWireMethodWithComplexParams extends LightningElement {
    // ... properties like listItemValue, numberValue, stringValue ...

    // The parameterObject itself is not directly decorated for reactivity in the wire.
    // Instead, the getter 'variables' (or in this case, 'parameterObject' directly via '$parameterObject')
    // constructs the object, and LWC tracks changes to its constituent reactive properties.
    parameterObject = {
      /* initial structure */
    };

    @wire(checkApexTypes, { wrapper: "$parameterObject" })
    apexResponse;

    handleStringChange(event) {
      // Re-assign parameterObject to trigger reactivity for the wire
      this.parameterObject = {
        ...this.parameterObject,
        someString: event.target.value,
      };
    }
    // ... other handlers that re-assign this.parameterObject ...
  }
  ```

  A common pattern for complex reactive parameters is using a getter:

  ```javascript
  // graphqlVariables.js (illustrative pattern for complex reactive @wire params)
  import { LightningElement, wire } from 'lwc';
  import { gql, graphql } from 'lightning/uiGraphQLApi';
  // ...
  export default class GraphqlVariables extends LightningElement {
      searchKey = '';

      @wire(graphql, { /* ... query ... */, variables: '$variables' })
      contacts;

      get variables() { // Getter constructs the reactive parameter object
          return {
              searchKey: this.searchKey === '' ? '%' : `%${this.searchKey}%`
          };
      }
      // ...
  }
  ```

## `@track` Decorator (Use with Caution)

- **Purpose (Historically):** Used to make fields in an object or elements in an array reactive. If a property held an object or array, LWC's reactivity system would not by default observe mutations _inside_ that object/array (e.g., changing `obj.field` or `arr.push(item)`). `@track` was needed to tell LWC to observe these internal changes.
- **Current Status:**

  - **Primitives:** For primitive data types (String, Number, Boolean), `@track` is **not needed**. They are inherently reactive.
  - **Objects/Arrays:** If you re-assign the entire object or array (e.g., `this.myObject = { ...newObjectData };` or `this.myArray = [...newArrayElements];`), `@track` is **not needed**. This is the recommended modern approach.
  - **Internal Mutations:** `@track` is **still needed** if you directly mutate the contents of an object or array _without re-assigning the variable itself_, and you want the template to react to these internal changes.
    - Example: `this.trackedObject.someProperty = 'newValue';` (if `trackedObject` is decorated with `@track`)
    - Example: `this.trackedArray.push('newItem');` (if `trackedArray` is decorated with `@track`)

- **Recommendation:** Minimize the use of `@track`. Prefer re-assigning objects and arrays to trigger reactivity, as it often leads to more predictable state management.

  ```javascript
  // Example of when @track might still be used (though often avoidable)
  // lwcObjectMutation.js (hypothetical example from lwc-llm.txt, adapted for clarity)
  import { LightningElement, track } from "lwc";

  export default class LwcObjectMutation extends LightningElement {
    // Without @track, mutating 'name.raw' directly might not trigger re-render in older LWC versions
    // or if the template binds to deeper properties of 'name'.
    // In modern LWC, re-assigning `this.name = {...}` is preferred.
    @track
    name = {
      raw: "Web components ",
      normalized: "Web Components",
    };

    updateRawName() {
      this.name.raw = "Updated Web Components"; // Mutation observed due to @track
    }
  }
  ```

## Standard JavaScript Getters/Setters

- Can be used to compute values dynamically or perform logic when a property is accessed or set.
- Getters are reactive if the properties they depend on are reactive.

  ```javascript
  // helloExpressions.js
  import { LightningElement } from "lwc";
  export default class HelloExpressions extends LightningElement {
    firstName = "";
    lastName = "";
    // ... handleChange method ...

    get uppercasedFullName() {
      // Getter
      return `${this.firstName} ${this.lastName}`.trim().toUpperCase();
    }
  }
  ```

```

```
