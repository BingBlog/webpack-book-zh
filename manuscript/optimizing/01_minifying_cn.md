# 压缩 (Minifying)

The build output hasn't received attention yet and no doubt it's going to be chunky, especially as you included React in it. You can apply a variety of techniques to bring down the size of the vendor bundle. You can also leverage client level caching and load individual assets lazily as you saw earlier.

目前输出代码仍未得到过关注，而毫无疑问它看起来会有点臃肿，尤其是将React涵盖进去之后。你可以使用各种各样地技术来降低包（vendor bundle）的体积。还可使用客户端缓存或你之前曾看到地惰性地加载特别的资源。

**Minification** is a process where the code is simplified without losing any meaning that matters to the interpreter. As a result, your code most likely looks jumbled, and it's hard to read. But that's the point.

**压缩**是一个在不丢失任何语义以影响解析器处理的情况下将代码简化的处理过程。因此，这也使得你的代码看起来十分混乱，而更为关键地是，其难以被阅读。

T> Even if you minify the build, you can still generate source maps through the `devtool` option that was discussed earlier to gain a better debugging experience, even production code if you want.

T> 即使压缩了构建代码，仍然可通过webpack的`devtool`选项来生成之前曾讨论过的source maps以获得更好地debug的体验，只要你想哪怕生产版本的代码也可以。

## 生成基本版本的构建代码 (Generating a Baseline Build)

To get started, you should generate a baseline build, so you have something to optimize. Execute `npm run build` to see output below:

首先，你应该先生成一个基本版本的构建，这样你才可以有东西可以去优化。执行`npm run build`命令可以看到以下的输出结果：

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

150 kB for a vendor bundle is a lot! Minification should bring the size down.

150kb对于一个集成库的包来说非常大！而压缩应该减少其体积。

## 支持性能预算 (Enabling a Performance Budget)

Webpack allows you to define a **performance budget**. The idea is that it gives your build size constraint which it has to follow. The feature is disabled by default, but if enabled it defaults to 250 kB limit per entries and assets. The calculation includes extracted chunks to entry calculation.

Webpakc允许你定义**性能预算**。这一想法旨在限制你的构建代码的体积且其必须遵守。该特性默认被关闭，但若支持该特性，则每个资源和入口文件限制的大小默认为250kb。入口文件大小的计算会涵盖被提取的代码块。

Performance budget can be configured to provide warnings or errors. If a budget isn't met and it has been configured to emit an error, it would terminate the entire build.

性能预算能够设置为支持警告或错误两种方式。若未满足预算且已被配置为触发错误，则webpack会中断整个构建环节。

{pagebreak}

To integrate the feature into the project, adjust the configuration:

为了将该特性集成进项目中，将配置调整为：

**webpack.config.js**

```javascript
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
```

In practice you want to maintain lower limits. The current ones are enough for this demonstration. If you build now (`npm run build`), you should see a warning within the output:

实际上你想要主张将限制设置得更低。当前值仅仅是用于示范。如果你现在进行构建，你应该看到在输出结果中包含一条警告信息：

```bash
...
WARNING in entrypoint size limit: The following entrypoint(s) combined asset size exceeds the recommended limit (100 kB). This can impact web performance.
Entrypoints:
  app (156 kB)
      vendor.js
,      app.js
,      app.css
...
```

If minification works, the warning should disappear. That's the next challenge.

如果压缩生效了，那么警告应该会消失。而这是下一环节地挑战。

{pagebreak}

## 压缩JavaScript (Minifying JavaScript)

The point of **minification** is to convert the code into a smaller form. Safe **transformations** do this without losing any meaning by rewriting code. Good examples of this include renaming variables or even removing entire blocks of code based on the fact that they are unreachable (`if (false)`).

压缩的关键是将代码转换为更小的形式。安全的转换方式是在不丢失任何语义的情况下重写代码。可行的例子包括重命名变量或者移除代码中的整个代码块，而实际上他们是不可达，如（`if(false)`）。

Unsafe transformations can break code as they can lose something implicit the underlying code relies upon. For example, Angular 1 expects specific function parameter naming when using modules. Rewriting the parameters breaks code unless you take precautions against it in this case.

