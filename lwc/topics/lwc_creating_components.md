# Creating Lightning Web Components

Creating Lightning Web Components involves defining their structure (HTML), behavior (JavaScript), styling (CSS), and metadata (XML).

## Basic Component Structure

An LWC is a bundle of files in a directory named after the component (e.g., `myComponent/`). The root element in HTML templates must always be `<template>`.

1.  **JavaScript (`myComponent.js`):**
    The core logic. Extends `LightningElement`.

    ```javascript
    import { LightningElement } from "lwc";

    export default class MyComponent extends LightningElement {
      greeting = "World"; // Reactive property
    }
    ```

2.  **HTML Template (`myComponent.html`):**
    Defines the component's UI structure. Uses `{propertyName}` for data binding.

    ```html
    <template>
      <p>Hello, {greeting}!</p>
    </template>
    ```

3.  **CSS File (`myComponent.css`):** (Optional)
    Styles are scoped to the component.

    ```css
    /* Example: Styles for p element in this component */
    p {
      font-weight: bold;
    }
    ```

4.  **Metadata Configuration (`myComponent.js-meta.xml`):**
    Defines component visibility and properties for tools like App Builder.
    ```xml
    <?xml version="1.0" encoding="UTF-8"?>
    <LightningComponentBundle xmlns="http://soap.sforce.com/2006/04/metadata">
        <apiVersion>62.0</apiVersion>
        <isExposed>true</isExposed>
        <targets>
            <target>lightning__AppPage</target>
            <target>lightning__HomePage</target>
        </targets>
    </LightningComponentBundle>
    ```

## Public Properties (`@api`)

Expose properties to parent components or App Builder using the `@api` decorator.

```javascript
// childComponent.js
import { LightningElement, api } from "lwc";

export default class ChildComponent extends LightningElement {
  @api itemName = "Default Item"; // Public property
}
```

```html
<!-- parentComponent.html -->
<template>
  <c-child-component item-name="Passed from Parent"></c-child-component>
</template>
```

## Data Binding and User Input

Properties in the JavaScript class are reactive. Changes automatically reflect in the template.

```javascript
// helloBinding.js
import { LightningElement } from "lwc";

export default class HelloBinding extends LightningElement {
  greeting = "World";

  handleChange(event) {
    this.greeting = event.target.value; // Update property on input change
  }
}
```

```html
<!-- helloBinding.html -->
<template>
  <p>Hello, {greeting}!</p>
  <lightning-input
    label="Name"
    value="{greeting}"
    onchange="{handleChange}"
  ></lightning-input>
</template>
```

## Conditional Rendering

Use `lwc:if`, `lwc:else`, and `lwc:elseif` directives to render HTML conditionally.

```javascript
// helloConditionalRendering.js
import { LightningElement } from "lwc";

export default class HelloConditionalRendering extends LightningElement {
  areDetailsVisible = false;

  handleChange(event) {
    this.areDetailsVisible = event.target.checked;
  }
}
```

```html
<!-- helloConditionalRendering.html -->
<template>
  <lightning-input
    type="checkbox"
    label="Show details"
    onchange="{handleChange}"
  ></lightning-input>
  <template lwc:if="{areDetailsVisible}">
    <p>These are the details!</p>
  </template>
  <template lwc:else>
    <p>Not showing details.</p>
  </template>
</template>
```

## Iterating Over Lists

LWC provides two directives for iterating over lists:

- **`for:each={array}` and `for:item="currentItem"`**:

  - Iterates over an array. `currentItem` holds the current item in the iteration.
  - Optionally, use `for:index="currentIndex"` to get the index of the current item.
  - A `key={uniqueId}` attribute is **required** on the first element inside the `template` tag for efficient re-rendering. The key must be a string or number and unique among its siblings.

  ```javascript
  // helloForEach.js
  import { LightningElement } from "lwc";
  export default class HelloForEach extends LightningElement {
    contacts = [
      { Id: "1", Name: "Amy Taylor", Title: "VP of Engineering" },
      { Id: "2", Name: "Michael Jones", Title: "VP of Sales" },
    ];
  }
  ```

  ```html
  <!-- helloForEach.html -->
  <template>
    <ul>
      <template for:each="{contacts}" for:item="contact">
        <li key="{contact.Id}">{contact.Name}, {contact.Title}</li>
      </template>
    </ul>
  </template>
  ```

- **`iterator:iteratorName={array}`**:

  - Similar to `for:each`, but provides additional properties:
    - `iteratorName.value`: The current item
    - `iteratorName.index`: The current item's index
    - `iteratorName.first`: Boolean, true if it's the first item
    - `iteratorName.last`: Boolean, true if it's the last item
  - Requires a `key={iteratorName.value.uniqueId}`

  ```html
  <!-- helloIterator.html -->
  <template>
    <ul>
      <template iterator:it="{contacts}">
        <li key="{it.value.Id}">
          <div lwc:if="{it.first}" class="list-first"></div>
          {it.value.Name}, {it.value.Title}
          <div lwc:if="{it.last}" class="list-last"></div>
        </li>
      </template>
    </ul>
  </template>
  ```

