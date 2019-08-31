# Production Deployment

Most of the tips below are enabled by default if you are using Vue CLI. This section is only relevant if you are using a custom build setup.

## Turn on Production Mode

During development, Vue provides a lot of warnings to help you with common errors and pitfalls. However, these warning strings become useless in production and bloat your app’s payload size. In addition, some of these warning checks have small runtime costs that can be avoided in production mode.

### Without Build Tools

If you are using the full build, i.e. directly including Vue via a script tag without a build tool, make sure to use the minified version (vue.min.js) for production. Both versions can be found in the Installation guide.

### With Build Tools

When using a build tool like Webpack or Browserify, the production mode will be determined by process.env.NODE_ENV inside Vue’s source code, and it will be in development mode by default. Both build tools provide ways to overwrite this variable to enable Vue’s production mode, and warnings will be stripped by minifiers during the build. All vue-cli templates have these pre-configured for you, but it would be beneficial to know how it is done:

#### Webpack

In Webpack 4+, you can use the mode option:

```js
module.exports = {
  mode: 'production'
}
```

But in Webpack 3 and earlier, you’ll need to use DefinePlugin:

```js
var webpack = require('webpack')

module.exports = {
  // ...
  plugins: [
    // ...
    new webpack.DefinePlugin({
      'process.env.NODE_ENV': JSON.stringify('production')
    })
  ]
}
```

#### Browserify

* Run your bundling command with the actual NODE_ENV environment variable set to "production". This tells vueify to avoid including hot-reload and development related code.

* Apply a global envify transform to your bundle. This allows the minifier to strip out all the warnings in Vue’s source code wrapped in env variable conditional blocks. For example:

```shell
NODE_ENV=production browserify -g envify -e main.js | uglifyjs -c -m > build.js
```

* Or, using envify with Gulp:

```js
// Use the envify custom module to specify environment variables
var envify = require('envify/custom')

browserify(browserifyOptions)
  .transform(vueify)
  .transform(
    // Required in order to process node_modules files
    { global: true },
    envify({ NODE_ENV: 'production' })
  )
  .bundle()
```

* Or, using envify with Grunt and grunt-browserify:

```js
// Use the envify custom module to specify environment variables
var envify = require('envify/custom')

browserify: {
  dist: {
    options: {
      // Function to deviate from grunt-browserify's default order
      configure: b => b
        .transform('vueify')
        .transform(
          // Required in order to process node_modules files
          { global: true },
          envify({ NODE_ENV: 'production' })
        )
        .bundle()
    }
  }
}
```

#### Rollup

Use rollup-plugin-replace:

```js
const replace = require('rollup-plugin-replace')

rollup({
  // ...
  plugins: [
    replace({
      'process.env.NODE_ENV': JSON.stringify( 'production' )
    })
  ]
}).then(...)
```

## Pre-Compiling Templates

When using in-DOM templates or in-JavaScript template strings, the template-to-render-function compilation is performed on the fly. This is usually fast enough in most cases, but is best avoided if your application is performance-sensitive.

The easiest way to pre-compile templates is using Single-File Components - the associated build setups automatically performs pre-compilation for you, so the built code contains the already compiled render functions instead of raw template strings.

If you are using Webpack, and prefer separating JavaScript and template files, you can use vue-template-loader, which also transforms the template files into JavaScript render functions during the build step.

## Extracting Component CSS

When using Single-File Components, the CSS inside components are injected dynamically as `<style>` tags via JavaScript. This has a small runtime cost, and if you are using server-side rendering it will cause a “flash of unstyled content”. Extracting the CSS across all components into the same file will avoid these issues, and also result in better CSS minification and caching.

Refer to the respective build tool documentations to see how it’s done:

* Webpack + vue-loader (the vue-cli webpack template has this pre-configured)
* Browserify + vueify
* Rollup + rollup-plugin-vue

## Tracking Runtime Errors

If a runtime error occurs during a component’s render, it will be passed to the global `Vue.config.errorHandler` config function if it has been set. It might be a good idea to leverage this hook together with an error-tracking service like Sentry, which provides an official integration for Vue.

