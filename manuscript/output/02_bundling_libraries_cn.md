# 打包代码库（Bundling Libraries）

To understand webpack's library targets better, you could set up a small library to bundle. The idea is to end up with a non-minified, a minified version, and a version compatible with *package.json* `module` field. The first two can be used for standalone consumption. You can also point to the non-minified version through *package.json* `main`.

为更好理解webpack的库目标，你可以配置一个小代码库用于打包。这一理念可以通过一个非压缩版本、压缩版本和兼容*package.json*的`module`字段的版本来完成。之前的两个可以被单独使用。你也可以通过*package.json*`main`字段来指向非压缩版本。

## 配置代码库（Setting Up a Library）

To have something to build, set up a module as follows:

为有东西可以构建，如下配置模块：

**lib/index.js**

```javascript
const add = (a, b) => a + b;

export {
  add,
};
```

The idea is that this file becomes the entry point for the entire library and represents the API exposed to the consumers. If you want to support both CommonJS and ES6, it can be a good idea to use the CommonJS module definition here. If you go with ES6 `export default`, using such an export in a CommonJS environment often requires extra effort.

该文件变成了整个代码库的入口并暴露API给使用者。如果你想支持CommonJS和ES6，理想的方式是使用CommonJS模块定义方式。如果你使用ES6的`export default`，在类似CommonJS的环境里使用这样的输出则需要一些额外的操作。

{pagebreak}

## 配置npm脚本（Setting Up a npm Script）

Given the `build` target of the project has been taken already by the main application, you should set up a separate one for generating the library. It points to a library specific configuration file to keep things nice and tidy.

考虑到项目的`构建`目标已经被主应用占据，你需要单独配置一个用于生成代码库。其指向代码库的具体配置文件已保持整体的整洁美观。

**package.json**

```json
"scripts": {
leanpub-start-insert
  "build:lib": "webpack --config webpack.lib.js",
leanpub-end-insert
  ...
},
```

## 配置Webpack（Setting Up Webpack）

Webpack configuration itself can be adapted from the one you built. This time, however, you have to generate two files - a non-minified version and a minified one. This can be achieved by running webpack in so called *multi-compiler mode*. It means you can expose an array of configurations for webpack and it executes each.

然而，这次你需要生成两个文件 - 一个非压缩版本和一个压缩版本。可以通过让webpack运行在一个称为*多编译器模式*下来实现。意味着你可以暴露一个包含配置的数组给webpack，其会依次执行每一项。

**webpack.lib.js**

```javascript
const path = require('path');
const merge = require('webpack-merge');

const parts = require('./webpack.parts');

const PATHS = {
  lib: path.join(__dirname, 'lib'),
  build: path.join(__dirname, 'dist'),
};

const commonConfig = merge([
  {
    entry: {
      lib: PATHS.lib,
    },
    output: {
      path: PATHS.build,
      library: 'Demo',
      libraryTarget: 'var', // Default
    },
  },
  parts.attachRevision(),
  parts.generateSourceMaps({ type: 'source-map' }),
  parts.loadJavaScript({ include: PATHS.app }),
]);

const libraryConfig = merge([
  commonConfig,
  {
    output: {
      filename: '[name].js',
    },
  },
]);

const libraryMinConfig = merge([
  commonConfig,
  {
    output: {
      filename: '[name].min.js',
    },
  },
  parts.minifyJavaScript({ useSourceMap: true }),
]);

module.exports = [
  libraryConfig,
  libraryMinConfig,
];
```

If you execute `npm run build:lib` now, you should see output:

若你此时执行`npm run build:lib`，你应该看到输出：

```bash
Hash: 760c4d25403432782e1079cf0c3f76bbd168a80c
Version: webpack 2.2.1
Child
    Hash: 760c4d25403432782e10
    Time: 302ms
         Asset     Size  Chunks             Chunk Names
        lib.js  2.96 kB       0  [emitted]  lib
    lib.js.map  2.85 kB       0  [emitted]  lib
       [0] ./lib/index.js 55 bytes {0} [built]
Child
    Hash: 79cf0c3f76bbd168a80c
    Time: 291ms
             Asset       Size  Chunks             Chunk Names
        lib.min.js  695 bytes       0  [emitted]  lib
    lib.min.js.map    6.72 kB       0  [emitted]  lib
       [0] ./lib/index.js 55 bytes {0} [built]
```

Webpack ran twice in this case. It can be argued that it would be smarter to minify the initial result separately. In this case, the overhead is so small that it's not worth the extra setup.

在这一例子中，webpack运行了两次。可以认为，单独压缩初始化结果更为理想。在这一情况中，性能损耗很小并不值得被提出配置。

{pagebreak}

Examining the build output reveals more:

再检查一次构建输出结果：

**dist/lib.js**

```javascript
/*! 33c69fc */
var Demo =
/******/ (function(modules) { // webpackBootstrap
...
/******/ })
/************************************************************************/
/******/ ([
/* 0 */
/***/ (function(module, __webpack_exports__, __webpack_require__) {

"use strict";
Object.defineProperty(__webpack_exports__, "__esModule", { value: true });
/* harmony export (binding) */ __webpack_require__.d(__webpack_exports__, "add", function() { return add; });
var add = function add(a, b) {
  return a + b;
};

/***/ })
/******/ ]);
//# sourceMappingURL=lib.js.map
```

You can see familiar code there and more. Webpack's bootstrap script is in place, and it starts the entire execution process. It takes the majority of space for a small library, but that's not a problem as the library begins to grow.

