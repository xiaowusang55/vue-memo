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

* `update`: called after the containing component’s VNode has updated, but possibly before its children have updated. The directive’s value may or may not have changed, but you can skip unnecessary updates by comparing the binding’s current and old values (see below on hook arguments).

>We’ll cover VNodes in more detail later, when we discuss render functions.

* `componentUpdated`: called after the containing component’s VNode and **the VNodes of its children** have updated.

* `unbind`: called only once, when the directive is unbound from the element.

We’ll explore the arguments passed into these hooks (i.e. `el`, `binding`, `vnode`, and `oldVnode`) in the next section.