不安全地转换能够中断代码由于他们将底层代码的间接依赖丢失。举例来说，Angular 1当使用模块时函数期望的是具体的命名参数。重写参数将会中断代码运行除非你采用预防措施来防止这样的案例。

Minification in webpack can be enabled through `webpack -p` (same as `--optimize-minimize`). This uses webpack's `UglifyJsPlugin` underneath. The problem is that UglifyJS doesn't support ES6 syntax yet making it problematic if Babel and *babel-preset-env* are used while targeting specific browsers. You have to go another route in this case for this reason.

在webpack中，压缩可以通过`webpack -p`（与 `--optimize-minimize`相同）。其背后使用的是webpack的`UglifyJsPlugin`。然而问题在于若Babel和*babel-prest-env*需要针对具体的浏览器时将会成为问题因为目前UglifyJS仍然不支持ES 6语法。出于这个原因，在这一情况下你只能另寻出路。

### 设置JavaScript压缩 (Setting Up JavaScript Minification)

[babili](https://www.npmjs.com/package/babili) is a JavaScript minifier maintained by the Babel team and it provides support for ES6 and newer features. [babili-webpack-plugin](https://www.npmjs.com/package/babili-webpack-plugin) makes it possible to use it through webpack.

[babili](https://www.npmjs.com/package/babili)是一个由Babel团队维护的JavaScript压缩工具，其为ES 6和更新的JavaScript特性提供支持。. [babili-webpack-plugin](https://www.npmjs.com/package/babili-webpack-plugin) 使其能够通过webpack来进行调用。

To get started, include the plugin to the project:

首先，将该插件引入项目中：

```bash
npm install babili-webpack-plugin --save-dev
```

{pagebreak}

To attach it to the configuration, define a part for it first:

为了将该插件引入配置中，我们首先先定义一部分

**webpack.parts.js**

```javascript
...
leanpub-start-insert
const BabiliPlugin = require('babili-webpack-plugin');
leanpub-end-insert

...

leanpub-start-insert
exports.minifyJavaScript = () => ({
  plugins: [
    new BabiliPlugin(),
  ],
});
leanpub-end-insert
```

The plugin exposes more functionality, but having the possibility of toggling source maps is enough. Hook it up with the configuration:

该插件提供了更多的功能，但有可能切换为source maps就足够了。将其挂载到wepack配置中：

**webpack.config.js**

```javascript
const productionConfig = merge([
  ...
  parts.clean(PATHS.build),
leanpub-start-insert
  parts.minifyJavaScript(),
leanpub-end-insert
  ...
]);
```

{pagebreak}

If you execute `npm run build` now, you should see smaller results:

如果你现在执行`npm run build`命令，你应该能看到更小的结果：

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

Given it needs to do more work, it took longer to execute the build. But on the plus side, the build is now smaller, the size limit warning disappeared, and the vendor build went from 150 kB to roughly 45 kB.

考虑到需要更多地工作，其需要耗费更长的时间来进行构建。但好处是，其构建结果现在变得更小了，体积限制警告也消失了，代码库的构建结构从150kb直接减少到45kb。

You should check *babili-webpack-plugin* and Babili documentation for more options. Babili gives you control over how to handle code comments for example.

你应该查看下*babili-webpack-plugin*和Babili文档来了解更多选项。Babili会提供你更多控制，例如如何处理代码评论。

{pagebreak}

## 其他压缩JavaScript的方式 (Other Ways to Minify JavaScript)

Although Babili works for this use case, there are more options you can consider:

* [webpack-closure-compiler](https://www.npmjs.com/package/webpack-closure-compiler) runs parallel and gives even smaller result than Babili at times.
* [optimize-js-plugin](https://www.npmjs.com/package/optimize-js-plugin) complements the other solutions by wrapping eager functions and it enhances the way your JavaScript code gets parsed initially. The plugin relies on [optimize-js](https://github.com/nolanlawson/optimize-js) by Nolan Lawson.
* [webpack.optimize.UglifyJsPlugin](https://webpack.js.org/plugins/uglifyjs-webpack-plugin/) is the official UglifyJS plugin for webpack. It doesn't support ES6 yet.
* [uglifyjs-webpack-plugin](https://www.npmjs.com/package/uglifyjs-webpack-plugin) allows you to try out an experimental version of UglifyJS that provides better support for ES6 than the stable version.
* [uglify-loader](https://www.npmjs.com/package/uglify-loader) gives more granular control than webpack's `UglifyJsPlugin` in case you prefer to use UglifyJS.
* [webpack-parallel-uglify-plugin](https://www.npmjs.com/package/webpack-parallel-uglify-plugin) allows you to parallelize the minifying step and can yield extra performance as webpack doesn't run in parallel by default.

尽管Babili用于该例中，但仍然有更多的选项可以考虑：
*  [webpack-closure-compiler](https://www.npmjs.com/package/webpack-closure-compiler) 能并发运行，同时有时甚至其构建结果要比Babili更小。
* [optimize-js-plugin](https://www.npmjs.com/package/optimize-js-plugin)通过包装热函数(eager functions)来完善其他的解决方案，该方案提高了你的JavaScript代码在初始环节的解析效率。该插件依赖于Nolan Lawson开发的[optimize-js](https://github.com/nolanlawson/optimize-js)。 
* [webpack.optimize.UglifyJsPlugin](https://webpack.js.org/plugins/uglifyjs-webpack-plugin/) 是为Webpack打造的UglifyJS的官方插件。其至今仍不支持ES 6语法。
*  [uglifyjs-webpack-plugin](https://www.npmjs.com/package/uglifyjs-webpack-plugin) 能够让你尝试试验版本的UglifyJS，其能够比稳定版提供更多关于ES 6语法的支持。
*  [uglify-loader](https://www.npmjs.com/package/uglify-loader) 比wepback的UglifyJsPlugin提供更精细化的控制权若你更倾向于使用UglifyJs的话。
*  [webpack-parallel-uglify-plugin](https://www.npmjs.com/package/webpack-parallel-uglify-plugin) 允许你并行地执行压缩并产生额外的性能损耗由于webpack默认并不支持并行。

## 压缩CSS (Minifying CSS)

*css-loader* allows minifying CSS through [cssnano](http://cssnano.co/). Minification needs to be enabled explicitly using the `minimize` option. You can also pass [cssnano specific options](http://cssnano.co/optimisations/) to the query to customize the behavior further.

*css-loader* 能够通过[cssnano](http://cssnano.co/)压缩CSS。压缩需要通过显示的指定`minimize`选项以获得支持。你也可以通过传递[cssnano具体选项](http://cssnano.co/optimisations/)查询来获得更多定制化行为。

[clean-css-loader](https://www.npmjs.com/package/clean-css-loader) allows you to use a popular CSS minifier [clean-css](https://www.npmjs.com/package/clean-css).

[clean-css-loader](https://www.npmjs.com/package/clean-css-loader) 允许你使用一个受欢迎的CSS压缩器 [clean-css](https://www.npmjs.com/package/clean-css)。

[optimize-css-assets-webpack-plugin](https://www.npmjs.com/package/optimize-css-assets-webpack-plugin) is a plugin based option that applies a chosen minifier on CSS assets. Using `ExtractTextPlugin` can lead to duplicated CSS given it only merges text chunks. `OptimizeCSSAssetsPlugin` avoids this problem by operating on the generated result and thus can lead to a better result.

[optimize-css-assets-webpack-plugin](https://www.npmjs.com/package/optimize-css-assets-webpack-plugin) 是一个基于可选的用于CSS资源的压缩器的插件。只要合并CSS文本，使用`ExtractTextPlugin`会产生重复的CSS代码。`OptimizeCSSAssetsPlugin`通过对生成的结果进行处理避免了这一问题，而这也会得到一个更好的结果。

W> In webpack 1 `minimize` was set on by default if `UglifyJsPlugin` was used. This confusing behavior was fixed in webpack 2, and now you have explicit control over minification.

W> 在Webpack 1中，如果使用`UglifyJsPlugin`，则压缩是默认的。这一困惑的行为在Webpack 2中被修复，现在你可以直接对压缩环节进行控制。

###  设置CSS压缩 (Setting Up CSS Minification)

Out of the available solutions, `OptimizeCSSAssetsPlugin` composes the best. To attach it to the setup, install it and *cssnano* first:

所有便利的解决方案中，`OptimizeCSSAssetsPlugin`压缩的效果最好，为了将其配置到构建环节中，需要先安装它和*cssnano*

```bash
npm install optimize-css-assets-webpack-plugin cssnano --save-dev
```

Like for JavaScript, you can wrap the idea in a configuration part:

类似于JavaScript，你可以将这一想法包装在部分的配置中：

**webpack.parts.js**

```javascript
...
leanpub-start-insert
const OptimizeCSSAssetsPlugin = require('optimize-css-assets-webpack-plugin');
const cssnano = require('cssnano');
leanpub-end-insert

...

leanpub-start-insert
exports.minifyCSS = ({ options }) => ({
  plugins: [
    new OptimizeCSSAssetsPlugin({
      cssProcessor: cssnano,
      cssProcessorOptions: options,
      canPrint: false,
    }),
  ],
});
leanpub-end-insert
```

W> If you use `--json` output with webpack as discussed in the *Analyzing Build Statistics* chapter, you should set `canPrint: false` to avoid output. You can solve by exposing the flag as a parameter so you can control it based on the environment.

W > 如果你采用之前在*构建静态资源*那一章所讨论的在webpack中使用`--json`格式输出，你应该设置`canPrint: false`以避免输出。你可以通过暴露一个标志参数来解决该问题，这样你可基于不同的环境来控制。

Then, connect with main configuration:

然后将该配置导入主配置中：

**webpack.config.js**

```javascript
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
```

{pagebreak}

If you build the project now (`npm run build`), you should notice that CSS has become smaller as it's missing comments:

如果你现在构建项目（`npm run build`），你可以注意到CSS变得更小了，由于其移除了评论：

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

[cssnano](http://cssnano.co/) has a lot more options to try out.

[cssnano](http://cssnano.co/) 有许多选项可以尝试。

{pagebreak}

## 压缩HTML (Minifying HTML)

If you consume HTML templates through your code using [html-loader](https://www.npmjs.com/package/html-loader), you can preprocess it through [posthtml](https://www.npmjs.com/package/posthtml) with [posthtml-loader](https://www.npmjs.com/package/posthtml-loader). You can use [posthtml-minifier](https://www.npmjs.com/package/posthtml-minifier) to minify your HTML through it.

压缩是能让你减小构建结果体积最容易的一步。概括如下：

## Conclusion

Minification is the easiest step you can take to make your build smaller. To recap:

压缩是能让你减小构建结果体积最容易的一步。概括如下：

* **Minification** process analyzes your source code and turns it into a smaller form with the same meaning if you use safe transformations. Certain unsafe transformations allow you to reach even smaller results while potentially breaking code that relies, for example, on exact parameter naming.

* **压缩**过程分析你的源代码并在保持相同语义的情况下将代码转变成更小的形式，若你使用的是安全转换的方式。明确的非安全转换方式能够让你得到体积更小的结果，然而这可能会打破代码依赖，如具体的命名参数。

* **Performance budget** allows you to set limits to the build size. Maintaining a budget can keep developers more conscious of the size of the generated bundles.

* **性能预算**特性允许你对代码构建结果的体积进行限制。维护预算能够让开发者更好的意识到生成包的大小。

* Webpack includes `UglifyJsPlugin` for minification. Other solutions, such as Babili, provide similar functionality with costs of their own. While Babili supports ES6, it can be less performant that UglifyJS.

* Webpack包含了`UglifyJsPlugin`用以压缩。其他解决方案，如Babili，提供了与其相同的功能。然而Babili支持ES 6，其性能要稍逊于UglifyJS。

* Besides JavaScript, it's possible to minify other assets, such as CSS and HTML, too. Minifying these requires specific technologies that have to be applied through loaders and plugins of their own.

* 除了JavaScript，奇还能够压缩其他资源，如CSS何HTML。压缩这些资源要求使用特定的技术如通过加载器(loaders)和其相关的插件。

You'll learn to apply tree shaking against code in the next chapter.

在下一章节，你讲学会应用tree shaking来优化代码。