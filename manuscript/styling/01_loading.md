# 加载样式文件

Webpack不能直接处理样式文件。需要你自己来根据自己的需求配置加载器和插件。

本章节，我们将在项目中使用CSS，并见证它是如何能够在浏览器中自动刷新并生效。更完美的是，webpack并不会全部刷新。它将更聪明的处理这种情况，我们稍后将见到。

## 加载CSS


为了加载CSS，我们需要使用[css-loader](https://www.npmjs.com/package/css-loader) 和 [style-loader](https://www.npmjs.com/package/style-loader)。*css-loader* 遍历可能的`@import`和`url()`，并查找与之匹配的文件，将其视为普通的ES6`import`。

这个过程允许我们基于其它加载器，例如[file-loader](https://www.npmjs.com/package/file-loader) or [url-loader](https://www.npmjs.com/package/url-loader)。如果一个`@import`指向一个外部资源，*css-loader*将会跳过。只有内部资源将会被webpack处理。

当*css-loader*处理完这部分后，*style-loader*获取输出，并将CSS注入到最终的构建包中。默认情况下，CSS将会以行内形式注入到JavaScript中，它实现了HMR的接口。由于行内形式在生产环境中并不是一个好主意，所以用`ExtractTextPlugin`来生成单独的样式文件就很有意义了。我们将在下一章节详细讨论。

首先，执行安装命令

```bash
npm install css-loader style-loader --save-dev
```

现在，我们得保证webpack可以处理它们。如下配置：

**webpack.parts.js**

```javascript
...

exports.loadCSS = function({ include, exclude } = {}) {
  return {
    module: {
      rules: [
        {
          test: /\.css$/,
          include,
          exclude,

          use: ['style-loader', 'css-loader'],
        },
      ],
    },
  };
};
```

我们也需要将配置文件同主配置文件联系起来。

**webpack.config.js**

```javascript
...

const commonConfig = merge([
  {
    ...
  },
leanpub-start-insert
  parts.loadCSS(),
leanpub-end-insert
]);

...
```

我们添加的配置表示：以`.css`结尾的文件将会执行分配的加载器。`test`匹配一个JavaScript格式的正则表达式。加载器是从右往左执行的。

T> 加载器就是转换器，接受被应用的文件，返回新的资源。加载器可以被串起来，就像Unix中的管道。`loaders: ['style-loader', 'css-loader']` 可读作为 `styleLoader(cssLoader(input))`.

T> 如果你想禁用*css-loader* `url`解析，可设置`url: false`。禁止解析imports也一样，可通过加载器配置项设置`import: false`。

## 创建初始化的CSS

我们仅缺少1bit：CSS代码如下：

**app/main.css**

```css
body {
  background: cornsilk;
}
```

Also, we’ll need to make webpack aware of it. Without having an entry pointing to it somehow, webpack won’t be able to find the file:
同样，我们也需要让webpack能够识别它。如果没有一个入口文件引用它，webpack将找不到该文件：

**app/index.js**

```javascript
leanpub-start-insert
import './main.css';
leanpub-end-insert
...
```

Execute `npm start` now. Browse to `http://localhost:8080` if you are using the default port and open up *main.css* and change the background color to something like `lime` (`background: lime`). Develop styles as needed to make it look a little nicer. Note that it does **not** perform a hard refresh on the browser since we have HMR setup in place.
现在执行`npm start`。如果你使用的是默认的端口在浏览器中打开`http://localhost:8080`，打开*main.css*文件，更改背景颜色，例如`lime` (`background: lime`)。样式开发如我们所需要使它更好看一点儿。请注意，由于我们已经设置了HMR，浏览器*并没有*表现出硬刷新，


We’ll continue from here in the next chapter. Before that, though, I will discuss styling-related techniques you may find useful. If you want, integrate some of them to your project.
我们将会从此处继续下一章节。在那之前，我将会讨论一些也许对你有用的样式相关的技术。如果你希望，你也可以在你的项目进行使用。


![Hello cornsilk world](images/hello_02.png)

T> An alternate way to load CSS would be to define a separate entry and point to the CSS there. Coupling styling to application code can be a nice way to handle it, though, as then you can see which styling is related to what file.
T> 加载CSS的一个可选的方案是定义单独的入口文件，并指向CSS。将样式和应用代码组合起来是一个很好的处理方式，那样你可以知道哪个样式与哪个文件相关。

## Understanding CSS Scoping and CSS Modules
## 理解CSS块和CSS模块

Perhaps the biggest challenge of CSS is that all rules exist within **global scope**. Due to this reason, specific conventions that work around this feature have been developed. The [CSS Modules](https://github.com/css-modules/css-modules) specification solves the problem by introducing **local scope** per `import`. As it happens, this makes CSS more bearable to use as you don’t have to worry about namespace collisions anymore.
也许，CSS最大的挑战就是*全局* 中存在的所有规则。由于这个原因，围绕解决该特征的特定约定被发展起来了。[CSS Modules](https://github.com/css-modules/css-modules)规范通过每个`import`引入局部变量。因为它的出现，使得CSS使用起来更可靠，你不必再担心命名空间冲突了。


Webpack’s *css-loader* supports CSS Modules. You can enable it through a loader definition like this:
Webpack的*css-loader*支持CSS模块。你可以通过如下的加载器定义来启用它：


```javascript
{
  loader: 'css-loader',
  options: {
    modules: true,
  },
},
```

After this change, your class definitions will remain local to the files. In case you want global class definitions, you’ll need to wrap them within `:global(.redButton) { ... }` kind of declarations.
如此更改后，你的样式class定义将保留在本地。当你想定义全局的类时，你需要用类似于`:global(.redButton) { ... }`的申明来包裹它们。

In this case, the `import` statement will give you the local classes you can then bind to elements. Assume we had CSS like this:
在这中情况下，`import` 语法将向你提供本地类，你可以将其绑定到元素上。假设我们又下面的CSS文件：

**app/main.css**

```css
body {
  background: cornsilk;
}

.redButton {
  background: red;
}
```

We could then bind the resulting class to a component like this:
我们可以将结果通过如下方式绑定到一个组件：

**app/component.js**

```javascript
import styles from './main.css';

...

// Attach the generated class name
element.className = styles.redButton;
```

Note that `body` remains as a global declaration still. It’s that `redButton` that makes the difference. You can build component-specific styles that don’t leak elsewhere this way.
请注意，`body`仍然保留了一个全局的声明。被改变的仅仅是`redButton`。你可以创建特定于组件的样式，从而不会污染其它地方。

CSS Modules provides also features like composition to make it even easier to work with your styles. You can also combine it with other loaders as long as you apply them before *css-loader*.
CSS Modules还提供了如组合的特性使其更容易让你编写样式。你也可以将其同其它加载器一起使用，只需要在*css-loader*之前引用即可。

T> CSS Modules behavior can be modified [as discussed in the official documentation](https://www.npmjs.com/package/css-loader#local-scope). You have control over the names it generates for instance.
T> CSS Modules 的行为可以被更改，如在[官方文档](https://www.npmjs.com/package/css-loader#local-scope)中被讨论的。例如你可以控制其生成的名称。

T> [eslint-plugin-css-modules](https://www.npmjs.com/package/eslint-plugin-css-modules) is useful for tracking CSS Modules related problems.
T> [eslint-plugin-css-modules](https://www.npmjs.com/package/eslint-plugin-css-modules)在跟踪CSS Modules相关的问题上非常有用。

## Loading Less

![Less](images/less.png)

[Less](http://lesscss.org/) is a CSS processor packed with functionality. Using Less doesn’t take a lot of effort through webpack as [less-loader](https://www.npmjs.com/package/less-loader) deals with the heavy lifting. You should install [less](https://www.npmjs.com/package/less) as well given it’s a peer dependency of *less-loader*.
[Less](http://lesscss.org/)是一个CSS预处理器。借助[less-loader](https://www.npmjs.com/package/less-loader)来处理复杂的加载工作，在Webpack中简单配置一下就可以使用Less。你需要安装Less依赖包，和对应的加载器`less-loader`。

Consider the following minimal setup:
如下面的一小段代码进行配置：

```javascript
{
  test: /\.less$/,
  use: ['style-loader', 'css-loader', 'less-loader'],
},
```

The loader supports Less plugins, source maps, and so on. To understand how those work you should check out the project itself.
这个加载器支持Less插件，source maps等。如果你想更好的理解其工作原理，请查看[Less](http://lesscss.org/)项目本身。

## Loading Sass
## 加载Sass

![Sass](images/sass.png)

[Sass](http://sass-lang.com/) is a widely used CSS preprocessor. You should use [sass-loader](https://www.npmjs.com/package/sass-loader) with it. Remember to install [node-sass](https://www.npmjs.com/package/node-sass) to your project as it’s a peer dependency.
[Sass](http://sass-lang.com/) 是一个被广泛使用的CSS预处理器。你需要安装[sass-loader](https://www.npmjs.com/package/sass-loader)。请记住在项目中安装依赖包[node-sass](https://www.npmjs.com/package/node-sass)。

Webpack doesn’t take much configuration:
Webpack仅需如下配置：

```javascript
{
  test: /\.scss$/,
  use: ['style-loader', 'css-loader', 'sass-loader'],
},
```

T> If you want more performance, especially during development, check out [fast-sass-loader](https://www.npmjs.com/package/fast-sass-loader).
T> 如果你希望更好的性能，尤其是在开发环境中，请查看[fast-sass-loader](https://www.npmjs.com/package/fast-sass-loader)。


## Loading Stylus and Yeticss

![Stylus](images/stylus.png)

[Stylus](http://stylus-lang.com/) is yet another example of a CSS processor. It works well through [stylus-loader](https://github.com/shama/stylus-loader). [yeticss](https://www.npmjs.com/package/yeticss) is a pattern library that works well with it. Consider the following configuration:
[Stylus](http://stylus-lang.com/)是另一个CSS预处理器。通过[stylus-loader](https://github.com/shama/stylus-loader)来工作。[yeticss](https://www.npmjs.com/package/yeticss)是其一个对应的模式库。Webpack如下配置：

```javascript
{
  ...
  module: {
    rules: [
      {
        test: /\.styl$/,
        use: ['style-loader', 'css-loader', 'stylus-loader'],
      },
    ],
  },
  plugins: [
    new webpack.LoaderOptionsPlugin({
      options: {
        // yeticss
        stylus: {
          use: [require('yeticss')],
        },
      },
    }),
  ],
},
```

To start using yeticss with Stylus, you must import it to one of your app’s *.styl* files:
为了将yeticss和Stylus结合起来使用，你必须将它引入到一个*.styl*文件：

```javascript
@import 'yeticss'

//or
@import 'yeticss/components/type'
```

## PostCSS

![PostCSS](images/postcss.png)

[PostCSS](http://postcss.org/) allows you to perform transformations over CSS through JavaScript plugins. You can even find plugins that provide you Sass-like features. PostCSS is the equivalent of Babel for styling. [postcss-loader](https://www.npmjs.com/package/postcss-loader) allows using it with webpack.
[PostCSS](http://postcss.org/)允许你通过JavaScript插件来对CSS进行转换。你甚至能找到可提供Sass类似特性的插件。PostCSS对于样式来说就像Babel。[postcss-loader](https://www.npmjs.com/package/postcss-loader)允许你将它和webpack一起使用。

The example below illustrates how to set up autoprefixing using PostCSS. It also sets up [precss](https://www.npmjs.com/package/precss), a PostCSS plugin that allows you to use Sass-like markup in your CSS. You can mix this technique with other loaders to allow autoprefixing there.
下面的例子向你展示了如何使用PostCSS来设置自动前缀的功能。它还设置了[precss](https://www.npmjs.com/package/precss)，一个PostCSS插件，让你能够在CSS中使用Sass一样的标记。你可以将这个技术和其它加载器结合起来实现自动前缀。

```javascript
{
  test: /\.css$/,
  use: [
    'style-loader',
    'css-loader',
    {
      loader: 'postcss-loader',
      options: {
        plugins: () => ([
          require('autoprefixer'),
          require('precss'),
        ]),
      },
    },
  ],
},
```

For this to work, you will have to remember to include [autoprefixer](https://www.npmjs.com/package/autoprefixer) and [precss](https://www.npmjs.com/package/precss) to your project. The technique is discussed in detail in the *Autoprefixing* chapter.
为了让其生效，你需要在项目中安装[autoprefixer](https://www.npmjs.com/package/autoprefixer) 和 [precss](https://www.npmjs.com/package/precss)

T> PostCSS supports also *postcss.config.js* based configuration. It relies on [cosmiconfig](https://www.npmjs.com/package/cosmiconfig) internally. It can pick up configuration from your *package.json*, JSON or YAML, or that you can even push your configuration below an arbitrary directory. *cosmiconfig* will find it. The problem is that this style is harder to compose than inline configuration.
T> PostCSS还支持基于*postcss.config.js*的配置。它内部依赖于[cosmiconfig](https://www.npmjs.com/package/cosmiconfig)。它可以从你的 *package.json*、JSON、YAML 中找到配置信息。或者甚至可以将配置放到任意目录。*cosmiconfig*能够找到它。问题是这种风格比内联的配置方法更难撰写（组合）。


### cssnext

![cssnext](images/cssnext.jpg)

[cssnext](http://cssnext.io/) is a PostCSS plugin that allows us to experience the future now. It comes with some restrictions as it’s not possible to adapt to each feature, but it may be worth a go. You can use it through [postcss-cssnext](https://www.npmjs.com/package/postcss-cssnext), and you can enable it as follows:
[cssnext](http://cssnext.io/)是一个PostCSS插件，允许我们现在就可以体验未来的特性。它有一些限制，因为它不可能适应每个功能，但它可能值得尝试。 您可以通过[postcss-cssnext]（https://www.npmjs.com/package/postcss-cssnext）使用它，您可以按如下所示启用它：
```javascript
{
  test: /\.css$/,
  use: [
    'style-loader',
    'css-loader',
    {
      loader: 'postcss-loader',
      options: {
        plugins: () => ([
          require('cssnext'),
        ]),
      },
    },
  ],
},
```

See [the usage documentation](http://cssnext.io/usage/) for available options.
有关可用选项，请参见[使用文档]（http://cssnext.io/usage/）。

T> Note that cssnext includes autoprefixer! You don’t have to configure autoprefixing separately for it to work in this case.
T> 注意cssnext包含自动前缀功能！ 在这种情况下，你不必单独配置自动前缀。


## Understanding Lookups
## 理解查找

To get most out of *css-loader*, you should understand how it performs its lookups. Even though *css-loader* handles relative imports by default, it won’t touch absolute imports (`url("/static/img/demo.png")`). If you rely on imports like this, you will have to copy the files to your project.
要充分利用*css-loader*，您应该了解它如何执行查找。 *css-loader*默认处理相对导入，它不会执行绝对导入（`url（“/static/img/demo.png”`）。 如果你依赖这样的导入，你必须将文件复制到你的项目。

[copy-webpack-plugin](https://www.npmjs.com/package/copy-webpack-plugin) works for this purpose, but you can also copy the files outside of webpack. The benefit of the former approach is that webpack-dev-server can pick that up.
[copy-webpack-plugin](https://www.npmjs.com/package/copy-webpack-plugin) 可实现这个目的，但您也可以将文件复制到webpack外部。 前一种方法的好处是，webpack-dev-server可以选择。

T> [resolve-url-loader](https://www.npmjs.com/package/resolve-url-loader) will come in handy if you use Sass or Less. It adds support for relative imports to the environments.
T> [resolve-url-loader](https://www.npmjs.com/package/resolve-url-loader)将会在你使用Sass、Less时派上用场。它增加了对环境想对导入的支持。

### Processing *css-loader* Imports

If you want to process *css-loader* imports in a specific way, you should set up `importLoaders` option to a number that tells the loader how many loaders after the *css-loader* should be executed against the imports found. If you import other CSS files from your CSS through the `@import` statement and want to process the imports through specific loaders, this is particularly useful.
如果你想以特定的方式处理*css-loader*的引入，你应该设置`importLoaders`选项为一个数字，告诉加载器*css-loader*之后的还有多少个加载器，需要对找到的内容执行操作。 如果通过`@import`语句从CSS文件中导入其他CSS文件，并且希望通过特定加载器处理导入，这是特别有用的。

Consider the following import from a CSS file:
考虑如下的情景，从CSS文件中引入另外一个Sass文件：

```css
@import "./variables.sass";
```

To process the Sass file, you would have to write configuration like this:
为了对Sass文件进行处理，你需要如下配置：

```javascript
{
  test: /\.css$/,
  use: [
    'style-loader',
    {
      loader: 'css-loader',
      options: {
        importLoaders: 1,
      },
    },
    'sass-loader',
  ],
},
```

If you added more loaders, such as *postcss-loader*, to the chain, you would have to adjust the `importLoaders` option accordingly.
如果你在加载器链中添加了更多的加载器，例如*postcss-loader*，你必须相应地调整`importLoaders`选项。

### Loading from *node_modules* Directory
### 从*node_modules*文件夹加载

You can load files directly from your node_modules directory. Consider Bootstrap and its usage for example:
你可以从*node_modules*文件夹加载文件。对于*Bootstrap*，用法如下：

```less
@import "~bootstrap/less/bootstrap";
```

The tilde character (`~`) tells webpack that it’s not a relative import as by default. If tilde is included, it will perform a lookup against `node_modules` (default setting) although this is configurable through the [resolve.modules](https://webpack.js.org/configuration/resolve/#resolve-modules) field.
波浪符号（`〜`）告诉webpack它不是默认的相对导入。 如果包括波浪符号（`〜`），它将对`node_modules`（默认设置）执行查找，尽管这可以通过[resolve.modules]（https://webpack.js.org/configuration/resolve/#resolve-modules）字段配置 。

W> If you are using *postcss-loader*, you can skip using `~` as discussed in [postcss-loader issue tracker](https://github.com/postcss/postcss-loader/issues/166). *postcss-loader* can resolve the imports without a tilde.
W> 
## Enabling Source Maps

If you want to enable source maps for CSS, you should enable `sourceMap` option for *css-loader* and set `output.publicPath` to an absolute url pointing to your development server. If you have multiple loaders in a chain, you have to enable source maps separately for each. *css-loader* [issue 29](https://github.com/webpack/css-loader/issues/29) discusses this problem further.
如果你使用* postcss-loader *，你可以跳过使用`〜`，如[postcss-loader issue tracker]（https://github.com/postcss/postcss-loader/issues/166）中所讨论的。 * postcss-loader *可以解析导入，而不使用波浪号。

## Using Bootstrap

There are a couple of ways to use [Bootstrap](https://getbootstrap.com/) through webpack. One option is to point to the [npm version](https://www.npmjs.com/package/bootstrap) and perform loader configuration as above.
有几种方法可以通过webpack使用[Bootstrap]（https://getbootstrap.com/）。 一个选项是指向[npm版本]（https://www.npmjs.com/package/bootstrap），并执行如上所述的加载程序配置。

The [Sass version](https://www.npmjs.com/package/bootstrap-sass) is another option. In this case, you should set `precision` option of *sass-loader* to at least 8. This is [a known issue](https://www.npmjs.com/package/bootstrap-sass#sass-number-precision) explained at *bootstrap-sass*.
[Sass版本]（https://www.npmjs.com/package/bootstrap-sass）是另一个选项。 在这种情况下，您应该将*sass-loader*的'precision'选项设置为至少8.这是一个[a known issue](https://www.npmjs.com/package/bootstrap-sass#sass-number-precision),在*bootstrap-sass*有详细解释。

The third option is to go through [bootstrap-loader](https://www.npmjs.com/package/bootstrap-loader). It does a lot more but allows customization.
第三个选项是通过[bootstrap-loader]（https://www.npmjs.com/package/bootstrap-loader）。 它可以做的梗多，但也可以定制。

## Converting CSS to Strings
## 将CSS转换为字符串

Especially with Angular 2, it can be useful if you can get CSS in a string format that can be pushed to components. [css-to-string-loader](https://www.npmjs.com/package/css-to-string-loader) achieves exactly this.
特别是对于Angular 2，它可以是很有用的，如果你可以获得CSS的字符串格式，进而可以推送到组件。 [css-to-string-loader]（https://www.npmjs.com/package/css-to-string-loader）实现了这一点。

## Conclusion
## 小结

Loading styles through webpack is easy. It even supports advanced specifications like [CSS Modules](https://github.com/css-modules/webpack-demo). The approaches covered here inline the styling by default.
通过webpack加载样式很容易。 它甚至支持高级规范，如[CSS模块]（https://github.com/css-modules/webpack-demo）。 这里默认介绍的方法内联样式方法。

To recap:
回顾：

* *css-loader* evaluates the `@import` and `url()` definitions of your styling. *style-loader* converts it to JavaScript and implements webpack’s Hot Module Replacement interface.
* *css-loader*执行处理了样式文件中定义的`@import` 和 `url()`。*style-loader*将其转换为JavaScript，并实现了webpack的热加载接口。

* *css-loader* supports the CSS Modules specification. CSS Modules allow you maintain CSS in a local scope by default solving the biggest issue of CSS.
* *css-loader*支持CSS模块化定义。CSS模块允许你将CSS维护在局部，进而解决了CSS中最大的问题。

* Webpack supports a large variety of formats compiling to CSS through loaders. These include Sass, Less, and Stylus.
* Webpack支持通过加载器将各种格式的样式文件编译为CSS。包括Sass, Less, and Stylus。

* PostCSS allows you to inject functionality to CSS in through its plugin system. cssnext is an example of a collection of plugins for PostCSS that implements future features of CSS.
* PostCSS允许你通过其插件系统向CSS中注入功能。cssnext是这个插件集合中的一个例子，实现了可以是我们使用用CSS一些未来特性的功能。

* *css-loader* won’t touch absolute imports by default. It allows customization of loading behavior through the `importLoaders` option. You can perform lookups against *node_modules* by prefixing your imports with a tilde (`~`) character.
* *css-loader*默认情况下，将不会处理绝对路径的引入。可以通过`importLoaders`配置项来实现加载行为的自定义。你可以通过在路径中增加波浪符号（`~`）来执行对*node_modules*目录的查找。


* To use source maps, you have to enable `sourceMap` boolean through each style loader you are using except for *style-loader*. You should also set `output.publicPath` to an absolute url that points to your development server.
* 为了能够使用source maps，你需要通过对除了*style-loader*之外的每一个样式加载器启用`sourceMap`。您还应该将`output.publicPath`设置为指向开发服务器的绝对URL。

* Using Bootstrap with webpack requires special care. You can either go through generic loaders or a bootstrap specific loader for more customization options.
* 使用Bootstrap和webpack需要特别小心。 您可以通过通用加载程序或引导程序特定加载程序来获得更多自定义选项。


Although the loading approach covered here is enough for development purposes, it’s not ideal for production as it inlines the styling to the JavaScript bundles. We’ll cover this problem in the next chapter as we learn to separate CSS.
虽然这里描述的加载方法足够用于开发目的，但它对于生产来说并不理想，因为它在JavaScript中内联了样式代码。 我们将在下一章学习分离CSS时，讨论这个问题。