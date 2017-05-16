# Eliminating Unused CSS
# 去掉无用的CSS

Frameworks like [Bootstrap](https://getbootstrap.com/) tend to come with a lot of CSS. Often you use only a small part of it. Typically, you bundle even the unused CSS. It's possible, however, to eliminate the portions you aren't using.

类似于[Bootstrap](https://getbootstrap.com/)这样的框架有很多CSS。通常情况下，你仅仅是用了其中的一部分。但是你将没有使用的CSS也打到包里了。然而，将没有被使用的部分去掉，也是可行的。

[PurifyCSS](https://www.npmjs.com/package/purifycss) is a tool that can achieve this by analyzing files. It walks through your code and figures out which CSS classes are being used. Often there is enough information for it to strip unused CSS from your project. It also works with single page applications to an extent.

[PurifyCSS](https://github.com/purifycss/purifycss)是一个通过分析文件来实现这个目的的插件。它遍历你的代码，找出哪些CSS被使用了。大多数时候，可以找到足够的信息来删除项目中未被使用的代码。在某种程度上，它也适用于单页应用程序。

[uncss](https://www.npmjs.com/package/uncss) is a good alternative to PurifyCSS. It operates through PhantomJS and performs its work in a different manner. You can use uncss itself as a PostCSS plugin.

[uncss](https://www.npmjs.com/package/uncss)是另外一个很好的替代方案。它通过PhantomJS进行操作，以一种不同的方式来执行它的工作。你可以将它作为一个PostCSS的插件来使用。

[!delete] W> The approach does **not** work with CSS Modules and `ExtractTextPlugin` yet. See [issue
 97](https://github.com/webpack-contrib/purifycss-webpack/issues/97) for more information.

[!delete] W> 该方式还**不能**同CSS Modules和`ExtractTextPlugin`一起使用。详情可查看[issue 97](https://github
.com/webpack-contrib/purifycss-webpack/issues/97)。

W> You have to be careful if you are using CSS Modules. You have to **whitelist** the related classes as discussed in [purifycss-webpack readme](https://github.com/webpack-contrib/purifycss-webpack#usage-with-css-modules).

## Setting Up Pure.css
## 设置Pure.css

To make the demo more realistic, let's install [Pure.css](http://purecss.io/), a small CSS framework, as well and refer to it from the project so that you can see PurifyCSS in action. These two projects aren't related in any way despite the naming.

为了使我们的例子更加真实，我们安装一个小型框架[Pure.css](http://purecss.io/)，在项目中引用这个框架，这样你就可以看到PurifyCSS的效果。这两个项目除了名字相似外，并没有其它关系。

```bash
npm install purecss --save
```

{pagebreak}

To make the project aware of Pure.css, `import` it:

为了让项目使用Pure.css，引入它：

**app/index.js**

```javascript
leanpub-start-insert
import 'purecss';
leanpub-end-insert
...
```

You should also make the demo component use a Pure.css class, so there is something to work with:

我们还需要使我们项目中的组件，使用一个`Pure.css`的样式类，所以我们可以如下修改：

**app/component.js**

```javascript
module.exports = () => {
  const element = document.createElement('div');

leanpub-start-insert
  element.className = 'pure-button';
leanpub-end-insert
  element.innerHTML = 'Hello world';

  return element;
};
```

If you run the application (`npm start`), the "Hello world" should look like a button.

如果现在执行`npm start`，"Hello world"看起来应该是一个按钮。

![Styled hello](https://raw.githubusercontent.com/survivejs-translations/webpack-book-zh/master/manuscript/images/styled-button.png)

{pagebreak}

Building the application (`npm run build`) should yield output:

构建应用，执行`npm run build`，将会输出如下信息：

```bash
Hash: e8601af01b5c5a5e722c
Version: webpack 2.2.1
Time: 1255ms
     Asset       Size  Chunks             Chunk Names
    app.js    4.15 kB       0  [emitted]  app
   app.css    16.5 kB       0  [emitted]  app
index.html  218 bytes          [emitted]
   [0] ./app/component.js 185 bytes {0} [built]
   [1] ./app/main.css 41 bytes {0} [built]
   [2] ./~/purecss/build/pure-min.css 41 bytes {0} [built]
...
```

As you can see, the size of the CSS file grew. This is something to fix with PurifyCSS.

正如你所见，CSS文件的大小有所增加。我们将用PurifyCSS来解决这个问题。

## Enabling PurifyCSS
## 启用PurifyCSS

Using PurifyCSS can lead to significant savings. In the official example of the project, they purify and minify Bootstrap (140 kB) in an application using ~40% of its selectors to mere ~35 kB. That's a big difference.

使用PurifyCSS可以显着节省成本。在有些例子中，他们在一个应用程序中使用大约40％的选择器来纯化和缩减Bootstrap（140 kB），直到35 kB左右。这是一个很大的区别。

[purifycss-webpack](https://www.npmjs.com/package/purifycss-webpack) allows to achieve similar results. You should use the `ExtractTextPlugin` with it for the best results. Install it and a [glob](https://www.npmjs.org/package/glob) helper first:

[purifycss-webpack](https://www.npmjs.com/package/purifycss-webpack)允许我们实现这样的效果。你应该使用`ExtractTextPlugin`来获得最好的结果。首先安装它和[glob](https://www.npmjs.org/package/glob)助手：

```bash
npm install glob purifycss-webpack --save-dev
```

{pagebreak}

You also need PurifyCSS configuration as below:

我们还需要对PurifyCSS设置配置。如下扩展parts文件：

**webpack.parts.js**

```javascript
...
leanpub-start-insert
const PurifyCSSPlugin = require('purifycss-webpack');
leanpub-end-insert

...

leanpub-start-insert
exports.purifyCSS = ({ paths }) => ({
  plugins: [
    new PurifyCSSPlugin({ paths }),
  ],
});
leanpub-end-insert
```

Next, the part has to be connected with the configuration. It's important the plugin is used *after* the `ExtractTextPlugin`; otherwise it doesn't work:

接下来，我们将它挂在到配置中。重要的是，我们需要使这个插件在`ExtractTextPlugin`*之后*被使用；否则它将不会生效。

**webpack.config.js**

```javascript
...
leanpub-start-insert
const glob = require('glob');
leanpub-end-insert

const parts = require('./webpack.parts');

...

const productionConfig = merge([
  ...
leanpub-start-insert
  parts.purifyCSS({
    paths: glob.sync(`${PATHS.app}/**/*.js`, { nodir: true }),
  }),
leanpub-end-insert
]);
```

W> The order matters. CSS extraction has to happen before purifying.

W> 请注意顺序很重要。CSS提取必须在CSS纯化之前发生。

If you execute `npm run build` now, you should see something:

如果你现在`npm run build`，你将的到如下信息：

```bash
Hash: e8601af01b5c5a5e722c
Version: webpack 2.2.1
Time: 1363ms
     Asset       Size  Chunks             Chunk Names
    app.js    4.15 kB       0  [emitted]  app
   app.css    2.19 kB       0  [emitted]  app
index.html  218 bytes          [emitted]
   [0] ./app/component.js 185 bytes {0} [built]
   [1] ./app/main.css 41 bytes {0} [built]
   [2] ./~/purecss/build/pure-min.css 41 bytes {0} [built]
...
```

The size of the style has decreased noticeably. Instead of 16k, you have roughly 2k now. The difference would be even bigger for heavier CSS frameworks.

样式文件的体积明显减少了。由之前的16k到现在的2k。这个差距对于大型CSS框架来说将会更加明显。

PurifyCSS supports [additional options](https://github.com/purifycss/purifycss#the-optional-options-argument) including `minify`. You can enable these through the `purifyOptions` field when instantiating the plugin. Given PurifyCSS cannot pick all of the classes you are using always, you should use `purifyOptions.whitelist` array to define selectors which it should leave in the result no matter what.

PurifyCSS支持[附加选项](https://github.com/purifycss/purifycss#the-optional-options-argument)，包括`minify`。 您可以在实例化插件时通过`purifyOptions`字段启用它们。由于PurifyCSS可能不会选择您正在使用的所有类，您应该使用`purifyOptions.whitelist`数组来定义，应该留在结果中的选择器。


W> Using PurifyCSS loses CSS source maps even if you have enabled them through loader specific configuration due to the way it works underneath.

W> 因为它的工作方式,使用PurifyCSS将丢失CSS source maps，即使您已通过加载器明确配置启用它们。

{pagebreak}

### Critical Path Rendering

The idea of [critical path rendering](https://developers.google.com/web/fundamentals/performance/critical-rendering-path/) takes a look at CSS performance from a different angle. Instead of optimizing for size, it optimizes for render order and puts emphasis on **above-the-fold** CSS.

[关键路径渲染](https://developers.google.com/web/fundamentals/performance/critical-rendering-path/)的想法是从不同的角度来看待CSS性能。而不是优化大小，它优化渲染顺序和强调**above-the-fold** CSS。


[isomorphic-style-loader](https://www.npmjs.com/package/isomorphic-style-loader) achieves this using webpack and React. [critical-path-css-tools](https://github.com/addyosmani/critical-path-css-tools) listing by Addy Osmani lists other related tools.

[isomorphic-style-loader](https://www.npmjs.com/package/isomorphic-style-loader) 使用webpack和React实现这一点。 Addy Osmani列出的[critical-path-css-tools](https://github.com/addyosmani/critical-path-css-tools)列出了其他相关工具。

## Conclusion

Using PurifyCSS can lead to a significant decrease in file size. It's particularly valuable for static sites that rely on a heavy CSS framework. The more dynamic a site or an application becomes, the harder it becomes to analyze reliably.

使用PurifyCSS可以使文件大小显著减少。 它对依赖于大型CSS框架的静态网站特别有用。站点或应用程序变得越动态，就越难以可靠地分析。

To recap:

回顾：

* Eliminating unused CSS is possible using PurifyCSS. It performs static analysis against the source.

* 使用PurifyCSS可以消除未使用的CSS。它对源代码执行静态分析。

* The functionality can be enabled through *purifycss-webpack* and the plugin should be applied *after* `ExtractTextPlugin`.

* 该功能可以通过*purifycss-webpack*启用，插件应该在`ExtractTextPlugin`之后应用。

* At best, PurifyCSS can eliminate most, if not all, unused CSS rules.

* 充其量，如果不是全部，PurifyCSS也可以消除大多数未使用的CSS规则。

* Critical path rendering is another CSS technique that puts emphasis on rendering the above-the-fold CSS first. The idea is to render something as fast as possible instead of waiting for all CSS to load.

* 关键路径渲染是另一种CSS技术，强调首先渲染above-the-fold的CSS。 这个想法是尽可能快地渲染，而不是等待所有的CSS加载。

The styling portion of the demo is in good shape. It can be made easier to develop by including CSS linting to the project.

我们演示项目的样式部分现在已经初具形状了。我们可以通过在项目中支持CSS语法检测来使开发更加容易。下一章节，我们将实现这个。
