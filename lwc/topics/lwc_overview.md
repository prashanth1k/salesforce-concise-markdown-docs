# Lightning Web Components (LWC): Overview

Lightning Web Components (LWC) is a Salesforce UI framework for building web applications using modern web standards (HTML, JavaScript, CSS). LWCs are custom HTML elements, encapsulated for reusability and performance.

## Core Structure

The foundation of an LWC is a JavaScript class that extends `LightningElement`:

```javascript
import { LightningElement } from "lwc";

class MyComponent extends LightningElement {
  // Component logic, properties, and methods
  greeting = "World";
}
```

An LWC typically includes:

- **HTML Template (`.html`):** Defines the component's structure. Data binding uses `{propertyName}`.

  ```html
  <template>
    <p>Hello, {greeting}!</p>
  </template>
  ```

- **JavaScript File (`.js`):** Contains the component's logic, extending `LightningElement`.
- **CSS File (`.css`):** (Optional) For component-specific styling.
- **Metadata File (`.js-meta.xml`):** Defines how the component is used in Salesforce (e.g., targets, public properties).
  ```xml
  <?xml version="1.0" encoding="UTF-8" ?>
  <LightningComponentBundle xmlns="http://soap.sforce.com/2006/04/metadata">
      <apiVersion>62.0</apiVersion>
      <isExposed>true</isExposed>
      <targets>
          <target>lightning__AppPage</target>
          <!-- Other targets -->
      </targets>
  </LightningComponentBundle>
  ```

## Key Features & Usage

- **Standards-Based:** Uses Web Components, ES Modules, and modern JavaScript.
- **Performance:** Lightweight and optimized for browser performance.
- **Encapsulation:** Shadow DOM for style and behavior isolation.
- **Reusability:** Build complex UIs by composing smaller components.
- **Salesforce Integration:**
  - **Data Access:** `@wire` service for reactive data (e.g., `getRecord`, Apex).
  - **UI Elements:** Utilizes Base Lightning Components (e.g., `lightning-input`, `lightning-card`).
  - **Platform Features:** Navigation, toast notifications, custom permissions.
- **Developer Experience:**
  - **Decorators:** `@api` (public properties), `@wire` (data binding), `@track` (explicit reactivity for older object/array mutations).
  - **Communication:** Custom DOM events for child-to-parent; Lightning Message Service (LMS) for cross-DOM communication.

The LWC Recipes app provides many examples of these features in action.
