# Custom Events

## Event Names

Unlike components and props, event names don’t provide any automatic case transformation. Instead, the name of an emitted event must exactly match the name used to listen to that event. For example, if emitting a camelCased event name:

```js
this.$emit('myEvent')
```

Listening to the kebab-cased version will have no effect:

```html
<!-- Won't work -->
<my-component v-on:my-event="doSomething"></my-component>
```

Unlike components and props, event names will never be used as variable or property names in JavaScript, so there’s no reason to use camelCase or PascalCase. Additionally, `v-on` event listeners inside DOM templates will be automatically transformed to lowercase (due to HTML’s case-insensitivity), so `v-on:myEvent` would become `v-on:myevent` – making `myEvent` impossible to listen to.

For these reasons, we recommend you **always use kebab-case for event names**.

### Customizing Component `v-model`

***New in 2.2.0+***

By default, `v-model` on a component uses `value` as the prop and `input` as the event, but some input types such as checkboxes and radio buttons may want to use the `value` attribute for a **different purpose**. Using the `model` option can avoid a conflict in such cases:

```js
Vue.component('base-checkbox', {
    model: {
        props: 'checked',
        event: 'change'
    },
    props: {
        checked: Boolean
    },
    template: `
        <input
            type="checkbox"
            v-bind:checked="checked"
            v-on:change="$emit('change', $event.target.checked)"
        >
    `
})
```

Now when using `v-model` on this component:

```html
<base-checkbox v-model="lovingVue"></base-checkbox>
```

the value of `lovingVue` will be passed to the `checked` prop. The `lovingVue` property will then be updated when `<base-checkbox>` emits a `change` event with a new value.

>Note that you still have to declare the `checked` prop in component’s props option.

