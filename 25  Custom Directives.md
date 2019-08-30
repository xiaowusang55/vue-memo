# Custom Directives

## Intro

In addition to the default set of directives shipped in core (`v-model` and `v-show`), Vue also allows you to register your own custom directives. Note that in Vue 2.0, the primary form of code reuse and abstraction is components - however there may be cases where you need some low-level DOM access on plain elements, and this is where custom directives would still be useful. An example would be focusing on an input element, like this one:

When the page loads, that element gains focus (note: `autofocus` doesn’t work on mobile Safari). In fact, if you haven’t clicked on anything else since visiting this page, the input above should be focused now. Now let’s build the directive that accomplishes this:

```js
// Register a global custom directive called `v-focus`
Vue.directive('focus', {
  // When the bound element is inserted into the DOM...
  inserted: function (el) {
    // Focus the element
    el.focus()
  }
})
```

If you want to register a directive locally instead, components also accept a `directives` option:

```js
directives: {
  focus: {
    // directive definition
    inserted: function (el) {
      el.focus()
    }
  }
}
```

Then in a template, you can use the new v-focus attribute on any element, like this:

```html
<input v-focus>
```

## Hook Functions

A directive definition object can provide several hook functions (all optional):

* `bind`: called only once, when the directive is first bound to the element. This is where you can do one-time setup work.

* `inserted`: called when the bound element has been inserted into its parent node (this only guarantees parent node presence, not necessarily in-document).

* `update`: called after the containing component’s VNode has updated, **but possibly before its children have updated.** The directive’s value may or may not have changed, but you can skip unnecessary updates by comparing the binding’s current and old values (see below on hook arguments).

>We’ll cover VNodes in more detail later, when we discuss render functions.

* `componentUpdated`: called after the containing component’s VNode and **the VNodes of its children** have updated.

* `unbind`: called only once, when the directive is unbound from the element.

We’ll explore the arguments passed into these hooks (i.e. `el`, `binding`, `vnode`, and `oldVnode`) in the next section.

## Directive Hook Arguments

Directive hooks are passed these arguments:

* el: The element the directive is bound to. This can be used to directly manipulate the DOM.
* binding: An object containing the following properties.
---name: The name of the directive, without the v- prefix.

---value: The value passed to the directive. For example in v-my-directive="1 + 1", the value would be 2.

---oldValue: The previous value, only available in update and componentUpdated. It is available whether or not the value has changed.

---expression: The expression of the binding as a string. For example in v-my-directive="1 + 1", the expression would be "1 + 1".

---arg: The argument passed to the directive, if any. For example in v-my-directive:foo, the arg would be "foo".

---modifiers: An object containing modifiers, if any. For example in v-my-directive.foo.bar, the modifiers object would be { foo: true, bar: true }.

* vnode: The virtual node produced by Vue’s compiler. See the VNode API for full details.

* oldVnode: The previous virtual node, only available in the update and componentUpdated hooks.

>Apart from el, you should treat these arguments as read-only and never modify them. If you need to share information across hooks, it is recommended to do so through element’s dataset.

An example of a custom directive using some of these properties:

```html
<div id="hook-arguments-example" v-demo:foo.a.b="message"></div>
```

```js
Vue.directive('demo', {
  bind: function (el, binding, vnode) {
    var s = JSON.stringify
    el.innerHTML =
      'name: '       + s(binding.name) + '<br>' +
      'value: '      + s(binding.value) + '<br>' +
      'expression: ' + s(binding.expression) + '<br>' +
      'argument: '   + s(binding.arg) + '<br>' +
      'modifiers: '  + s(binding.modifiers) + '<br>' +
      'vnode keys: ' + Object.keys(vnode).join(', ')
  }
})

new Vue({
  el: '#hook-arguments-example',
  data: {
    message: 'hello!'
  }
})
```

### Dynamic Directive Arguments

Directive arguments can be dynamic. For example, in `v-mydirective:[argument]="value"`, the `argument` can be updated based on data properties in our component instance! This makes our custom directives flexible for use throughout our application.

Let’s say you want to make a custom directive that allows you to pin elements to your page using fixed positioning. We could create a custom directive where the value updates the vertical positioning in pixels, like this:

```html
<div id="baseexample">
  <p>Scroll down the page</p>
  <p v-pin="200">Stick me 200px from the top of the page</p>
</div>
```

```js
Vue.directive('pin', {
  bind: function (el, binding, vnode) {
    el.style.position = 'fixed'
    el.style.top = binding.value + 'px'
  }
})

new Vue({
  el: '#baseexample'
})
```

This would pin the element 200px from the top of the page. But what happens if we run into a scenario when we need to pin the element from the left, instead of the top? Here’s where a dynamic argument that can be updated per component instance comes in very handy:

```html
<div id="dynamicexample">
  <h3>Scroll down inside this section ↓</h3>
  <p v-pin:[direction]="200">I am pinned onto the page at 200px to the left.</p>
</div>
```

```js
Vue.directive('pin', {
  bind: function (el, binding, vnode) {
    el.style.position = 'fixed'
    var s = (binding.arg == 'left' ? 'left' : 'top')
    el.style[s] = binding.value + 'px'
  }
})

new Vue({
  el: '#dynamicexample',
  data: function () {
    return {
      direction: 'left'
    }
  }
})
```

## Function Shorthand

In many cases, you may want the same behavior on bind and update, but don’t care about the other hooks. For example:

```js
Vue.directive('color-swatch', function (el, binding) {
  el.style.backgroundColor = binding.value
})
```

## Object Literals

If your directive needs multiple values, you can also pass in a JavaScript object literal. Remember, directives can take any valid JavaScript expression.

```html
<div v-demo="{ color: 'white', text: 'hello!' }"></div>
```

```js
Vue.directive('demo', function (el, binding) {
  console.log(binding.value.color) // => "white"
  console.log(binding.value.text)  // => "hello!"
})
```

