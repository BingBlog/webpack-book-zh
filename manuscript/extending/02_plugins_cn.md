# 扩展插件（Extending with Plugins）

Compared to loaders, plugins are a more flexible means to extend webpack. You have access to webpack's **compiler** and **compilation** processes. It's possible to run child compilers and plugins can work in tandem with loaders as `ExtractTextPlugin` shows.

相较于加载器，对于扩展webpack插件是一个更为灵活的方式。可以访问webpack的**编译器**和**编辑**进程。如`ExtractTextPlugin`所示，子编译器和插件可以协同工作。

Plugins allow you to intercept webpack's execution through hooks. Webpack itself has been implemented as a collection of plugins. Underneath it relies on [tapable](https://www.npmjs.com/package/tapable) plugin interface that allows webpack to apply plugins in different ways.

插件允许你通过钩子拦截webpack的执行过程。webpack自身实现了一系列的插件。底层依赖于[tapable](https://www.npmjs.com/package/tapable)插件接口，其能让webpack以不同方式应用插件。

You'll learn to develop a couple of small plugins next. Unlike for loaders, there is no separate environment where you can run plugins so you have to run them against webpack itself. It's possible to push smaller logic outside of the webpack facing portion, though, as this allows you to unit test it in isolation.

你会在接下来学习开发一系列小的插件。不同于加载器，没有一个独立的环境能让你运行插件，因此你不得不通过webpack来运行他们。尽管如此，还是可以将更小的逻辑推到webpack外部，这可以独立地运行单元测试。

## Webpack插件的基本流程（The Basic Flow of Webpack Plugins）

A webpack plugin is expected to expose an `apply(compiler)` method. JavaScript allows multiple ways to do this. You could use a function and then attach methods to its `prototype`. To follow the newest syntax, you could use a `class` to model the same idea.

wepback插件会暴露一个`apply(compiler)`方法。JavaScript能够有多种方式来处理。可以使用一个函数，并将该函数绑定到其`prototype`上。为遵循最新语法，可以使用`class`来实现相同的理念。

Regardless of your approach, you should capture possible options passed by a user at the constructor. It's a good idea to declare a schema to communicate them to the user. [schema-utils](https://www.npmjs.com/package/schema-utils) allows validation and works with loaders too.

无论何种方式，你应该在构造函数中捕获用户可能传递的参数。声明一个用于和用户沟通的模式是一个不错的想法。[schema-utils](https://www.npmjs.com/package/schema-utils) 支持验证以及和加载器协作。

When the plugin is connected to webpack configuration, webpack will run its constructor and call `apply` with a compiler object passed to it. The object exposes webpack's plugin API and allows you to use its hooks as listed by [the official compiler reference](https://webpack.js.org/api/plugins/compiler/).

当插件连到wepback配置上时，webpack将会运行期构造函数并通过传递给它的编译器对象调用`apply`方法。该对象暴露了webpack的插件API并允许使用罗列在[官方编译器参考](https://webpack.js.org/api/plugins/compiler/)中的钩子。

T> [webpack-defaults](https://www.npmjs.com/package/webpack-defaults) works as a starting point for webpack plugins. It contains the infrastructure used to develop official webpack loaders and plugins.

T> [webpack-defaults](https://www.npmjs.com/package/webpack-defaults) 可以作为webpack插件的起点。其包含了被用于开发webpack官方加载器和插件的基础设施。

## 配置开发环境（Setting Up a Development Environment）

Since plugins have to be run against webpack, you have to set up one to run a demo plugin that will be developed further:

由于插件需要通过webpack来运行，你不得不配置用以运行的将进一步开发的插件样例：

**webpack.plugin.js**

```javascript
const path = require('path');

const DemoPlugin = require('./plugins/demo-plugin.js');

const PATHS = {
  lib: path.join(__dirname, 'lib'),
  build: path.join(__dirname, 'build'),
};

module.exports = {
  entry: {
    lib: PATHS.lib,
  },
  output: {
    path: PATHS.build,
    filename: '[name].js',
  },
  plugins: [
    new DemoPlugin(),
  ],
};
```

T> If you don't have a `lib` entry file set up yet, write one. The contents doesn't matter as long as it's JavaScript that webpack can parse.

T> 若目前仍没配置`lib`入口文件，写一个。内容无关紧要，只要是能被webpack解析的JavaScript即可。

To make it convenient to run, set up a build shortcut:

为方便运行，配置构建捷径：

**package.json**

```json
"scripts": {
leanpub-start-insert
  "build:plugin": "webpack --config webpack.plugin.js",
leanpub-end-insert
  ...
},
```

Executing it should result in an `Error: Cannot find module` failure as the actual plugin is still missing.

执行其应该会得到一个失败结果`Error: Cannot find module`，由于真正的插件仍缺失。

T> If you want an interactive development environment, consider setting up [nodemon](https://www.npmjs.com/package/nodemon) against the build. Webpack's own watcher won't work in this case.

T> 若想要一个交互式开发环境，考虑在构建环节配置 [nodemon](https://www.npmjs.com/package/nodemon)。Webpack自身的监听器在这一例子中并不会生效。

## 实现基础插件（Implementing a Basic Plugin）

The simplest plugin should do two things: capture options and provide `apply` method:

最简单地参数应该做两件事：捕获选项和提供`apply`方法：

**plugins/demo-plugin.js**

```javascript
module.exports = class DemoPlugin {
  apply() {
    console.log('applying ');
  }
};
```

If you run the plugin (`npm run build:plugin`), you should see `applying` message at console. Given most plugins accept options, it's a good idea to capture those and pass them to `apply`.

若你运行插件 (`npm run build:plugin`)，你应该在console日志中看到`applying`的信息。考虑到大多数插件接受参数，捕获并传递给`apply`方法是个不错的想法。

{pagebreak}

## 捕获选项（Capturing Options）

Options can be captured through a `constructor`:

选项可以通过`constructor`被捕获：

**plugins/demo-plugin.js**

```javascript
module.exports = class DemoPlugin {
  constructor(options) {
    this.options = options;
  }
  apply() {
    console.log('apply', this.options);
  }
};
```

Running the plugin now would result in `apply undefined` kind of message given no options were passed.

由于没有参数传递，现在运行插件会得到了类似`apply undefined`的信息。

Adjust the configuration to pass an option:

调整配置用以传递选项：

**webpack.plugin.js**

```javascript
module.exports = {
  ...
  plugins: [
leanpub-start-delete
    new DemoPlugin(),
leanpub-end-delete
leanpub-start-insert
    new DemoPlugin({ name: 'demo' }),
leanpub-end-insert
  ],
};
```

Now you should see `apply { name: 'demo' }` after running.

运行后，你现在应该看到`apply { name: 'demo' }`。

{pagebreak}

## 理解编译器和编译（Understanding Compiler and Compilation）

`apply` receives webpack's compiler as a parameter. Printing reveals more:

`apply` 接受webpack的编译器作为参数。打印输出更多信息：

**plugins/demo-plugin.js**

```javascript
module.exports = class DemoPlugin {
  constructor(options) {
    this.options = options;
  }
  apply(compiler) {
    console.log(compiler);
  }
};
```

After running, you should see a lot of data. Especially `options` should look familiar as it contains webpack configuration. You can also see familiar names like `records`.

运行之后，应该能看到大量数据。尤其是`options`看起来很熟悉，因为其包含了webpack配置。你应该也对一些名字如`records`看起来很熟悉。

If you go through webpack's [plugin development documentation](https://webpack.js.org/api/plugins/), you'll see a compiler provides a large amount of hooks. Each hook corresponds with a specific stage. For example, to emit files, you could listen to the `emit` event and then write.

若查阅webpack的[插件开发文档](https://webpack.js.org/api/plugins/)，你将会看到编译器提供大量的钩子函数。每个钩子函数对应某一具体场景。例如，为生成文件，你可以监听`emit`事件而后写入。

Change the implementation to listen and capture `compilation`:

调整实现以监听和捕获`compilation`：

**plugins/demo-plugin.js**

```javascript
module.exports = class DemoPlugin {
  constructor(options) {
    this.options = options;
  }
  apply(compiler) {
leanpub-start-delete
    console.log(compiler);
leanpub-end-delete
leanpub-start-insert
    compiler.plugin('emit', (compilation, cb) => {
      console.log(compilation);

      cb();
    });
leanpub-end-insert
  }
};
```

W> Forgetting the callback and running the plugin makes webpack fail silently!

W> 忘记回调函数并运行插件使得webpack会静默失败。

Running the build should show more information than before. This is because a compilation object contains whole dependency graph traversed by webpack. You have access to everything related to it here including entries, chunks, modules, assets, and more.

执行构建应该能看到比之前更多的信息。这是由于编译对象包含了webpack遍历的所有依赖图。你可以在这里访问所有相关的东西，包括入口，文件块，模块，资源和其他。

T> Many of the available hooks expose compilation, but sometimes they expose a more specific structure and it takes more specific study to understand those.

T> 大量可用钩子暴露了编译环节，但有时他们暴露了一个更为特定的结构并需要花费更多精力去了解。

T> Loaders have a dirty access to `compiler` and `compilation` through underscore (`this._compiler`/`this._compilation`).

T> 通过下划线（`this._compiler`/`this._compilation`），加载器具有访问`compiler` and `compilation`的脏权限。

## 通过编译写文件（Writing Files Through Compilation）

The `assets` object of compilation can be used for writing new files. You can also capture already created assets, manipulate them, and write them back.

To write an asset, you have to use [webpack-sources](https://www.npmjs.com/package/webpack-sources) file abstraction. Install it first:

```bash
npm install webpack-sources --save-dev
```

{pagebreak}

Adjust the code as follows to write through `RawSource`:

如下调整代码以通过`RawSource`写入：

**plugins/demo-plugin.js**

```javascript
const { RawSource } = require('webpack-sources');

module.exports = class DemoPlugin {
  constructor(options) {
    this.options = options;
  }
  apply(compiler) {
leanpub-start-insert
    const { name } = this.options;
leanpub-end-insert

    compiler.plugin('emit', (compilation, cb) => {
leanpub-start-delete
      console.log(compilation);
leanpub-end-delete
leanpub-start-insert
      compilation.assets[name] = new RawSource('demo');
leanpub-end-insert

      cb();
    });
  }
};
```

After building, you should see output:

构建之后，你应该看到输出：

```bash
Hash: 62abc7fe06a7360b9735
Version: webpack 2.2.1
Time: 58ms
 Asset     Size  Chunks             Chunk Names
lib.js  2.89 kB       0  [emitted]  lib
  demo  4 bytes          [emitted]
   [0] ./lib/index.js 49 bytes {0} [built]
```

If you examine *build/demo* file, you'll see it contains the word *demo* as per code above.

若你检查*build/demo*文件，你将会看到每段代码前都包含了单词*demo*。

T> Compilation has a set of hooks of its own as covered in [the official compilation reference](https://webpack.js.org/api/plugins/compiler/).

T> 编译有一套自己的钩子函数，其在[官方的编译文档](https://webpack.js.org/api/plugins/compiler/)中有详细介绍。

## 管理警告错误（Managing Warnings and Errors）

Plugin execution can be caused to fail by throwing (`throw new Error('Message')`). If you validate options, you can use this method.

通过抛出（`throw new Error('Message')`）可以让插件执行失败。若你验证选项，你可以使用这一方法。

In case you want to give the user a warning or an error message during compilation, you should use `compilation.warnings` and `compilation.errors`. Example:

若想在编译期间给用户一个警告或错误信息，应该使用`compilation.warnings`和`compilation.errors`。例如：

```javascript
compilation.warnings.push('warning');
compilation.errors.push('error');
```

There is no way pass information messages to webpack yet although there is [a logging proposal](https://github.com/webpack/webpack/issues/3996). If you want to use `console.log` for this purpose, push it behind a `verbose` flag. The problem is that `console.log` will print to stdout and it will end up in webpack's `--json` output as a result. A flag will allow the user to work around this problem.

尽管有[日志建议](https://github.com/webpack/webpack/issues/3996)，但目前仍然没办法将信息消息传递给wepback。若你想处于这一目的使用`console.log`，将其放在`verbose`之后。问题在于`console.log`将会打印系统输出，并以webpack的`--json`输出结果作为结束。该标识符允许用户采用变通方式来处理这一问题。

## 插件可以使用插件（Plugins Can Have Plugins）

A plugin can provide hooks of its own. [html-webpack-plugin](https://www.npmjs.com/package/html-webpack-plugin) uses plugins to extend itself as discussed in the *Getting Started* chapter.

插件可以提供自己的钩子函数。如*开始*一章所讨论，[html-webpack-plugin](https://www.npmjs.com/package/html-webpack-plugin)使用插件扩展其自身。

## 插件可以使用自带编译器（Plugins Can Run Compilers of Their Own）

In special cases, like [offline-plugin](https://www.npmjs.com/package/offline-plugin), it makes sense to run a child compiler. This gives full control over related entries and output. Arthur Stolyar, the author of the plugin has explained [the idea of child compilers at Stack Overflow](https://stackoverflow.com/questions/38276028/webpack-child-compiler-change-configuration).

在特殊例子中，如[offline-plugin](https://www.npmjs.com/package/offline-plugin)，其运行子编译器是有意义的。其对相关入口和输出提供了完成的控制权。Arthur Stolyar，插件作者已经[在StackOverflow解释了子编译器的理念](https://stackoverflow.com/questions/38276028/webpack-child-compiler-change-configuration)。

## 结论（Conclusion）

When you begin to design a plugin, spend time studying existing plugins that are close enough. Develop plugins piece-wise so that you validate one piece at a time. Studying webpack source can give more insight given it's a collection of plugins itself.

当开始设计插件时，花时间学习现有插件足够了。分段开发插件以让你可以在某一时刻可以验证一部分。了解webpack的资源可以提供更多视角，其本身就是个插件集合。

To recap:

* **Plugins** can intercept webpack's execution and extend it making them more flexible than loaders.
* Plugins can be combined with loaders. `ExtractTextPlugin` works this way. There loaders are used to mark assets to extract.
* Plugins have access to webpack's **compiler** and **compilation** processes. Both provide hooks for different stages of webpack's execution flow and allow you to manipulate it. This is how webpack itself works.
* Plugins can emit new assets and shape existing assets.
* Plugins can implement plugin systems of their own. `HtmlWebpackPlugin` is an example of a such plugin.
* Plugins can run compilers of their own. The isolation gives more control and allows plugins like *offline-plugin* to be written.

本章回顾：

* **插件**可以拦截webpack的执行过程并比加载器更易扩展。
* 插件可以同加载器相结合。`ExtractTextPlugin`以这一方式工作。这些加载器被用来标记资源用以提取。
* 插件可以有访问webpack的**编译器**和**编译**过程的权限。二者提供了在webpack不同的执行阶段流的钩子函数并允许你操作。这即webpack如何工作。
* 插件可以生成新资源和改造现有资源。
* 插件可以形成自己的插件系统。`HtmlWebpackPlugin`就是一个这样的插件。
* 插件可以运行自身的编译器。其隔离方案提供了更多的控制权同时允许写入类似*offline-plugin* 的插件。