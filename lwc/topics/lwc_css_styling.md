# LWC CSS Styling

Styling in Lightning Web Components ensures that component styles are encapsulated and do not unintentionally affect other parts of the page. LWC uses standard CSS, with some specific considerations for how styles are applied and scoped.

## Component-Scoped CSS

- Each LWC can have its own CSS file (e.g., `myComponent.css`) in its component bundle.
- Styles defined in this file are scoped to the component by default. This means CSS rules only apply to the elements within that component's template.
- This encapsulation is achieved through the Shadow DOM, which isolates the component's DOM tree and styles.

  ```css
  /* myComponent.css - These styles only apply to elements within myComponent.html */
  p {
    color: blue;
    font-size: 16px;
  }

  .important-text {
    font-weight: bold;
    text-decoration: underline;
  }
  ```

## Importing Additional Stylesheets

You can import multiple CSS files into a single component. This is useful for organizing styles or sharing common styles among a group of related components (though global styling is often better handled via SLDS or custom styling hooks).

- Import the CSS file as a module in your component's JavaScript file.
- Assign an array of imported style modules to the `static stylesheets` property of the component class.
- Styles from imported stylesheets are also scoped to the component.

```javascript
// stylesheets.js
import { LightningElement } from "lwc";
import baseStyles from "./base-styles.css"; // An imported CSS file
import textStyles from "./text-styles.css"; // Another imported CSS file

export default class Stylesheets extends LightningElement {
  static stylesheets = [baseStyles, textStyles]; // Apply both
}
```

````

```css
/* text-styles.css (Example of an imported stylesheet) */
.fancy-text {
  color: #d54d7b;
  font-family: "Great Vibes", cursive;
  text-shadow: 0 1px 1px #fff;
}
```

```html
<!-- stylesheets.html -->
<template>
  <lightning-card title="Stylesheets" icon-name="custom:custom3">
    <div class="card-body">
      <!-- card-body might be styled in base-styles.css -->
      <p class="fancy-text">This is a paragraph with style!</p>
      <!-- fancy-text styled in text-styles.css -->
    </div>
  </lightning-card>
</template>
```

## Styling Hooks (CSS Custom Properties)

Styling hooks are CSS custom properties exposed by Salesforce Lightning Design System (SLDS) and Base Lightning Components. They allow you to customize the look and feel of these components without breaking encapsulation or resorting to complex CSS overrides.

- You can set these CSS custom properties in your component's CSS file, typically within the `:host` pseudo-class to apply them to the component itself and its children (including base components).
- They provide a standardized way to theme components.

```css
/* stylingHooks.css */
:host {
  /* Example: Customizing a lightning-card base component */
  --slds-c-card-color-background: linear-gradient(
    115deg,
    #020024 60%,
    #a7a400 100%
  );
  --slds-c-card-text-color: #e3df00;
  --slds-c-card-radius-border: 1.5rem 0;

  /* Example: Customizing a lightning-button base component */
  --slds-c-button-text-color: #e3df00;
  --slds-c-button-neutral-color-background: #020024;
  --slds-c-button-neutral-color-border: #e3df00;
}

/* Specific styling for a toggle if needed */
.toggle-red {
  --slds-c-checkbox-toggle-color-border-checked: #990000;
  --sds-c-checkbox-toggle-color-background-checked: #990000;
}
```

```html
<!-- stylingHooks.html -->
<template>
  <!-- The lightning-card will inherit styles from the :host CSS custom properties -->
  <lightning-card title="StylingHooks" icon-name="custom:custom3">
    <div class="slds-var-m-around_medium">
      <lightning-input
        type="toggle"
        label="Some toggle control"
        checked
        class="slds-var-m-bottom_medium"
      ></lightning-input>
      <lightning-input
        type="toggle"
        label="Another toggle control"
        checked
        class="slds-var-m-bottom_medium toggle-red"
        <!--
        This
        toggle
        gets
        additional
        specific
        hook
        overrides
        --
      >
        ></lightning-input
      >
      <lightning-button label="A button"></lightning-button>
      <!-- This button also inherits styles -->
    </div>
  </lightning-card>
</template>
```

The `c-view-source` component in the example also demonstrates consuming styling hooks (`--source-text-color`, `--source-link-color`) defined by its parent (`stylingHooks.css`).

## Light DOM Styling

Components explicitly configured to use Light DOM (`static renderMode = 'light';`) do not have their styles scoped by Shadow DOM.

- Styles defined in a Light DOM component's CSS file behave like standard global CSS within the context of that component's rendered markup.
- This means they can affect parent or sibling elements if selectors are not carefully written, and they can also be affected by global stylesheets.
- Care should be taken with Light DOM styling to avoid unintended style conflicts.

## Styling Child Components from a Parent

- **Standard (Shadow DOM) Children:** You generally cannot directly style the internal elements of a child component from a parent's CSS due to Shadow DOM encapsulation.
  - Use public `@api` properties on the child to accept class names or style-related data.
  - The child component can then use these properties to apply styles internally.
  - Utilize SLDS styling hooks if the child is a base component or exposes its own hooks.
- **Light DOM Children:** Styles from a parent component's CSS _can_ affect the elements within a Light DOM child component if the CSS selectors match elements rendered by the child.
- **CSS Custom Properties (Styling Hooks):** This is the recommended way to allow parent components to influence the styling of child components (both Shadow DOM and Light DOM) by defining CSS custom properties that the child component consumes.

  ```css
  /* parent.css */
  :host {
    --child-text-color: blue;
  }
  ```

  ```css
  /* child.css */
  .child-text {
    color: var(
      --child-text-color,
      black
    ); /* Consumes parent-defined hook, with a fallback */
  }
  ```

## SLDS (Salesforce Lightning Design System)

LWC is designed to work seamlessly with SLDS. Base Lightning Components are already styled according to SLDS. For custom components, you can use SLDS utility classes directly in your HTML templates to apply consistent styling.

```html
<template>
  <!-- Using SLDS utility classes for margin and padding -->
  <div class="slds-var-m-around_medium slds-var-p-around_small">
    <p class="slds-text-heading_large">Large Heading</p>
  </div>
</template>
```

````
