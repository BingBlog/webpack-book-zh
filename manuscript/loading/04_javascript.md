# Loading JavaScript
# 加载JavaScript

Webpack processes ES6 module definitions by default and transforms them into code. It does **not** transform ES6 specific syntax apart, such as `const`. The resulting code can be problematic especially in the older browsers.

Webpack默认就可以处理ES6模块化定义并将其转化为代码。但是它并不会转化那些ES6特定的语法，如`const`。最终导致代码会有很多问题，特别是在一些老的浏览器中。

To get a better idea of the default transform, consider the example output below:

为了更好地理解默认的转换机制，思考如下的输出：

**build/app.js**

```javascript
webpackJsonp([1],{

/* 0 */
/***/ (function(module, __webpack_exports__, __webpack_require__) {

"use strict";
/* harmony default export */ __webpack_exports__["a"] = function (text = 'Hello world') {
  const element = document.createElement('div');

  element.className = 'fa fa-hand-spock-o fa-1g';
  element.innerHTML = text;

  return element;
};

...
```

The problem can be worked around by processing the code through [Babel](https://babeljs.io/), a popular JavaScript compiler that supports ES6 features and more. It resembles ESLint in that it's built on top of presets and plugins. Presets are collections of plugins, and you can define your own as well.

这个问题可以通过使用[Babel](https://babeljs.io/)对代码进行处理来解决，Babel是一个很受欢迎的JavaScript编译器，它支持ES6以及其它的特性。它类似于ESLint，因为它建立在预设（preset）和插件的基础上。预设是插件的集合，你也可以定义自己的预设。

T> Given sometimes extending existing presets is not be enough, [modify-babel-preset](https://www.npmjs.com/package/modify-babel-preset) allows you to go a step further and configure the base preset in a more flexible way.

T> 由于有时候扩展已有的预设并不足以满足需要，[modify-babel-preset](https://www.npmjs.com/package/modify-babel-preset)允许你更进一步，以更灵活的方式来配置一个基本的预设。

## Using Babel with Webpack Configuration
## 结合Webpack配置使用Babel

Even though Babel can be used standalone, as you can see in the *Package Authoring Techniques* chapter, you can hook it up with webpack as well. During development, it can make sense to skip processing if you are using language features supported by your browser.

如你将在*Package Authoring Techniques*章节看到的，尽管Babel可以被单独使用，你也可以将其挂在到webpack上。在开发环境中，如果你使用的是你的浏览器可以支持的语言特性，忽略处理也是有意义的。

Skipping processing is a good option especially if you don't rely on any custom language features and work using a modern browser. Processing through Babel becomes almost a necessity when you compile your code for production, though.

如果你不依赖于任何特别的语言特性，并使用现代浏览器进行开发，忽略处理是一个不错的选择。但是为了在生产环境中正常运行，编译代码时通过Babel进行处理就变得很有必要了。

You can use Babel with webpack through [babel-loader](https://www.npmjs.com/package/babel-loader). It can pick up project level Babel configuration or you can configure it at the webpack loader itself. [babel-webpack-plugin](https://www.npmjs.com/package/babel-webpack-plugin) is another lesser known option.

可以在webpack中通过[babel-loader](https://www.npmjs.com/package/babel-loader)来使用Babel。它可以查找系统级别的Babel配置，或者你也可以在webpack的加载器自身中配置它。[babel-webpack-plugin](https://www.npmjs.com/package/babel-webpack-plugin)是另外一个较少为人知晓的选项。

Connecting Babel with a project allows you to process webpack configuration through it. To achieve this, name your webpack configuration using the *webpack.config.babel.js* convention. [interpret](https://www.npmjs.com/package/interpret) package enables this and it supports other compilers as well.

将Babel和项目结合起来后，你可以通过它来处理你的webpack配置文件。为了实现这个，可以将你的webpack配置文件用约定的*webpack.config.babel.js*来命名。[interpret](https://www.npmjs.com/package/interpret)可以提供支持而且也支持其他的编译器。

T> Given that [Node supports the ES6 specification well](http://node.green/) these days, you can use a lot of ES6 features without having to process configuration through Babel.

T> 由于目前[Node已经能够很好地支持ES6规范](http://node.green/)，你可以不用通过Babel对webpack配置进行处理，就可以使用很多ES6特性。

T> Babel isn't the only option although it's the most popular one. [Buble](https://buble.surge.sh) by Rich Harris is another compiler worth checking out. There's experimental [buble-loader](https://www.npmjs.com/package/buble-loader) that allows you to use it with webpack. Buble doesn't support ES6 modules, but that's not a problem as webpack provides that functionality.

T> 尽管Babel是最受欢迎的，但不是唯一的选择。Rich Harris的[Buble](https://buble.surge.sh)是另外一个值得尝试的编译器。试验中的[buble-loader](https://www.npmjs.com/package/buble-loader)可以让你通过webpack使用它。Buble不支持ES6模块，但因为webpack提供了对应的功能所以不会造成什么问题。

W> If you use *webpack.config.babel.js*, take care with the `"modules": false,` setting. If you want to use ES6 modules, you could skip the setting in your global Babel configuration and then configure it per environment as discussed below.

W> 如果你使用了*webpack.config.babel.js*，请注意`"modules": false,`的设置。如果你希望使用ES6模块化，你可以在全局的Babel配置中忽略这个设置，并在每个环境中如下面讨论的那样配置。

### Setting Up *babel-loader*
### 设置*babel-loader*

The first step towards configuring Babel to work with webpack is to set up [babel-loader](https://www.npmjs.com/package/babel-loader). It takes the code and turns it into a format older browsers can understand. Install *babel-loader* and include its peer dependency *babel-core*:

配置Babel与webpack一起工作的第一步就是设置[babel-loader](https://www.npmjs.com/package/babel-loader)。它将代码转换为老的浏览器可以理解的格式。如下安装*babel-loader*和其对应的依赖*babel-core*：

```bash
npm install babel-loader babel-core --save-dev
```

{pagebreak}

As usual, let's define a part for Babel:

和前面的一样，为Babel定义一个方法：

**webpack.parts.js**

```javascript
exports.loadJavaScript = ({ include, exclude }) => ({
  module: {
    rules: [
      {
        test: /\.js$/,
        include,
        exclude,

        loader: 'babel-loader',
        options: {
          // Enable caching for improved performance during
          // development.
          // It uses default OS directory by default. If you need
          // something more custom, pass a path to it.
          // I.e., { cacheDirectory: '<path>' }
          cacheDirectory: true,
        },
      },
    ],
  },
});
```

Next, you need to connect this with the main configuration. If you are using a modern browser for development, you can consider processing only the production code through Babel. To play it safe, it's used for both production and development environments in this case. In addition, only application code is processed through Babel.

接下来，需要将其挂在到主配置文件中。如果你在开发时使用的是一个现代浏览器，你可以考虑只用Babel对生产环境的代码进行处理。为了确保可以正常运行，这里在开发环境和生产环境都使用了Babel。补充一下，只有应用的代码通过Babel进行了处理。

{pagebreak}

Adjust as below:

**webpack.config.js**

```javascript
const commonConfig = merge([
  {
  ...
leanpub-start-insert
  parts.loadJavaScript({ include: PATHS.app }),
leanpub-end-insert
]);
```

Even though you have Babel installed and set up, you are still missing one bit: Babel configuration. This can be achieved using a *.babelrc* dotfile as other tooling can pick it up as well.

尽管你已经安装并设置了Babel，你仍然忽略了一件事：Babel的配置。可以使用*.babelrc*文件来实现，这个文件也支持其它工具。

W> There are times when caching Babel compilation can surprise you if your dependencies change in a way that *babel-loader* default caching mechanism doesn't notice. Override `cacheIdentifier` with a string that has been derived based on data that should invalidate the cache for better control. [Node crypto API](https://nodejs.org/api/crypto.html) and especially its MD5 related functions can come in handy.

W> 有时候缓存Bable编译会让你感到惊讶，果你的依赖发生了改变，但是*babel-loader*的默认缓存机制可能没有注意到这个变化。用一个数据的字符串来改写`cacheIdentifier`字段，该字符串可以使缓存无效以便更好的控制缓存。[Node crypto API](https://nodejs.org/api/crypto.html),尤其是MD5相关的功能特别有用。

W> If you try to import files **outside** of your configuration root directory and then process them through *babel-loader*, this fails. It's [a known issue](https://github.com/babel/babel-loader/issues/313), and there are workarounds including maintaining *.babelrc* at a higher level in the project and resolving against Babel presets through `require.resolve` at webpack configuration.

W> 如果你希望引入在配置文件根目录**之外**的文件，然后通过*babel-loader*对其进行处理，将会失败。这是一个[已知的问题](https://github.com/babel/babel-loader/issues/313)，有几种解决办法，包括在比项目更高一层的目录中来维护*.babelrc*文件，在webpack的配置中通过`require.resolve`对Babel预设

### Setting Up *.babelrc*

At a minimum, you need [babel-preset-env](https://www.npmjs.com/package/babel-preset-env). It's a Babel preset that enables the needed plugins based on the environment definition you pass to it. It follows the **browserslist** definition discussed in the *Autoprefixing* chapter.

Install the preset first:

```bash
npm install babel-preset-env --save-dev
```

To make Babel aware of the preset, you need to write a *.babelrc*. Given webpack supports ES6 modules out of the box, you can tell Babel to skip processing them. Skipping this step would break webpack's HMR mechanism although the production build would still work. You can also constrain the build output to work only in recent versions of Chrome.

Adjust the target definition as you like. As long as you follow [browserslist](https://www.npmjs.com/package/browserslist), it should work. Here's a sample configuration:

**.babelrc**

```json
{
  "presets": [
    [
      "env",
      {
        "modules": false,
        "targets": {
          "browsers": ["last 2 Chrome versions"]
        }
      }
    ]
  ]
}
```

T> If you omit the `targets` definition, *babel-preset-env* compiles to ES5 compatible code. If you are using UglifyJS, see the *Minifying* chapter for more information on why this is required. You can also target Node through the `node` field. Example: `"node": "current"`.

W> **babel-preset-env** does **not** support *browserslist* file yet. [See issue #26](https://github.com/babel/babel-preset-env/issues/26) for more information.

If you execute `npm run build` now and examine *build/app.js*, the result should be similar to the earlier since it supports the features you are using in the code.

To see that the target definition works, change it to work such as `"browsers": ["IE 8"]`. Since IE 8 doesn't support `const`s, the code should change. If you build (`npm run build`), now, you should see something different:

**build/app.js**

```javascript
webpackJsonp([1],{

/* 0 */
/***/ (function(module, __webpack_exports__, __webpack_require__) {

"use strict";
/* harmony default export */ __webpack_exports__["a"] = function () {
  var text = arguments.length > 0 && arguments[0] !== undefined ? arguments[0] : 'Hello world';

  var element = document.createElement('div');

  element.className = 'fa fa-hand-spock-o fa-1g';
  element.innerHTML = text;

  return element;
};

...
```

Note especially how the function was transformed. You can try out different browser definitions and language features to see how the output changes based on the selection.

{pagebreak}

## Polyfilling Features

*babel-preset-env* allows you to polyfill certain language features for older browsers. For this to work, you should enable its `useBuiltIns` option (`"useBuiltIns": true`) and install [babel-polyfill](https://babeljs.io/docs/usage/polyfill/). You have include it to your project either through an import or an entry (`app: ['babel-polyfill', PATHS.app]`). *babel-preset-env* rewrites the import based on your browser definition and loads only the polyfills that are needed.

*babel-polyfill* pollutes the global scope with objects like `Promise`. Given this can be problematic for library authors, there's [transform-runtime](https://babeljs.io/docs/plugins/transform-runtime/) option. It can be enabled as a Babel plugin, and it avoids the problem of globals by rewriting the code in such way that they aren't be needed.

W> Certain webpack features, such as *Code Splitting*, write `Promise` based code to webpack's bootstrap after webpack has processed loaders. The problem can be solved by applying a shim before your application code is executed. Example: `entry: { app: ['core-js/es6/promise', PATHS.app] }`.

## Babel Tips

There are other possible [*.babelrc* options](https://babeljs.io/docs/usage/options/) beyond the ones covered here. Like ESLint, *.babelrc* supports [JSON5](https://www.npmjs.com/package/json5) as its configuration format meaning you can include comments in your source, use single quoted strings, and so on.

Sometimes you want to use experimental features that fit your project. Although you can find a lot of them within so-called stage presets, it's a good idea to enable them one by one and even organize them to a preset of their own unless you are working on a throwaway project. If you expect your project to live a long time, it's better to document the features you are using well.

{pagebreak}

## Babel Presets and Plugins

Perhaps the greatest thing about Babel is that it's possible to extend with presets and plugins:

* [babel-preset-es2015](https://www.npmjs.org/package/babel-preset-es2015) includes ES2015 features.
* [babel-preset-es2016](https://www.npmjs.org/package/babel-preset-es2016) includes **only** ES2016 features. Remember to include the previous preset as well if you want both!
* [babel-plugin-import](https://www.npmjs.com/package/babel-plugin-import) rewrites module imports so that you can use a form such as `import { Button } from 'antd';` instead of pointing to the module through an exact path.
* [babel-plugin-import-asserts](https://www.npmjs.com/package/babel-plugin-import-asserts) asserts that your imports have been defined.
* [babel-plugin-log-deprecated](https://www.npmjs.com/package/babel-plugin-log-deprecated) adds `console.warn` to functions that have `@deprecate` annotation in their comment.
* [babel-plugin-annotate-console-log](https://www.npmjs.com/package/babel-plugin-annotate-console-log) annotates `console.log` calls with information about invocation context so it's easier to see where they logged.
* [babel-plugin-webpack-loaders](https://www.npmjs.com/package/babel-plugin-webpack-loaders) allows you to use certain webpack loaders through Babel.
* [babel-plugin-syntax-trailing-function-commas](https://www.npmjs.com/package/babel-plugin-syntax-trailing-function-commas) adds trailing comma support for functions.
* [babel-react-optimize](https://github.com/thejameskyle/babel-react-optimize) implements a variety of React specific optimizations you can experiment with.
* [babel-plugin-transform-react-remove-prop-types](https://www.npmjs.com/package/babel-plugin-transform-react-remove-prop-types) allows you to remove `propType` related code from your production build. It also allows component authors to generate code that's wrapped so that setting environment at `DefinePlugin` can kick in as discussed in the book.

T> It's possible to connect Babel with Node through [babel-register](https://www.npmjs.com/package/babel-register) or [babel-cli](https://www.npmjs.com/package/babel-cli). These packages can be handy if you want to execute your code through Babel without using webpack.

## Enabling Presets and Plugins per Environment

Babel allows you to control which presets and plugins are used per environment through its [env option](https://babeljs.io/docs/usage/babelrc/#env-option). You can manage Babel's behavior per build target this way.

`env` checks both `NODE_ENV` and `BABEL_ENV` and functionality to your build based on that. If `BABEL_ENV` is set, it overrides any possible `NODE_ENV`. Consider the example below:

```json
{
  ...
  "env": {
    "development": {
      "plugins": [
        "react-hot-loader/babel"
      ]
    }
  }
}
```

Any shared presets and plugins are available to all targets still. `env` allows you to specialize your Babel configuration further.

{pagebreak}

It's possible to pass the webpack environment to Babel with a tweak:

**webpack.config.js**

```javascript
module.exports = (env) => {
leanpub-start-insert
  process.env.BABEL_ENV = env;
leanpub-end-insert

  ...
};
```

T> The way `env` works is subtle. Consider logging `env` and make sure it matches your Babel configuration or otherwise the functionality you expect is not applied to your build.

T> The technique is used in the *Server Side Rendering* chapter to enable the Babel portion of *react-hot-loader* for development target only.

## Setting Up TypeScript

Microsoft's [TypeScript](http://www.typescriptlang.org/) is a compiled language that follows a similar setup as Babel. The neat thing is that in addition to JavaScript, it can emit type definitions. A good editor can pick those up and provide enhanced editing experience. Stronger typing is valuable for development as it becomes easier to state your type contracts.

Compared to Facebook's type checker Flow, TypeScript is a more established option. As a result, you find more premade type definitions for it, and overall, the quality of support should be better.

{pagebreak}

You can use TypeScript with webpack using the following loaders:

* [ts-loader](https://www.npmjs.com/package/ts-loader)
* [awesome-typescript-loader](https://www.npmjs.com/package/awesome-typescript-loader)
* [light-ts-loader](https://www.npmjs.com/package/light-ts-loader)

T> There's a [TypeScript parser for ESLint](https://www.npmjs.com/package/typescript-eslint-parser). It's also possible to lint it through [tslint](https://www.npmjs.com/package/tslint).

## Setting Up Flow

[Flow](https://flow.org/) performs static analysis based on your code and its type annotations. You have to install it as a separate tool and then run it against your code. There's [flow-status-webpack-plugin](https://www.npmjs.com/package/flow-status-webpack-plugin) that allows you to run it through webpack during development.

If you use React, the React specific Babel preset does most of the work through [babel-plugin-syntax-flow](https://www.npmjs.com/package/babel-plugin-syntax-flow). It can strip Flow annotations and convert your code into a format that is possible to transpile further.

There's also [babel-plugin-typecheck](https://www.npmjs.com/package/babel-plugin-typecheck) that allows you to perform runtime checks based on your Flow annotations. [flow-runtime](https://codemix.github.io/flow-runtime/) goes a notch further and provides more functionality. These approaches complement Flow static checker and allow you to catch even more issues.

T> [flow-coverage-report](https://www.npmjs.com/package/flow-coverage-report) shows how much of your code is covered by Flow type annotations.

## Conclusion

Babel has become an indispensable tool for developers given it bridges the standard with older browsers. Even if you targeted modern browsers, transforming through Babel is an option.

To recap:

* Babel gives you control over what browsers to support. It can compile ES6 features to a form the older browser understand. *babel-preset-env* is valuable as it can choose which features to compile and which polyfills to enable based on your browser definition.
* Babel allows you to use experimental language features. You can find numerous plugins that improve development experience and the production build through optimizations.
* Babel functionality can be enabled per development target. This way you can be sure you are using the correct plugins at the right place.
* Besides Babel, webpack supports other solutions like TypeScript of Flow. Flow can complement Babel while TypeScript represents an entire language compiling to JavaScript.
