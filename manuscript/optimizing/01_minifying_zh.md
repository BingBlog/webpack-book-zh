# 压缩构建（minifying building）

我们目前还没有考虑到去优化输出，而毫无疑问它看起来会有那么一点臃肿，尤其是当React被包含进去后。我们可以应用各类技术去降低包体积，还可以利用客户端缓存和之前提到惰性加载个别的资源。

**压缩**是一个简化代码同时不会丢失任何语义以影响编译器解析代码的一个环节。而这使得输出的代码看起来犹如一团乱麻。更为关键的是，其很难被阅读。

T > 即使压缩了代码，我们依然可以通过devtool选项来生成source maps（详见第八章），以获得更好地debug，即便在生成环境也可以利用这一技巧。

## 生成基本版本的构建代码（generating a Baseline build）

一开始，我们应首先生成一个基本的版本，这样我们才会有东西可以去优化。执行命令`npm run build`。然后可以在命令行终端看到如下信息：

```bash
Hash: 12aec469d54202150429
Version: webpack 2.2.1
Time: 2863ms
        Asset       Size  Chunks                    Chunk Names
       app.js    2.42 kB       1  [emitted]         app
  ...font.eot     166 kB          [emitted]
...font.woff2    77.2 kB          [emitted]
 ...font.woff      98 kB          [emitted]
  ...font.svg     444 kB          [emitted]  [big]
     logo.png      77 kB          [emitted]
         0.js    1.89 kB       0  [emitted]
  ...font.ttf     166 kB          [emitted]
leanpub-start-insert
    vendor.js     150 kB       2  [emitted]         vendor
leanpub-end-insert
      app.css     3.9 kB       1  [emitted]         app
     0.js.map    2.22 kB       0  [emitted]
   app.js.map    2.13 kB       1  [emitted]         app
  app.css.map   84 bytes       1  [emitted]         app
vendor.js.map     178 kB       2  [emitted]         vendor
   index.html  274 bytes          [emitted]
   [3] ./~/react/lib/ReactElement.js 11.2 kB {2} [built]
  [18] ./app/component.js 461 bytes {1} [built]
  [19] ./~/font-awesome/css/font-awesome.css 41 bytes {1} [built]
...
```

如上图所示，150kb对于一个库代码来说显得过大。而压缩则是要减小其包体积。


## 性能预算支持(Enabling a Performance Bugdet)

webpack允许定义性能预算。这一想法主要是用于强制限制每一个构建代码模块的大小。该特性默认关闭，但若支持的该特性，在默认情况下每个资源和入口文件（entries，即JavaScript代码文件，单页应用中由于大部分情况下仅有一个JavaScript文件，故又称为入口文件）。需要注意的是，入口文件的计算涵盖了已被提取出来的公共部分，如代码库等。

性能预算（Performance Bugdet）能够提供警告和错误两种配置。在已将性能预算设置为错误了之后，如果其未达到性能预算的要求，则将终止整个项目构建。

为了将该特性集成到配置文件中，将配置文件调整至如下：

**webpck.config.js**

```javascript
...

const productionConfig = merge([
leanpub-start-insert
  {
    performance: {
      hints: 'warning', // 'error' or false are valid too
      maxEntrypointSize: 100000, // in bytes
      maxAssetSize: 450000, // in bytes
    },
  },
leanpub-end-insert
  ...
]);

...
```

在实践中，你可能希望将限制设置更低。当前代码仅仅是做展示用。若现在运行构建命令（npm run build），你可能会看到以下警告：

```bash

WARNING in entrypoint size limit: The following entrypoint(s) combined asset size exceeds the recommended limit (100 kB). This can impact web performance.
Entrypoints:
  app (156 kB)
      vendor.js
,      app.js
,      app.css
```

若已经在开发中对每个构建代码正确处理，我们就能满足所提供的预算并在开发环节中消除这一警告。


## 压缩JavaScript

理想情况下，压缩能够在不丢任何失语义的情况下减小代码体积。通常，这意味着通过预定义地转换方式在一定程度上地改写代码。好的例子如重命名变量或者移除实际上不可达的代码块如表达式`if (false)`。

有时压缩也会破坏代码，当他重写了一部分代码。Angular 1就是其中之一，由于其依赖于具体的命名参数，重写函数参数则会让代码异常，除非对其采取预防手段。

在Webpack中最简单地支持压缩方式是调用`webpack -p`（效果与`--optimize-minize`相同）。其支持Webpack的`UglifyJsPlugin`插件，然而问题在于UglifyJS至今并不支持ES 6语法使得其产生问题，若Babel或*babel-preset-env*使用了此那我们只能另寻出路了。

### 配置JavaScript压缩

