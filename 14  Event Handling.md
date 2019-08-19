# Event Handling

We can use the `v-on` directive to listen to DOM events and run some JavaScript when they’re triggered.

```html
<div id="example-1">
    <button v-on:click="counter += 1">Add 1</button>
    <p>The button above has been clicked {{ counter }} times.</p>
</div>
```

```js
var example1 = new Vue({
    el: '#example-1',
    data: {
        counter: 0
    }
})
```

result: <https://jsfiddle.net/xiaowusang/9qygt0ed/25/>

## Method Event Handlers

The logic for many event handlers will be more complex though, so keeping your JavaScript in the value of the `v-on` attribute isn’t feasible. That’s why `v-on` can also accept the name of a method you’d like to call.

```html
<div id="example-2">
  <!-- `greet` is the name of a method defined below -->
  <button v-on:click="greet">Greet</button>
</div>
```

```js
var example2 = new Vue({
  el: '#example-2',
  data: {
    name: 'Vue.js'
  },
  // define methods under the `methods` object
  methods: {
    greet: function (event) {
      // `this` inside methods points to the Vue instance
      alert('Hello ' + this.name + '!')
      // `event` is the native DOM event
      if (event) {
        alert(event.target.tagName)
      }
    }
  }
})

// you can invoke methods in JavaScript too
example2.greet() // => 'Hello Vue.js!'
```

result: <https://jsfiddle.net/xiaowusang/9qygt0ed/26/>

## Methods in Inline Handlers

Instead of binding directly to a method name, we can also use methods in an inline JavaScript statement:

```html
<div id="example-3">
  <button v-on:click="say('hi')">Say hi</button>
  <button v-on:click="say('what')">Say what</button>
</div>
```

```js
new Vue({
  el: '#example-3',
  methods: {
    say: function (message) {
      alert(message)
    }
  }
})
```

result: <https://jsfiddle.net/xiaowusang/9qygt0ed/28/>

Sometimes we also need to access the original DOM event in an inline statement handler. You can pass it into a method using the special `$event` variable:

```html
<button v-on:click="warn('Form cannot be submitted yet.', $event)">
  Submit
</button>
```

```js
// ...
methods: {
  warn: function (message, event) {
    // now we have access to the native event
    if (event) event.preventDefault()
    alert(message)
  }
}
```

result: <https://jsfiddle.net/xiaowusang/9qygt0ed/29/>

## Event Modifiers

It is a very common need to call `event.preventDefault()` or `event.stopPropagation()` inside event handlers. Although we can do this easily inside methods, it would be better if the methods can be purely about data logic rather than having to deal with DOM event details.

To address this problem, Vue provides event modifiers for `v-on`. Recall that modifiers are directive postfixes denoted by a dot.

- `.stop`
- `.prevent`
- `.capture`
- `.self`
- `.once`
- `.passive`

```html
<!-- the click event's propagation will be stopped -->
<a v-on:click.stop="doThis"></a>

<!-- the submit event will no longer reload the page -->
<form v-on:submit.prevent="onSubmit"></form>

<!-- modifiers can be chained -->
<a v-on:click.stop.prevent="doThat"></a>

<!-- just the modifier -->
<form v-on:submit.prevent></form>

<!-- use capture mode when adding the event listener -->
<!-- i.e. an event targeting an inner element is handled here before being handled by that element -->
<div v-on:click.capture="doThis">...</div>

<!-- only trigger handler if event.target is the element itself -->
<!-- i.e. not from a child element -->
<div v-on:click.self="doThat">...</div>
```

>Order matters when using modifiers because the relevant code is generated in the same order. Therefore using `v-on:click.prevent.self` will prevent all clicks while `v-on:click.self.prevent` will only prevent clicks on the element itself.

***New in 2.1.4+***

```html
<!-- the click event will be triggered at most once -->
<a v-on:click.once="doThis"></a>
```

Unlike the other modifiers, which are exclusive to native DOM events, the .once modifier can also be used on component events. If you haven’t read about components yet, don’t worry about this for now.

