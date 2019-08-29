# Enter/Leave & List Transitions

## Overview

Vue provides a variety of ways to apply transition effects when items are inserted, updated, or removed from the DOM. This includes tools to:

* automatically apply classes for CSS transitions and animations
* integrate 3rd-party CSS animation libraries, such as Animate.css
* use JavaScript to directly manipulate the DOM during transition hooks
* integrate 3rd-party JavaScript animation libraries, such as Velocity.js

On this page, we’ll only cover entering, leaving, and list transitions, but you can see the next section for managing state transitions.

## Transitioning Single Elements/Components

Vue provides a `transition` wrapper component, allowing you to add entering/leaving transitions for any element or component in the following contexts:

* Conditional rendering (using v-if)
* Conditional display (using v-show)
* Dynamic components
* Component root nodes

This is what an example looks like in action:

```html
<div id="demo">
  <button v-on:click="show = !show">
    Toggle
  </button>
  <transition name="fade">
    <p v-if="show">hello</p>
  </transition>
</div>
```

```js
new Vue({
  el: '#demo',
  data: {
    show: true
  }
})
```

```css
.fade-enter-active, .fade-leave-active {
  transition: opacity .5s;
}
.fade-enter, .fade-leave-to /* .fade-leave-active below version 2.1.8 */ {
  opacity: 0;
}
```

When an element wrapped in a `transition` component is inserted or removed, this is what happens:

1. Vue will automatically sniff whether the target element has CSS transitions or animations applied. If it does, CSS transition classes will be added/removed at appropriate timings.

2. If the transition component provided JavaScript hooks, these hooks will be called at appropriate timings.

3. If no CSS transitions/animations are detected and no JavaScript hooks are provided, the DOM operations for insertion and/or removal will be executed immediately on next frame (Note: this is a browser animation frame, different from Vue’s concept of `nextTick`).

### Transition Classes

There are six classes applied for enter/leave transitions.

1. `v-enter`: Starting state for enter. Added before element is inserted, removed one frame after element is inserted.

2. `v-enter-active`: Active state for enter. Applied during the entire entering phase. Added before element is inserted, removed when transition/animation finishes. This class can be used to define the duration, delay and easing curve for the entering transition.

3. `v-enter-to`: Only available in versions 2.1.8+. Ending state for enter. Added one frame after element is inserted (at the same time v-enter is removed), removed when transition/animation finishes.

4. `v-leave`: Starting state for leave. Added immediately when a leaving transition is triggered, removed after one frame.

5. `v-leave-active`: Active state for leave. Applied during the entire leaving phase. Added immediately when leave transition is triggered, removed when the transition/animation finishes. This class can be used to define the duration, delay and easing curve for the leaving transition.

6. `v-leave-to`: Only available in versions 2.1.8+. Ending state for leave. Added one frame after a leaving transition is triggered (at the same time v-leave is removed), removed when the transition/animation finishes.

Each of these classes will be prefixed with the name of the transition. Here the `v-` prefix is the default when you use a `<transition>` element with no name. If you use `<transition name="my-transition">` for example, then the `v-enter` class would instead be `my-transition-enter`.

`v-enter-active` and `v-leave-active` give you the ability to specify different easing curves for enter/leave transitions, which you’ll see an example of in the following section.

### CSS Transitions

One of the most common transition types uses CSS transitions. Here’s an example:

```html
<div id="example-1">
  <button @click="show = !show">
    Toggle render
  </button>
  <transition name="slide-fade">
    <p v-if="show">hello</p>
  </transition>
</div>
```

```js
new Vue({
  el: '#example-1',
  data: {
    show: true
  }
})
```

```css
/* Enter and leave animations can use different */
/* durations and timing functions.              */
.slide-fade-enter-active {
  transition: all .3s ease;
}
.slide-fade-leave-active {
  transition: all .8s cubic-bezier(1.0, 0.5, 0.8, 1.0);
}
.slide-fade-enter, .slide-fade-leave-to
/* .slide-fade-leave-active below version 2.1.8 */ {
  transform: translateX(10px);
  opacity: 0;
}
```

### CSS Animations

CSS animations are applied in the same way as CSS transitions, the difference being that `v-enter` is not removed immediately after the element is inserted, but on an `animationend` event.

Here’s an example, omitting prefixed CSS rules for the sake of brevity:

```html
<div id="example-2">
  <button @click="show = !show">Toggle show</button>
  <transition name="bounce">
    <p v-if="show">Lorem ipsum dolor sit amet, consectetur adipiscing elit. Mauris facilisis enim libero, at lacinia diam fermentum id. Pellentesque habitant morbi tristique senectus et netus.</p>
  </transition>
</div>
```

```js
new Vue({
  el: '#example-2',
  data: {
    show: true
  }
})
```

```css
.bounce-enter-active {
  animation: bounce-in .5s;
}
.bounce-leave-active {
  animation: bounce-in .5s reverse;
}
@keyframes bounce-in {
  0% {
    transform: scale(0);
  }
  50% {
    transform: scale(1.5);
  }
  100% {
    transform: scale(1);
  }
}
```

### Custom Transition Classes

You can also specify custom transition classes by providing the following attributes:

* `enter-class`
* `enter-active-class`
* `enter-to-class` (2.1.8+)
* `leave-class`
* `leave-active-class`
* `leave-to-class` (2.1.8+)

These will override the conventional class names. This is especially useful when you want to combine Vue’s transition system with an existing CSS animation library, such as **Animate.css**.

Here’s an example:

```html
<link href="https://cdn.jsdelivr.net/npm/animate.css@3.5.1" rel="stylesheet" type="text/css">

<div id="example-3">
  <button @click="show = !show">
    Toggle render
  </button>
  <transition
    name="custom-classes-transition"
    enter-active-class="animated tada"
    leave-active-class="animated bounceOutRight"
  >
    <p v-if="show">hello</p>
  </transition>
</div>
```

```js
new Vue({
  el: '#example-3',
  data: {
    show: true
  }
})
```

### Using Transitions and Animations Together

Vue needs to attach event listeners in order to know when a transition has ended. It can either be `transitionend` or `animationend`, depending on the type of CSS rules applied. If you are only using one or the other, Vue can automatically detect the correct type.

However, in some cases you may want to have both on the same element, for example having a CSS animation triggered by Vue, along with a CSS transition effect on hover. In these cases, you will have to explicitly declare the type you want Vue to care about in a `type` attribute, with a value of either `animation` or `transition`.

### Explicit Transition Durations

New in 2.2.0+

In most cases, Vue can automatically figure out when the transition has finished. By default, Vue waits for the first `transitionend` or `animationend` event on the root transition element. However, this may not always be desired - for example, we may have a choreographed transition sequence where some nested inner elements have a delayed transition or a longer transition `duration` than the root `transition` element.

```html
<transition :duration="1000">...</transition>
```

You can also specify separate values for enter and leave durations:

```html
<transition :duration="{ enter: 500, leave: 800 }">...</transition>
```

### JavaScript Hooks

You can also define JavaScript hooks in attributes:

```html
<transition
  v-on:before-enter="beforeEnter"
  v-on:enter="enter"
  v-on:after-enter="afterEnter"
  v-on:enter-cancelled="enterCancelled"

  v-on:before-leave="beforeLeave"
  v-on:leave="leave"
  v-on:after-leave="afterLeave"
  v-on:leave-cancelled="leaveCancelled"
>
  <!-- ... -->
</transition>
```

```js
// ...
methods: {
  // --------
  // ENTERING
  // --------

  beforeEnter: function (el) {
    // ...
  },
  // the done callback is optional when
  // used in combination with CSS
  enter: function (el, done) {
    // ...
    done()
  },
  afterEnter: function (el) {
    // ...
  },
  enterCancelled: function (el) {
    // ...
  },

  // --------
  // LEAVING
  // --------

  beforeLeave: function (el) {
    // ...
  },
  // the done callback is optional when
  // used in combination with CSS
  leave: function (el, done) {
    // ...
    done()
  },
  afterLeave: function (el) {
    // ...
  },
  // leaveCancelled only available with v-show
  leaveCancelled: function (el) {
    // ...
  }
}
```

These hooks can be used in combination with CSS transitions/animations or on their own.

>When using JavaScript-only transitions, **the `done` callbacks are required for the `enter` and `leave` hooks**. Otherwise, the hooks will be called synchronously and the transition will finish immediately.
>It’s also a good idea to explicitly add `v-bind:css="false"` for JavaScript-only transitions so that Vue can skip the CSS detection. This also prevents CSS rules from accidentally interfering with the transition.