一个可行的通过Babel压缩代码方式是使用[babili](https://www.npmjs.com/package/babili)。其又Babel团队维护的一个JavaScript压缩器，并提供了ES 6语法和新特性支持。[babili-webpack-plugin](https://www.npmjs.com/package/babili-webpack-plugin)使其能够在webpack中使用成为可能。

首先，需要将该插件引入项目：

```bash
npm install babili-webpack-plugin --save-dev
```

为了能够将其加入配置中，需要先将进行定义：

**webpack.parts.js**

```javascript
...
leanpub-start-insert
const BabiliPlugin = require('babili-webpack-plugin');
leanpub-end-insert

...

leanpub-start-insert
exports.minifyJavaScript = function() {
  return {
    plugins: [
      new BabiliPlugin(),
    ],
  };
};
leanpub-end-insert
```

该插件提供了许多地功能，但提供了source maps的转换已经能够达成我们的目标了。
而后将该插件集成进我们的配置中：

**webpack.config.js**

```javascript
...

const productionConfig = merge([
  ...
  parts.clean(PATHS.build),
leanpub-start-insert
  parts.minifyJavaScript(),
leanpub-end-insert
  ...
]);

...
```

如果你现在执行`npm run build`，那么你应该看到一个小得多的结果：

```bash
Hash: 12aec469d54202150429
Version: webpack 2.2.1
Time: 5265ms
        Asset       Size  Chunks             Chunk Names
       app.js  669 bytes       1  [emitted]  app
  ...font.eot     166 kB          [emitted]
...font.woff2    77.2 kB          [emitted]
 ...font.woff      98 kB          [emitted]
  ...font.svg     444 kB          [emitted]
     logo.png      77 kB          [emitted]
         0.js  399 bytes       0  [emitted]
  ...font.ttf     166 kB          [emitted]
leanpub-start-insert
    vendor.js    45.2 kB       2  [emitted]  vendor
leanpub-end-insert
      app.css     3.9 kB       1  [emitted]  app
     0.js.map    2.07 kB       0  [emitted]
   app.js.map    1.64 kB       1  [emitted]  app
  app.css.map   84 bytes       1  [emitted]  app
leanpub-start-insert
vendor.js.map     169 kB       2  [emitted]  vendor
leanpub-end-insert
   index.html  274 bytes          [emitted]
   [3] ./~/react/lib/ReactElement.js 11.2 kB {2} [built]
  [18] ./app/component.js 461 bytes {1} [built]
  [19] ./~/font-awesome/css/font-awesome.css 41 bytes {1} [built]
...
```

考虑到需要处理更多地工作，因而构建也将花费更长的时间。但好处就是，其构建后的代码体积更小了，限制警告消失了，同时我们的代码库从150kb降到约45kb。

可以通过查看*babili-webpack-plugin*和Babili文档来了解更多功能。举例来说，Babili甚至能让你决定如何处理代码评论。

### 其他压缩JavaScript的方式（Other Ways to Minify JavaScript）

尽管我们采用Babili来举例，但仍然有很多的工具可供考虑：

* [webpack-closure-compiler](https://www.npmjs.com/package/webpack-closure-compiler) 并发执行同时其构建结果能够比Babili更小。
* [optimize-js-plugin](https://www.npmjs.com/package/optimize-js-plugin)  通过对热代码（eager functions）进行包装来补充完善其他解决方案，而这提升了JavaScript代码首次编译速度。这一插件依赖于Nolan Lawson提供的[optimize-js](https://github.com/nolanlawson/optimize-js) 。
* [webpack.optimize.UglifyJsPlugin](https://webpack.js.org/plugins/uglifyjs-webpack-plugin/) UglifiJS官方的webpack插件。但其目前仍不支持ES 6。
* [uglifyjs-webpack-plugin](https://www.npmjs.com/package/uglifyjs-webpack-plugin) 可以体验下试验版的UglifyJS，比稳定版提供更多ES 6语法支持。
* [uglify-loader](https://www.npmjs.com/package/uglify-loader) 比webpack的`UglifyJSPlugin`提供更精细粒度地选项以防你可能更喜欢使用UglifyJS。
* [webpack-parallel-uglify-plugin](https://www.npmjs.com/package/webpack-parallel-uglify-plugin) 允许并发地执行压缩步骤但可能产生一些性能损耗，默认情况下webpack并不支持并行。

## 压缩CSS（Minifying CSS）

*css-loader*可以通过[cssnano](http://cssnano.co/)来压缩CSS。压缩CSS需要其直接配置`minimize`选项。可以通过[cssnano specific options](http://cssnano.co/optimisations/)来查询更多定制化的行为。

[clean-css-loader](https://www.npmjs.com/package/clean-css-loader)支持使用当前一款比较受欢迎的CSS压缩工具[clean-css](https://www.npmjs.com/package/clean-css)。

[optimize-css-assets-webpack-plugin](https://www.npmjs.com/package/optimize-css-assets-webpack-plugin)是一款用于压缩CSS资源的基本可选的插件。需要考虑的是，在合并文件时，使用`ExtractTextPlugin`会导致CSS重复。`OptimizeCSSAssetsPlugin`通过对生成的结果进行编辑避免了这一问题，同时也能获得一个较好的结果。

W > 在Weback 1中，`minimize`选项默认打开若使用了`UglifyJSPlugin`的话。这一让人困扰的行为在Webpack 2中被修复，现在需要再配置中显示声明压缩。

### 配置CSS压缩 （Setting Up CSS Minification）

在可用的解决方案中，`OptimizeCSSAssetsPlugin`处理的效果最好。为了能够加入配置中，首选需要安装它和cssnano

```bash
npm install optimize-css-assets-webpack-plugin cssnano --save-dev
```

类似于处理JavaScript，我们将这一实现包装在配置的一部分中：

**webpack.parts.js**

```javascript
...
leanpub-start-insert
const OptimizeCSSAssetsPlugin = require('optimize-css-assets-webpack-plugin');
const cssnano = require('cssnano');
leanpub-end-insert

...

leanpub-start-insert
exports.minifyCSS = function({ options }) {
  return {
    plugins: [
      new OptimizeCSSAssetsPlugin({
        cssProcessor: cssnano,
        cssProcessorOptions: options,
      }),
    ],
  };
};
leanpub-end-insert
```

而后，将其与主配置关联上：

**webpack.config.js**

```javascript
...

const productionConfig = merge([
  ...
  parts.minifyJavaScript({ useSourceMap: true }),
leanpub-start-insert
  parts.minifyCSS({
    options: {
      discardComments: {
        removeAll: true,
      },
      // Run cssnano in safe mode to avoid
      // potentially unsafe transformations.
      safe: true,
    },
  }),
leanpub-end-insert
  ...
]);

...
```

若你现在进行项目构建（执行npm run build），你应当注意到CSS变得更小了由于其移除了注释：

```bash
Hash: 12aec469d54202150429
Version: webpack 2.2.1
Time: 4764ms
        Asset       Size  Chunks             Chunk Names
       app.js  669 bytes       1  [emitted]  app
  ...font.eot     166 kB          [emitted]
...font.woff2    77.2 kB          [emitted]
 ...font.woff      98 kB          [emitted]
  ...font.svg     444 kB          [emitted]
     logo.png      77 kB          [emitted]
         0.js  399 bytes       0  [emitted]
  ...font.ttf     166 kB          [emitted]
    vendor.js    45.2 kB       2  [emitted]  vendor
leanpub-start-insert
      app.css    2.48 kB       1  [emitted]  app
leanpub-end-insert
     0.js.map    2.07 kB       0  [emitted]
   app.js.map    1.64 kB       1  [emitted]  app
  app.css.map   84 bytes       1  [emitted]  app
vendor.js.map     169 kB       2  [emitted]  vendor
   index.html  274 bytes          [emitted]
   [3] ./~/react/lib/ReactElement.js 11.2 kB {2} [built]
  [18] ./app/component.js 461 bytes {1} [built]
  [19] ./~/font-awesome/css/font-awesome.css 41 bytes {1} [built]
...
```

[cssnano](http://cssnano.co/)提供许多选项可以试试。


## 压缩HTML（Minifying HTML）

若代码中含有HTML模板可以使用[html-loader](https://www.npmjs.com/package/html-loader)，你也可以使用[posthtml](https://www.npmjs.com/package/posthtml) 和 [posthtml-loader](https://www.npmjs.com/package/posthtml-loader)对其进行预处理。 或者你可以使用[posthtml-minifier](https://www.npmjs.com/package/posthtml-minifier)来进行压缩。


## 结论（Conclusion）
压缩是最能够减小代码构建体积的一个环节。本章概括：

* **压缩**环节对源码进行解析同时将其转换为相同含义但更小代码若你采用安全地转换策略。不安全的转换方式能够得到更小的代码体积，但潜在地也会导致代码异常而中断代码执行。举例来说，明确的命名参数就是情况之一。
* **性能预算** 允许你限制构建代码包的大小。维护预算能够让开发者更加了解生成的包（bundles）的体积的大小。
* Webpack包含了`UglifyJsPlugin`用以压缩。其他解决方案，如Babili提供了和Wepack等价的功能。然而Babili提供了ES 6的支持，而这也使其在性能上略低于UglifyJS。
* 除了JavaScript，Wepack还能够压缩其他资源，例如CSS和HTML。压缩这些资源需要使用之对应的装载器（loader）或插件。

在一下小节我们将讨论**Tree shaking**，它是减小代码体积的另一种方法。