***New in 2.3.0+***

Vue also offers the `.passive` modifier, corresponding to a`ddEventListener‘s passive` option.

```html
<div v-on:scroll.passive="onScroll">...</div>
```

## Key Modifiers

When listening for keyboard events, we often need to check for specific keys. Vue allows adding key modifiers for `v-on` when listening for key events:

```html
<!-- only call `vm.submit()` when the `key` is `Enter` -->
<input v-on:keyup.enter="submit">
```

You can directly use any valid key names exposed via `KeyboardEvent.key` as modifiers by converting them to kebab-case.

```html
<input v-on:keyup.page-down="onPageDown">
```

In the above example, the handler will only be called if `$event.key` is equal to '`PageDown`'.

### Key Codes

>The use of `keyCode` events is deprecated and may not be supported in new browsers.

Using keyCode attributes is also permitted:

```html
<input v-on:keyup.13="submit">
```

- `.enter`
- `.tab`
- `.delete (captures both “Delete” and “Backspace” keys)`
- `.esc`
- `.space`
- `.up`
- `.down`
- `.left`
- `.right`

>A few keys (.`esc` and all arrow keys) have inconsistent `key` values in IE9, so these built-in aliases should be preferred if you need to support IE9.

You can also d`efine custom key modifier aliases` via the global `config.keyCodes` object:

```js
// enable `v-on:keyup.f1`
Vue.config.keyCodes.f1 = 112
```

## System Modifier Keys

***New in 2.1.0+***

You can use the following modifiers to trigger mouse or keyboard event listeners only when the corresponding modifier key is pressed:

- `.ctrl`
- `.alt`
- `.shift`
- `.meta`

Note: On Macintosh keyboards, meta is the command key (⌘). On Windows keyboards, meta is the Windows key (⊞). On Sun Microsystems keyboards, meta is marked as a solid diamond (◆). On certain keyboards, specifically MIT and Lisp machine keyboards and successors, such as the Knight keyboard, space-cadet keyboard, meta is labeled “META”. On Symbolics keyboards, meta is labeled “META” or “Meta”.

For example:

```html
<!-- Alt + C -->
<input @keyup.alt.67="clear">

<!-- Ctrl + Click -->
<div @click.ctrl="doSomething">Do something</div>
```

>Note that modifier keys are different from regular keys and when used with keyup events, they have to be pressed when the event is emitted. In other words, keyup.ctrl will only trigger if you release a key while holding down ctrl. It won’t trigger if you release the ctrl key alone. If you do want such behaviour, use the keyCode for ctrl instead: keyup.17.

### .exact Modifier

***New in 2.5.0+***

The `.exact` modifier allows control of the exact combination of system modifiers needed to trigger an event.

```html
<!-- this will fire even if Alt or Shift is also pressed -->
<button @click.ctrl="onClick">A</button>

<!-- this will only fire when Ctrl and no other keys are pressed -->
<button @click.ctrl.exact="onCtrlClick">A</button>

<!-- this will only fire when no system modifiers are pressed -->
<button @click.exact="onClick">A</button>
```

### Mouse Button Modifiers

***New in 2.2.0+***

- `.left`
- `.right`
- `.middle`

These modifiers restrict the handler to events triggered by a specific mouse button.

## Why Listeners in HTML?

You might be concerned that this whole event listening approach violates the good old rules about “separation of concerns”. Rest assured - since all Vue handler functions and expressions are strictly bound to the ViewModel that’s handling the current view, it won’t cause any maintenance difficulty. In fact, there are several benefits in using `v-on`:

1、It’s easier to locate the handler function implementations within your JS code by skimming the HTML template.

2、Since you don’t have to manually attach event listeners in JS, your ViewModel code can be pure logic and DOM-free. This makes it easier to test.

3、When a ViewModel is destroyed, all event listeners are automatically removed. You don’t need to worry about cleaning it up yourself.