Now let’s dive into an example. Here’s a JavaScript transition using Velocity.js:

```html
<!--
Velocity works very much like jQuery.animate and is
a great option for JavaScript animations
-->
<script src="https://cdnjs.cloudflare.com/ajax/libs/velocity/1.2.3/velocity.min.js"></script>

<div id="example-4">
  <button @click="show = !show">
    Toggle
  </button>
  <transition
    v-on:before-enter="beforeEnter"
    v-on:enter="enter"
    v-on:leave="leave"
    v-bind:css="false"
  >
    <p v-if="show">
      Demo
    </p>
  </transition>
</div>
```

```js
new Vue({
  el: '#example-4',
  data: {
    show: false
  },
  methods: {
    beforeEnter: function (el) {
      el.style.opacity = 0
    },
    enter: function (el, done) {
      Velocity(el, { opacity: 1, fontSize: '1.4em' }, { duration: 300 })
      Velocity(el, { fontSize: '1em' }, { complete: done })
    },
    leave: function (el, done) {
      Velocity(el, { translateX: '15px', rotateZ: '50deg' }, { duration: 600 })
      Velocity(el, { rotateZ: '100deg' }, { loop: 2 })
      Velocity(el, {
        rotateZ: '45deg',
        translateY: '30px',
        translateX: '30px',
        opacity: 0
      }, { complete: done })
    }
  }
})
```

## Transitions on Initial Render

If you also want to apply a transition on the initial render of a node, you can add the `appear` attribute:

```html
<transition appear>
  <!-- ... -->
</transition>
```

By default, this will use the transitions specified for entering and leaving. If you’d like however, you can also specify custom CSS classes:

```html
<transition
  appear
  appear-class="custom-appear-class"
  appear-to-class="custom-appear-to-class" (2.1.8+)
  appear-active-class="custom-appear-active-class"
>
  <!-- ... -->
</transition>
```

and custom JavaScript hooks:

```html
<transition
  appear
  v-on:before-appear="customBeforeAppearHook"
  v-on:appear="customAppearHook"
  v-on:after-appear="customAfterAppearHook"
  v-on:appear-cancelled="customAppearCancelledHook"
>
  <!-- ... -->
</transition>
```

In the example above, either `appear` attribute or `v-on:appear` hook will cause an appear transition.

## Transitioning Between Elements

We discuss transitioning between components later, but you can also transition between raw elements using `v-if`/`v-else`. One of the most common two-element transitions is between a list container and a message describing an empty list:

```html
<transition>
  <table v-if="items.length > 0">
    <!-- ... -->
  </table>
  <p v-else>Sorry, no items found.</p>
</transition>
```

This works well, but there’s one caveat to be aware of:

When toggling between elements that have **the same tag name**, you must tell Vue that they are distinct elements by giving them unique `key` attributes. Otherwise, Vue’s compiler will only replace the content of the element for efficiency. Even when technically unnecessary though, **it’s considered good practice to always key multiple items within a `<transition>` component**.

For example:

```html
<transition>
  <button v-if="isEditing" key="save">
    Save
  </button>
  <button v-else key="edit">
    Edit
  </button>
</transition>
```

In these cases, you can also use the `key` attribute to transition between different states of the same element. Instead of using `v-if` and `v-else`, the above example could be rewritten as:

```html
<transition>
  <button v-bind:key="isEditing">
    {{ isEditing ? 'Save' : 'Edit' }}
  </button>
</transition>
```

It’s actually possible to transition between any number of elements, either by using multiple `v-if`s or binding a single element to a dynamic property. For example:

```html
<transition>
  <button v-if="docState === 'saved'" key="saved">
    Edit
  </button>
  <button v-if="docState === 'edited'" key="edited">
    Save
  </button>
  <button v-if="docState === 'editing'" key="editing">
    Cancel
  </button>
</transition>
```

Which could also be written as:

```html
<transition>
  <button v-bind:key="docState">
    {{ buttonMessage }}
  </button>
</transition>
```

```js
// ...
computed: {
  buttonMessage: function () {
    switch (this.docState) {
      case 'saved': return 'Edit'
      case 'edited': return 'Save'
      case 'editing': return 'Cancel'
    }
  }
}
```

### Transition Modes

