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