可以在这里发现更多熟悉的代码。Webpack的驱动脚本已放置在合适位置，其将启动整个脚本的执行过程。其占据了整个小代码库的主要空间，随着代码库的增长这并非是个问题。

T> Instead of using the multi-compiler mode, it would be possible to define two targets. One of them would generate the non-minified version while the other would generate the minified one. The other npm script could be called as `build:lib:dist` and you could define a `build:lib:all` script to build both.

T> 除了使用多编译器模式，其也能够定义两类目标。其中的一个生成非压缩的版本而另一个生成压缩版本。另一个npm脚本可以被称为`build:lib:dist`，同时也可以定义一个`build:lib:all`脚本来生成二者。

## 构建之前清理和校验（Cleaning and Linting Before Building）

It's a good idea to clean the build directory and lint the code before building the library. You could expand webpack configuration:

在构建代码库之前清理构建目录和校验代码格式是个好的想法。你可以扩展webpack配置：

```javascript
...

const libraryConfig = merge([
  commonConfig,
  {
    output: {
      filename: '[name].js',
    },
  },
leanpub-start-insert
  parts.clean(PATHS.build),
  parts.lintJavaScript({ include: PATHS.lib }),
leanpub-end-insert
]);

...
```

`parts.clean` and `parts.lintJavaScript` were included to `libraryConfig` on purpose as it makes sense to run them only once at the beginning of the execution. This solution would be problematic with *parallel-webpack* though as it can run configurations out of order.

`parts.clean` 和 `parts.lintJavaScript`有目的地被包含进了`libraryConfig`，因为其在执行开始时运行一次是有意义的。这一解决方案在*并发webpack*下运行是存在问题的，尽管其可以无序地运行配置。

T> There's [a proposal to improve the situation](https://github.com/webpack/webpack/issues/4271) by introducing the concepts of pre- and post-processing to webpack.

T> 这有一份通过介绍关于wepack的预处理和后期处理的概念用以提升[处境](https://github.com/webpack/webpack/issues/4271)。

{pagebreak}

## 通过npm进行清理和校验（Cleaning and Linting Through npm）

Another, and in this case a more fitting, way would be to handle the problem through an npm script. As discussing in the *Package Authoring Techniques* chapter, npm provides pre- and post-script hooks. To keep this solution cross-platform, install [rimraf](https://www.npmjs.com/package/rimraf) first:

此外，在这一例子中，一个更合适的处理该问题的方式是通过npm脚本。如在*包编程技术*一章的探讨，npm提供了预处理和后期处理脚本的钩子。为了让这一方案能够跨平台，首先安装 [rimraf](https://www.npmjs.com/package/rimraf)：

```bash
npm install rimraf --save-dev
```

Then, to remove the build directory and lint the source before building, adjust as follows:

而后，为在构建之前移除某并对源码进行校验，调整如下：

**package.json**

```json
"scripts": {
leanpub-start-insert
  "prebuild:lib": "npm run lint:js && rimraf dist",
leanpub-end-insert
  ...
},
```

If either process fails, npm doesn't proceed to the `lib` script. You can verify this by breaking a linting rule and seeing what happens when you build (`npm run build:lib`). Instead, it gives you an error.

若其中一个任务失败，npm则不再处理`lib`脚本。你可以破坏一个校验规则来验证并看看在你构建（`npm run build:lib`）时都发生了什么。其反而会提供一个错误。

T> To get cleaner error output, run either `npm run build:lib --silent` or `npm run build:lib -s`.

T> 为了得到一个更清晰的错误输出，运行`npm run build:lib --silent`或`npm run build:lib -s`。

T> The same idea can be used for post-processes, such as deployment. For example, you could set up a `postpublish` script to deploy the library site after you have published it to npm.

T> 相同的原理也可用于后期处理，如开发。例如，设置一段`postpublish`来部署代码库站点在发布到npm之后。

{pagebreak}

## 结论（Conclusion）

Webpack can be used for bundling libraries. You can use it to generate multiple different output files based on your exact needs.

Webpack可用于打包代码库。你可以基于明确的需求使用其生成多个不同输出文件。

To recap:

* If you bundle libraries with webpack, you should set the `output` options carefully to get the result you want.
* Webpack can generate both a non-minified and a minified version of a library through its **multi-compiler** mode. It's possible to minify also as a post-process using an external tool.
* Performing tasks, such as cleaning and linting JavaScript, while using the multi-compiler mode is problematic at the moment. Instead, it can be a good idea to handle these tasks outside of webpack or run multiple webpack instances separately.

本章回归：

* 若使用webpack打包代码库，你应该小心设置`output`项以获得你想要的结果。
* Webpack可以通过其**多编译器**模式生成一个非压缩版本和一个压缩版本的代码库。也能够使用额外的工具在后期进行压缩。
* 在此时使用多编译器模式运行如清理和校验JavaScript代码格式的任务是一个问题。相反，在webpack外处理这些任何或运行多个丢李的webpack实例是一个不错想法。

If you try to import *./dist/lib.js* through Node, you notice it emits `{}`. The problem has to do with the output type that was chosen. To understand better which output to use and why, the next chapter covers them in detail.

若你尝试通过Node引入 *./dist/lib.js*，你会注意到期触发为`{}`。这一问题在选择输出类型时已经产生。为更好理解选择使用哪一个输出类型及原因，下一章将包含详细内容。

T> The *Package Authoring Techniques* chapter discusses npm specific techniques in detail.

T> *包编程技术*一章详细讨论了npm的具体技术细节。
