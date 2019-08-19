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

