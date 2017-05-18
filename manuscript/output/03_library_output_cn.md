# 代码库输出（Library Output）

The example of the previous chapter can be expanded further to study webpack's library output options in detail.

上一章的例子可以进一步扩展以详细了解webpack的代码库输出项。

The library target is controlled through the `output.libraryTarget` field. `output.library` comes into play as well and individual targets have additional fields related to them.

代码库目标通过`output.libraryTarget`字段控制。`output.library`也可以在个别与之相关的其他领域发挥作用。

## `var`

The demonstration project of ours uses `var` by default through configuration:

我们演示项目在配置中默认使用了`var`：

**webpack.lib.js**

```javascript
const commonConfig = merge([
  {
    ...
    output: {
      path: PATHS.build,
      library: 'Demo',
      libraryTarget: 'var',
    },
  },
  ...
]);
```

{pagebreak}

The output configuration maps `library` and `libraryTarget` to the following form:

输出配置将`library`和`libraryTarget`映射到如下形式：

```javascript
var Demo =
/******/ (function(modules) { // webpackBootstrap
...
/******/ })
...
/******/ ([
  ...
/******/ ]);
//# sourceMappingURL=lib.js.map
```

This tells it generates `var <output.library> = <webpack bootstrap>` kind of code and also explains why importing the code from Node does not give access to any functionality.

这表明其生成类似`var <output.library> = <webpack bootstrap>`类似代码并也解释了为何从Node中引入的代码无法使用任何功能。

## `window`, `global`, `assign`, `this`

Most of the available options vary the first line of the output as listed below:

* `window` - `window["Demo"] =`
* `global` - `global["Demo"] =`
* `assign` - `Demo =` - If you executed this code in the right context, it would associate it to a global `Demo`.
* `this` - `this["Demo"] =` - Now the code would associate to context `this` and work through Node.

大多数可选项如下列所示会改变输出的第一行：

* `window` - `window["Demo"] =`
* `global` - `global["Demo"] =`
* `assign` - `Demo =` - 若在正确地上下文中执行该代码，其会关联到全局的`Demo`。
* `this` - `this["Demo"] =` - 现在代码会关联到`this`的上下文中并能通过Node运行。

T> You can try running the resulting code through Node REPL (use `node` at project root and use `require('./dist/lib')`).

T> 你可以通过Node REPL 尝试运行结果代码（在项目根路径使用`node`同时使用`require('./dist/lib')`）。

## CommonJS

