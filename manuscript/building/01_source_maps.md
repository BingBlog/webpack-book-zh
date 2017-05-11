# Source Maps
# Source Maps

![Source maps in Chrome](images/sourcemaps.png)

When your source code has gone through any transformations, debugging becomes a problem. When debugging in a browser, how to tell where the original code is? **Source maps** solve this problem by providing a mapping between the original and the transformed source code. In addition to source compiling to JavaScript, this works for styling as well.

当你的源码经过任何转换后，调试就变成了一个问题。在浏览器中调试时，如何告知原始的代码在哪里？**Source maps**通过提供一个源码和转换后的代码的映射来解决这个问题。除了源码编译后的JavaScript，也可以支持styling。

One approach is to simply skip source maps during development and rely on browser support of language features. If you use ES6 without any extensions and develop using a modern browser, this can work. The advantage of doing this is that you avoid all the problems related to source maps while gaining better performance.

另一个办法是，在开发中忽略source map并仅依赖于浏览器支持的语言特性。如果你使用没有任何扩展的ES6并且使用现代浏览器，这种方法就可行。这样做的好处是避免了source map相关的所有问题且获得了更好地性能。

T> If you want to understand the ideas behind source maps in greater detail, [read Ryan Seddon's introduction to the topic](https://www.html5rocks.com/en/tutorials/developertools/sourcemaps/).

T> 如果你希望进一步了解source map背后的思想，可以阅读[Ryan Seddon对该话题的介绍](https://www.html5rocks.com/en/tutorials/developertools/sourcemaps/)。

## Inline Source Maps and Separate Source Maps
## 内联的Source Map和单独的Source Map

Webpack can generate both inline source maps included within bundles or separate source map files. The former are valuable during development due to better performance while the latter are handy for production usage as it keeps the bundle size small. In this case, loading source maps is optional.

Webpack可以生成包含在代码包中的内联的source map，也可以生成单独的source map文件。前者因较好的性能更适用于开发中，而后者可以保持较小的代码包体积因而在生产环境中特别有用。在次情况下，加载source map也是可选的。

It's possible you **don't** want to generate a source map for your production bundle as this makes it effortless to inspect your application. By disabling them you are performing a sort of obfuscation. Whether or not you want to enable source maps for production, they are handy for staging. Skipping source maps entirely speeds up your build as generating source maps at the best quality can be a complex operation.

你也可能并**不**想为你的生产环境代码包生成source map，因为它会使得窥探你的应用变得更加轻松。通过禁用它们，你实际上执行了一些迷惑措施。不论你是否在生产环境中使用source map，它们都可以分级。不启用source map将会极大的提高构建的速度，而想要以最好的质量来支持source map也将非常复杂。

**Hidden source maps** give stack trace information only. You can connect them with a monitoring service to get traces as the application crashes allowing you to fix the problematic situations. While this isn't ideal, it's better to know about possible problems than not.

隐藏的source map仅提供堆栈跟踪信息。您可以将它们与监视服务连接，以便在应用程序崩溃时获取跟踪，从而可以解决有问题的情况。虽然这样不是很理想，但了解可能出现的问题也是不错的。

T> It's a good idea to study the documentation of the loaders you are using to see loader specific tips. For example, with TypeScript, you have to set a particular flag to make it work as you expect.

T> 学习你正在使用的加载器的文档以了解特定于该加载器的提示是一个很好的注意。例如，使用TypeScript时，您必须设置一个特定的标志，使其按照您的期望工作。

## Enabling Source Maps
## 启用Source Map

Webpack provides two ways to enable source maps. There's a `devtool` shortcut field. You can also find two plugins that give more options to tweak. The plugins are be discussed briefly at the end of this chapter. Beyond webpack, you also have to enable support for source maps at the browsers you are using for development.

Webpack提供了两种方式来启用source map。提供了一个`devtool`的缩写字段。你也可以找到提供了更多可调整选项的两个插件。这两个插件在本章节的后面简要的进行了讨论。除了webpack，你同样还要启用开发中所用浏览器的source map功能。

{pagebreak}

### Enabling Source Maps in Webpack
### 在Webpack启用Source Maps

To get started, you can wrap the core idea within a configuration part. You can convert this to use the plugins later if you want:

首先，你可以将主要的想法都囊括到一个配置中。你可以在之后将其转换为使用插件的方法，如你所希望的那样：

**webpack.parts.js**

```javascript
exports.generateSourceMaps = ({ type }) => ({
  devtool: type,
});
```

Webpack supports a wide variety of source map types. These vary based on quality and build speed. For now, you can enable `eval-source-map` for development and `source-map` for production. This way you get good quality while trading off performance, especially during development.

Webpack支持很多种类的source map类型。他们种类取决于质量和构建速度。显现，你可以在开发中启用`eval-source-map` ，而在生产中启用`source-map`。如此，你可以获得较高的质量而牺牲较少的性能，特别是在开发环境中。

Set these up as follows:

如下设置：

**webpack.config.js**

```javascript
const productionConfig = merge([
leanpub-start-insert
  parts.generateSourceMaps({ type: 'source-map' }),
leanpub-end-insert
  ...
]);

const developmentConfig = merge([
leanpub-start-insert
  {
    output: {
      devtoolModuleFilenameTemplate: 'webpack:///[absolute-resource-path]',
    },
  },
  parts.generateSourceMaps({ type: 'cheap-module-source-map' }),
leanpub-end-insert
  ...
]);
```

`eval-source-map` builds slowly initially, but it provides fast rebuild speed. More rapid development specific options, such as `cheap-module-source-map` and `eval`, produce lower quality source maps. All `eval` options emit source maps as a part of your JavaScript code.

`eval-source-map`初始化构建时比较慢，但是在重构时速度较快。更快的特定于开发环境的选项，例如`cheap-module-source-map`和`eval`，生产出质量较低的source map。所有的`eval`选项，都将source map作为JavaScript代码的一部分进行输出。

`source-map` is the slowest and highest quality option of them all, but that's fine for a production build.

`source-map`是最慢但是质量最高的选项，所以更适用于生产构建。

If you build the project now (`npm run build`), you should see source maps in the output:

如果你现在构建项目(`npm run build`)，你将会在输出中看到source map：

```bash
Hash: 79905cd66e14d3455b7d
Version: webpack 2.2.1
Time: 2817ms
        Asset       Size  Chunks                    Chunk Names
     logo.png      77 kB          [emitted]
  ...font.eot     166 kB          [emitted]
...font.woff2    77.2 kB          [emitted]
 ...font.woff      98 kB          [emitted]
  ...font.svg     444 kB          [emitted]  [big]
  ...font.ttf     166 kB          [emitted]
       app.js    4.46 kB       0  [emitted]         app
      app.css    3.89 kB       0  [emitted]         app
leanpub-start-insert
   app.js.map    4.15 kB       0  [emitted]         app
  app.css.map   84 bytes       0  [emitted]         app
leanpub-end-insert
   index.html  218 bytes          [emitted]
   [0] ./app/component.js 272 bytes {0} [built]
   [1] ./~/font-awesome/css/font-awesome.css 41 bytes {0} [built]
   [2] ./app/main.css 41 bytes {0} [built]
...
```

Take a good look at those *.map* files. That's where the mapping between the generated and the original source happens. During development, it writes the mapping information in the bundle itself.

仔细查看这些 *.map*文件。这就是生成的文件和源文件之间的映射发生的地方。在开发过程中，它将映射信息写入包中。

### Enabling Source Maps in Browsers
### 开启浏览器的Source Map

To use source maps within a browser, you have to enable source maps explicitly as per browser-specific instructions:

要在浏览器中使用source map，你需要按照每个浏览器的介绍启用source map：

* [Chrome](https://developer.chrome.com/devtools/docs/javascript-debugging). Sometimes source maps [will not update in Chrome inspector](https://github.com/webpack/webpack/issues/2478). For now, the temporary fix is to force the inspector to reload itself by using *alt-r*.
* [Firefox](https://developer.mozilla.org/en-US/docs/Tools/Debugger/How_to/Use_a_source_map)
* [IE Edge](https://developer.microsoft.com/en-us/microsoft-edge/platform/documentation/f12-devtools-guide/debugger/#source-maps)
* [Safari](https://developer.apple.com/library/safari/documentation/AppleApplications/Conceptual/Safari_Developer_Guide/ResourcesandtheDOM/ResourcesandtheDOM.html#//apple_ref/doc/uid/TP40007874-CH3-SW2)

W> If you want to use breakpoints (i.e., a `debugger;` statement or ones set through the browser), the `eval`-based options won't work in Chrome!

W> 如果你希望使用断点(例如一个`debugger;`语法或者通过浏览器来设置)，基于`eval`的选项在Chrome将不会生效。

## Source Map Types Supported by Webpack
## Webpack支持的Source Map类型

Source map types supported by webpack can be split into two categories:

webpack所支持的source map类型可以分为两类：

* **Inline** source maps add the mapping data directly to the generated files.
* **内联** source map将映射数据直接添加到生成的代码文件中。

* **Separate** source maps emit the mapping data to separate source map files and link the original source to them using a comment. Hidden source maps omit the comment on purpose.

* **单独的** source map将映射数据输出到单独的source map文件中，并使用注解将原始文件链接到其自身。隐藏的source map忽略了对目的的注解。

Thanks to their speed, inline source maps are ideal for development. Given they make the bundles big, separate source maps are the preferable solution for production. Separate source maps work during development as well if the performance overhead is acceptable.

由于他们的速度，内联的source map对于开发来说是理想的。由于他们使得代码包变大，单独的source map是生产环境的最佳选择。单独的source map也可以用于开发环境，如果性能上的开销可以接受的话。

{pagebreak}

## Inline Source Map Types
## 内联的Source Map类型

Webpack provides multiple inline source map variants. Often `eval` is the starting point and [webpack issue #2145](https://github.com/webpack/webpack/issues/2145#issuecomment-294361203) recommends `cheap-module-source-map` with `output.devtoolModuleFilenameTemplate: 'webpack:///[absolute-resource-path]'` as it's a good compromise between speed and quality while working reliably in Chrome and Firefox browsers.

Webpack提供了多种内联的Source Map变种。通常情况下从`eval`开始，[webpack issue #2145](https://github.com/webpack/webpack/issues/2145#issuecomment-294361203)推荐了`cheap-module-source-map`和`output.devtoolModuleFilenameTemplate: 'webpack:///[absolute-resource-path]'`，因为它在速度和质量之间找到了一个很好的平衡，并且Chrome和Firefox浏览器都可以有效地支持它。

To get a better idea of the available options, they are listed below while providing a small example for each. The source code contains only a single `console.log('Hello world')` and `webpack.NamedModulesPlugin` is used to keep the output easier to understand. In practice, you would see a lot more code to handle the mapping.

为了更深入的理解可靠的选项，如下列出了这些选项并为每一个选项给出了例子。源码仅包含了`console.log('Hello world')`一行代码，`webpack.NamedModulesPlugin`使输出的代码更容易理解。实际开发中，你将看到有非常多的代码来处理这个映射。

T> `webpack.NamedModulesPlugin` replaces number based module IDs with paths. It's discussed in the *Hot Module Replacement* appendix.

T> `webpack.NamedModulesPlugin`将模ID称替换为路径。在*Hot Module Replacement*附录中有对其的讨论。

### `devtool: 'eval'`

`eval` generates code in which each module is wrapped within an `eval` function:

`eval`生成的代码的每个模块都被一个`eval`函数包裹着：

```javascript
webpackJsonp([1, 2], {
  "./app/index.js": function(module, exports) {
    eval("console.log('Hello world');\n\n//////////////////\n// WEBPACK FOOTER\n// ./app/index.js\n// module id = ./app/index.js\n// module chunks = 1\n\n//# sourceURL=webpack:///./app/index.js?")
  }
}, ["./app/index.js"]);
```

{pagebreak}

### `devtool: 'cheap-eval-source-map'`

`cheap-eval-source-map` goes a step further and it includes base64 encoded version of the code as a data url. The result includes only line data while losing column mappings.

`cheap-eval-source-map`更进一步，它以数据url的形式包含了将代码进行64位编码后的版本。结果中也仅有行的数据而丢失了列的映射。

```javascript
webpackJsonp([1, 2], {
  "./app/index.js": function(module, exports) {
    eval("console.log('Hello world');//# sourceMappingURL=data:application/json;charset=utf-8;base64,eyJ2ZXJzaW9uIjozLCJmaWxlIjoiLi9hcHAvaW5kZXguanMuanMiLCJzb3VyY2VzIjpbIndlYnBhY2s6Ly8vLi9hcHAvaW5kZXguanM/MGUwNCJdLCJzb3VyY2VzQ29udGVudCI6WyJjb25zb2xlLmxvZygnSGVsbG8gd29ybGQnKTtcblxuXG4vLy8vLy8vLy8vLy8vLy8vLy9cbi8vIFdFQlBBQ0sgRk9PVEVSXG4vLyAuL2FwcC9pbmRleC5qc1xuLy8gbW9kdWxlIGlkID0gLi9hcHAvaW5kZXguanNcbi8vIG1vZHVsZSBjaHVua3MgPSAxIl0sIm1hcHBpbmdzIjoiQUFBQSIsInNvdXJjZVJvb3QiOiIifQ==")
  }
}, ["./app/index.js"]);
```

If you decode that base64 string, you get output containing the mapping:

如果你将64位的字符串解码，你将会得到包含如下信息的输入：

```json
{
  "file": "./app/index.js.js",
  "mappings": "AAAA",
  "sourceRoot": "",
  "sources": [
    "webpack:///./app/index.js?0e04"
  ],
  "sourcesContent": [
    "console.log('Hello world');\n\n\n//////////////////\n// WEBPACK FOOTER\n// ./app/index.js\n// module id = ./app/index.js\n// module chunks = 1"
  ],
  "version": 3
}
```

{pagebreak}

### `devtool: 'cheap-module-eval-source-map'`

`cheap-module-eval-source-map` is the same idea, except with higher quality and lower performance:

`cheap-module-eval-source-map`也是同样的想法，除了拥有更高的质量和较低的性能：

```javascript
webpackJsonp([1, 2], {
  "./app/index.js": function(module, exports) {
    eval("console.log('Hello world');//# sourceMappingURL=data:application/json;charset=utf-8;base64,eyJ2ZXJzaW9uIjozLCJmaWxlIjoiLi9hcHAvaW5kZXguanMuanMiLCJzb3VyY2VzIjpbIndlYnBhY2s6Ly8vYXBwL2luZGV4LmpzPzIwMTgiXSwic291cmNlc0NvbnRlbnQiOlsiY29uc29sZS5sb2coJ0hlbGxvIHdvcmxkJyk7XG5cblxuLy8gV0VCUEFDSyBGT09URVIgLy9cbi8vIGFwcC9pbmRleC5qcyJdLCJtYXBwaW5ncyI6IkFBQUEiLCJzb3VyY2VSb290IjoiIn0=")
  }
}, ["./app/index.js"]);
```

Again, decoding the data reveals more:

同样，解码后的数据可以展示更多信息：

```json
{
  "file": "./app/index.js.js",
  "mappings": "AAAA",
  "sourceRoot": "",
  "sources": [
    "webpack:///app/index.js?2018"
  ],
  "sourcesContent": [
    "console.log('Hello world');\n\n\n// WEBPACK FOOTER //\n// app/index.js"
  ],
  "version": 3
}
```

In this particular case, the difference between the options is minimal.

在这个特殊情况下，两者之间的差异非常小。

{pagebreak}

### `devtool: 'eval-source-map'`

`eval-source-map` is the highest quality option of the inline options. It's also the slowest one as it emits the most data:

`eval-source-map`是内联选项中质量最好的选择。因为其生成了更多数据，所以处理中也最花时间。

```javascript
webpackJsonp([1, 2], {
  "./app/index.js": function(module, exports) {
    eval("console.log('Hello world');//# sourceMappingURL=data:application/json;charset=utf-8;base64,eyJ2ZXJzaW9uIjozLCJzb3VyY2VzIjpbIndlYnBhY2s6Ly8vLi9hcHAvaW5kZXguanM/ZGFkYyJdLCJuYW1lcyI6WyJjb25zb2xlIiwibG9nIl0sIm1hcHBpbmdzIjoiQUFBQUEsUUFBUUMsR0FBUixDQUFZLGFBQVoiLCJmaWxlIjoiLi9hcHAvaW5kZXguanMuanMiLCJzb3VyY2VzQ29udGVudCI6WyJjb25zb2xlLmxvZygnSGVsbG8gd29ybGQnKTtcblxuXG4vLyBXRUJQQUNLIEZPT1RFUiAvL1xuLy8gLi9hcHAvaW5kZXguanMiXSwic291cmNlUm9vdCI6IiJ9")
  }
}, ["./app/index.js"]);
```

This time around there's more mapping data available for the browser:

这一次，浏览器有更多的映射数据可用：

```json
{
  "file": "./app/index.js.js",
  "mappings": "AAAAA,QAAQC,GAAR,CAAY,aAAZ",
  "names": [
    "console",
    "log"
  ],
  "sourceRoot": "",
  "sources": [
    "webpack:///./app/index.js?dadc"
  ],
  "sourcesContent": [
    "console.log('Hello world');\n\n\n// WEBPACK FOOTER //\n// ./app/index.js"
  ],
  "version": 3
}
```

{pagebreak}

## Separate Source Map Types
## 单独的Source Map类型

Webpack can also generate production usage friendly source maps. These end up in separate files ending with `.map` extension and are loaded by the browser only when required. This way your users get good performance while it's easier for you to debug the application.

Webpack也可以生成生产环境友好的source maps。这些方式会将结果输出到单独的文件并以`.map`作为扩展名，然后再浏览器需要的时候才被调用。如此，你的用户将获得很好的性能体验而你也可以很容易的调试应用。

`source-map` is a good default here. Even though it takes longer to generate the source maps this way, you get the best quality. If you don't care about production source maps, you can simply skip the setting there and get better performance in return.

`source-map`是一个很好的默认选项。经管这种方式在生成source map时花费了更多的时间，但是你得到了更好地质量。如果你不考虑生产环境的source map，你可以跳过这个设置，可以让你获得更好的性能。

### `devtool: 'cheap-source-map'`

`cheap-source-map` is similar to the cheap options above. The result is going to miss column mappings. Also, source maps from loaders, such as *css-loader*, are not going to be used.

`cheap-source-map`同如上的cheap选项一样。结果将会丢失列映射。而且，对于加载器的source map，如*css-loader*，将无法适用。

Examining the `.map` file reveals the following output in this case:

查看`.map`作为后缀的文件，当前情况下将展示如下输出：

```json
{
  "file": "app.9aff3b1eced1f089ef18.js",
  "mappings": "AAAA",
  "sourceRoot": "",
  "sources": [
    "webpack:///app.9aff3b1eced1f089ef18.js"
  ],
  "sourcesContent": [
    "webpackJsonp([1,2],{\"./app/index.js\":function(o,n){console.log(\"Hello world\")}},[\"./app/index.js\"]);\n\n\n// WEBPACK FOOTER //\n// app.9aff3b1eced1f089ef18.js"
  ],
  "version": 3
}
```

The original source contains `//# sourceMappingURL=app.9a...18.js.map` kind of comment at its end to map to this file.

源码文件的底部包含了`//# sourceMappingURL=app.9a...18.js.map`类似的注释来映射该文件。

### `devtool: 'cheap-module-source-map'`

`cheap-module-source-map` is the same as previous except source maps from loaders are simplified to a single mapping per line. It yields the following output in this case:

`cheap-module-source-map`同上，除了来自加载器的source map被简化为每一行的映射。当前情况下，将输出如下信息：

```json
{
  "file": "app.9aff3b1eced1f089ef18.js",
  "mappings": "AAAA",
  "sourceRoot": "",
  "sources": [
    "webpack:///app.9aff3b1eced1f089ef18.js"
  ],
  "version": 3
}
```

W> `cheap-module-source-map` is [currently broken if minification is used](https://github.com/webpack/webpack/issues/4176) and this is a good reason to avoid the option for now.

W> `cheap-module-source-map`[在压缩被使用时会崩溃](https://github.com/webpack/webpack/issues/4176)，当前来说，这也是一个避免使用该选项的原因。

{pagebreak}

### `devtool: 'source-map'`

`source-map` provides the best quality with the complete result, but it's also the slowest option. The output reflects this:

`source-map`提供了最好的质量和最完整的结果，但它也是最慢的选项。

```json
{
  "file": "app.9aff3b1eced1f089ef18.js",
  "mappings": "AAAAA,cAAc,EAAE,IAEVC,iBACA,SAAUC,EAAQC,GCHxBC,QAAQC,IAAI,kBDST",
  "names": [
    "webpackJsonp",
    "./app/index.js",
    "module",
    "exports",
    "console",
    "log"
  ],
  "sourceRoot": "",
  "sources": [
    "webpack:///app.9aff3b1eced1f089ef18.js",
    "webpack:///./app/index.js"
  ],
  "sourcesContent": [
    "webpackJsonp([1,2],{\n\n/***/ \"./app/index.js\":\n/***/ (function(module, exports) {\n\nconsole.log('Hello world');\n\n/***/ })\n\n},[\"./app/index.js\"]);\n\n\n// WEBPACK FOOTER //\n// app.9aff3b1eced1f089ef18.js",
    "console.log('Hello world');\n\n\n// WEBPACK FOOTER //\n// ./app/index.js"
  ],
  "version": 3
}
```

{pagebreak}

### `devtool: 'hidden-source-map'`

`hidden-source-map` is the same as `source-map` except it doesn't write references to the source maps to the source files. If you don't want to expose source maps to development tools directly while you want proper stack traces, this is handy.

`hidden-source-map`同`source-map`一样，除了它并不将source map的引用写到源文件中。如果您不希望将源代码直接暴露给开发工具，而希望正确的堆栈跟踪，这是方便的。

T> [The official documentation](https://webpack.js.org/configuration/devtool/#devtool) contains more information about `devtool` options.

T> [官方文档](https://webpack.js.org/configuration/devtool/#devtool)包含更多的关于`devtool`选项的信息

## Other Source Map Options
## 其它Source Map选项

There are a couple of other options that affect source map generation:

还要一些其它选项会影响source map的生成。

```javascript
{
  output: {
    // Modify the name of the generated source map file.
    // You can use [file], [id], and [hash] replacements here.
    // The default option is enough for most use cases.
    sourceMapFilename: '[file].map', // Default

    // This is the source map filename template. It's default
    // format depends on the devtool option used. You don't
    // need to modify this often.
    devtoolModuleFilenameTemplate: 'webpack:///[resource-path]?[loaders]'
  },
}
```

T> The [official documentation](https://webpack.js.org/configuration/output/#output-sourcemapfilename) digs into `output` specifics.

T> [官方文档](https://webpack.js.org/configuration/output/#output-sourcemapfilename)更详细的探讨了`output`

W> If you are using any `UglifyJsPlugin` and still want source maps, you need to enable `sourceMap: true` for the plugin. Otherwise, the result isn't be what you expect because UglifyJS will perform a further transformation on the code, breaking the mapping. The same has to be done with other plugins and loaders performing transformations. *css-loader* and related loaders are a good example.

W> 如果你使用了`UglifyJsPlugin`并且仍想支持source map，你需要启用该插件的`sourceMap: true`。否则，结果将不是你所期望的，因为UglifyJS将会对代码执行进一步的转换，破坏了映射。对于其它执行了转化的加载器和插件也要进行相同的处理。*css-loader*以及相关的加载器就是一个很好的例子。

## `SourceMapDevToolPlugin` and `EvalSourceMapDevToolPlugin`

If you want more control over source map generation, it's possible to use the `SourceMapDevToolPlugin` or `EvalSourceMapDevToolPlugin` instead. The latter is a more limited alternative, and as stated by its name, it's handy for generating `eval` based source maps.

如果你希望对source map的生成进行更多的控制，可以使用`SourceMapDevToolPlugin`和`EvalSourceMapDevToolPlugin`来替代。后者是一个更有限的选择，真如其名称所表示的，对于生成基于`eval`的source map比较方便。

Both plugins can allow more granular control over which portions of the code you want to generate source maps for, while also having strict control over the result with `SourceMapDevToolPlugin`. Using either plugin allows you to skip the `devtool` option altogether.

这两个插件都允许更精细地控制哪部分代码要生成source map，同时还可以使用`SourceMapDevToolPlugin`严格控制结果。使用任一插件可以让您跳过`devtool`选项。

You could model a configuration part using `SourceMapDevToolPlugin` (adapted from [the official documentation](https://webpack.js.org/plugins/source-map-dev-tool-plugin/)):

您可以使用`SourceMapDevToolPlugin`（从[官方文档](https://webpack.js.org/plugins/source-map-dev-tool-plugin/)改编）创建配置部分的模型）：

```javascript
exports.generateSourceMaps = ({
  test, include, separateSourceMaps, columnMappings
}) => ({
  // Enable functionality as you want to expose it
  plugins: [
    new webpack.SourceMapDevToolPlugin({
      // Match assets like for loaders. This is
      // convenient if you want to match against multiple
      // file types.
      test: test, // string | RegExp | Array,
      include: include, // string | RegExp | Array,

      // `exclude` matches file names, not package names!
      // exclude: string | RegExp | Array,

      // If filename is set, output to this file.
      // See `sourceMapFileName`.
      // filename: string,

      // This line is appended to the original asset processed.
      // For instance '[url]' would get replaced with an url
      // to the source map.
      // append: false | string,

      // See `devtoolModuleFilenameTemplate` for specifics.
      // moduleFilenameTemplate: string,
      // fallbackModuleFilenameTemplate: string,

      // If false, separate source maps aren't generated.
      module: separateSourceMaps,

      // If false, column mappings are ignored.
      columns: columnMappings,

      // Use plain line to line mappings for the matched modules.
      // lineToLine: bool | {test, include, exclude},

      // Remove source content from source maps. This is handy
      // especially if your source maps are big (over 10 MB)
      // as browsers can struggle with those.
      // See https://github.com/webpack/webpack/issues/2669.
      // noSources: bool,
    }),
  ],
});
```

Given webpack matches only `.js` and `.css` files by default for source maps, you can use `SourceMapDevToolPlugin` to overcome this issue. This can be achieved by passing a `test` pattern like `/\.(js|jsx|css)($|\?)/i`.

由于webpack仅默认为source map匹配`.js`和`.css`文件，你可以使用`SourceMapDevToolPlugin`来解决这个问题。可以通过传入一个如`/\.(js|jsx|css)($|\?)/i`的`test`正则来实现。

`EvalSourceMapDevToolPlugin` accepts only `module` and `lineToLine` options as described above. Therefore it can be considered as an alias to `devtool: 'eval'` while allowing a notch more flexibility.

`EvalSourceMapDevToolPlugin`如上所述的那样仅接受`module`和`lineToLine`选项。因此，它可以被认为是`devtool:'eval'`的别名，同时允许更多的灵活性。

## Changing Source Map Prefix
## 改变Source Map前缀

You can prefix a source map option with a **pragma** character that gets injected to the source map reference. Webpack uses `#` by default that is supported by modern browsers so you don't have to set it.

你可以在source map选项前加一个**pragma**字符并注入到source map的引用中。Webpack默认使用现代浏览器支持的`#`，因此你也可以不用设置它。

To override this, you have to prefix your source map option with it (e.g., `@source-map`). After the change, you should see `//@` kind of reference to the source map over `//#` in your JavaScript files assuming a separate source map type was used.

要改写这个，你需要将这个字符添加到source map选项中（例如`@source-map`）。如此更改之后，你可以在代码中看见`//@`之类的source map引用而不是`//#`，如果是采用的单独的source map类型。

## Using Dependency Source Maps
## 启用依赖包的Source Map

Assuming you are using a package that uses inline source maps in its distribution, you can use [source-map-loader](https://www.npmjs.com/package/source-map-loader) to make webpack aware of them. Without setting it up against the package, you get minified debug output. Often you can skip this step as it's a special case.

如果你使用了一个在分发中使用了source map的代码包，你可以使用[source-map-loader](https://www.npmjs.com/package/source-map-loader)来使webpack知晓它们。如果没有对这些代码包设置它，你将会得到最小化的调试输出。通常情况下你可以跳过这一步，因为它仅是一个特殊情况。

## Source Maps for Styling
## 对于样式的Source Map

If you want to enable source maps for styling files, you can achieve this by enabling the `sourceMap` option. The same idea works with style loaders such as *css-loader*, *sass-loader*, and *less-loader*.

如果你希望为样式文件启用source map，你可以通过`sourceMap`选项来实现。该想法同样适用于样式加载器，诸如*css-loader*, *sass-loader*, 和 *less-loader*。

The *css-loader* is [known to have issues](https://github.com/webpack-contrib/css-loader/issues/232) when you are using relative paths in imports. To overcome this problem, you should set `output.publicPath` to resolve against the server url.

当你在import中使用相对路径的时候，*css-loader*有一个[已知的问题](https://github.com/webpack-contrib/css-loader/issues/232)。为了解决这个问题，你需要设置`output.publicPath`来对服务的url进行处理。

## Conclusion
## 小结

Source maps can be convenient during development. They provide better means to debug applications as you can still examine the original code over a generated one. They can be valuable even for production usage and allow you to debug issues while serving a client-friendly version of your application.

Source map在开发中提供了诸多便利。它们提供了更好的方式来调试应用，使你在生成了代码之后也可以检查原始代码。即使对于生产使用，它们也是有价值的，并允许您在为客户端版本的应用程序提供服务同时调试问题。

To recap:

回顾：

* **Source maps** can be helpful both during development and production. They provide more accurate information about what's going on and make it faster to debug possible problems.

* **Source maps**在开发环境和生产环境都可提供诸多帮助。它们提供了更准确的关于到底法身了什么的信息，并让调试问题变得更容易。

* Webpack supports a large variety of source map variants. They can be split into inline and separate source maps based on where they are generated. Inline source maps are handy during development due to their speed. Separate source maps work for production as then loading them becomes optional.

* Webpack提供了多种多样的source map变体。基于它们被生成的地方，可以被区分为内联的source map和独立source map。内联的source map因其速度快而适用于开发环境。独立source map适用于生产环境，而是否加载它们也是可选的。

* `devtool: 'source-map'` is the highest quality option making it valuable for production.

* `devtool: 'source-map'`是质量最好的选项，因而更适用于生产环境。

* `cheap-module-eval-source-map` is a good starting point for development.

* `cheap-module-eval-source-map`对开发环境来说是一个很好的起点。

* If you want to get only stack traces during production, use `devtool: 'hidden-source-map'`. You can capture the output and send it to a third party service for you to examine. This way you can capture errors and fix them.

* 如果你希望在生产环境中仅得到堆栈追踪信息，适用`devtool: 'hidden-source-map'`。你可以捕获其输出并发送到第三方服务来为你检测。如此你可以捕获错误并修复它们。

* `SourceMapDevToolPlugin` and `EvalSourceMapDevToolPlugin` provide more control over the result than the `devtool` shortcut.

* `SourceMapDevToolPlugin`和`EvalSourceMapDevToolPlugin`比`devtool`这种快捷方式提供了对结果的更多控制。

* *source-map-loader* can come in handy if your dependencies provide source maps.

* 如果你的依赖提供了source map，可以考虑*source-map-loader*

* Enabling source maps for styling requires additional effort. You have to enable `sourceMap` option per styling related loader you are using.

* 为样式启用source map需要更多额外的工作。你需要对每个样式相关的加载器都启用`sourceMap`选项。

In the next chapter, you'll learn to split bundles and separate the current bundle into application and vendor bundles.

在下一章节，你将会学习拆分代码包，并将当前的代码包拆分到应用代码包和公共代码包中。
