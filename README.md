# dom-factory

[![npm version](https://badge.fury.io/js/dom-factory.svg)](https://badge.fury.io/js/dom-factory)

The DOM factory provides a convenient API (inspired by Polymer) to enhance HTML elements with custom behavior, using plain JavaScript objects with advanced features like property change observers, property reflection to attributes on HTML elements and simplify mundane tasks like adding and removing event listeners.

## Compatibility

- Supports ES5-compliant browsers (IE9+)

## Installation

```bash
npm install dom-factory
```

## Usage

The DOM factory library exports to AMD, CommonJS and global.

### Global

```html
<script src="node_modules/dom-factory/dist/dom-factory.js"></script>
<script>
  var handler = domFactory.handler
</script>
```

### CommonJS

```js
var handler = require('dom-factory').handler
```

### ES6

```js
import { handler } from 'dom-factory'
```

### Components

The following ES6 example, illustrates a component definition for an enhanced button using properties as a public API, property reflection to attributes, property change observers and event listeners.

#### Component definition

A component definition is nothing more than a simple object factory (a function that creates an object) which accepts a `HTMLElement` node as it's argument.

```js
/**
 * A component definition
 * @param  {HTMLElement} element
 * @return {Object}
 */
const buttonComponent = (element) => ({
  element,

  /**
   * Properties part of the component's public API.
   * @type {Object}
   */
  properties: {

    /**
     * Maps to [a-property="value"] attribute
     * Also sets a default property value 
     * and updates the attribute on the HTMLElement
     * @type {Object}
     */
    aProperty: {
      reflectToAttribute: true,
      value: 'value for aProperty'
    },

    /**
     * Maps to [b-property] attribute
     * It removes the attribute when the property value is `false`
     * @type {Object}
     */
    bProperty: {
      type: Boolean,
      reflectToAttribute: true
    }
  },

  /**
   * Property change observers.
   * @type {Array}
   */
  observers: [
    '_onPropertyChanged(aProperty, bProperty)'
  ],

  /**
   * Event listeners.
   * @type {Array}
   */
  listeners: [
    '_onClick(click)'
  ],

  /**
   * Property change observer handler
   * @param  {?}      newValue The new property value
   * @param  {?}      oldValue The old property value
   * @param  {String} propName The property name
   */
  _onPropertyChanged (newValue, oldValue, propName) {
    switch (propName) {
      case 'aProperty':
        console.log('aProperty has changed', newValue, oldValue)
        break
      case 'bProperty':
        console.log('bProperty has changed', newValue, oldValue)
        break
    }

    // access from context, with the new values
    console.log(this.aProperty, this.bProperty)
  },

  /**
   * Click event listener
   * @param  {MouseEvent} event The Mouse Event
   */
  _onClick (event) {
    event.preventDefault()
    console.log('The button was clicked!')
  }
})
```

#### Register the component

```js
handler.register('my-button', buttonComponent)
```

#### Initialize the component

The component handler attempts to self-initialize all registered components which match the component CSS class. The CSS class is computed automatically from the component ID which was provided at registration.

In this example, since we registered the `buttonComponent` with a component ID of `my-button`, the handler will try to initialize all the HTML elements which have the `my-js-button` CSS class.

```html
<button class="my-js-button">Press me</button>
```

#### Interact with the component API

When the handler initializes a component on a HTML element, a reference to the created component is stored as a property on the HTML element, matching the `camelCase` version of the component ID.

```js
var buttonNode = document.querySelector('button')
var button = buttonNode.myButton
```

We can use this reference to interact with the component's API.

```js
console.log(button.aProperty)
// => 'value for aProperty'
```

#### Property reflection to attribute

```js
button.aProperty = 'something else'
button.bProperty = true
```

When using the `reflectToAttribute: true` property option, the property reflects a string representation of it's value to the corresponding attribute on the HTML element, which means you can use the attribute to target styles or to configure the property value.

When using a `Boolean` property type and assigning a property value of `true`, the attribute will be created with the same value as the attribute name and when assigning a property value of `false`, the attribute will be removed from the DOM.

```html
<button class="my-js-button" a-property="something else" b-property="b-property">
  Press me
</button>
```

#### Destroy

Calling the `destroy` method on the component reference, removes observers and event listeners. Useful before removing the HTML element from the DOM, for example when using libraries like Vue.js or Angular2 where you need to hook into the application lifecycle.

```js
button.destroy()
```