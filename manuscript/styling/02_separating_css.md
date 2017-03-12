# Separating CSS
# 分离CSS

Even though we have a nice build set up now, where did all the CSS go? As per our configuration, it has been inlined to JavaScript! Even though this can be convenient during development, it doesn’t sound ideal.
尽管我们现在有了一个非茶不错的构建设置，但是所有的CSS都去哪里了呢？对于目前的配置来说，CSS都被内联到了JavaScript中! 尽管这样在开发环境中非常方便，但它仍然不是很理想。

The current solution doesn’t allow us to cache CSS. In some cases, we might suffer from a **Flash of Unstyled Content** (FOUC). FOUC happens because the browser will take a while to load JavaScript and the styles would be applied only then. Separating CSS to a file of its own avoids the problem by letting the browser to manage it separately.
目前的方案让我们无法缓存CSS。在某些情况下，我们可能会遭受**Flash of Unstyled Content** (FOUC)。FOUC的发生是因为浏览器加载JavaScript需要花费一定的时间，当我们把样式內联到了JavaScript当中，只有JavaScript完毕后，样式才会生效。分离CSS到单独的文件，可以让浏览器分别处理它们来避免这个问题。

Webpack provides a means to generate a separate CSS bundles using [ExtractTextPlugin](https://www.npmjs.com/package/extract-text-webpack-plugin). It can aggregate multiple CSS files into one. For this reason, it comes with a loader that handles the extraction process. The plugin then picks up the result aggregated by the loader and emits a separate file.
Webpack提供了一种使用[ExtractTextPlugin]（https://www.npmjs.com/package/extract-text-webpack-plugin）生成单独的CSS包的方法。 它可以将多个CSS文件合并为一个。 为此，它配备了一个处理提取过程的加载器。 然后由插件拾取由加载器聚合的结果并生成单独的文件。

Due to this process, `ExtractTextPlugin` comes with overhead during the compilation phase. It won’t work with Hot Module Replacement (HMR) by design. Given we are using it only for production, that won’t be a problem.
由于这个处理过程，`ExtractTextPlugin`在编译阶段带来很大的开销。它从设计上不能同Hot Module Replacement (HMR)。由于我们仅会在生产环境中使用它，这并不是一个问题。

T> This same technique can be employed with other assets, like templates, too.
T> 这种技术也可以用于其他资源，例如模板。

W> It can be potentially dangerous to use inline styles within JavaScript in production as it represents an attack vector. **Critical path rendering** embraces the idea and inlines the critical CSS to the initial HTML payload improving perceived performance of the site. It is discussed in the next chapter in detail. In limited contexts inlining a small amount of CSS can be a viable option to speed up the initial load (fewer requests).
W> 在生产中使用JavaScript内联样式可能是有潜在危险的，因为它代表攻击向量。 **关键路径渲染**包含这个想法，将关键CSS添加到初始HTML有效载荷，提高网站的感知性能。 这将在下一章详细讨论。 在有限的上下文中内联少量的CSS可以是加速初始负载（更少的请求）的可行选项。


## Setting Up `ExtractTextPlugin`
## 设置`ExtractTextPlugin`

It will take some configuration to make it work. Install the plugin:
需要通过增加一些配置来使它生效。安装插件：

```bash
npm install extract-text-webpack-plugin --save-dev
```

`ExtractTextPlugin` includes a loader, `ExtractTextPlugin.extract` that marks the assets to be extracted. Then a plugin will perform its work based on this annotation.
`ExtractTextPlugin`包含了一个加载器。`ExtractTextPlugin.extract`标记了将要被提取的资源。然后插件将基于此执行其工作。

`ExtractTextPlugin.extract` accepts `use` and `fallback` definitions. `ExtractTextPlugin` processes content through `use` only from **initial chunks** by default and it uses `fallback` for the rest. It won’t touch any split bundles unless `allChunks: true` is set true. The *Splitting Bundles* chapter digs into greater detail.
`ExtractTextPlugin.extract`接受`use` 和 `fallback`两个定义项。`ExtractTextPlugin`默认通过`use`仅处理**initial chunks**的内容，使用`fallback`处理剩下的内容。

It is important to note that if you wanted to extract CSS from a more involved format, like Sass, you would have to pass multiple loaders to the `use` option. Both `use` and `fallback` accept a loader (string), a loader definition, or an array of loader definitions.
重要的是要注意，如果涉及到更多的格式，如从Sass提取CSS，你将不得不传递多个加载器到`use`选项。 `use`和`fallback`接受一个加载器（字符串）或一个加载器定义或者一个加载器定义数组。

The idea looks like this:
如下进行配置：

**webpack.parts.js**

```javascript
const webpack = require('webpack');
leanpub-start-insert
const ExtractTextPlugin = require('extract-text-webpack-plugin');
leanpub-end-insert

...

leanpub-start-insert
exports.extractCSS = function({ include, exclude, use }) {
  // Output extracted CSS to a file
  const plugin = new ExtractTextPlugin({
    filename: '[name].css',
  });

  return {
    module: {
      rules: [
        {
          test: /\.css$/,
          include,
          exclude,

          use: plugin.extract({
            use,
            fallback: 'style-loader',
          }),
        },
      ],
    },
    plugins: [ plugin ],
  };
};
leanpub-end-insert
```

That `[name]` placeholder will use the name of the entry where the CSS is referred. Placeholders and the overall idea are discussed in detail in the *Adding Hashes to Filenames* chapter.
这个`[name]`占位符将使用CSS引用的条目的名称。 占位符和整体思路在*添加哈希到文件名*一章中详细讨论。

T> If you wanted to output the resulting file to a specific directory, you could do it like this: `new ExtractTextPlugin('styles/[name].css')`.
T> 如果你想将结果文件输出到一个特定的目录，你可以这样做：`new ExtractTextPlugin（'styles / [name] .css'）`。

T> If you extract multiple CSS files from separate entries, you can use [merge-files-webpack-plugin](https://www.npmjs.com/package/merge-files-webpack-plugin) to concatenate them into one file.
T> 如果从单独的条目中提取多个CSS文件，可以使用[merge-files-webpack-plugin]（https://www.npmjs.com/package/merge-files-webpack-plugin）将它们连接成一个文件。

### Connecting with Configuration
### 应用到Configuration文件中

Connect the function with our configuration as below:
将这个方法如下同我们的配置文件连接起来：

**webpack.config.js**

```javascript
...

const commonConfig = merge([
  {
    ...
  },
leanpub-start-delete
  parts.loadCSS(),
leanpub-end-delete
]);

const productionConfig = merge([
leanpub-start-insert
  parts.extractCSS({ use: 'css-loader' }),
leanpub-end-insert
]);

const developmentConfig = merge([
  ...
leanpub-start-insert
  parts.loadCSS(),
leanpub-end-insert
]);

...
```

Using this setup, we can still benefit from the HMR during development. For a production build, we generate a separate CSS, though. *html-webpack-plugin* will pick it up automatically and inject it into our `index.html`.
使用这种设置，我们仍然可以从开发过程中的HMR中受益。 对于生产构建，我们生成一个单独的CSS。 * html-webpack-plugin *会自动获取并注入我们的`index.html`。

T> If you are using CSS Modules, remember to tweak `use` accordingly as discussed in the *Loading Styles* chapter. You may also want to maintain separate setups for normal CSS and CSS Modules so that they get loaded through separate logic.
T> 如果你使用CSS模块，请记住按照*载入样式*一章中的讨论相应地调整`use'。 您可能还需要为普通CSS和CSS模块维护单独的设置，以便通过单独的逻辑加载它们。


After running `npm run build`, you should see output similar to the following:
运行`npm run build`后，你应该看到类似下面的输出：

```bash
Hash: 57e54590377069688806
Version: webpack 2.2.1
Time: 952ms
     Asset       Size  Chunks             Chunk Names
    app.js    3.79 kB       0  [emitted]  app
   app.css   32 bytes       0  [emitted]  app
index.html  218 bytes          [emitted]
   [0] ./app/component.js 148 bytes {0} [built]
   [1] ./app/main.css 41 bytes {0} [built]
   [2] ./app/index.js 434 bytes {0} [built]
...
```

Now our styling has been pushed to a separate CSS file. Thus, our JavaScript bundle has become slightly smaller. We also avoid the FOUC problem. The browser doesn’t have to wait for JavaScript to load to get styling information. Instead, it can process the CSS separately, avoiding the flash.
现在我们的样式已经输出到了一个单独的CSS文件。 因此，我们的JavaScript包变得稍微小一些。 我们也避免FOUC问题。 浏览器不必等待JavaScript加载以获取样式信息。 相反，它可以单独处理CSS，避免FOUC。

T> If you are getting `Module build failed: CssSyntaxError:` or `Module build failed: Unknown word` error, make sure your `common` configuration doesn’t have a CSS-related section set up.
T> 如果你得到“模块构建失败：CssSyntaxError：`或`模块构建失败：未知的单词'错误，请确保你的`common`配置没有设置CSS相关的部分。

T> [extract-loader](https://www.npmjs.com/package/extract-loader) is a light alternative to `ExtractTextPlugin`. It does less, but can be enough for basic extraction needs.
T> [extract-loader]（https://www.npmjs.com/package/extract-loader）是“ExtractTextPlugin”的轻型替代品。 它做的少，但可以足够基本的提取需要。

## Managing Styles Outside of JavaScript
## 在JavaScript之外管理样式

Even though referring to styling through JavaScript and then bundling is a valid option, it is possible to achieve the same result through an `entry` and [globbing](https://www.npmjs.com/package/glob). The basic idea goes like this:
尽管通过JavaScript引用样式，然后构建，是一个有效的选项，但是也可以通过`entry'和[globbing]（https://www.npmjs.com/package/glob）实现相同的结果。 基本思想是这样的：

```javascript
...
const glob = require('glob');

// Glob CSS files as an array of CSS files. This can be
// potentially problematic due to CSS rule ordering so
// be careful!
const PATHS = {
  app: path.join(__dirname, 'app'),
  build: path.join(__dirname, 'build'),
  style: glob.sync('./app/**/*.css'),
};

...

const commonConfig = merge([
  {
    entry: {
      app: PATHS.app,
      style: PATHS.style,
    },
    ...
  },
  ...
]);
```

After this type of change, you would not have to refer to styling from your application code. It also means that CSS Modules won’t work anymore. As a result, you should get both *style.css* and *style.js*. The latter file will contain roughly content like `webpackJsonp([1,3],[function(n,c){}]);` and it doesn’t do anything useful as discussed in [webpack issue 1967](https://github.com/webpack/webpack/issues/1967).
进行此类更改后，您不必从应用程序代码引用样式。 这也意味着CSS模块将不再工作。 因此，你应该得到*style.css*和*style.js*。 *style.js*文件将包含大概内容像`webpackJsonp（[1,3]，[function（n，c）{}]）;`。它没有任何作用，正如在[webpack issue 1967](https://github.com/webpack/webpack/issues/1967)讨论的那样。

The approach can be useful if you have to port a legacy project relying on CSS concatenation. If you want strict control over the ordering, you can set up a single CSS entry and then use `@import` to bring the rest to the project through it. Another option would be to set up a JavaScript entry and go through `import` to get the same effect.
如果您必须移植依赖于CSS连接的旧项目，该方法可能很有用。 如果你想要严格控制排序，你可以设置一个单独的CSS入口，然后使用`@import`把剩余的项目通过它引入。 另一个选项是设置一个JavaScript入口并通过`import`获得相同的效果。

## Conclusion
## 小结

Our current setup separates styling from JavaScript neatly. Even though the technique is most useful with CSS, it can be used to extract HTML templates or any other files types you might consume. The hard part about `ExtractTextPlugin` has to do with its setup, but the complexity can be hidden behind an abstraction.
我们当前的设置整齐地分离了造型和JavaScript。 即使该技术对CSS最有用，它也可用于提取HTML模板或任何其他可能使用的文件类型。 关于`ExtractTextPlugin`的困难部分与其设置有关，但是复杂性可以隐藏在抽象抽象后面（使用时，并没有想象的那么复杂）。

To recap:
回顾：

* Using `ExtractTextPlugin` with styling solves the problem of Flash of Unstyled Content (FOUC). Separating CSS from JavaScript also improves caching behavior and removes a potential attack vector.
* 使用`ExtractTextPlugin`与样式解决了无格式内容（FOUC）Flash的问题。 将CSS与JavaScript分离还可以提高缓存行为并消除潜在的被攻击因素。

* `ExtractTextPlugin` is not the only way to handle the problem. *extract-loader* can give the same result in more limited contexts.
* `ExtractTextPlugin`不是处理问题的唯一方法。 * extract-loader *可以在更有限的上下文中给出相同的结果。

* If you don’t prefer to maintain references to styling through JavaScript, an alternative is to handle them through an entry. You will have to be careful with style ordering in this case, though.
* 如果你不喜欢通过JavaScript来保持对样式的引用，一个替代方法是通过单独的入口来处理它们。 在这种情况下，你必须小心的样式排序。

In the next chapter, we will discuss **autoprefixing**. Enabling the feature will make it more convenient to develop complex CSS setups that work with older browsers as well.
在下一章中，我们将讨论** autoprefixing **。 启用该功能将使开发复杂的CSS设置更加方便，这些设置也适用于旧版的浏览器。