# Routing

## Official Router

For most Single Page Applications, it’s recommended to use the officially-supported vue-router library. For more details, see vue-router’s documentation.

## Simple Routing From Scratch

If you only need very simple routing and do not wish to involve a full-featured router library, you can do so by dynamically rendering a page-level component like this:

```js
const NotFound = { template: '<p>Page not found</p>' }
const Home = { template: '<p>home page</p>' }
const About = { template: '<p>about page</p>' }

const routes = {
  '/': Home,
  '/about': About
}

new Vue({
  el: '#app',
  data: {
    currentRoute: window.location.pathname
  },
  computed: {
    ViewComponent () {
      return routes[this.currentRoute] || NotFound
    }
  },
  render (h) { return h(this.ViewComponent) }
})
```

Combined with the HTML5 History API, you can build a very basic but fully-functional client-side router. To see that in practice, check out this example app.

## Integrating 3rd-Party Routers

If there’s a 3rd-party router you prefer to use, such as Page.js or Director, integration is similarly easy. Here’s a complete example using Page.js.
\