## Component Composition

Build complex UIs by nesting child components within parent components.

- **Passing Data:** Use public `@api` properties on the child.

  ```html
  <!-- compositionBasics.html (Parent) -->
  <template>
    <c-contact-tile contact="{contact}"></c-contact-tile>
  </template>
  ```

  ```javascript
  // compositionBasics.js (Parent)
  import { LightningElement } from 'lwc';
  export default class CompositionBasics extends LightningElement {
      contact = { Name: 'Amy Taylor', Title: 'VP of Engineering', ... };
  }
  ```

  ```javascript
  // contactTile.js (Child)
  import { LightningElement, api } from "lwc";
  export default class ContactTile extends LightningElement {
    @api contact;
  }
  ```

- **Spreading Properties (`lwc:spread`):** Pass an object's properties directly to a child.
  ```html
  <!-- apiSpread.html (Parent) -->
  <template>
    <c-child lwc:spread="{props}"></c-child>
  </template>
  ```
  ```javascript
  // apiSpread.js (Parent)
  import { LightningElement } from "lwc";
  export default class ApiSpread extends LightningElement {
    props = { firstName: "Amy", lastName: "Taylor" };
  }
  ```
  ```javascript
  // child.js (Child)
  import { LightningElement, api } from "lwc";
  export default class Child extends LightningElement {
    @api firstName;
    @api lastName;
  }
  ```

## Base Lightning Components

LWC provides a rich set of pre-built UI components for common use cases. Import and use them in your templates. Examples include:

- `lightning-card`: For structured content display.
- `lightning-input`: For user input (text, number, checkbox, etc.).
- `lightning-button`: For user actions.
- `lightning-combobox`: For dropdown selection.
- `lightning-record-form`: Displays/edits a record with fields determined by layout.
- `lightning-record-edit-form`: Customizable form for editing a record's fields.
- `lightning-record-view-form`: Customizable form for viewing a record's fields.
- `lightning-datatable`: For displaying tabular data with features like inline editing and custom data types.
- `lightning-record-picker`: For searching and selecting records.
- `lightning-pill-container`: For displaying a collection of pills.
- `lightning-formatted-text`, `lightning-formatted-number`, `lightning-formatted-date-time`: For locale-aware formatting.

## Event Handling

- **Handling DOM Events:** Use `on<eventtype>` attributes in HTML (e.g., `onclick`, `onchange`).

  ```html
  <lightning-button label="Save" onclick="{handleSave}"></lightning-button>
  ```

  ```javascript
  // In JS
  handleSave(event) {
      // Logic for save button click
  }
  ```

- **Dispatching Custom Events (Child to Parent):** Use `CustomEvent` to communicate from child to parent.
  ```javascript
  // contactListItem.js (Child)
  handleClick(event) {
      event.preventDefault();
      const selectEvent = new CustomEvent('select', {
          detail: this.contact.Id // Pass data with the event
      });
      this.dispatchEvent(selectEvent);
  }
  ```
  ```html
  <!-- eventWithData.html (Parent) -->
  <c-contact-list-item
    contact="{contact}"
    onselect="{handleSelect}"
  ></c-contact-list-item>
  ```
  ```javascript
  // eventWithData.js (Parent)
  handleSelect(event) {
      const contactId = event.detail;
      // Logic to handle selected contactId
  }
  ```
- **Event Bubbling:** Events can bubble up the DOM tree if `bubbles: true` is set in `CustomEvent` options.

## Using Static Resources and Content Assets

Import static resources (images, scripts, stylesheets) and content assets for use in your components.

- **Static Resource:**

  ```javascript
  // miscStaticResource.js
  import { LightningElement } from "lwc";
  import trailheadLogo from "@salesforce/resourceUrl/trailhead_logo";
  import trailheadCharacters from "@salesforce/resourceUrl/trailhead_characters";

  export default class MiscStaticResource extends LightningElement {
    trailheadLogoUrl = trailheadLogo;
    einsteinUrl = trailheadCharacters + "/images/einstein.png"; // Path within a zip file
  }
  ```

  ```html
  <!-- miscStaticResource.html -->
  <template>
    <img src="{trailheadLogoUrl}" alt="Trailhead logo" />
    <img src="{einsteinUrl}" alt="Einstein logo" />
  </template>
  ```

- **Content Asset:**

  ```javascript
  // miscContentAsset.js
  import { LightningElement } from "lwc";
  import recipesLogo from "@salesforce/contentAssetUrl/recipes_sq_logo";

  export default class MiscContentAsset extends LightningElement {
    recipesLogoUrl = recipesLogo;
  }
  ```

  ```html
  <!-- miscContentAsset.html -->
  <template>
    <img src="{recipesLogoUrl}" alt="Recipes logo" />
  </template>
  ```

