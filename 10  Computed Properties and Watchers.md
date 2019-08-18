# Computed Properties and Watchers

## Computed Properties

In-template expressions are very convenient, but they are meant for simple operations. Putting too much logic in your templates can make them bloated and hard to maintain. For example:

```html
    <div id="example">
        {{ message.split('').reverse().join('') }}
    </div>
```

>At this point, the template is no longer simple and declarative. You have to look at it for a second before realizing that it displays **`message`** in reverse. The problem is made worse when you want to include the reversed message in your template more than once.

That’s why for any complex logic, you should use a **computed property**.

### Basic Example

```html
    <div id="example">
        <p>Original message: "{{ message }}"</p>
        <P>Computed reversed message: "{{ reversedMessage }}"</P>
    </div>
```

```js
    let vm = new Vue({
        el: '#example',
        data: {
            message: 'Hello'
        },
        computed: {
            //a computed getter
            reversedMessage: function () {
                //'this' points to the vm instance
                return this.message.split('').reverse().join('')
            }
        }
    })
```

demo-result(<https://jsfiddle.net/xiaowusang/8cLqyx5s/6/>)

Here we have declared a computed property `reversedMessage`.The function we provided
will be used as the getter function for the property `vm.reversedMessage`:

```js
    console.log(vm.reversedMessage) // => 'olleH'
    vm.message = 'Goodbye'
    console.log(vm.reversedMessage) // => 'eybdooG'
```

>You can data-bind to computed properties in templates just like a normal property. Vue is
aware that vm.reversedMessage depends on vm.message, so it will update any bindings that depend on vm.reversedMessage when vm.message changes. And the best part is
that we’ve created this dependency relationship declaratively:
the computed getter function has no side effects, which makes it easier to test and understand.

### Computed caching vs Methods

You may have noticed we can achieve the same result by invoking a method in the expression:

```html
    <P>Reserved message: "{{ reverseMessage() }}"</P>
```

```js
    methods: {
        reverseMessage: function () {
            return this.message.split('').reverse().join('')
        }
    }
```

Instead of a computed property, we can define the same function as a method. For the end result, the two approaches are indeed exactly the same. However, the difference is that computed properties are cached based on their reactive dependencies. A computed property will only re-evaluate when some of its reactive dependencies have changed. This means as long as `message` has not changed, multiple access to the `reversedMessage` computed property will immediately return the previously computed result without having to run the function again.

This also means the following computed property will never update, because Date.now() is not a reactive dependency:

```js
    computed: {
        now: function () {
            return Date.now()
        }
    }
```

In comparison, a method invocation will **always** run the function whenever a re-render happens.

>Why do we need caching? Imagine we have an expensive computed property A, which requires looping through a huge Array and doing a lot of computations. Then we may have other computed properties that in turn depend on A. Without caching, we would be executing A’s getter many more times than necessary! In cases where you do not want caching, use a method instead.

### Computed vs Watched Property

Vue does provide a more generic way to observe and react to data changes on a Vue instance: **watch properties**. When you have some data that needs to change based on some other data, it is tempting to overuse `watch` - especially if you are coming from an AngularJS background. However, it is often a better idea to use a computed property rather than an imperative `watch` callback. Consider this example:

```html
    <div id="demo">{{ fullName }}</div>
```

```js
    let vm = new Vue({
        el: '#demo',
        data: {
            firstName: 'foo',
            lastName: 'Bar',
            fullName: 'Foo Bar'
        },
        watch: {
            firstName: function (val) {
                this.fullName = val + ' ' + this.lastName
            },
            lastName: function (val) {
                this.fullName = this.firstName + ' ' + val
            }
        }
    })
```

>The above code is imperative and repetitive. Compare it with a computed property version:

```js
    let vm = new Vue({
        el: '#demo',
        data: {
            firstName: 'foo',
            lastName: 'Bar'
        },
        computed: {
            fullName: function () {
                return this.firstName + ' ' + this.lastName
            }
        }
    })
```

***Much better, isn't it?***

### Computed Setter

Computed properties are by default **getter-only**, but you can also provide a **setter** when you need it:

```js
    computed: {
        fullName: {
            //getter
            get: function () {
                return this.firstName + ' ' + this.lastName
            },
            //setter
            set: function (newValue) {
                let names = newValue.split(' ');
                this.firstName = names[0];
                this.lastName = names[names.length - 1];
            }
        }
    }
```

demo-result(<https://jsfiddle.net/xiaowusang/8cLqyx5s/8/>)

Now when you run `vm.fullName = 'John Doe'`, the setter will be invoked and `vm.firstName` and `vm.lastName` will be updated accordingly.

## Watchers

While computed properties are more appropriate in most cases, there are times when a custom watcher is necessary. That’s why Vue provides a more generic way to react to data changes through the watch option. This is most useful when you want to perform **asynchronous** or **expensive operations** in response to changing data.