The CommonJS specific targets are handy when it comes to Node. There are two options: `commonjs` and `commonjs2`. These refer to different interpretations of the [CommonJS specification](http://wiki.commonjs.org/wiki/CommonJS). Let's explore the difference.

当涉及Node时，CommonJS特定目标就会变得很方便。有两个可选项：`commonjs` 和 `commonjs2`。涉及到对[CommonJS 规范](http://wiki.commonjs.org/wiki/CommonJS)不同的解释。现在我们就探究下差异。

### `commonjs`

If you used the `commonjs` option, you would end up with code that expects global `exports` in which to associate:

若你使用`commonjs`选项，则代码会期望用全局的`exports`作为结束以用来关联：

```javascript
exports["Demo"] =
...
```

If this code was imported from Node, you would get `{ Demo: { add: [Getter] } }`.

如果这段代码是从Node中引入，则会得到 `{ Demo: { add: [Getter] } }`。

### `commonjs2`

`commonjs2` expects a global `module.exports` instead:

`commonjs2`则期望一个全局的`module.exports`：

```javascript
module.exports =
...
```

The library name, `Demo`, isn't used anywhere. As a result importing the module yields `{ add: [Getter] }`. You lose the extra wrapping.

代码库名`Demo`不再在任何地方被用。最终引入模块生成`{ add: [Getter] }`。丢弃了额外的包装。

## AMD

If you remember [RequireJS](http://requirejs.org/), you recognize the AMD format it uses. In case you can use the `amd` target, you get output:

如果你记得 [RequireJS](http://requirejs.org/)，你应该能认得出它使用的AMD格式。在若你使用`amd`，你会得到输出：

```javascript
define("Demo", [], function() { return /******/ (function(modules) { // webpackBootstrap
...
```

In other words, webpack has generated a named AMD module. The result doesn't work from Node as there is no support for the AMD format.

换句话说，webpack可以生成一个命名的AMD模块。该结果并不能在Node中运行，由于其不支持ADM格式。

## UMD

[Universal Module Definition](https://github.com/umdjs/umd) (UMD) was developed to solve the problem of consuming the same code from different environments. Webpack implements two output variants: `umd` and `umd2`. To understand the idea better, let's see what happens when the options are used.

### `umd`

Basic UMD output looks complicated:

基础的UMD输出看起来比较复杂：

```javascript
(function webpackUniversalModuleDefinition(root, factory) {
  if(typeof exports === 'object' && typeof module === 'object')
    module.exports = factory();
  else if(typeof define === 'function' && define.amd)
    define("Demo", [], factory);
  else if(typeof exports === 'object')
    exports["Demo"] = factory();
  else
    root["Demo"] = factory();
})(this, function() {
```

The code performs checks based on the environment and figures out what kind of export to use. The first case covers Node, the second is for AMD, the third one for Node again, while the last one includes a global environment.

代码会基于当前环境进行检查并识别出所使用的导出类型。第一个涵盖了Node，第二个针对AMD。最后一个则再次包含Node，但最后一个也引入了一个全局环境。

The output can be modified further by setting `output.umdNamedDefine: false`:

通过设置`output.umdNamedDefine: false`，输出可以被进一步修改:

```javascript
(function webpackUniversalModuleDefinition(root, factory) {
  if(typeof exports === 'object' && typeof module === 'object')
    module.exports = factory();
  else if(typeof define === 'function' && define.amd)
    define([], factory);
  else if(typeof exports === 'object')
    exports["Demo"] = factory();
  else
    root["Demo"] = factory();
})(this, function() {
...
```

To understand `umd2` option, you have to understand *optional externals* first.

为了解`umd2`，你必须首先了解*可选扩展*：

### 可选扩展（Optional Externals）

In webpack terms, externals are dependencies that are resolved outside of webpack and are available through the environment. Optional externals are dependencies that can exist in the environment, but if they don't, they get skipped instead of failing hard.

在webpack配置项中，扩展是在webpack外部所需要的且在当前环境可用的依赖。可选扩展是存在于当前环境的依赖，若他们不存在，则会被忽略而不是发生错误。

Consider the following example where jQuery is loaded if it exists:

考虑如下例子，当jQuery存在时加载jQuery：

**lib/index.js**

```javascript
var optionaljQuery;
try {
  optionaljQuery = require('jquery');
} catch(err) {} // eslint-disable-line no-empty

function add(a, b) {
  return a + b;
}

export {
  add,
};
```

To treat jQuery as an external, you should configure as follows:

为将jQuery作为一个扩展，你应该做如下配置

```javascript
{
  externals: {
    jquery: 'jQuery',
  },
},
```

If `libraryTarget: 'umd'` is used after these changes, you get output:

若在这些改变之后使用`libraryTarget: 'umd'`，得到输出：

```javascript
(function webpackUniversalModuleDefinition(root, factory) {
  if(typeof exports === 'object' && typeof module === 'object')
    module.exports = factory((
      function webpackLoadOptionalExternalModule() {
        try { return require("jQuery"); } catch(e) {}
      }())
    );
  else if(typeof define === 'function' && define.amd)
    define(["jQuery"], factory);
  else if(typeof exports === 'object')
    exports["Demo"] = factory((
      function webpackLoadOptionalExternalModule() {
        try { return require("jQuery"); } catch(e) {}
      }())
    );
  else
    root["Demo"] = factory(root["jQuery"]);
})(this, function(__WEBPACK_EXTERNAL_MODULE_0__) {
return /******/ (function(modules) { // webpackBootstrap
...
```

Webpack wrapped the optional externals in `try`/`catch` blocks.

webpack将可选扩展包装在 `try`/`catch`代码块中。

{pagebreak}

### `umd2`

To understand what the `umd2` option does, consider the following output:

为了解`umd2`选项做了什么，考虑如下输出：

```javascript
/*! fd0ace9 */
(function webpackUniversalModuleDefinition(root, factory) {
  if(typeof exports === 'object' && typeof module === 'object')
    module.exports = factory((
      function webpackLoadOptionalExternalModule() {
        try { return require("jQuery"); } catch(e) {}
      }())
    );
  else if(typeof define === 'function' && define.amd)
    define([], function webpackLoadOptionalExternalModuleAmd() {
      return factory(root["jQuery"]);
    });
  else if(typeof exports === 'object')
    exports["Demo"] = factory((
      function webpackLoadOptionalExternalModule() {
        try { return require("jQuery"); } catch(e) {}
      }())
    );
  else
    root["Demo"] = factory(root["jQuery"]);
})(this, function(__WEBPACK_EXTERNAL_MODULE_0__) {
return /******/ (function(modules) { // webpackBootstrap
```

You can see one important difference: the AMD block contains more code than earlier. The output follows non-standard Knockout.js convention as [discussed in the related pull request](https://github.com/webpack/webpack/pull/362). 

你可以看到一个重要的不同：之前的AMD代码块包含更多代码。输出遵循了如[之前在相关PR讨论](https://github.com/webpack/webpack/pull/362)非标准的Knockout.js规范。

In most of the cases using `output.libraryTarget: 'umd'` is enough as optional dependencies and AMD tend to be a rare configuration especially if you use modern technologies.

在多数情况下使用 `output.libraryTarget: 'umd'` 作为可选依赖已能满足同时AMD正逐渐成为罕见配置尤其是在使用现代化技术上。

## JSONP

There's one more output option: `jsonp`. It generates output as below:

还有一个可选输出项：`jsonp`。其生成输出如下：

```javascript
Demo(/******/ (function(modules) { // webpackBootstrap
...
```

In short, `output.library` maps to the JSONP function name. The idea is that you could load a file across domains and have it call the named function. Specific APIs implement the pattern although there is no official standard for it.

简单说，`output.library`映射到JSONP的函数名。这一理念允许你快于加载文件并能够调用该命名函数。尽管官方文档没有将其标准化，但具体的API实现了该模式。

## SystemJS

[SystemJS](https://www.npmjs.com/package/systemjs) is an emerging standard. [webpack-system-register](https://www.npmjs.com/package/webpack-system-register) plugin allows you to wrap your output in a `System.register` call making it compatible with the scheme. If you want to support SystemJS this way, set up another build target.

[SystemJS](https://www.npmjs.com/package/systemjs) 是一个新兴的规范。[webpack-system-register](https://www.npmjs.com/package/webpack-system-register) 插件允许包装输出到一个`System.register`调用使其与该方案兼容。如果你想支持SystemJS，配置其他的构建目标。

## 结论（Conclusion）

Webpack supports a large variety of library output formats. `umd` is the most valuable for a package author. The rest are more specialized and require specific use cases to be valuable.

Webpack支持大量关于代码库输出格式的变量。`umd`对于包作者来说是最具价值的。剩下的则更为专业化且要求具体的使用案例才会有价值。

To recap:

* Most often `umd` is all you need. The other library targets exist more specialized usage in mind.
* The CommonJS variants are handy if you target only Node or consume the output through bundlers alone. UMD implements support for CommonJS, AMD, and globals.
* It's possible to target SystemJS through a plugin. Webpack does not support it out of the box.

本章回顾：

* 大部分情况下`umd`是你所需要的。其他的代码库输出目标存在更专门的用法。
* 若你仅针对Node或通过代码包单独使用其构建输出，CommoneJS变体非常方便。UMD实现支持CommonJS，AMD和全局变量。
* 若是通过插件针对SystemJS。Webpack原本就不支持。

You'll learn to manage multi-page setups in the next chapter.

你会在下一章节了解管理多页面的配置。
