# Form Input Bindings

## Basic Usage

You can use the `v-model` directive to create two-way data bindings on form input, textarea, and select elements. It automatically picks the correct way to update the element based on the input type. Although a bit magical, `v-model` is essentially syntax sugar for updating data on user input events, plus special care for some edge cases.

>v`-model` will ignore the initial `value`, `checked` or `selected` attributes found on any form elements. It will always treat the Vue instance data as the source of truth. You should declare the initial value on the JavaScript side, inside the `data` option of your component.

`v-model` internally uses different properties and emits different events for different input elements:

- text and textarea elements use `value` property and `input` event;
- checkboxes and radiobuttons use `checked` property and `change` event;
- select fields use `value` as a prop and `change` as an event.

>tFor languages that require an IME (Chinese, Japanese, Korean etc.), you’ll notice that `v-model` doesn’t get updated during IME composition. If you want to cater for these updates as well, use `input` event instead.

### Text

```html
<input v-model="message" placeholder="edit me">
<p>Message is: {{ message }}</p>
```

result: <https://jsfiddle.net/xiaowusang/9qygt0ed/32/>

### Multiline text

```html
<span>Multiline message is:</span>
<p style="white-space: pre-line;">{{ message }}</p>
<br>
<textarea v-model="message" placeholder="add multiple lines"></textarea>
```

result: <https://jsfiddle.net/xiaowusang/9qygt0ed/33/>

>Interpolation on textareas (`<textarea>{{text}}</textarea>`) won't work. Use v-model instead.

### Checkbox

Single checkbox, boolean value:

```html
<input type="checkbox" id="checkbox" v-model="checked">
<label for="checkbox">{{ checked }}</label>
```

result: <https://jsfiddle.net/xiaowusang/9qygt0ed/37/>

Multiple checkboxes, bound to the same Array:

```html
<div id='example-3'>
  <input type="checkbox" id="jack" value="Jack" v-model="checkedNames">
  <label for="jack">Jack</label>
  <input type="checkbox" id="john" value="John" v-model="checkedNames">
  <label for="john">John</label>
  <input type="checkbox" id="mike" value="Mike" v-model="checkedNames">
  <label for="mike">Mike</label>
  <br>
  <span>Checked names: {{ checkedNames }}</span>
</div>
```

```js
new Vue({
  el: '#example-3',
  data: {
    checkedNames: []
  }
})
```

result: <https://jsfiddle.net/xiaowusang/9qygt0ed/39/>

### Radio

```html
<input type="radio" id="one" value="One" v-model="picked">
<label for="one">One</label>
<br>
<input type="radio" id="two" value="Two" v-model="picked">
<label for="two">Two</label>
<br>
<span>Picked: {{ picked }}</span>
```

result: <https://jsfiddle.net/xiaowusang/9qygt0ed/40/>

### Select

Single select:

```html
<select v-model="selected">
  <option disabled value="">Please select one</option>
  <option>A</option>
  <option>B</option>
  <option>C</option>
</select>
<span>Selected: {{ selected }}</span>
```

```js
new Vue({
  el: '...',
  data: {
    selected: ''
  }
})
```

result: <https://jsfiddle.net/xiaowusang/9qygt0ed/43/>

>If the initial value of your `v-model` expression does not match any of the options, the `<select>`element will render in an “unselected” state. On iOS this will cause the user not being able to select the first item because iOS does not fire a change event in this case. It is therefore recommended to provide a disabled option with an empty value, as demonstrated in the example above.

```html
<select v-model="selected" multiple>
  <option>A</option>
  <option>B</option>
  <option>C</option>
</select>
<br>
<span>Selected: {{ selected }}</span>
```

result: <https://jsfiddle.net/xiaowusang/9qygt0ed/45/>

Dynamic options rendered with `v-for`:

```html
<select v-model="selected">
  <option v-for="option in options" v-bind:value="option.value">
    {{ option.text }}
  </option>
</select>
<span>Selected: {{ selected }}</span>
```

```js
new Vue({
  el: '...',
  data: {
    selected: 'A',
    options: [
      { text: 'One', value: 'A' },
      { text: 'Two', value: 'B' },
      { text: 'Three', value: 'C' }
    ]
  }
})
```

result: <https://jsfiddle.net/xiaowusang/9qygt0ed/47/>

## Value Bindings

For radio, checkbox and select options, the `v-model` binding values are usually static strings (or booleans for checkbox):

```html
<!-- `picked` is a string "a" when checked -->
<input type="radio" v-model="picked" value="a">

<!-- `toggle` is either true or false -->
<input type="checkbox" v-model="toggle">

<!-- `selected` is a string "abc" when the first option is selected -->
<select v-model="selected">
  <option value="abc">ABC</option>
</select>
```

But sometimes we may want to bind the value to a dynamic property on the Vue instance. We can use `v-bind` to achieve that. In addition, using `v-bind` allows us to bind the input value to non-string values.

### Checkbox

```html
<input
  type="checkbox"
  v-model="toggle"
  true-value="yes"
  false-value="no"
>
```

```js
// when checked:
vm.toggle === 'yes'
// when unchecked:
vm.toggle === 'no'
```

>The `true-value` and `false-value` attributes don’t affect the input’s `value` attribute, because browsers don’t include unchecked boxes in form submissions. To guarantee that one of two values is submitted in a form (e.g. “yes” or “no”), use radio inputs instead.

### Radio

```html
<input type="radio" v-model="pick" v-bind:value="a">
```

```js
// when checked:
vm.pick === vm.a
```

### Select Options

```html
<select v-model="selected">
  <!-- inline object literal -->
  <option v-bind:value="{ number: 123 }">123</option>
</select>
```

```js
// when selected:
typeof vm.selected // => 'object'
vm.selected.number // => 123
```

## Modifiers

### `.lazy`

By default, `v-model` syncs the input with the data after each `input` event (with the exception of IME composition as stated above). You can add the `lazy` modifier to instead sync after `change` events:

```html
<!-- synced after "change" instead of "input" -->
<input v-model.lazy="msg">
```

results: <https://jsfiddle.net/xiaowusang/325px9qj/1/>

### `.number`

If you want user input to be automatically typecast as a number, you can add the `number` modifier to your `v-model` managed inputs:

```html
<input v-model.number="age" type="number">
```

results: <https://jsfiddle.net/xiaowusang/325px9qj/2/>

This is often useful, because even with `type="number"`, the value of HTML input elements always returns a string. If the value cannot be parsed with `parseFloat()`, then the original value is returned.

### `.trim`

If you want whitespace from user input to be trimmed automatically, you can add the `trim` modifier to your `v-model`-managed inputs:

```html
<input v-model.trim="msg">
```

results: <https://jsfiddle.net/xiaowusang/325px9qj/1/>

## `v-model` with Components

>HTML’s built-in input types won’t always meet your needs. Fortunately, Vue components allow you to build reusable inputs with completely customized behavior. These inputs even work with `v-model`! To learn more, read about custom inputs in the Components guide.
