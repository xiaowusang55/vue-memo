# Components Basics

## Base Example

Here’s an example of a Vue component:

```js
    Vue.component("button-counter", {
        data: () {
            return {
                count: 0
            }
        },
        template: `
            <button
                v-on:click="count++"
            >You clicked me {{ count }} times.</button>
        `
    })

    new Vue({
        el: '#components-demo'
    })
```

Components are reusable Vue instances with a name: in this case, `<button-counter>`. We can use this component as a custom element inside a root Vue instance created with `new Vue`:

```html
    <div id="components-demo">
        <button-counter></button-counter>
    </div>
```

Since components are reusable Vue instances, they accept the same options as `new Vue`, such as `data`, `computed`, `watch`, `methods`, and lifecycle hooks. The only exceptions are a few root-specific options like `el`.

## Reusing Components

Components can be reused as many times as you want:

```html
<div id="components-demo">
  <button-counter></button-counter>
  <button-counter></button-counter>
  <button-counter></button-counter>
</div>
```

Notice that when clicking on the buttons, each one maintains its own, separate `count`. That’s because each time you use a component, a new **instance** of it is created.

### `data` Must Be a Function

When we defined the `<button-counter>` component, you may have noticed that `data` wasn’t directly provided an object, like this:

```js
data: {
  count: 0
}
```

Instead, **a component’s `data` option must be a function**, so that each instance can maintain an independent copy of the returned data object:

```js
data: function () {
  return {
    count: 0
  }
}
```

If Vue didn’t have this rule, clicking on one button would affect the data of all other instances.

## Organizing Components

It’s common for an app to be organized into a tree of nested components:

For example, you might have components for a header, sidebar, and content area, each typically containing other components for navigation links, blog posts, etc.

To use these components in templates, they must be registered so that Vue knows about them. There are two types of component registration: global and local. So far, we’ve only registered components globally, using `Vue.component`:

```js
    Vue.component("my-component-name", {
        // ...option...
    })
```

Globally registered components can be used in the template of any root Vue instance (`new Vue`) created afterwards – and even inside all subcomponents of that Vue instance’s component tree.

That’s all you need to know about registration for now, but once you’ve finished reading this page and feel comfortable with its content, we recommend coming back later to read the full guide on **Component Registration**.

## Passing Data to Child Components with Props

Earlier, we mentioned creating a component for blog posts. The problem is, that component won’t be useful unless you can pass data to it, such as the title and content of the specific post we want to display. That’s where props come in.

Props are custom attributes you can register on a component. **When a value is passed to a prop attribute, it becomes a property on that component instance**. To pass a title to our blog post component, we can include it in the list of props this component accepts, using a props option:

```js
Vue.component('blog', {
    props: ['title'],
    template: `
        <h3>{{ title }}</h3>
    `
})
```

```html
<blog-post title="My journey with Vue"></blog-post>
<blog-post title="Blogging with Vue"></blog-post>
<blog-post title="Why Vue is so fun"></blog-post>
```

In a typical app, however, you’ll likely have an array of posts in `data`:

```js
new Vue({
  el: '#blog-post-demo',
  data: {
    posts: [
      { id: 1, title: 'My journey with Vue' },
      { id: 2, title: 'Blogging with Vue' },
      { id: 3, title: 'Why Vue is so fun' }
    ]
  }
})
```

Then want to render a component for each one:

```js
    <blog-post
        v-for="post in posts"
        v-bind:key="post.id"
        v-bind:title = "post.title"
    ></blog-post>
```

Above, you’ll see that we can use `v-bind` to dynamically pass props. This is especially useful when you don’t know the exact content you’re going to render ahead of time, like when fetching posts from an API.

That’s all you need to know about props for now, but once you’ve finished reading this page and feel comfortable with its content, we recommend coming back later to read the full guide on **Props**.

## A Single Root Element

When building out a `<blog-post>`component, your template will eventually contain more than just the title:

```html
<h3>{{ title }}</h3>
```

At the very least, you’ll want to include the post’s content:

```html
<h3>{{ title }}</h3>
<div v-html="content"></div>
```

If you try this in your template however, Vue will show an error, explaining that **every component must have a single root element**. You can fix this error by wrapping the template in a parent element, such as:

```html
<div class="blog-post">
  <h3>{{ title }}</h3>
  <div v-html="content"></div>
</div>
```

As our component grows, it’s likely we’ll not only need the title and content of a post, but also the published date, comments, and more. Defining a prop for each related piece of information could become very annoying:

```html
<blog-post
  v-for="post in posts"
  v-bind:key="post.id"
  v-bind:title="post.title"
  v-bind:content="post.content"
  v-bind:publishedAt="post.publishedAt"
  v-bind:comments="post.comments"
></blog-post>
```

So this might be a good time to refactor the `<blog-post>` component to accept a single `post` prop instead:

```html
<blog-post
  v-for="post in posts"
  v-bind:key="post.id"
  v-bind:post="post"
></blog-post>
```

```js
Vue.component('blog-post', {
  props: ['post'],
  template: `
    <div class="blog-post">
      <h3>{{ post.title }}</h3>
      <div v-html="post.content"></div>
    </div>
  `
})
```

Now, whenever a new property is added to `post` objects, it will automatically be available inside `<blog-post>`.

## Listening to Child Components Events

As we develop our `<blog-post>` component, some features may require communicating back up to the parent. For example, we may decide to include an accessibility feature to enlarge the text of blog posts, while leaving the rest of the page its default size:

In the parent, we can support this feature by adding a `postFontSize` data property:

```js
new Vue({
    el: '#blog-posts-events-demo',
    data: {
        posts: [],
        postFontSize: 1
    }
})
```

Which can be used in the template to control the font size of all blog posts:

```html
    <div id="blog-posts-events-demo">
        <div :style="{ fontSize: postFontSize + 'em' }">
            <blog-post
                v-for="post in posts"
                v-bind:key="post.id"
                v-bind:post="post"
            ></blog-post>
        </div>
    </div>
```

Now let’s add a button to enlarge the text right before the content of every post:

```js
Vue.component('blog-post', {
    props: ['post'],
    template: `
        <div class="blog-post">
            <h3>{{ post.title }}</h3>
            <button>
                Enlarge text
            </button>
            <div v-html="post.content"></div>
        </div>
    `
})
```

The problem is, this button doesn’t do anything:

```html
<button>
    Enlarge text
</button>
```

When we click on the button, we need to communicate to the parent that it should enlarge the text of all posts. Fortunately, Vue instances provide a custom events system to solve this problem. The parent can choose to listen to any event on the child component instance with `v-on`, just as we would with a native DOM event:

```html
<blog-post
    ...
    v-on:enlarge-text="postFontSize += 0.1"
></blog-post>
```

Then the child component can emit an event on itself by calling the built-in `$emit` method, passing the name of the event:

```html
<button v-on:click="$emit('enlarge-text')">
    Enlarge text
</button>
```

Thanks to the `v-on:enlarge-text="postFontSize += 0.1"` listener, the parent will receive the event and update `postFontSize` value.

### Emitting a Value With an Event

It’s sometimes useful to emit a specific value with an event. For example, we may want the `<blog-post>`component to be in charge of how much to enlarge the text by. In those cases, we can use `$emit`‘s 2nd parameter to provide this value:

```html
<button v-on:click="$emit('enlarge-text', 0.1)">
  Enlarge text
</button>
```

Then when we listen to the event in the parent, we can access the emitted event’s value with `$event`:

```html
<blog-post
  ...
  v-on:enlarge-text="postFontSize += $event"
></blog-post>
```

Or, if the event handler is a method:

```html
<blog-post
  ...
  v-on:enlarge-text="onEnlargeText"
></blog-post>
```

Then the value will be passed as the first parameter of that method:

```js
methods: {
    onEnlargeText: function (enlargeAmount) {
    this.postFontSize += enlargeAmount
  }
}
```

### Using `v-model` on Components

Custom events can also be used to create custom inputs that work with `v-model`. Remember that:

```html
<input v-model="searchText">
```

does the same thing as:

```html
<input
    v-bind:value="searchText"
    v-on:input="searchText = $event.target.value"
></input>
```

When used on a component, v-model instead does this:

```html
<custom-input
    v-bind:value="searchText"
    v-on:input="searchText = $event"
></custom-input>
```

For this to actually work though, the `<input>` inside the component must:

* Bind the `value` attribute to a `value` prop
* On `input`, emit its own custom `input` event with the new value

Here’s that in action:

```js
Vue.component('custom-input', {
    props: ['value'],
    template: `
        <input
            v-bind:value="value"
            v-on:input="$emit('input', $event.target.value)"
        >
    `
})
```

Now `v-model` should work perfectly with this component:

```html
<custom-input v-model="searchText"></custom-input>
```

