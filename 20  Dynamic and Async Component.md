# Dynamic & Asynac Components

This page assumes you’ve already read the Components Basics. Read that first ifyou are new to components.

## `keep-alive` with Dynamic Components

Earlier, we used the `is` attribute to switch between components in a tabbed interface:

```html
<component v-bind:is="currentTabComponent"></component>
```

When switching between these components though, you’ll sometimes want to maintain their state or avoid re-rendering for performance reasons. For example, when expanding our tabbed interface a little:

You’ll notice that if you select a post, switch to the Archive tab, then switch back to Posts, it’s no longer showing the post you selected. That’s because each time you switch to a new tab, Vue creates a new instance of the `currentTabComponent`.

Recreating dynamic components is normally useful behavior, but in this case, we’d really like those tab component instances to be cached once they’re created for the first time. To solve this problem, we can wrap our dynamic component with a `<keep-alive>` element:

```html
<!-- Inactive components will be cached! -->
<keep-alive>
  <component v-bind:is="currentTabComponent"></component>
</keep-alive>
```

Check out the result below:(<https://jsfiddle.net/chrisvfritz/Lp20op9o/>)

>Note that `<keep-alive>` requires the components being switched between to all have names, either using the name option on a component, or through local/global registration.

## Async Components

In large applications, we may need to divide the app into smaller chunks and only load a component from the server when it’s needed. To make that easier, Vue allows you to define your component as a factory function that asynchronously resolves your component definition. Vue will only trigger the factory function when the component needs to be rendered and will cache the result for future re-renders. For example:

```js
Vue.component('async-example', function (resolve, reject) {
    setTimeout(function () {
        //Pass the component definition to the resolve callback
        resolve({
            template: '<div>I am async!</div>'
        })
    }, 1000)
})
```

As you can see, the factory function receives a `resolve` callback, which should be called when you have retrieved your component definition from the server. You can also call `reject(reason)` to indicate the load has failed. The `setTimeout` here is for demonstration; how to retrieve the component is up to you. One recommended approach is to use async components together with **Webpack’s code-splitting** feature:

```js
Vue.component('async-webpack-example', function (resolve) {
  // This special require syntax will instruct Webpack to
  // automatically split your built code into bundles which
  // are loaded over Ajax requests.
  require(['./my-async-component'], resolve)
})
```

You can also return a Promise in the factory function, so with Webpack 2 and ES2015 syntax you can do:

```js
Vue.component('async-webpack-example',
    //The 'import' function returns a Promise.
    () => import('./my-async-component')
)
```

When using **local registration**, you can also directly provide a function that returns a `Promise`:

```js
new Vue({
    //...
    components: {
        'my-component': () => import('./my-async-component')
    }
})
```

>If you’re a Browserify user that would like to use async components, its creator has unfortunately made it clear that async loading “is not something that Browserify will ever support.” Officially, at least. The Browserify community has found some workarounds, which may be helpful for existing and complex applications. For all other scenarios, we recommend using Webpack for built-in, first-class async support.

### Handling Loading State

New in 2.3.0+

The async component factory can also return an object of the following format:

```js
const AsyncComponent = () => ({
    //The component to load (shoud be a Promise)
    component: import('./MyComponent.vue'),
    //A component to use while the async component is loading
    loading: LoadingComponent,
    //A component to use if the load fails
    error: ErrorComponent,
    //Delay before showing the laoding component. Default: 200ms.
    delay: 200,
    //The error component will be displayed if a timeout is 
    //provided and exceeded. Default: Infinity.
    timeout: 3000
})
```

Note that you must use Vue Router 2.4.0+ if you wish to use the above syntax for route components.
