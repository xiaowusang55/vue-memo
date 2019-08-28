# Slot

This page assumes you’ve already read the Components Basics. Read that first if you arenew to components.

In 2.6.0, we introduced a new unified syntax (the v-slot directive) for named and scopedslots. It replaces the slot and slot-scope attributes, which are now deprecated, but havenot been removed and are still documented here. The rationale for introducing the newsyntax is described in this RFC.

## Slot Content

Vue implements a content distribution API inspired by the Web Components spec draft, using the `<slot>` element to serve as distribution outlets for content.

This allows you to compose components like this:

```html
<navigation-link url="/profile">
  Your Profile
</navigation-link>
```

Then in the template for `<navigation-link>`, you might have:

```html
<a
  v-bind:href="url"
  class="nav-link"
>
  <slot></slot>
</a>
```

When the component renders, `<slot></slot>` will be replaced by “Your Profile”. Slots can contain any template code, including HTML:

```html
<navigation-link url="/profile">
  <!-- Add a Font Awesome icon -->
  <span class="fa fa-user"></span>
  Your Profile
</navigation-link>
```

Or even other components:

```html
<navigation-link url="/profile">
  <!-- Use a component to add an icon -->
  <font-awesome-icon name="user"></font-awesome-icon>
  Your Profile
</navigation-link>
```

If `<navigation-link>`‘s template did not contain a `<slot>` element, any content provided between its opening and closing tag would be discarded.

## Compilation Scope

When you want to use data inside a slot, such as in:

```html
<navigation-link url="/profile">
  Logged in as {{ user.name }}
</navigation-link>
```

That slot has access to the same instance properties (i.e. the same “scope”) as the rest of the template. The slot does **not** have access to `<navigation-link>`‘s scope. For example, trying to access `url` would not work:

```html
<navigation-link url="/profile">
  Clicking here will send you to: {{ url }}
  <!--
  The `url` will be undefined, because this content is passed
  _to_ <navigation-link>, rather than defined _inside_ the
  <navigation-link> component.
  -->
</navigation-link>
```

As a rule, remember that:

**Everything in the parent template is compiled in parent scope**
**everything in the child template is compiled in the child scope.**

## Fallback Content

There are cases when it’s useful to specify fallback (i.e. default) content for a slot, to be rendered only when no content is provided. For example, in a `<submit-button>` component:

```html
<button type="submit">
  <slot></slot>
</button>
```

We might want the text “Submit” to be rendered inside the `<button>` most of the time. To make “Submit” the fallback content, we can place it in between the `<slot>` tags:

```html
<button type="submit">
  <slot>Submit</slot>
</button>
```

Now when we use `<submit-button>` in a parent component, providing no content for the slot:

```html
<submit-button></submit-button>
```

will render the fallback content, “Submit”:

```html
<button type="submit">
  Submit
</button>
```

But if we provide content:

```html
<submit-button>
  Save
</submit-button>
```

Then the provided content will be rendered instead:

```html
<button type="submit">
  Save
</button>
```

## Named Slots

Updated in 2.6.0+. See here for the deprecated syntax using the `slot` attribute.

There are times when it’s useful to have multiple slots. For example, in a `<base-layout>` component with the following template:

```html
<div class="container">
  <header>
    <!-- We want header content here -->
  </header>
  <main>
    <!-- We want main content here -->
  </main>
  <footer>
    <!-- We want footer content here -->
  </footer>
</div>
```

For these cases, the `<slot>` element has a special attribute, `name`, which can be used to define additional slots:

```html
<div class="container">
    <header>
        <slot name="header"></slot>
    </header>
    <main>
        <slot></slot>
    </main>
    <footer>
        <slot name="footer"></slot>
    </footer>
</div>
```

A `<slot>` outlet without name implicitly has the name “default”.

To provide content to named slots, we can use the `v-slot` directive on a `<template>`, providing the name of the slot as `v-slot`‘s argument:

```html
<base-layout>
    <template v-slot:header>
        <h1>Here might be a page title</h1>
    </template>

    <p>A paragraph for the main content.</p>
    <p>And another one.</p>

    <template v-slot:footer>
        <p>Here's some contact info</p>
    </template>
</base-layout>
```

Now everything inside the `<template>` elements will be passed to the corresponding slots. Any content not wrapped in a `<template>` using v-slot is assumed to be for the default slot.

However, you can still wrap default slot content in a `<template>` if you wish to be explicit:

```html
<base-layout>
  <template v-slot:header>
    <h1>Here might be a page title</h1>
  </template>

  <template v-slot:default>
    <p>A paragraph for the main content.</p>
    <p>And another one.</p>
  </template>

  <template v-slot:footer>
    <p>Here's some contact info</p>
  </template>
</base-layout>
```

Either way, the rendered HTML will be:

```html
<div class="container">
  <header>
    <h1>Here might be a page title</h1>
  </header>
  <main>
    <p>A paragraph for the main content.</p>
    <p>And another one.</p>
  </main>
  <footer>
    <p>Here's some contact info</p>
  </footer>
</div>
```

Note that v-slot can only be added to a `<template>` (with one exception), unlike the deprecated slot attribute.

## Scoped Slots

Updated in 2.6.0+. See here for the deprecated syntax using the slot-scope attribute.

Sometimes, it’s useful for slot content to have access to data only available in the child component. For example, imagine a `<current-user>` component with the following template:

```html
<span>
  <slot>{{ user.lastName }}</slot>
</span>
```

We might want to replace this fallback content to display the user’s first name, instead of last, like this:

```html
<current-user>
  {{ user.firstName }}
</current-user>
```

That won’t work, however, because only the `<current-user>` component has access to the `user` and the content we’re providing is rendered in the parent.

To make `user` available to the slot content in the parent, we can bind `user` as an attribute to the `<slot>` element:

```html
<span>
  <slot v-bind:user="user">
    {{ user.lastName }}
  </slot>
</span>
```

Attributes bound to a `<slot>` element are called **slot props**. Now, in the parent scope, we can use `v-slot` with a value to define a name for the slot props we’ve been provided:

```html
<current-user>
  <template v-slot:default="slotProps">
    {{ slotProps.user.firstName }}
  </template>
</current-user>
```

In this example, we’ve chosen to name the object containing all our slot props `slotProps`, but you can use any name you like.

## Abbreviated Syntax for Lone Default Slots