That’s all you need to know about custom component events for now, but once you’ve finished reading this page and feel comfortable with its content, we recommend coming back later to read the full guide on Custom Events.

## Content Distribution with Slots

Just like with HTML elements, it’s often useful to be able to pass content to a component, like this:

```html
<alert-box>
    Something bad happend.
</alert-box>
```

Which might render something like:

>Error! Something bad happend.

Fortunately, this task is made very simple by Vue’s custom `<slot>` element:

```js
Vue.component('alert-box', {
  template: `
    <div class="demo-alert-box">
      <strong>Error!</strong>
      <slot></slot>
    </div>
  `
})
```

As you’ll see above, we just add the slot where we want it to go – and that’s it. We’re done!

That’s all you need to know about slots for now, but once you’ve finished reading this page and feel comfortable with its content, we recommend coming back later to read the full guide on Slots.

## Dynamic Components

Sometimes, it’s useful to dynamically switch between components, like in a tabbed interface:

```html
<script src="https://unpkg.com/vue"></script>

<div id="dynamic-component-demo" class="demo">
  <button
    v-for="tab in tabs"
    v-bind:key="tab"
    v-bind:class="['tab-button', { active: currentTab === tab }]"
    v-on:click="currentTab = tab"
  >{{ tab }}</button>

  <component
    v-bind:is="currentTabComponent"
    class="tab"
  ></component>
</div>
```

```js
Vue.component('tab-home', {
    template: '<div>Home component</div>'
})
Vue.component('tab-posts', {
    template: '<div>Posts component</div>'
})
Vue.component('tab-archive', {
    template: '<div>Archive component</div>'
})

new Vue({
  el: '#dynamic-component-demo',
  data: {
    currentTab: 'Home',
    tabs: ['Home', 'Posts', 'Archive']
  },
  computed: {
    currentTabComponent: function () {
      return 'tab-' + this.currentTab.toLowerCase()
    }
  }
})
```

```html
<script src="https://unpkg.com/vue"></script>

<div id="dynamic-component-demo" class="demo">
  <button
    v-for="tab in tabs"
    v-bind:key="tab.name"
    v-bind:class="['tab-button', { active: currentTab.name === tab.name }]"
    v-on:click="currentTab = tab"
  >{{ tab.name }}</button>

  <component
    v-bind:is="currentTab.component"
    class="tab"
  ></component>
</div>
```

```js
var tabs = [
  {
    name: 'Home',
    component: {
      template: '<div>Home component</div>'
    }
  },
  {
    name: 'Posts',
    component: {
      template: '<div>Posts component</div>'
    }
  },
  {
    name: 'Archive',
    component: {
      template: '<div>Archive component</div>',
    }
  }
]

new Vue({
  el: '#dynamic-component-demo',
  data: {
    tabs: tabs,
    currentTab: tabs[0]
  }
})
```

The above is made possible by Vue’s `<component> `element with the is special attribute:

```html
<!-- Component changes when currentTabComponent changes -->
<component v-bind:is="currentTabComponent"></component>
```

In the example above, `currentTabComponent` can contain either:

* the name of a registered component, or
* a component’s options object

That’s all you need to know about dynamic components for now, but once you’ve finished reading this page and feel comfortable with its content, we recommend coming back later to read the full guide on **Dynamic & Async Components**.

## DOM Template Parsing Caveats

Some HTML elements, such as `<ul>`, `<ol>`, `<table>` and `<select>` have restrictions on what elements can appear inside them, and some elements such as `<li>`, `<tr>`, and `<option>` can only appear inside certain other elements.

This will lead to issues when using components with elements that have such restrictions. For example:

```html
<table>
  <blog-post-row></blog-post-row>
</table>
```

The custom component `<blog-post-row> `will be hoisted out as invalid content, causing errors in the eventual rendered output. Fortunately, the `is` special attribute offers a workaround:

```html
<table>
  <tr is="blog-post-row"></tr>
</table>
```

It should be noted that **this limitation does not apply if you are using string templates from one of the following sources**:

* String templates (e.g. `template: '...'`)
* Single-file (`.vue`) components
* `<script type="text/x-template">`

That’s all you need to know about DOM template parsing caveats for now – and actually, the end of Vue’s Essentials. Congratulations! There’s still more to learn, but first, we recommend taking a break to play with Vue yourself and build something fun.

Once you feel comfortable with the knowledge you’ve just digested, we recommend coming back to read the full guide on **Dynamic & Async Components**, as well as the other pages in the Components In-Depth section of the sidebar.