## Multiple Templates

A component can render different HTML templates based on conditions using the `render()` method.

```javascript
// miscMultipleTemplates.js
import { LightningElement } from "lwc";
import templateOne from "./templateOne.html";
import templateTwo from "./templateTwo.html";

export default class MiscMultipleTemplates extends LightningElement {
  showTemplateOne = true;

  render() {
    return this.showTemplateOne ? templateOne : templateTwo;
  }

  switchTemplate() {
    this.showTemplateOne = !this.showTemplateOne;
  }
}
```

`templateOne.html` and `templateTwo.html` would be separate HTML files in the same component bundle.

## Slots

Slots are placeholders in a component's template that a parent component can populate with its own markup. This allows for flexible and reusable component structures.

- **Default Slot:** Content passed without a `slot` attribute from the parent goes into the unnamed slot.
- **Named Slots:** Define slots with a `name` attribute in the child (e.g., `<slot name="footer"></slot>`). The parent provides content for that slot using the `slot="footer"` attribute.

```html
<!-- viewSource.html (Child component with slots) -->
<template>
  <div class="description"><slot></slot></div>
  <!-- Default slot -->
  <p>
    <a href="{sourceURL}" target="source">View Source</a>
  </p>
</template>
```

```html
<!-- miscStaticResource.html (Parent component using the child) -->
<template>
  <lightning-card title="MiscStaticResource" icon-name="custom:custom19">
    <!-- Card content -->
    <c-view-source source="lwc/miscStaticResource" slot="footer">
      <!-- Content for the 'footer' slot in lightning-card -->
      Use static resources.
      <!-- This content goes into the default slot of c-view-source -->
    </c-view-source>
  </lightning-card>
</template>
```

In the example above, `lightning-card` itself uses a named slot "footer". The `c-view-source` component is placed into that slot. The text "Use static resources." inside `c-view-source` goes into the _default_ slot of `c-view-source`.

## Spreading Properties (`lwc:spread`)

The `lwc:spread={object}` directive allows you to pass all properties of an object directly as attributes to a child component. This can simplify templates when many properties need to be passed down.

```html
<!-- apiSpread.html (Parent) -->
<template>
  <!-- props object might contain { firstName: 'Amy', lastName: 'Taylor' } -->
  <c-child lwc:spread="{props}"></c-child>
</template>
```

The `c-child` component would then receive `firstName` and `lastName` as individual `@api` properties.

## DOM Element Access

While direct DOM manipulation is discouraged in favor of reactive properties, you can query elements within a component's shadow DOM using:

- `this.template.querySelector()`
- `this.template.querySelectorAll()`

For components using Light DOM (`static renderMode = 'light';`), use:

- `this.querySelector()`
- `this.querySelectorAll()`

```javascript
// miscDomQuery.js
import { LightningElement } from "lwc";
export default class MiscDomQuery extends LightningElement {
  selection;
  handleCheckboxChange() {
    const checked = Array.from(
      this.template.querySelectorAll("lightning-input") // Querying elements
    )
      .filter((element) => element.checked)
      .map((element) => element.label);
    this.selection = checked.join(", ");
  }
}
```

## CSS and Styling

Styles in LWC are scoped by default to prevent unintended style leakage. Here are the key styling features:

1. **Component-Scoped CSS:**

   ```css
   /* myComponent.css */
   .my-component {
     color: blue;
   }
   ```

2. **External Stylesheets:**

   ```javascript
   // stylesheets.js
   import { LightningElement } from "lwc";
   import textStyles from "./text-styles.css"; // Importing another CSS file

   export default class Stylesheets extends LightningElement {
     static stylesheets = [textStyles];
   }
   ```

3. **CSS Custom Properties (Styling Hooks):**
   Use these to customize base Lightning components and custom components:

   ```css
   /* stylingHooks.css */
   :host {
     /* Card styling hooks */
     --slds-c-card-color-background: navy;
     --slds-c-card-text-color: white;
   }
   ```

4. **Shadow DOM Styling:**

   - Use `:host` to style the component's container
   - Use `:host()` with a selector to apply conditional styles

   ```css
   :host {
     display: block;
     background: white;
   }

   :host(.selected) {
     background: yellow;
   }
   ```

5. **Global Stylesheets:**
   For rare cases where global styles are needed, use the `@salesforce/resourceUrl` import:

   ```javascript
   import { loadStyle } from "lightning/platformResourceLoader";
   import globalStyles from "@salesforce/resourceUrl/globalStyles";

   // In connectedCallback or other lifecycle hook:
   loadStyle(this, globalStyles)
     .then(() => {
       console.log("Styles loaded");
     })
     .catch((error) => {
       console.error(error);
     });
   ```
