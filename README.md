# rollup-plugin-postcss

[![NPM version](https://img.shields.io/npm/v/rollup-plugin-postcss.svg?style=flat)](https://npmjs.com/package/rollup-plugin-postcss) [![NPM downloads](https://img.shields.io/npm/dm/rollup-plugin-postcss.svg?style=flat)](https://npmjs.com/package/rollup-plugin-postcss) [![Build Status](https://img.shields.io/circleci/project/egoist/rollup-plugin-postcss/master.svg?style=flat)](https://circleci.com/gh/egoist/rollup-plugin-postcss) [![codecov](https://codecov.io/gh/egoist/rollup-plugin-postcss/branch/master/graph/badge.svg)](https://codecov.io/gh/egoist/rollup-plugin-postcss)
 [![donate](https://img.shields.io/badge/$-donate-ff69b4.svg?maxAge=2592000&style=flat)](https://github.com/egoist/donate)

<img align="right" width="95" height="95"
     title="Philosopher’s stone, logo of PostCSS"
     src="http://postcss.github.io/postcss/logo.svg">

Seamless integration between [Rollup](https://github.com/rollup/rollup) and [PostCSS](https://github.com/postcss/postcss).

## Install

```bash
yarn add rollup-plugin-postcss --dev
```

## Usage

You are viewing the docs for `v1.0`, for `v0.5` please see [here](https://github.com/egoist/rollup-plugin-postcss/tree/0.5).

```js
// rollup.config.js
import postcss from 'rollup-plugin-postcss'

export default {
  plugins: [
    postcss({
      plugins: []
    })
  ]
}
```

Then you can use CSS files:

```js
import './style.css'
```

Note that the generated CSS will be injected to `<head>` by default, and the CSS string is also available as default export unless `extract: true`:

```js
// Inject to `<head>` and also available as `style`
import style from './style.css'
```

It will also automatically use local PostCSS config files.

### Extract CSS
To extract the processed CSS file, set the `extract` configuration option to
the path where the file should be saved.

```js
postcss({
  extract: './dist/mystyles.css'
})
```

### CSS modules

```js
postcss({
  modules: true,
  // Or with custom options for `postcss-modules`
  modules: {}
})
```

### With Sass/Stylus/Less

Install corresponding dependency:

- For `Sass` install `node-sass`: `yarn add node-sass --dev`
- For `Stylus` Install `stylus`: `yarn add stylus --dev`
- For `Less` Install `less`: `yarn add less --dev`

That's it, you can now import `.styl` `.scss` `.sass` `.less` files in your library.

#### imports

__For Sass/Scss Only.__

Similar to how webpack's [sass-loader](https://github.com/webpack-contrib/sass-loader#imports) works, you can prepend the path with `~` to tell this plugin to resolve in `node_modules`:

```sass
@import "~bootstrap/dist/css/bootstrap";
```

## Options

### plugins

Type: `Array`

PostCSS Plugins.

### inject

Type: `boolean` `object`<br>
Default: `true`

Inject CSS into `<head>`, it's always `false` when `extract` is not `false`.

You can also use it as options for [`style-inject`](https://github.com/egoist/style-inject#options).

### extract

Type: `boolean` `string`<br>
Default: `false`

Extract CSS to a separate file.  The recommended settings are either `false`, to not extract the CSS file, or
a path to the location where the file should be saved.

Setting this option to `true` will extract CSS to the same location where JS file is generated but with `.css` extension.  Unfortunately, PostCSS plugins might behave incorrectly in this case.  See [Issue #65](https://github.com/egoist/rollup-plugin-postcss/issues/65) for more details.

### modules

Type: `boolean` `object`<br>
Default: `false`

Enable CSS modules or set options for `postcss-modules`.

### autoModules

Type: `boolean`<br>
Default: `true`

Automatically enable CSS modules for `.module.css` `.module.sss` `.module.scss` `.module.sass` `.module.styl` `.module.stylus` `.module.less` files.

### namedExports

Type: `boolean` `function`<br>
Default: `false`

Use named exports alongside default export.

You can supply a function to control how exported named is generated:

```js
namedExports(name) {
  // Maybe you simply want to convert dash to underscore
  return name.replace(/-/g, '_')
}
```

If you set it to `true`, the following will happen when importing specific classNames:

- dashed class names will be transformed by replacing all the dashes to `$` sign wrapped underlines, eg. `--` => `$__$`
- js protected names used as your style class names, will be transformed by wrapping the names between `$` signs, eg. `switch` => `$switch$`

All transformed names will be logged in your terminal like:

```bash
Exported "new" as "$new$" in test/fixtures/named-exports/style.css
```

The original will not be removed, it's still available on `default` export:

```js
import style, { class$_$name, class$__$name, $switch$ } from './style.css'
console.log(style['class-name'] === class$_$name) // true
console.log(style['class--name'] === class$__$name) // true
console.log(style['switch'] === $switch$) // true
```

### minimize

Type: `boolean` `object`<br>
Default: `false`

Minimize CSS, `boolean` or options for `cssnano`.

### sourceMap

Type: `boolean` `"inline"`

Enable sourceMap.

### parser

Type: `string` `function`

PostCSS parser, like `sugarss`.

### stringifier

Type: `string` `function`

PostCSS Stringifier.

### syntax

Type: `string` `function`

PostCSS Syntax.

### exec

Type: `boolean`

Enable PostCSS Parser support in `CSS-in-JS`.

### config

Type: `boolean` `object`<br>
Default: `true`

Load PostCSS config file.

#### config.path

Type: `string`

The path to config file, so that we can skip searching.

#### config.ctx

Type: `object`

[`ctx`](https://github.com/michael-ciniawsky/postcss-load-config#context) argument for PostCSS config file.

Note: Every keys you pass to `config.ctx` will be available under `options` inside
the postcss config.

```js
// rollup.config.js
postcss({
  config: {
    ctx: {
      foo: 'bar'
    }
  }
})

// postcss.config.js
module.exports = context => {
  console.log(context.options.foo) // 'bar'

  return {}
}
```

### use

Type: `name[]` `[name, options][]`<br>
Default: `['sass', 'stylus', 'less']`

Use a loader, currently built-in loaders are:

- `sass` (Support `.scss` and `.sass`)
- `stylus` (Support `.styl` and `.stylus`)
- `less` (Support `.less`)

They are executed from right to left.

### loaders

Type: `Loader[]`

An array of custom loaders, check out our [sass-loader](./src/sass-loader.js) as example.

```js
interface Loader {
  name: string,
  test: RegExp,
  process: (this: Context, input: Payload) => Promise<Payload> | Payload
}

interface Context {
  /** Loader options */
  options: any
  /** Sourcemap */
  sourceMap: any
  /** Resource path */
  id: string
}

interface Payload {
  /** File content */
  code: string
  /** Sourcemap */
  map?: string | SourceMap
}
```

### onImport

Type: `id => void`

A function to be invoked when an import for CSS file is detected.

## License

MIT &copy; [EGOIST](https://github.com/egoist)
