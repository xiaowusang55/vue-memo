# Single File Components

## Introduction

In many Vue projects, global components will be defined using `Vue.component`, followed by `new Vue({ el: '#container' })` to target a container element in the body of every page.

This can work very well for small to medium-sized projects, where JavaScript is only used to enhance certain views. In more complex projects however, or when your frontend is entirely driven by JavaScript, these disadvantages become apparent:

* Global definitions force unique names for every component
* String templates lack syntax highlighting and require ugly slashes for multiline HTML
* No CSS support means that while HTML and JavaScript are modularized into components, CSS is conspicuously left out
* No build step restricts us to HTML and ES5 JavaScript, rather than preprocessors like Pug (formerly Jade) and Babel

All of these are solved by single-file components with a .vue extension, made possible with build tools such as Webpack or Browserify.

### What About Separation of Concerns

One important thing to note is that separation of concerns is not equal to separation of file types. In modern UI development, we have found that instead of dividing the codebase into three huge layers that interweave with one another, it makes much more sense to divide them into loosely-coupled components and compose them. Inside a component, its template, logic and styles are inherently coupled, and collocating them actually makes the component more cohesive and maintainable.

Even if you don’t like the idea of Single-File Components, you can still leverage its hot-reloading and pre-compilation features by separating your JavaScript and CSS into separate files:

```html
<!-- my-component.vue -->
<template>
  <div>This will be pre-compiled</div>
</template>
<script src="./my-component.js"></script>
<style src="./my-component.css"></style>
```

## Getting Started

### Example Sandbox

If you want to dive right in and start playing with single-file components, check out this simple todo app on CodeSandbox.

### For Users New to Module Build Systems in JavaScript

With .vue components, we’re entering the realm of advanced JavaScript applications. That means learning to use a few additional tools if you haven’t already:

* Node Package Manager (NPM): Read the Getting Started guide through section 10: Uninstalling global packages.

* Modern JavaScript with ES2015/16: Read through Babel’s Learn ES2015 guide. You don’t have to memorize every feature right now, but keep this page as a reference you can come back to.

After you’ve taken a day to dive into these resources, we recommend checking out Vue CLI 3. Follow the instructions and you should have a Vue project with .vue components, ES2015, Webpack and hot-reloading in no time!

### For Advanced Users

The CLI takes care of most of the tooling configurations for you, but also allows fine-grained customization through its own **config options**.

In case you prefer setting up your own build setup from scratch, you will need to manually configure webpack with **vue-loader**. To learn more about webpack itself, check out **their official docs** and **Webpack Academy**.

