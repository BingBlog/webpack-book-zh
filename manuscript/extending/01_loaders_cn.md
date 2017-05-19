# 扩展加载器（Extending with Loaders）

As you have seen so far, loaders are one of the building blocks of webpack. If you want to load an asset, you most likely need to set up a matching loader definition. Even though there are a lot of [available loaders](https://webpack.js.org/loaders/), it's possible you are missing one fitting your purposes.

如当前所见，加载器是webpack构建的一部分。若你想要加载资源，你很有可能需要配置一个相匹配的加载器。即使已经有很多[可用的加载器](https://webpack.js.org/loaders/)，但可能还是缺少能够满足你的条件一个。

You'll learn to develop a couple of small loaders next. But before that, it's good to understand how to debug them in isolation.

你将会在接下来了解如何开发一系列小的加载器。但在那之前，最好还是先了解下如何单独对他们进行调试。

T> If you want a good starting point for a standalone loader or plugin project, consider using [webpack-defaults](https://github.com/webpack-contrib/webpack-defaults). It provides an opinionated starting point that comes with linting, testing, and other goodies.

T> 若你想对单独的加载器或插件项目有一个好的起点，考虑使用[webpack-defaults](https://github.com/webpack-contrib/webpack-defaults)。其提供了一个关于代码格式校验，测试机其他相关等一个相对不错的起点。

## 通过*loader-runner*调试加载器（Debugging Loaders with *loader-runner*）

[loader-runner](https://www.npmjs.com/package/loader-runner) allows you to run loaders without webpack making it a good tool for learning more about loader development. Install it first:

[loader-runner](https://www.npmjs.com/package/loader-runner) 允许在没有webpack情况下运行加载器，使得其成为一个了解加载器开发不错的学习工具。首先进行安装：

```bash
npm install loader-runner --save-dev
```

{pagebreak}

To have something to test with, set up a loader that returns twice what's passed to it:

为了能够进行测试，配置一个返回传入参数两倍的值的加载器：

**loaders/demo-loader.js**

```javascript
module.exports = (input) => input + input;
```

Set up a file to process:

配置一个文件用以处理：

**demo.txt**

```
foobar
```

There's nothing webpack specific in the code yet. The next step is to run the loader through *loader-runner*:

目前仍没有具体的webpack代码。下一步是通过*loader-runner*运行加载器：

**run-loader.js**

```javascript
const fs = require('fs');
const path = require('path');
const { runLoaders } = require('loader-runner');

runLoaders({
  resource: './demo.txt',
  loaders: [
    path.resolve(__dirname, './loaders/demo-loader'),
  ],
  readResource: fs.readFile.bind(fs),
},
(err, result) => err ?
  console.error(err) :
  console.log(result)
);
```

{pagebreak}

If you run the script now (`node run-loader.js`), you should see output:

若现在运行脚本（`node run-loader.js`），你应该看到输出：

```javascript
{ result: [ 'foobar\nfoobar\n' ],
  resourceBuffer: <Buffer 66 6f 6f 62 61 72 0a>,
  cacheable: true,
  fileDependencies: [ './demo.txt' ],
  contextDependencies: [] }
```

The output tells the `result` of the processing, the resource that was processed as a buffer, and other meta information. This is enough to develop more complicated loaders.

输出告诉处理的`结果`，被处理的资源作为一个缓冲和其他元信息。足以用来开发更为复杂的加载器。

T> If you want to capture the output to a file, use either `fs.writeFileSync('./output.txt', result.result)` or its asynchronous version as discussed in [Node documentation](https://nodejs.org/api/fs.html).

T> 若你要捕获输出到一个文件，使用`fs.writeFileSync('./output.txt', result.result)`或在[Node 文档](https://nodejs.org/api/fs.html)提及的它的异步版本。

T> It's possible to refer to loaders installed to the local project by name instead of resolving a full path to them. Example: `loaders: ['raw-loader']`.

T> 可以通过名字来引用安装到本地项目的加载器而非其完整路径。如：`loaders: ['raw-loader']`。

## 实现一个异步加载器（Implementing an Asynchronous Loader）

Even though you can implement a lot of loaders using the synchronous interface, there are times when asynchronous calculation is required. Wrapping a third party package as a loader can force you to this.

即使你可以使用同步接口来实现大量加载器，但有时仍需要异步计算。包装一个第三方吧作为一个加载器能够帮助你处理此。

The example above can be adapted to asynchronous form by using webpack specific API through `this.async()`. Webpack sets this and the function returns a callback following Node conventions (error first, result second).

通过使用webpack的特定API`this.async()`可以将之前的例子调整成为异步形式。Webpack这样设置并返回一个遵循Node规范的回调函数（错误优先，结果第二）。

{pagebreak}

Tweak as follows:

调整如下：

**loaders/demo-loader.js**

```javascript
module.exports = function(input) {
  const callback = this.async();

  // No callback -> return synchronous results
  // if (callback) { ... }

  callback(null, input + input);
};
```

W> Given webpack injects its API through `this`, the shorter function form (`() => ...`) cannot be used here.

W> 由于webpack通过`this`来注入器API，因此短函数形式（`() => ...`）无法在这里使用。

T> If you want to pass a source map to webpack, pass it as the third parameter of the callback.

T> 若你想传递source map给webpack，将其作为回调函数的第三个参数传递进去。

Running the demo script (`node run-loader.js`) again should give exactly the same result as before. To raise an error during execution, try the following:

再次运行展示脚本（`node run-loader.js`）应该能够看到和之前一样的结果。为了在运行期间触发异常，可以做以下尝试：

**loaders/demo-loader.js**

```javascript
module.exports = function(input) {
  const callback = this.async();

  callback(new Error('Demo error'));
};
```

The result should contain `Error: Demo error` with a stack trace showing where the error originates.

结果应该包含带有显示错误源的堆栈跟踪信息`Error: Demo error`。

{pagebreak}

## 仅返回输出（Returning Only Output）

Loaders can be used to output code alone. You could have implementation as below:

加载器可以被用于仅返回代码。可以做如下调整：

**loaders/demo-loader.js**

```javascript
module.exports = function() {
  return 'foobar';
};
```

But what's the point? You can pass to loaders through webpack entries. Instead of pointing to pre-existing files as you would in majority of the cases, you could pass to a loader that generates code dynamically.

但什么是关键？你可以通过webpack入口传递给加载器。而不是和大多数情况一样将其指向预生成文件，可以将动态生成的代码传递给加载器。

T> If you want to return `Buffer` output, set `module.exports.raw = true`. The flag overrides the default behavior which expects a string is returned.

T> 若想输出`Buffer`，设置`module.exports.raw = true`。该标志重写了返回字符串的默认行为。

## 写文件（Writing Files）

Loaders, like *file-loader*, emit files. Webpack provides a single method, `this.emitFile`, for this. Given *loader-runner* does not implement it, you have to mock it:

加载器，如 *file-loader*，生成文件。webpack提供一个单独的方法，`this.emitFile`用于处理这个。考虑到*loader-runner*并没有实现该功能，因此需要模仿一个：

**run-loader.js**

```javascript
const fs = require('fs');
const { runLoaders } = require('loader-runner');

runLoaders({
  resource: './demo.txt',
  loaders: [
    path.resolve(__dirname, './loaders/demo-loader'),
  ],
leanpub-start-insert
  context: {
    emitFile: () => {},
  },
leanpub-end-insert
  readResource: fs.readFile.bind(fs),
},
(err, result) => err ?
  console.error(err) :
  console.log(result)
);
```

To implement the essential idea of *file-loader*, you have to do two things: emit the file and return path to it. You could implement it as below:

为实现*file-loader*的基本原理，不得不处理两件事：生成文件以及返回其所在路径。可以通过如下方式实现：

**loaders/demo-loader.js**

```javascript
const loaderUtils = require('loader-utils');

module.exports = function(content) {
  const url = loaderUtils.interpolateName(
    this, '[hash].[ext]', { content }
  );

  this.emitFile(url, content);

  const path = `__webpack_public_path__ + ${JSON.stringify(url)};`;

  return `export default ${path}`;
};
```

Webpack provides two additional `emit` methods:

Webpack提供两个额外的`emit`方法：

* `this.emitWarning(<string>)`
* `this.emitError(<string>)`

These calls should be used over `console` based alternatives. As with `this.emitFile`, you have to mock them for *loader-runner* to work.

这些调用应该可以用于`console`的替代方案，与`this.emitFile`一致，你不得不为*loader-runner*模拟它们以能正常工作。

The next question is, how to pass file name to the loader.

下一个问题是，如何传递文件名给加载器。

{pagebreak}

## 传递选项给加载器（Passing Options to Loaders）

To demonstrate passing options, the runner needs a small tweak:

为演示传递选项，代码需要一些微调：

**run-loader.js**

```javascript
runLoaders({
  resource: './demo.txt',
  loaders: [
leanpub-start-delete
    path.resolve(__dirname, './loaders/demo-loader')
leanpub-end-delete
leanpub-start-insert
    {
      loader: path.resolve(__dirname, './loaders/demo-loader'),
      options: {
        name: 'demo.[ext]',
      },
    },
leanpub-end-insert
  ],
  ...
},
...
);
```

To capture the option, you need to use [loader-utils](https://www.npmjs.com/package/loader-utils). It has been designed to parse loader options and queries. Install it:

为捕获选项，需要使用[loader-utils](https://www.npmjs.com/package/loader-utils)。其被设计用来解析加载器选项和查询。安装：

```bash
npm install loader-utils --save-dev
```

{pagebreak}

To connect it to the loader, set it to capture `name` and pass it through webpack's interpolator:

为将其连接到加载器，将其配置用以捕获`name`参数和通过webpack的插入器传递它：

**loaders/demo-loader.js**

```javascript
const loaderUtils = require('loader-utils');

module.exports = function(content) {
leanpub-start-insert
  const { name } = loaderUtils.getOptions(this);
leanpub-end-insert
  const url = loaderUtils.interpolateName(
leanpub-start-delete
    this, '[hash].[ext]', { content }
leanpub-end-delete
leanpub-start-insert
    this, name, { content }
leanpub-end-insert
  );

  this.emitFile(url, content);

  const filePath = `__webpack_public_path__+${JSON.stringify(url)};`;

  return `export default ${filePath}`;
};
```

After running (`node ./run-loader.js`), you should see something:

运行（`node ./run-loader.js`）后，你应该看到些东西：

```javascript
{ result: [ 'export default __webpack_public_path__+"demo.txt";' ],
  resourceBuffer: <Buffer 66 6f 6f 62 61 72 0a>,
  cacheable: true,
  fileDependencies: [ './demo.txt' ],
  contextDependencies: [] }
```

You can see that the result matches what the loader should have returned. You can try to pass more options to the loader or use query parameters to see what happens with different combinations.

你可以看到与加载器应该返回的相匹配的结果。你可以尝试传递更多选项给加载器或使用查询参数以查看不同组合产生什么结果。

T> It's a good idea to validate options and rather fail hard than silently if the options aren't what you expect. [schema-utils](https://www.npmjs.com/package/schema-utils) has been designed for this purpose.

## 通过Webpack连接自定义加载器（Connecting Custom Loaders with Webpack）

To get most out of loaders you have to connect them with webpack. To achieve this, you can go through imports:

为充分利用加载器，你不得不将其和webpack拼接。为实现这个，你可以通过引入机制：

**app/component.js**

```javascript
leanpub-start-insert
import '!../loaders/demo-loader?name=foo!./main.css';
leanpub-end-insert
```

Given the definition is verbose, the loader can be aliased:

考虑到定义过于冗杂，加载器可以被简写：

**webpack.config.js**

```javascript
const commonConfig = merge([
  {
  ...
leanpub-start-insert
    resolveLoader: {
      alias: {
        'demo-loader': path.resolve(
          __dirname, 'loaders/demo-loader.js'
        ),
      },
    },
leanpub-end-insert
  },
  ...
]);
```

With this change the import can be simplified:

随着这一改变，引入可以被简化：

```javascript
leanpub-start-delete
import '!../loaders/demo-loader?name=foo!./main.css';
leanpub-end-delete
leanpub-start-insert
import '!demo-loader?name=foo!./main.css';
leanpub-end-insert
```

{pagebreak}

You could also handle the loader definition through `rules`. Once the loader is stable enough, set up a project based on *webpack-defaults*, push the logic there, and begin to consume the loader as a package.

你也可以通过`rules`来处理加载器定义。一旦加载器足够稳定，基于*webpack-defaults*配置项目，将逻辑放入这里，而后开始将加载器作为包来使用。

W> Although using *loader-runner* can be convenient for developing and testing loaders, implement integration tests that run against webpack. Subtle differences between environments make this essential.

W> 尽管使用*loader-runner*非常便于开发和测试加载器，实现针对webpack的集成测试。环境减细微不同使得其变得关键。

## 捕获加载器（Pitch Loaders）

Webpack evaluates loaders in two phases: pitching and running. If you are used to web event semantics, these map to capturing and bubbling. The idea is that webpack allows you to intercept execution during the pitching (capturing) phase. It goes through the loaders left to right first and executes them from right to left after that.

Webpack在两个阶段执行加载器：捕获和运行。若习惯于web事件的语义，其分别对应捕获和冒泡。webpack的这一理念允许在捕获阶段拦截执行。其第一次自左向右运行插件，而后自右向左运行。

A pitch loader allows you shape the request and even terminate it. Set it up:

拦截加载器允许你修改请求甚至是中断。配置：

**loaders/pitch-loader.js**

```javascript
const loaderUtils = require('loader-utils');

module.exports = function(input) {
  const { text } = loaderUtils.getOptions(this);

  return input + text;
};
module.exports.pitch = function(remainingReq, precedingReq, input) {
  console.log(`
Remaining request: ${remainingReq}
Preceding request: ${precedingReq}
Input: ${JSON.stringify(input, null, 2)}
  `);

  return 'pitched';
};
```

{pagebreak}

To connect it to the runner, add it to the loader definition:

为将其和运行脚本连接，将其添加至加载器中：

**run-loader.js**

```javascript
runLoaders({
  resource: './demo.txt',
  loaders: [
    ...
leanpub-start-insert
    path.resolve(__dirname, './loaders/pitch-loader'),
leanpub-end-insert
  ],
  ...
},
...
);
```

If you run (`node ./run-loader.js`) now, the pitch loader should log intermediate data and intercept the execution:

若此刻运行（`node ./run-loader.js`），捕获加载器应该会打印中间数据并拦截运行：

```javascript
Remaining request: ./demo.txt
Preceding request: .../webpack-demo/loaders/demo-loader?{"name":"demo.[ext]"}
Input: {}

{ result: [ 'export default __webpack_public_path__ + "demo.txt";' ],
  resourceBuffer: null,
  cacheable: true,
  fileDependencies: [],
  contextDependencies: [] }
```

T> The [official documentation](https://webpack.js.org/api/loaders/) covers the loader API in detail. You can see all fields available through `this` there.

[官方文档](https://webpack.js.org/api/loaders/)涵盖了详细的加载器API。你可以在这里查看绑定在`this`上的所有可用字段。

## 通过加载器缓存（Caching with Loaders）

Although webpack caches loaders by default unless they set `this.cacheable(false)`, writing a caching loader can be a good exercise as it helps you to understand how loader stages can work together. The example below shows how to achieve this (courtesy of Vladimir Grenaderov):

尽管webpack默认缓存加载器，除非设置`this.cacheable(false)`。写一个缓存加载器是一个很好的联系，因为能帮助理解装载结果如何协同工作。如下例子表明了如何实现（由Vladimir Grenaderov提供）：

```javascript
const cache = new Map();

module.exports = function(content) {
  // Calls only once for given resourcePath
  const callbacks = cache.get(this.resourcePath);
  callbacks.forEach(callback => callback(null, content));

  cache.set(this.resourcePath, content);

  return content;
};
module.exports.pitch = function() {
  if (cache.has(this.resourcePath)) {
    const item = cache.get(this.resourcePath);

    if (item instanceof Array) {
      // Load to cache
      item.push(this.async());
    } else {
      // Hit cache
      return item;
    }
  } else {
    // Missed cache
    cache.set(this.resourcePath, []);
  }
};
```

A pitch loader can be used to attach metadata to the input to use later. In this example, cache was constructed during the pitching stage and it was accessed during normal execution.

捕获加载器可以备用将元信息绑定到输入中以便稍后使用。在这一例子中，在拦截期间缓存被构建并在正常执行期间被访问。

## 总结（Conclusion）

Writing loaders is fun in the sense that they describe transformations from a format to another. Often you can figure out how to achieve something specific by either studying either the API documentation or the existing loaders.

编写加载器在某种意义上挺有趣。他们描述了从一种形式到另一种形式的转换。通常你通过学习或者API文档或者目前的加载器能够明白如何具体实现。

To recap:

* *loader-runner* is a valuable tool for understanding how loaders work. Use it for debugging how loaders work.
* Webpack **loaders** accept input and produce output based on it.
* Loaders can be either synchronous or asynchronous. In the latter case, you should use `this.async()` webpack API to capture the callback exposed by webpack.
* If you want to generate code dynamically for webpack entries, that's where loaders can come in handy. A loader does not have to accept input. It's acceptable that it returns only output in this case.
* Use **loader-utils** to parse possible options passed to a loader and consider validating them using **schema-utils**.
* When developing loaders locally, consider setting up a `resolveLoader.alias` to clean up references.
* Pitching stage complements the default behavior allowing you to intercept and to attach metadata.

本章回顾：

*  *loader-runner*对于了解加载器如何工作是个十分有价值的工具。通过它调试加载器如何工作。
*  Webpack**加载器**接收输入并基于此产生输出。
*  加载器可以同步或异步。在之后的例子，你应该使用webpack的`this.async()`API用以捕获webpack暴露的回调函数。
*  若想要为webpack入口动态生成代码，那对于加载器来说很方便。加载器并不要求接收输出。在这一情况下，仅返回输出是能接受的。
*  使用**loader-utils**以解析可能被传递给加载器的选项，同时使用 **schema-utils**用于验证合法性。
*  当本地开发加载器，考虑设置`resolveLoader.alias`以清理引用。
*  捕获阶段完善了默认行为，允许你截取和附加元数据。

You'll learn to write plugins in the next chapter. Plugins allow you to intercept webpack's execution process and they can be combined with loaders to develop more advanced functionality.

你会在下一章节了解编写插件。插件允许你阶段webpack的执行过程，同时能和加载器结合开发更高级的功能。
