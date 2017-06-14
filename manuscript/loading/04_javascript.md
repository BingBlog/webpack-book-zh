# Loading JavaScript
# 加载JavaScript

Webpack processes ES6 module definitions by default and transforms them into code. It does **not** transform ES6 specific syntax apart, such as `const`. The resulting code can be problematic especially in the older browsers.

Webpack默认就可以处理ES6模块化定义并将其转化为普通的代码。但是它并不会转化那些ES6特定的语法，如`const`。导致代码会有很多问题，特别是在一些老的浏览器中。

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

如你将在*Package Authoring Techniques*章节看到的，尽管Babel可以被单独使用，你也可以将其挂在到webpack上。在开发环境中，如果你使用的是你的浏览器可以支持的语言特性，也可以不用对代码进行处理。

Skipping processing is a good option especially if you don't rely on any custom language features and work using a modern browser. Processing through Babel becomes almost a necessity when you compile your code for production, though.

如果你不依赖于任何特别的语言特性，并使用现代浏览器进行开发，忽略处理也是一个不错的选择。但是为了在生产环境中正常运行，编译代码时通过Babel进行处理就显得很有必要了。

You can use Babel with webpack through [babel-loader](https://www.npmjs.com/package/babel-loader). It can pick up project level Babel configuration or you can configure it at the webpack loader itself. [babel-webpack-plugin](https://www.npmjs.com/package/babel-webpack-plugin) is another lesser known option.

可以在webpack中通过[babel-loader](https://www.npmjs.com/package/babel-loader)来使用Babel。它可以查找系统级别的Babel配置，或者你也可以在webpack的加载器自身中配置它。[babel-webpack-plugin](https://www.npmjs.com/package/babel-webpack-plugin)是另外一个较少为人知晓的选择。

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
  ...
leanpub-start-insert
  parts.loadJavaScript({ include: PATHS.app }),
leanpub-end-insert
]);
```

Even though you have Babel installed and set up, you are still missing one bit: Babel configuration. This can be achieved using a *.babelrc* dotfile as other tooling can pick it up as well.

尽管你已经安装并设置了Babel，你仍然忽略了一件事：Babel的配置。可以使用 *.babelrc*文件来实现，这个文件也支持其它工具。

W> There are times when caching Babel compilation can surprise you if your dependencies change in a way that *babel-loader* default caching mechanism doesn't notice. Override `cacheIdentifier` with a string that has been derived based on data that should invalidate the cache for better control. [Node crypto API](https://nodejs.org/api/crypto.html) and especially its MD5 related functions can come in handy.

W> 有时候缓存Bable编译会让你感到惊讶，果你的依赖发生了改变，但是*babel-loader*的默认缓存机制可能没有注意到这个变化。用一个数据的字符串来改写`cacheIdentifier`字段，该字符串可以使缓存无效以便更好的控制缓存。[Node crypto API](https://nodejs.org/api/crypto.html),尤其是MD5相关的功能特别有用。

W> If you try to import files **outside** of your configuration root directory and then process them through *babel-loader*, this fails. It's [a known issue](https://github.com/babel/babel-loader/issues/313), and there are workarounds including maintaining *.babelrc* at a higher level in the project and resolving against Babel presets through `require.resolve` at webpack configuration.

W> 如果你希望引入在配置文件根目录**之外**的文件，然后通过*babel-loader*对其进行处理，将会失败。这是一个[已知的问题](https://github.com/babel/babel-loader/issues/313)，有几种解决办法，包括在比项目更高一层的目录中来维护 *.babelrc*文件，在webpack的配置中通过`require.resolve`对Babel进行预设。

### Setting Up *.babelrc*

### 设置 *.babelrc*

At a minimum, you need [babel-preset-env](https://www.npmjs.com/package/babel-preset-env). It's a Babel preset that enables the needed plugins based on the environment definition you pass to it. It follows the **browserslist** definition discussed in the *Autoprefixing* chapter.

最低限度，你需要[babel-preset-env](https://www.npmjs.com/package/babel-preset-env)。它是一个Babel预设器，可以支持所需的插件，这些插件基于你传入
{pagebreak}

Install the preset first:

首先安装预设器：

```bash
npm install babel-preset-env --save-dev
```

To make Babel aware of the preset, you need to write a *.babelrc*. Given webpack supports ES6 modules out of the box, you can tell Babel to skip processing them. Skipping this step would break webpack's HMR mechanism although the production build would still work. You can also constrain the build output to work only in recent versions of Chrome.

为了使Babel意识到这个预设器，你需要编写一个 *.babelrc*。由于webpack支持直接使用ES6模块化，你可以告知Babel忽略处理它们。跳过这一步将打破webpack的HMR机制，尽管生产构建仍然可行。您也可以将构建输出限制为仅在最新版本的Chrome中运行。

Adjust the target definition as you like. As long as you follow [browserslist](https://www.npmjs.com/package/browserslist), it should work. Here's a sample configuration:

根据需要调整目标定义。只要你遵循[browserslist](https://www.npmjs.com/package/browserslist)，它就应该会生效。 以下是一个示例配置：

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

T> 如果你忽略了`targets`的定义，*babel-preset-env*将会编译代码为ES5的兼容代码。如果你使用了UglifyJS，关于为何要使用它，更多信息请见*Minifying*章节。你也可以通过`node`字段将目标设置为Node。例如：`"node": "current"`。

W> **babel-preset-env** does **not** support *browserslist* file yet. [See issue #26](https://github.com/babel/babel-preset-env/issues/26) for more information.

W> **babel-preset-env**目前还**不**支持*browserslist*。更多请[查看issue #26](https://github.com/babel/babel-preset-env/issues/26)

If you execute `npm run build` now and examine *build/app.js*, the result should be similar to the earlier since it supports the features you are using in the code.

如果你现在执行`npm run build`并检查*build/app.js*，结果应该类似于早期版本，因为它支持您在代码中使用的功能。

To see that the target definition works, change it to work such as `"browsers": ["IE 8"]`. Since IE 8 doesn't support `const`s, the code should change. If you build (`npm run build`), now, you should see something different:

为了验证目标定义的有效性，将其改变为 `"browsers": ["IE 8"]`，由于IE8并不支持`const`语法，所以代码也会改变。如果你现在构建（`npm run build`），将会看到一些不一样结果。

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

特别要注意该功能是如何转换的。您可以尝试不同的浏览器定义和语言功能，来了解输出是如何基于选择而改变的。

{pagebreak}

## Polyfilling Features

## Polyfilling特性

*babel-preset-env* allows you to polyfill certain language features for older browsers. For this to work, you should enable its `useBuiltIns` option (`"useBuiltIns": true`) and install [babel-polyfill](https://babeljs.io/docs/usage/polyfill/). You have include it to your project either through an import or an entry (`app: ['babel-polyfill', PATHS.app]`). *babel-preset-env* rewrites the import based on your browser definition and loads only the polyfills that are needed.

*babel-preset-env*允许你为一些老浏览器模拟实现特定的语言特性。为了实现这个，你需要启用其`useBuiltIns`选项（`"useBuiltIns": true`），并安装[babel-polyfill](https://babeljs.io/docs/usage/polyfill/)。你需要在项目中通过import引入或者在入口文件(`app: ['babel-polyfill', PATHS.app]`)将其引入。*babel-preset-env*会基于你定义的浏览器改写这个引入，并仅加载需要的模拟实现。

*babel-polyfill* pollutes the global scope with objects like `Promise`. Given this can be problematic for library authors, there's [transform-runtime](https://babeljs.io/docs/plugins/transform-runtime/) option. It can be enabled as a Babel plugin, and it avoids the problem of globals by rewriting the code in such way that they aren't be needed.

*babel-polyfill*用类似于`Promise`的对象污染了全局作用域。由于这将对一些库作者造成麻烦，可以使用[transform-runtime](https://babeljs.io/docs/plugins/transform-runtime/)。它作为一个Babel插件，可以通过重写代码来避免全局的问题。

W> Certain webpack features, such as *Code Splitting*, write `Promise` based code to webpack's bootstrap after webpack has processed loaders. The problem can be solved by applying a shim before your application code is executed. Example: `entry: { app: ['core-js/es6/promise', PATHS.app] }`.

W> 一些特定的webpack特性，注入*Code Splitting*，在webpack已经处理了加载器之后编写基于`Promise`的代码到webpack的引导脚本中。这个问题可以通过在应用代码执行前使用一个垫片来解决。如：`entry: { app: ['core-js/es6/promise', PATHS.app] }`。

## Babel Tips

There are other possible [*.babelrc* options](https://babeljs.io/docs/usage/options/) beyond the ones covered here. Like ESLint, *.babelrc* supports [JSON5](https://www.npmjs.com/package/json5) as its configuration format meaning you can include comments in your source, use single quoted strings, and so on.

Sometimes you want to use experimental features that fit your project. Although you can find a lot of them within so-called stage presets, it's a good idea to enable them one by one and even organize them to a preset of their own unless you are working on a throwaway project. If you expect your project to live a long time, it's better to document the features you are using well.

{pagebreak}

## Babel Presets and Plugins

## Babel预设器和插件

Perhaps the greatest thing about Babel is that it's possible to extend with presets and plugins:

Babel最了不起的地方就是可以通过预设器和插件来扩展：

* [babel-preset-es2015](https://www.npmjs.org/package/babel-preset-es2015) includes ES2015 features.
* [babel-preset-es2015](https://www.npmjs.org/package/babel-preset-es2015)包含了ES2015特性。

* [babel-preset-es2016](https://www.npmjs.org/package/babel-preset-es2016) includes **only** ES2016 features. Remember to include the previous preset as well if you want both!
* [babel-preset-es2016](https://www.npmjs.org/package/babel-preset-es2016)**仅**包含了ES2016的特性。记住你还需要使用ES2015，两者都得安装。

* [babel-plugin-import](https://www.npmjs.com/package/babel-plugin-import) rewrites module imports so that you can use a form such as `import { Button } from 'antd';` instead of pointing to the module through an exact path.
* [babel-plugin-import](https://www.npmjs.com/package/babel-plugin-import)重写了模块引入，使你可以使用如`import { Button } from 'antd';`的形式代替额外指向模块的路径。

* [babel-plugin-import-asserts](https://www.npmjs.com/package/babel-plugin-import-asserts) asserts that your imports have been defined.
* [babel-plugin-import-asserts](https://www.npmjs.com/package/babel-plugin-import-asserts)声明你的导入已经被定义。

* [babel-plugin-log-deprecated](https://www.npmjs.com/package/babel-plugin-log-deprecated) adds `console.warn` to functions that have `@deprecate` annotation in their comment.
* [babel-plugin-log-deprecated](https://www.npmjs.com/package/babel-plugin-log-deprecated)向备注了不建议（`@deprecate`）方法中添加警告信息（`console.warn`）。

* [babel-plugin-annotate-console-log](https://www.npmjs.com/package/babel-plugin-annotate-console-log) annotates `console.log` calls with information about invocation context so it's easier to see where they logged.
* [babel-plugin-annotate-console-log](https://www.npmjs.com/package/babel-plugin-annotate-console-log)向`console.log`中添加上下文信息使你更容易观察该日志在代码中的位置。

* [babel-plugin-webpack-loaders](https://www.npmjs.com/package/babel-plugin-webpack-loaders) allows you to use certain webpack loaders through Babel.
* [babel-plugin-webpack-loaders](https://www.npmjs.com/package/babel-plugin-webpack-loaders)允许你通过Babel使用特定的加载器。

* [babel-plugin-syntax-trailing-function-commas](https://www.npmjs.com/package/babel-plugin-syntax-trailing-function-commas) adds trailing comma support for functions.
* [babel-plugin-syntax-trailing-function-commas](https://www.npmjs.com/package/babel-plugin-syntax-trailing-function-commas)可以支持在函数的参数结尾添加一个逗号。（让你在更改参数个数时，最少限度的更改代码）

* [babel-react-optimize](https://github.com/thejameskyle/babel-react-optimize) implements a variety of React specific optimizations you can experiment with.
* [babel-react-optimize](https://github.com/thejameskyle/babel-react-optimize)实现各种可以尝试的对于React的特定的优化。

* [babel-plugin-transform-react-remove-prop-types](https://www.npmjs.com/package/babel-plugin-transform-react-remove-prop-types) allows you to remove `propType` related code from your production build. It also allows component authors to generate code that's wrapped so that setting environment at `DefinePlugin` can kick in as discussed in the book.
* [babel-plugin-transform-react-remove-prop-types](https://www.npmjs.com/package/babel-plugin-transform-react-remove-prop-types) 可以让你从生产环境的代码中去掉`propType`相关的代码。它还允许组件的作者生成被包裹的代码，并如本书中已经讨论的在`DefinePlugin`中设置环境参数来解决问题。

T> It's possible to connect Babel with Node through [babel-register](https://www.npmjs.com/package/babel-register) or [babel-cli](https://www.npmjs.com/package/babel-cli). These packages can be handy if you want to execute your code through Babel without using webpack.

T> 可以通过[babel-register](https://www.npmjs.com/package/babel-register)或者[babel-cli](https://www.npmjs.com/package/babel-cli)将Babel和Node联系起来。如果你想在不使用webpack的情况下使用Babel来执行你的代码，这些代码包将会很有用。

## Enabling Presets and Plugins per Environment
## 对不同环境启用预设和插件

Babel allows you to control which presets and plugins are used per environment through its [env option](https://babeljs.io/docs/usage/babelrc/#env-option). You can manage Babel's behavior per build target this way.

Babel允许你通过其[环境选择](https://babeljs.io/docs/usage/babelrc/#env-option)控制在不同的环境中使用哪些预设器和插件。你可以用这种方式来管理Babel在不同的构建目标下的行为。

`env` checks both `NODE_ENV` and `BABEL_ENV` and functionality to your build based on that. If `BABEL_ENV` is set, it overrides any possible `NODE_ENV`. Consider the example below:

`env`检查`NODE_ENV`和`BABEL_ENV`并基于此构建出你构建的功能。如果设置了`BABEL_ENV`，他将会覆盖任何`NODE_ENV`。例子如下：

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

任何共享的预设和插件对所有的构建目标都仍然有效。`env`可以让你的Babel配置更加专一于某一构建目标。

{pagebreak}

It's possible to pass the webpack environment to Babel with a tweak:

可以如下调整代码，将webpack的环境传递给Babel：

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

T> `env`工作的方式比较微妙。可以考试输出`env`日志，确保其与你的Babel配置匹配，否则你期望的功能可能并没有应用到构建中。

T> The technique is used in the *Server Side Rendering* chapter to enable the Babel portion of *react-hot-loader* for development target only.

T> 在*Server Side Rendering*章节使用的技术可以仅在开发环境启用Babel的*react-hot-loader*功能。

## Setting Up TypeScript
## 设置TypeScript

Microsoft's [TypeScript](http://www.typescriptlang.org/) is a compiled language that follows a similar setup as Babel. The neat thing is that in addition to JavaScript, it can emit type definitions. A good editor can pick those up and provide enhanced editing experience. Stronger typing is valuable for development as it becomes easier to state your type contracts.

微软的[TypeScript](http://www.typescriptlang.org/)是一个遵循与Babel类似的设置的编译语言。作为JavaScript的补充，它支持类型定义使其更加整洁。一个很好的编辑器可以支持并提供更好的编辑体验。强制类型在开发中很有价值，因为它让你可以更加容易的声明约定的类型。

Compared to Facebook's type checker Flow, TypeScript is a more established option. As a result, you find more premade type definitions for it, and overall, the quality of support should be better.

同Facebook的类型检测器Flow相比，TypeScript是一个更被认可的选择。因此，你会发现更多的预先制定的类型定义，总体上来说，支持的质量应该更好。

{pagebreak}

You can use TypeScript with webpack using the following loaders:

你可以通过webpack结合如下加载器使用TypeScript：

* [ts-loader](https://www.npmjs.com/package/ts-loader)
* [awesome-typescript-loader](https://www.npmjs.com/package/awesome-typescript-loader)
* [light-ts-loader](https://www.npmjs.com/package/light-ts-loader)

T> There's a [TypeScript parser for ESLint](https://www.npmjs.com/package/typescript-eslint-parser). It's also possible to lint it through [tslint](https://www.npmjs.com/package/tslint).

T> 有一个[用于ESLint的TypeScript解析器]。还可以通过[tslint](https://www.npmjs.com/package/tslint)来进行语法检测。

## Setting Up Flow
## 设置Flow

[Flow](https://flow.org/) performs static analysis based on your code and its type annotations. You have to install it as a separate tool and then run it against your code. There's [flow-status-webpack-plugin](https://www.npmjs.com/package/flow-status-webpack-plugin) that allows you to run it through webpack during development.

[Flow](https://flow.org/)基于你的代码以及其类型注释进行静态分析。你必须单独安装这个工具，然后对你的代码执行它。[flow-status-webpack-plugin](https://www.npmjs.com/package/flow-status-webpack-plugin)允许你在开发环境中通过webpack运行它。

If you use React, the React specific Babel preset does most of the work through [babel-plugin-syntax-flow](https://www.npmjs.com/package/babel-plugin-syntax-flow). It can strip Flow annotations and convert your code into a format that is possible to transpile further.

如果你使用React，特定于React的Babel预设器通过[babel-plugin-syntax-flow](https://www.npmjs.com/package/babel-plugin-syntax-flow)执行了大部分工作。它可以剥离Flow注释，并将您的代码转换成可能进一步扩展的格式。

There's also [babel-plugin-typecheck](https://www.npmjs.com/package/babel-plugin-typecheck) that allows you to perform runtime checks based on your Flow annotations. [flow-runtime](https://codemix.github.io/flow-runtime/) goes a notch further and provides more functionality. These approaches complement Flow static checker and allow you to catch even more issues.

还可以使用[babel-plugin-typecheck](https://www.npmjs.com/package/babel-plugin-typecheck)在运行时基于你的Flow注释进行检查。[flow-runtime](https://codemix.github.io/flow-runtime/)更进一步并提供了更过功能。这些方法补充了Flow静态检查器，并让你您可以捕获更多的问题。

T> [flow-coverage-report](https://www.npmjs.com/package/flow-coverage-report) shows how much of your code is covered by Flow type annotations.

T> [flow-coverage-report](https://www.npmjs.com/package/flow-coverage-report)可以展示Flow类型检测覆盖了多少代码

## Conclusion
## 总结

Babel has become an indispensable tool for developers given it bridges the standard with older browsers. Even if you targeted modern browsers, transforming through Babel is an option.

Babel因其在标准和老的浏览器之间搭建了通行的桥梁，使其成为了一个开发者不可缺少的工具。即使你的目标仅是现代浏览器，通过Babel进行转换也是一个选择。

To recap:

回归：

* Babel gives you control over what browsers to support. It can compile ES6 features to a form the older browser understand. *babel-preset-env* is valuable as it can choose which features to compile and which polyfills to enable based on your browser definition.
* Babel让你可以控制哪些浏览器将被支持。它可以将ES6的特性编译成老浏览器可以理解的形式。*babel-preset-env*是一个很有价值的预设，它可以基于你的浏览器定义来选择支持哪些特性和模拟实现的功能。

* Babel allows you to use experimental language features. You can find numerous plugins that improve development experience and the production build through optimizations.
* Babel可以让你使用实验阶段的语言特性。你可以找到很多插件来提升开发体验，并通过优化来提升生产构建的性能。

* Babel functionality can be enabled per development target. This way you can be sure you are using the correct plugins at the right place.
* Babel的功能可以支持不同的开发目标。如此，你可以确保在正确定地方使用了正确的插件。

* Besides Babel, webpack supports other solutions like TypeScript of Flow. Flow can complement Babel while TypeScript represents an entire language compiling to JavaScript.
* 除了Babel，webpack还支持其他的方案如TypeScript何Flow。Flow仅作为Babel的补充，而TypeScript则表示可编译为JavaScript的整个语言。
