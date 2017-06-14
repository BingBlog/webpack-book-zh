# Separating CSS
# 分离CSS

Even though there is a nice build set up now, where did all the CSS go? As per configuration, it has been inlined to JavaScript! Even though this can be convenient during development, it doesn't sound ideal.

尽管我们现在已经有了一个很不错的构建设置，但是所有的CSS都去哪里了呢？对于目前的配置来说，CSS都被内联到了JavaScript中! 尽管这在开发环境中非常方便，但它看起来仍然不是很理想。

The current solution doesn't allow to cache CSS. You can also get a **Flash of Unstyled Content** (FOUC). FOUC happens because the browser takes a while to load JavaScript and the styles would be applied only then. Separating CSS to a file of its own avoids the problem by letting the browser to manage it separately.

目前的方案无法缓存CSS。在某些情况下，你可能会遭遇**Flash of Unstyled Content** (FOUC)的问题。FOUC的发生是因为浏览器加载JavaScript需要花费一定的时间，当我们把样式內联到了JavaScript当中后，只有JavaScript加载完毕了，样式才会生效。分离CSS到单独的文件，可以让浏览器分别管理它们，进而避免这个问题。

Webpack provides a means to generate a separate CSS bundles using [ExtractTextPlugin](https://www.npmjs.com/package/extract-text-webpack-plugin). It can aggregate multiple CSS files into one. For this reason, it comes with a loader that handles the extraction process. The plugin then picks up the result aggregated by the loader and emits a separate file.

Webpack提供了一种使用[ExtractTextPlugin](https://www.npmjs.com/package/extract-text-webpack-plugin)插件来生成单独的CSS代码包的方法。它可以将多个CSS文件合并为一个。为此，它配备了一个处理提取过程的加载器。该插件可提取由加载器集成的结果，并生成单独的文件。

Due to this process, `ExtractTextPlugin` comes with overhead during the compilation phase. It doesn't work with Hot Module Replacement by design. Given the plugin is used only for production, that is not a problem.

由于上述的处理过程，`ExtractTextPlugin`插件在编译阶段会带来很大的开销。它在设计上不能同Hot Module Replacement (HMR)一起使用。由于我们将仅在生产环境中使用它，所以这也并不是什么问题。

T> This same technique can be employed with other assets, like templates, too.

T> 这种技术也可以用于其他资源（的提取），例如模板。

W> It can be potentially dangerous to use inline styles within JavaScript in production as it represents an attack vector. **Critical path rendering** embraces the idea and inlines the critical CSS to the initial HTML payload improving perceived performance of the site. In limited contexts inlining a small amount of CSS can be a viable option to speed up the initial load (fewer requests).

W> 在生产中使用JavaScript内联样式，是有潜在危险的，因为它代表（一种被）攻击的因素。**关键路径渲染**包含了一个想法，将关键CSS添加到初始的HTML文件中，来提高加载效率，提升网站的感知性能。这将在下一章详细讨论。在有限的上下文中内联少量的CSS,是提高初始加载速度（更少的请求）的可行选项。

## Setting Up `ExtractTextPlugin`
## 设置`ExtractTextPlugin`

Install the plugin first:
首先需要安装这个插件：

```bash
npm install extract-text-webpack-plugin --save-dev
```

`ExtractTextPlugin` includes a loader, `ExtractTextPlugin.extract` that marks the assets to be extracted. Then a plugin performs its work based on this annotation.

`ExtractTextPlugin`包含了一个加载器。`ExtractTextPlugin.extract`标记了将要被提取的资源。插件将基于此执行工作。

`ExtractTextPlugin.extract` accepts `use` and `fallback` definitions. `ExtractTextPlugin` processes content through `use` only from **initial chunks** by default and it uses `fallback` for the rest. It doesn't touch any split bundles unless `allChunks: true` is set true. The *Splitting Bundles* chapter digs into greater detail.

`ExtractTextPlugin.extract`接受`use` 和 `fallback`两个定义项。`ExtractTextPlugin`默认通过`use`来处理**初始化代码块**的内容，使用`fallback`处理剩下的内容。如果`allChunks: true`没有被设置为`true`，它将不会提取任何其它的代码块。*Splitting Bundles*章节将会详细介绍。

If you wanted to extract CSS from a more involved format, like Sass, you would have to pass multiple loaders to the `use` option. Both `use` and `fallback` accept a loader (string), a loader definition, or an array of loader definitions.

如果你想从更多相关的格式中提取CSS，例如从Sass提取CSS，你将不得不传递多个加载器到`use`选项。 `use`和`fallback`接受一个加载器（字符串）或一个加载器定义或者一个加载器定义数组。

The idea can be modeled as below:

这个想法可如下实现：

**webpack.parts.js**

```javascript
leanpub-start-insert
const ExtractTextPlugin = require('extract-text-webpack-plugin');
leanpub-end-insert

...

leanpub-start-insert
exports.extractCSS = ({ include, exclude, use }) => {
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

That `[name]` placeholder uses the name of the entry where the CSS is referred. Placeholders and hashing are discussed in detail in the *Adding Hashes to Filenames* chapter.

`[name]`占位符将使用引用CSS的文件名称。占位符和哈希值将在*添加哈希到文件名*一章中详细讨论。

It would be possible to have multiple `plugin.extract` calls against different file types. This would allow you to aggregate them to a single CSS file. Another option would be to extract multiple CSS files through separate plugin definitions and then concatenate them using [merge-files-webpack-plugin](https://www.npmjs.com/package/merge-files-webpack-plugin).

对于不同的文件类型，可以使用多个`plugin.extract`调用。这样可以将它们集成到单个CSS文件中。另一个选择是通过单独的插件定义来提取多个CSS文件，然后使用[merge-files-webpack-plugin]（https://www.npmjs.com/package/merge-files-webpack-plugin）连接它们。

T> If you wanted to output the resulting file to a specific directory, you could do it by passing a path. Example: `filename: 'styles/[name].css'`.

T> 如果你想将结果文件输出到一个特定的目录，你可以通过传入一个路劲来实现：`filename: 'styles/[name].css'`。

{pagebreak}

### Connecting with Configuration
### 应用到Configuration文件中

Connect the function with the configuration as below:

将这个方法如下应用到我们的配置文件中：

**webpack.config.js**

```javascript
const commonConfig = merge([
  ...
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
```

Using this setup, you can still benefit from the HMR during development. For a production build, it's possible to generate a separate CSS, though. `HtmlWebpackPlugin` picks it up automatically and injects it into `index.html`.

如此设置后，我们在开发过程中仍然可以享受热加载的好处。对于在生产环境进行构建打包，我们可以生成一个单独的CSS。 *html-webpack-plugin*会自动获取（CSS的文件的路径），并将路劲注入我们的`index.html`文件中。

T> If you are using CSS Modules, remember to tweak `use` accordingly as discussed in the *Loading Styles* chapter. You can maintain separate setups for normal CSS and CSS Modules so that they get loaded through separate logic.

T> 如果你使用了CSS模块这个特性，请记住按照*加载样式*章节中的讨论的那样，相应地调整`use`配置项。您可能还需要对普通CSS和模块化的CSS分别维护单独的配置，以便通过不同的逻辑加载它们。

{pagebreak}

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

Now styling has been pushed to a separate CSS file. Thus, the JavaScript bundle has become slightly smaller. You also avoid the FOUC problem. The browser doesn't have to wait for JavaScript to load to get styling information. Instead, it can process the CSS separately, avoiding the flash.

现在我们的样式已经输出到了一个单独的CSS文件中。 因此，我们的JavaScript包变得稍微小了一些。浏览器不必等待JavaScript加载完毕以获取样式信息。它可以单独处理CSS，避免FOUC问题。

T> If you are getting `Module build failed: CssSyntaxError:` or `Module build failed: Unknown word` error, make sure your `common` configuration doesn't have a CSS-related section set up.

T> 如果你得到`Module build failed: CssSyntaxError:`或`Module build failed: Unknown word`类似的错误，请确保你的`common`配置中没有CSS相关的设置。

T> [extract-loader](https://www.npmjs.com/package/extract-loader) is a light alternative to `ExtractTextPlugin`. It does less, but can be enough for basic extraction needs.

T> [extract-loader](https://www.npmjs.com/package/extract-loader)是`ExtractTextPlugin`的轻量级替代品。它能做的要少一些，但也可以满足基本的提取需求。

{pagebreak}

## Managing Styles Outside of JavaScript
## 在JavaScript之外管理样式

Even though referring to styling through JavaScript and then bundling is the recommended option, it's possible to achieve the same result through an `entry` and [globbing](https://www.npmjs.com/package/glob):

尽管通过JavaScript引用样式，然后进行构建，是一个很好的选择，但是也可以通过`entry`和[globbing](https://www.npmjs.com/package/glob)实现相同的结果。 基本思想是这样的：

```javascript
...
const glob = require('glob');

// Glob CSS files as an array of CSS files. This can be
// problematic due to CSS rule ordering so be careful!
const PATHS = {
  ...
leanpub-start-insert
  style: glob.sync('./app/**/*.css'),
leanpub-end-insert
};

...

const commonConfig = merge([
  {
    entry: {
      ...
leanpub-start-insert
      style: PATHS.style,
leanpub-end-insert
    },
    ...
  },
  ...
]);
```

After this type of change, you would not have to refer to styling from your application code. It also means that CSS Modules stop working. As a result, you should get both *style.css* and *style.js*. The latter file contains content like `webpackJsonp([1,3],[function(n,c){}]);` and it doesn't do anything as discussed in the [webpack issue 1967](https://github.com/webpack/webpack/issues/1967).

进行此类更改后，您不必在应用程序的代码中引用样式。这也意味着CSS模块将不再可用。最终，你将得到*style.css*和*style.js*两个文件。*style.js*文件将包含像`webpackJsonp（[1,3]，[function（n，c）{}]）;`的内容。正如在[webpack issue 1967](https://github.com/webpack/webpack/issues/1967)讨论的那样，它没有任何作用。

If you want strict control over the ordering, you can set up a single CSS entry and then use `@import` to bring the rest to the project through it. Another option would be to set up a JavaScript entry and go through `import` to get the same effect.

如果你想要严格控制排序，你可以设置一个单独的CSS入口，然后通过它使用`@import`引入剩余的项目。另一个选项是设置一个JavaScript入口并通过`import`获得相同的效果。

T> [css-entry-webpack-plugin](https://www.npmjs.com/package/css-entry-webpack-plugin) has been designed to help with this usage pattern. The plugin is able to extract a CSS bundle from an entry without `ExtractTextPlugin`.

T> [css-entry-webpack-plugin](https://www.npmjs.com/package/css-entry-webpack-plugin)被设计出来帮助解决使用这种模式。该插件可以在不需要`ExtractTextPlugin`的情况下从入口文件中提取CSS代码包。

## Conclusion
## 小结

The current setup separates styling from JavaScript neatly. Even though the technique is most valuable with CSS, it can be used to extract HTML templates or any other files types you consume. The hard part about `ExtractTextPlugin` has to do with its setup, but the complexity can be hidden behind an abstraction.

我们当前的设置清晰地分离了CSS和JavaScript。该技术不仅对CSS非常有用，它也可用于提取HTML模板或任何其他可能被使用的文件类型。 `ExtractTextPlugin`难以搞定是其设置相关的部分，但是复杂性都被隐藏在抽象后面（我们使用时，并没有想象的那么复杂）。

To recap:
回顾：

* Using `ExtractTextPlugin` with styling solves the problem of Flash of Unstyled Content (FOUC). Separating CSS from JavaScript also improves caching behavior and removes a potential attack vector.
* 对样式使用`ExtractTextPlugin`解决了（FOUC）的问题。将CSS与JavaScript分离还可以提高缓存效率并消除潜在的被攻击因素。

* `ExtractTextPlugin` is not the only solution. *extract-loader* can give the same result in more limited contexts.
* `ExtractTextPlugin`不是处理这个问题的唯一方法。*extract-loader*可以在更有限的上下文中给出相同的结果。

* If you don't prefer to maintain references to styling through JavaScript, an alternative is to handle them through an entry. You have to be careful with style ordering in this case, though.
* 如果你不喜欢通过JavaScript来保持对样式的引用，一个替代方法是通过单独的入口来处理它们。 在这种情况下，你必须小心的样式的顺序。

In the next chapter, you'll learn to **autoprefix**. Enabling the feature makes it more convenient to develop complex CSS setups that work with older browsers as well.

在下一章中，我们将讨论**自动前缀**技术。启用该功能将使开发复杂的CSS设置更加方便，这些设置也适用于旧版的浏览器。