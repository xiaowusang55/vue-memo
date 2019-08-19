# List Rendering

## Mapping an Array to Elements with `v-for`

We can use the `v-for` directive to render a list of items based on an array. The `v-for` directive requires a special syntax in the form of `item in items`, where `items` is the source data array and `item` is an alias for the array element being iterated on:

```html
    <ul id="example-1">
        <li v-for="item in items">
            {{ item.message }}
        </li>
    </ul>
```

```js
    let example1 = new Vue({
        el: '#example-1',
        data: {
            items: [
                { message: 'foo' },
                { message: 'Bar' },
            ]
        }
    })
```

result: <https://jsfiddle.net/xiaowusang/9qygt0ed/>

Inside `v-for` blocks we have full access to **parent scope** properties. `v-for` also supports an optional second argument for the index of the current item.

```html
    <ul id="example-2">
        <li v-for="(item, index) in items">
            {{ parentMessage }} - {{ index }} - {{ item.message }}
        </li>
    </ul>
```

```js
    let example2 = new Vue({
        el: '#example-2',
        data: {
            parentMessage: 'Parent',
            items: [
                { message: 'Foo' },
                { message: 'Bar' }
            ]
        }
    })
```

result: <https://jsfiddle.net/xiaowusang/9qygt0ed/2/>

You can also use `of` as the delimiter instead of `in`, so that it is closer to JavaScript’s syntax for iterators:

```html
    <div v-for="item of items"></div>
```

## `v-for` with an Object

You can also use `v-for` to iterate through the properties of an object.

```html
    <ul id="v-for-object" class="demo">
        <li v-for="value in object">
            {{ value }}
        </li>
    </ul>
```

```js
    new Vue({
        el: '#v-for-object',
        data: {
            object: {
                title: 'How to do list in Vue',
                author: 'Jane Doe',
                publishedAt: '2016-04-10'
            }
        }
    })
```

result: <https://jsfiddle.net/xiaowusang/9qygt0ed/3/>

You can also provide a second argument for the property’s name (a.k.a. key):

```html
    <div v-for="(value, name) in object">
        {{ name }}: {{ value }}
    </div>
```

result: <https://jsfiddle.net/xiaowusang/9qygt0ed/7/>

And another for the index:

```html
    <div v-for="(value, name, index) in object">
        {{ index }}. {{ name }}: {{ value }}
    </div>
```

result: <https://jsfiddle.net/xiaowusang/9qygt0ed/8/>

>When iterating over an object, the order is based on the enumeration order of **Object.keys()**, which is **not** guaranteed to be consistent across JavaScript engine implementations.

## Maintaining State

When Vue is updating a list of elements rendered with `v-for`, by default it uses an “in-place patch” strategy. If the order of the data items has changed, instead of moving the DOM elements to match the order of the items, Vue will patch each element in-place and make sure it reflects what should be rendered at that particular index. This is similar to the behavior of `track-by="$index"` in Vue 1.x.

This default mode is efficient, but **only suitable when your list render output does not rely on child component state or temporary DOM state (e.g. form input values).**

To give Vue a hint so that it can track each node’s identity, and thus reuse and reorder existing elements, you need to provide a unique `key` attribute for each item:

```html
    <div
        v-for="item in items" v-bind:key="item.id"
    ></div>
```

It is recommended to provide a `key` attribute with `v-for` whenever possible, unless the iterated DOM content is simple, or you are intentionally relying on the default behavior for performance gains.

Since it’s a generic mechanism for Vue to identify nodes, the `key` also has other uses that are not specifically tied to `v-for`, as we will see later in the guide.

>Don’t use non-primitive values like objects and arrays as `v-for` keys. Use string or numeric values instead.

For detailed usage of the `key` attribute, please see the `key` **`API documentation`**.

## Array Change Detection

### Mutation Methods

Vue wraps an observed **array’s mutation methods** so they will also **trigger view updates**. The wrapped methods are:

- `push()`
- `pop()`
- `shift()`
- `unshift()`
- `splice()`
- `sort()`
- `reverse()`

