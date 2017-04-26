# 性能（Performance）

Webpack's performance out of the box is often enough for small projects. That said, it begins to hit limits as your project grows in scale. It's a common topic in webpack's issue tracker. [Issue 1905](https://github.com/webpack/webpack/issues/1905) is a good example.

开箱即用的webpack性能通常对于小项目来说足够了。那就是说，随着项目规模的增长性能成为了瓶颈。这在webpack问题追踪里是个常见话题。[Issue 1905](https://github.com/webpack/webpack/issues/1905) 是个好的范例。

There are a couple of ground rules when it comes to optimization:

当谈到优化时，其存在一系列的基本法则：

1. Know what to optimize.
2. Perform fast to implement tweaks first.
3. Perform more involved tweaks after.
4. Measure impact.


1. 知道优化什么。
2. 优先实行调整。
3. 更进一步调整
4. 测量效果。

Sometimes optimizations come with a cost. They can make your configuration harder to understand or tie it to a particular solution. Often the best optimization is to do less work or do it in a smarter way. The basic directions are covered in the next sections, so you know where to look when it's time to work on performance.

有时优化会带来一定损失。他们会让你的配置难以理解或者与某一个特殊实现绑定。通常最优的优化方案是处理更少的工作或采取巧妙方式来解决。基本方法将在下一小节介绍，因此你需要知道了解哪里需要优化以及何时需要优化。

{pagebreak}

## 高阶优化（High-Level Optimizations）

Webpack uses only a single instance by default meaning you aren't able to benefit from a multi-core processor without extra effort. This where third party solutions, such as [parallel-webpack](https://www.npmjs.com/package/parallel-webpack) and [HappyPack](https://www.npmjs.com/package/happypack) come in.

Webpack默认仅使用单线程这意味着在没有额外的帮助下你无法从多核处理中获益。早已有第三方的解决方案提出，如[parallel-webpack](https://www.npmjs.com/package/parallel-webpack) and [HappyPack](https://www.npmjs.com/package/happypack)。

### parallel-webpack - 多webpack并发运行（parallel-webpack - Run Multiple Webpack's in Parallel）

*parallel-webpack* allows you to parallelize webpack configuration in two ways. Assuming you have defined your webpack configuration as an array, it can run the configurations in parallel. In addition to this, *parallel-webpack* can generate builds based on given **variants**.

*parallel-webpack* 允许你以两种方式并发运行webpack。假设你已将webpack定义为一个数组，其可以并发地运行。除此之外，*parallel-webpack*还可以基于**variants**生成构建文件。

Using variants allows you to generate both production and development builds at once. Variants also allow you to generate bundles with different targets to make them easier to consume depending on the environment. Variants can be used to implement feature flags when combined with `DefinePlugin` as discussed in the *Environment Variables* chapter.

使用variant参数能够让你一次性生成开发版本和生产版本。Variants能够让你根据不同的目标路径来生成代码包以方便根据环境来使用。当与在*环境变量*一章所探讨的`DefinePlugin`结合时，Variants还能被用于实现一些特性。

The underlying idea can be implemented using a [worker-farm](https://www.npmjs.com/package/worker-farm). In fact, *parallel-webpack* relies on *worker-farm* underneath.

这一根本理念可通过使用[worker-farm](https://www.npmjs.com/package/worker-farm)来实现。实际上，*parallel-webpack*在底层依赖于 *worker-farm*。

*parallel-webpack* can be used by installing it to your project as a development dependency and then replacing `webpack` command with `parallel-webpack`.

*parallel-webpack* 可以作为一个开发依赖项安装在你的项目中使用，而后将`webpack`命令替换为`parallel-webpack`。

{pagebreak}

### HappyPack - 文件级别并发（HappyPack - File Level Parallelism）

Compared to *parallel-webpack*, HappyPack is a more involved option. The idea is that HappyPack intercepts the loader calls you specify and then runs them in parallel. You have to set up the plugin first:

相较于*parallel-webpack*，HappyPack是一个更为复杂的选择。其原理是HappyPack会监听加载器（loader）对具体文件的每一次调用而后并发地运行。首先你必须先安装该插件：

**webpack.config.js**

```javascript
...
const HappyPack = require('happypack');

...

const commonConfig = merge([{
  {
    plugins: [
      new HappyPack({
        loaders: [
          // Capture Babel loader
          'babel-loader'
        ],
      }),
    ],
  },
}];
```

{pagebreak}

To complete the connection, you have to replace the original Babel loader definition with a HappyPack one:

为了能完成这一工作，你必须将原始的Babel加载器定义替换成HappyPack：

```javascript
exports.loadJavaScript = ({ include, exclude }) => ({
  module: {
    rules: [
      {
        ...
leanpub-start-delete
        loader: 'babel-loader',
leanpub-end-delete
leanpub-start-insert
        loader: 'happypack/loader',
leanpub-end-insert
        ...
      },
    ],
  },
});
```

The example above contains enough information for webpack to run the given loader parallel. HappyPack comes with more advanced options, but applying this idea is enough to get started.

关于webpack并发运行给定的加载器，之前的示例涵盖了足够的信息。HappyPack涵盖了更多选项，但应用这一理念足以开始。

Perhaps the problem with HappyPack is that it couples your configuration with it. It would be possible to overcome this issue by design and make it easier to inject. One option would be to build a higher level abstraction that can perform the replacement on top of vanilla configuration.

HappyPack可能存在的一个问题是和你的配置耦合在了一起。通过设计和灵活插入方式可以克服这一问题。一个可选项是构建更高级别的抽象以替换顶层的原始配置。

## 低阶优化（Low-Level Optimizations）

Certain lower-level optimizations can be good to know. The key is to allow webpack to perform less work. You have already implemented a couple of these, but it's a good idea to enumerate them:

某些低阶优化方式也应该了解。其关键在于让webpack处理更少的工作。你已经实现了其中的某一些方法，但最好还是将其列举出来：

* Consider using faster source map variants during development or skip them. Skipping is possible if you don't process the code in any way.
* Use [babel-preset-env](https://www.npmjs.com/package/babel-preset-env) during development instead of source maps to transpile fewer features for modern browsers and make the code more readable and easier to debug.
* Skip polyfills during development. Attaching a package, such as [babel-polyfill](https://www.npmjs.com/package/babel-polyfill), to the development version of an application adds to the overhead.
* Disable the portions of the application you don't need during development. It can be a valid idea to compile only a small portion you are working on as then you have less to bundle.
* Push bundles that change less to **Dynamically Loaded Libraries** (DLL) to avoid unnecessary processing. It's one more thing to worry about, but can lead to speed increases as there is less to bundle. The [official webpack example](https://github.com/webpack/webpack/tree/master/examples/dll-user) gets to the point while [Rob Knight's blog post](https://robertknight.github.io/posts/webpack-dll-plugins/) explains the idea further.

* 考虑在开发期间使用更快的source map或者忽略。如果你完全不需要在处理代码将其忽略也是可能的。
* 在开发环节使用[babel-preset-env](https://www.npmjs.com/package/babel-preset-env) 而不是source map来为现代浏览器转换较少的特性和让代码更易阅读和debug。
* 在开发期间忽略polyfills实现。增加一个类似[babel-polyfill](https://www.npmjs.com/package/babel-polyfill)的工具包在应用的开发版本的顶部。
* 禁用你在开发环节中不需要的应用代码。这一有效的理念能够仅编译你正在处理的一小部分代码，而这样你得到一个更小的代码包。
* 将更改较少的代码库推入**动态加载库**（DLL）中以避免不必要的处理。这是一件需要担心的问题，但由于更少的代码库能够让速度有所提升。[webpack官方示例](https://github.com/webpack/webpack/tree/master/examples/dll-user) 提到了这一点同时[Rob Knight's 的文章](https://robertknight.github.io/posts/webpack-dll-plugins/) 做了更进一步的阐述。

### 加载器和插件具体优化（Loader and Plugin Specific Optimizations）

There are a series of loader and plugin specific optimizations to consider:

一系列详细的加载器和插件优化方式值得考虑：

* Perform less processing by skipping loaders during development. Especially if you are using a modern browser, you can skip using *babel-loader* or equivalent altogether.
* Use either `include` or `exclude` with JavaScript specific loaders. Webpack traverses *node_modules* by default and executes *babel-loader* over the files unless it has been configured correctly.
* Utilize caching through plugins like [hard-source-webpack-plugin](https://www.npmjs.com/package/hard-source-webpack-plugin) to avoid unnecessary work. The caching idea applies to loaders as well. For example, you can enable cache on *babel-loader*.
* Use equivalent, but lighter alternatives, of plugins and loaders during development. Replacing `HtmlWebpackPlugin` with a [HtmlPlugin](https://gist.github.com/bebraw/5bd5ebbb2a06936e052886f5eb1e6874) that does far less is one direction.
* Consider using parallel variants of plugins if they are available. [webpack-uglify-parallel](https://www.npmjs.com/package/webpack-uglify-parallel) is one example.

* 通过在开发环境忽略加载器来处理更少的工作。尤其是如果你在使用现代浏览器，你可以将*babel-loader*或其他类似插件忽略。
* 在JavaScript加载器中使用`include`或`exclude`。Webpack默认情况下会遍历*node_modules*文件并通过*babel-loader*执行编译其文件，除非其已经被正确配置。
* 通过如[hard-source-webpack-plugin](https://www.npmjs.com/package/hard-source-webpack-plugin)来利用缓存以避免不必要的工作。缓存的理念同样可用于加载器。例如，可以让*babel-loader*支持缓存。
* 在开发环节，使用等价但是更轻便的插件和加载器。用[HtmlPlugin](https://gist.github.com/bebraw/5bd5ebbb2a06936e052886f5eb1e6874)替换`HtmlWebpackPlugin`则背离了这一方向。
* 如果可惜考虑使用支持并行化的插件。[webpack-uglify-parallel](https://www.npmjs.com/package/webpack-uglify-parallel) 就是一个例子。

## 开发环节中优化再打包速度（Optimizing Rebundling Speed During Development）

It's possible to optimize rebundling times during development by pointing the development setup to a minified version of a library, such as React. In React's case, you lose `propType`-based validation. But if speed is more important, this technique is worth a go.

可以在开发环节通过将开发环境中的库将其指向一个压缩版本，如React。在React这一案例中，你失去了基本的`propType`验证。但如果速度更为重要的话，该技术是值得尝试的。

`module.noParse` accepts a RegExp or an array of RegExps. In addition to telling webpack not to parse the minified file you want to use, you also have to point `react` to it by using `resolve.alias`. The aliasing idea is discussed in detail in the *Package Consuming Techniques* chapter.

`module.noParse`接受一个正则表达式或一个包含正则表达式的数组。除了高速webpack不要解析你想要用的压缩文件外，你还应该通过`reslve.alias`将其指`react`指向它。关于别名这一概念在*代码包使用*章节详细讨论。

It's possible to encapsulate the core idea within a function:

可以将核心理念封装在一个函数中：

```javascript
exports.dontParse = ({ name, path }) => {
  const alias = {};
  alias[name] = path;

  return {
    module: {
      noParse: [
        new RegExp(path),
      ],
    },
    resolve: {
      alias,
    },
  };
};
```

{pagebreak}

To use the function, you would call it as follows:

为了使用该函数，你可以以如下方式调用：

```javascript
dontParse({
  name: 'react',
  path: path.resolve(
    __dirname, 'node_modules/react/dist/react.min.js',
  ),
}),
```

After this change, the application should be faster to rebuild depending on the underlying implementation. The technique can also be applied during production.

在处理这一改变之后，依赖于之前的实现，应用代码的再构建应该快了不少。这一技术也可以被用于生产环节。

Given `module.noParse` accepts a regular expression if you wanted to ignore all `*.min.js` files, you could set it to `/\.min\.js/`.

考虑到`module.noParse`接受正则表达式，若你想要忽略所有的`.min.js`文件，你可以将其设置为`/\.min\.js/`。

W> Not all modules support `module.noParse`. They should not have a reference to `require`, `define`, or similar, as that leads to an `Uncaught ReferenceError: require is not defined` error.

W> 不是所有模块都支持`module.noParse`。他们不应该包含`require`、`define`或其他类似的引用，否则会导致`Uncaught ReferenceError: require is not defined`的错误。

## 结论（Conclusion）

You can optimize webpack's performance in multiple ways. Often it's a good idea to start with easier techniques before moving to more involved ones. The exact methods you have to use, depend on the project.

你可以通过多种方式优化webpack的性能。通常一个好的方式是在使用更为复杂的技术前从简单的技术开始。你必须根据项目来采用具体的技术。

To recap:

* Start with higher level techniques that are fast to implement first.
* Lower level techniques are more involved but come with their wins.
* Since webpack runs using a single instance by default, parallelizing is worthwhile.
* Especially during development, skipping work can be acceptable thanks to modern browsers.

本章回顾：

* 首先进行高阶优化因为速度优先。
* 低阶优化技术更复杂但他们能更有效。
* 由于webpack默认仅能在单进程中使用，因而并发是值得的。
* 尤其在开发环节，由于现代浏览器忽略一些工作是可取的。