You can open the console and play with the previous examples’ `items` array by calling their mutation methods. For example: `example1.items.push({ message: 'Baz' })`.

### Replacing an Array

**Mutation methods, as the name suggests, mutate the original array they are called on**. In comparison, there are also **non-mutating methods, e.g. `filter()`, `concat()` and `slice()`, which do not mutate the original array but always return a new array**. When working with non-mutating methods, you can replace the old array with the new one:

```js
    example1.items = example1.items.filter(function (item) {
        return item.message.match(/Foo/)
    })
```

You might think this will cause Vue to throw away the existing DOM and re-render the entire list - luckily, that is not the case. Vue implements some smart heuristics to maximize DOM element reuse, so replacing an array with another array containing overlapping objects is a very efficient operation.

### Caveats

Due to limitations in JavaScript, Vue **cannot** detect the following changes to an array:

1、When you directly set an item with the index, e.g. `vm.items[indexOfItem] = newValue`
2、When you modify the length of the array, e.g. `vm.items.length = newLength`

For example:

```js
    let vm = new Vue({
        data: {
            items: ['a', 'b', 'c']
        }
    })
    vm.items[1] = 'x'    // is NOT reactive
    vm.items.length = 2  // is NOT reactive
```

To overcome caveat 1, both of the following will accomplish the same as `vm.items[indexOfItem] = newValue`, but will also trigger state updates in the reactivity system:

```js
    //Vue.set
    Vue.set(vm.items, indexOfItem, newValue)
```

```js
    //Array.prototype.splice
    vm.items.splice(indexOfItem, 1, newValue)
```

You can also use the **`vm.$set`** instance method, which is an **alias** for the global **`Vue.set`**:

```js
    vm.$set(vm.items, indexOfItem, newValue)
```

To deal with caveat 2, you can use `splice`:

```js
    vm.items.splice(newLength)
```

## Object Change Detection Caveats

Again due to limitations of modern JavaScript, **Vue cannot detect property addition or deletion.** For example:

```js
    let vm = new Vue({
        data: {
            a: 1
        }
    })
    // 'vm.a' is now reactive

    vm.b = 2
    // 'vm.b' is NOT reactive
```

Vue does not allow dynamically adding new root-level reactive properties to an already created instance. However, it’s possible to add reactive properties to a nested object using the `Vue.set(object, propertyName, value)` method. For example, given:

```js
    let vm = new Vue({
        data: {
            userProfile: {
                name: 'Anika'
            }
        }
    })
```

You could add a new `age` property to the nested `userProfile` object with:

```js
    Vue.set(vm.userProfile, 'age', 27)
```

You can also use the `vm.$set` instance method, which is an **alias** for the global `Vue.set`:

```js
    vm.$set(vm.userProfile, 'age', 27)
```

Sometimes you may want to assign a number of new properties to an existing object, for example using `Object.assign()` or `_.extend()`. In such cases, you should create a fresh object with properties from both objects. So instead of:

```js
    Object.assign(vm.userProfile, {
        age: 27,
        favoriteColor: 'Vue Green'
    })
```

You would add new, reactive properties with:

```js
    vm.userProfile = Object.assign({}, vm.userProfile, {
        age: 27,
        favoriteColor: 'Vue Green'
    })
```

## Displaying filtered/Sorted Results

Sometimes we want to display a filtered or sorted version of an array without actually mutating or resetting the original data. In this case, you can create a computed property that returns the filtered or sorted array.

For example:

```html
    <li v-for="n in evenNumbers">{{ n }}</li>
```

```js
    data: {
        numbers: [1, 2, 3, 4, 5]
    },
    computed: {
        evenNumbers: function () {
            return this.numbers.filter(function (number) {
                return number % 2 === 0
            })
        }
    }
```

In situations where computed properties are not feasible (e.g. inside nested `v-for` loops), you can use a method:

```html
    <li v-for="n in even(numbers)">{{ n }}</li>
```

```js
    data: {
        numbers: [1, 2, 3, 4, 5]
    },
    methods: {
        even: function (numbers) {
            return numbers.filter(function (number) {
                return number % 2 === 0
            })
        }
    }
```

