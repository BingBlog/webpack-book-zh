# Loading Styles
# 加载样式

Webpack doesn't handle styling out of the box and you will have to use loaders and plugins to allow loading style files. In this chapter, you will set up CSS with the project and see how it works out with automatic browser refreshing. When you make a change to the CSS webpack doesn't have to force a full refresh. Instead, it can patch the CSS without one.

Webpack不能直接处理样式文件。需要您自己配置加载器和插件，来使其可以加载样式文件。本章节，你将在项目中使用CSS，并见证它是如何能够在浏览器中自动刷新并生效。当你更改了CSS，并不需要正真的刷新浏览器，取而代之的是，webpack用打补丁的方式来避免刷新。

## Loading CSS
## 加载CSS

To load CSS, you need to use [css-loader](https://www.npmjs.com/package/css-loader) and [style-loader](https://www.npmjs.com/package/style-loader). *css-loader* goes through possible `@import` and `url()` lookups within the matched files and treats them as a regular ES6 `import`. If an `@import` points to an external resource, *css-loader* skips it as only internal resources get processed further by webpack.

为了加载CSS，你需要使用[css-loader](https://www.npmjs.com/package/css-loader) 和 [style-loader](https://www.npmjs.com/package/style-loader)。*css-loader* 遍历所有的`@import`和`url()`，并查找与之匹配的文件，将其视为普通的ES6`import`。如果一个`@import`指向外部资源，*css-loader*将会忽略它，因为只有内部资源将被webpack进一步处理。

*style-loader* injects the styling through a `style` element. The way it does this can be customized. It also implements the *Hot Module Replacement* interface providing good development experience.

*style-loader*通过一个`style`元素将样式注入。处理该步骤的方式也可以自定义。它通过实现*热加载(Hot Module Replacement)*接口提升了开发体验。

The matched files can be processed through loaders like [file-loader](https://www.npmjs.com/package/file-loader) or [url-loader](https://www.npmjs.com/package/url-loader) and these possibilities are discussed in the *Loading Assets* part of the book.

匹配的文件可以通过诸如[file-loader](https://www.npmjs.com/package/file-loader)和[url-loader](https://www.npmjs.com/package/url-loader)这些加载器来处理。在*Loading Assets*部分对这些方法进行了讨论。

Since inlining CSS isn't a good idea for production usage, it makes sense to use `ExtractTextPlugin` to generate a separate CSS file. You will do this in the next chapter.

默认情况下webpack会将CSS内联在JavaScript中，这在生产环境中并不是一个好主意，所以用`ExtractTextPlugin`来生成单独的样式文件就很有意义了。我们将在下一章节详细讨论。

{pagebreak}

To get started, invoke

首先，（终端）执行：

```bash
npm install css-loader style-loader --save-dev
```

Now let's make sure webpack is aware of them. Configure as follows:

现在，我们需要确认webpack能够识别它们。如下配置：

**webpack.parts.js**

```javascript
exports.loadCSS = ({ include, exclude } = {}) => ({
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
});
```

You also need to connect the fragment with the main configuration:

我们需要将这段代码应用到主配置文件中：

**webpack.config.js**

```javascript
const commonConfig = merge([
  ...
leanpub-start-insert
  parts.loadCSS(),
leanpub-end-insert
]);
```

The added configuration means that files ending with `.css` should invoke the given loaders. `test` matches against a JavaScript-style regular expression.

我们添加的配置表示以`.css`结尾的文件将会调用指定的加载器。`test`将匹配一个JavaScript格式的正则表达式。

Loaders are transformations that are applied to source files, and return the new source and can be chained together like a pipe in Unix. They evaluated from right to left. This means that `loaders: ['style-loader', 'css-loader']` can be read as `styleLoader(cssLoader(input))`.

 加载器就是转换器，接受需要被处理的文件，返回新的资源。加载器可以被串起来，就像Unix中的管道。它们按照从右到左的顺序执行。`loaders: ['style-loader', 'css-loader']` 可读作为 `styleLoader(cssLoader(input))`.

T> If you want to disable *css-loader* `url` parsing, set `url: false`. The same idea applies to `@import` as to disable parsing imports you can set `import: false` through the loader options.

T> 如果您想禁用*css-loader* 的`url`解析，可设置`url: false`。禁止解析imports也一样，可通过加载器配置项设置`import: false`。

## Setting Up the Initial CSS
## 创建初始化的CSS

You are missing the CSS still:

你现在还缺少CSS代码：

**app/main.css**

```css
body {
  background: cornsilk;
}
```

Also, you need to make webpack aware of it. Without having an entry pointing to it somehow, webpack is not able to find the file:

同样，我们也需要让webpack能够识别它。如果没有一个入口文件引用它，webpack将找不到该文件：

**app/index.js**

```javascript
leanpub-start-insert
import './main.css';
leanpub-end-insert
...
```

Execute `npm start` now. Browse to `http://localhost:8080` if you are using the default port and open up *main.css* and change the background color to something like `lime` (`background: lime`). Develop styles as needed to make it look a nicer.

现在执行`npm start`。如果您使用的是默认的端口，在浏览器中打开`http://localhost:8080`，打开*main.css*文件，更改背景颜色，例如`lime` (`background: lime`)。根据我们的需要，使它更好看一点儿。

You continue from here in the next chapter. Before that, though, you'll learn about styling-related techniques.
下一章节，我们将会从此处继续。接下来，我将会讨论一些样式相关的技术。

![Hello cornsilk world](images/hello_02.png)

## Understanding CSS Scoping and CSS Modules

Perhaps the biggest challenge of CSS is that all rules exist within **global scope**, meaning that two classes with the same name will collide. The limitation is inherent to the CSS specification but projects have workarounds for the issue. [CSS Modules](https://github.com/css-modules/css-modules) introduces **local scope** for every module by making every class declared within unique by including a hash in their name that is globally unique to the module.

也许，CSS最大的挑战就是所有规则都存在于**全局**中，这意味着具有相同名称的两个类将会相互冲突。这个限制源自于CSS规范，但有些项目也在解决这个问题。[CSS Modules](https://github.com/css-modules/css-modules)为每个模块引入**局部规则**，通过对每个类的声明中添加一个对该模块来说全局唯一的哈希值，来使每个类的声明都是唯一的。

Webpack's *css-loader* supports CSS Modules. You can enable it through a loader definition:

Webpack的*css-loader*支持CSS模块。您可以通过对加载器进行修改，设置如下配置项来启用它：

```javascript
{
  loader: 'css-loader',
  options: {
    modules: true,
  },
},
```

After this change, your class definitions remain local to the files. In case you want global class definitions, you need to wrap them within `:global(.redButton) { ... }` kind of declarations.

如此更改后，您的定义的类将仅在本地文件有效。当您想定义全局的类时，您需要用类似于`:global(.redButton) { ... }`的申明来包裹它们。

{pagebreak}

In this case, the `import` statement gives you the local classes you can then bind to elements. Assume you had CSS as below:

在这中情况下，`import` 语法将向您提供本地类，您可以将其绑定到元素上。假设我们有下面的CSS文件：

**app/main.css**

```css
body {
  background: cornsilk;
}

.redButton {
  background: red;
}
```

You could then bind the resulting class to a component:

我们可以将这个类，通过下面的方式绑定到一个组件：

**app/component.js**

```javascript
import styles from './main.css';

...

// Attach the generated class name
element.className = styles.redButton;
```

`body` remains as a global declaration still. It's that `redButton` that makes the difference. You can build component-specific styles that don't leak elsewhere this way.

请注意，`body`仍然保留了一个全局的声明。被改变的仅仅是`redButton`。您可以用这种方式创建特定于组件的样式，从而不会污染其它地方。

CSS Modules provides additional features like composition to make it easier to work with your styles. You can also combine it with other loaders as long as you apply them before *css-loader*.

CSS Modules还提供了组合的特性，使其更容易让您编写样式。您也可以将其同其它加载器一起使用，只需要在*css-loader*之前应用即可。

T> CSS Modules behavior can be modified [as discussed in the official documentation](https://www.npmjs.com/package/css-loader#local-scope). You have control over the names it generates for instance.

T> CSS Modules的行为可以被更改，如在[官方文档](https://www.npmjs.com/package/css-loader#local-scope)中被讨论的。比如说您可以控制其生成的名称。

T> [eslint-plugin-css-modules](https://www.npmjs.com/package/eslint-plugin-css-modules) is handy for tracking CSS Modules related problems.

T> [eslint-plugin-css-modules](https://www.npmjs.com/package/eslint-plugin-css-modules)在跟踪CSS Modules相关的问题方面非常有用。

### Using CSS Modules with Third Party Libraries and CSS
### 将第三方库和CSS模块一起使用

If you are using CSS Modules in your project, you should process normal CSS through a separate loader definition without the `modules` option of *css-loader* enabled. Otherwise all classes will be scoped to their module. In the case of third party libraries this is almost certainly not what you want.

如果你在项目中使用的CSS模块，你应该通过单独定义的没有`modules`配置项的加载器来处理普通的CSS。否则所有的类都将被包裹到它们自己的模块儿中。在使用第三方库的情境下，这肯定不是你所希望的。

You can solve the problem by processing third party CSS differently through an `include` definition against *node_modules*. Alternately, you could use a file extension (`.mcss`) to tell files using CSS Modules apart from the rest and then manage this situation in a loader `test`.

您可以通过对*node_modules*文件夹的`include`配置项进行定义，有区别的处理第三方CSS来解决这个问题。或者，您可以使用文件扩展名（`.mcss`）将使用CSS模块的文件同其它文件区分开来，然后在加载器的`test`配置项中管理这种情况。

## Loading Less
## 加载Less

![Less](images/less.png)

[Less](http://lesscss.org/) is a CSS processor packed with functionality. Using Less doesn't take a lot of effort through webpack as [less-loader](https://www.npmjs.com/package/less-loader) deals with the heavy lifting. You should install [less](https://www.npmjs.com/package/less) as well given it's a peer dependency of *less-loader*.

[Less](http://lesscss.org/)是一个包含很多功能的CSS预处理器。借助[less-loader](https://www.npmjs.com/package/less-loader)来处理复杂的加载工作，在Webpack中简单配置一下就可以使用Less。您仅仅需要安装Less依赖包和对应的加载器`less-loader`。

{pagebreak}

Consider the following minimal setup:

下面的一小段代码展示了如何配置：

```javascript
{
  test: /\.less$/,
  use: ['style-loader', 'css-loader', 'less-loader'],
},
```

The loader supports Less plugins, source maps, and so on. To understand how those work you should check out the project itself.

这个加载器支持Less插件，source maps等。如果您想更好的理解其工作原理，请查看[Less](http://lesscss.org/)项目本身。

## Loading Sass

## 加载Sass

![Sass](images/sass.png)

[Sass](http://sass-lang.com/) is a widely used CSS preprocessor. You should use [sass-loader](https://www.npmjs.com/package/sass-loader) with it. Remember to install [node-sass](https://www.npmjs.com/package/node-sass) to your project as it's a peer dependency.

[Sass](http://sass-lang.com/) 是一个被广泛使用的CSS预处理器。您需要安装[sass-loader](https://www.npmjs.com/package/sass-loader)。请记住在项目中安装依赖包[node-sass](https://www.npmjs.com/package/node-sass)。

{pagebreak}

Webpack doesn't take much configuration:

Webpack仅需如下配置：

```javascript
{
  test: /\.scss$/,
  use: ['style-loader', 'css-loader', 'sass-loader'],
},
```

T> If you want more performance, especially during development, check out [fast-sass-loader](https://www.npmjs.com/package/fast-sass-loader).

T> 如果您希望有更好的性能，尤其是在开发环境中，请查看[fast-sass-loader](https://www.npmjs.com/package/fast-sass-loader)。

## Loading Stylus and Yeticss

![Stylus](images/stylus.png)

[Stylus](http://stylus-lang.com/) is yet another example of a CSS processor. It works well through [stylus-loader](https://www.npmjs.com/package/stylus-loader). [yeticss](https://www.npmjs.com/package/yeticss) is a pattern library that works well with it.

[Stylus](http://stylus-lang.com/)是另一个CSS预处理器。通过[stylus-loader](https://github.com/shama/stylus-loader)来工作。[yeticss](https://www.npmjs.com/package/yeticss)是一个与之对应的模式库。

{pagebreak}

Consider the following configuration:

Webpack如下配置：

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

To start using yeticss with Stylus, you must import it to one of your app's *.styl* files:

为了将yeticss和Stylus结合起来使用，您必须将它引入到一个*.styl*文件中：

```javascript
@import 'yeticss'

//or
@import 'yeticss/components/type'
```

## PostCSS

![PostCSS](images/postcss.png)

[PostCSS](http://postcss.org/) allows you to perform transformations over CSS through JavaScript plugins. You can even find plugins that provide you Sass-like features. PostCSS is the equivalent of Babel for styling. [postcss-loader](https://www.npmjs.com/package/postcss-loader) allows using it with webpack.

[PostCSS](http://postcss.org/)允许您通过JavaScript插件来对CSS进行转换。您甚至能找到提供Sass类似特性的插件。PostCSS对于样式来说就像Babel(之于JavaScript)。[postcss-loader](https://www.npmjs.com/package/postcss-loader)允许您将它和webpack一起使用。

The example below illustrates how to set up autoprefixing using PostCSS. It also sets up [precss](https://www.npmjs.com/package/precss), a PostCSS plugin that allows you to use Sass-like markup in your CSS. You can mix this technique with other loaders to allow autoprefixing there.

下面的例子向您展示了如何使用PostCSS来设置自动前缀的功能。它还设置了[precss](https://www.npmjs.com/package/precss)，一个PostCSS插件，让您能够在CSS中使用Sass一样的标记。您可以将这个技术和其它加载器结合起来实现自动前缀。

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

You have to remember to include [autoprefixer](https://www.npmjs.com/package/autoprefixer) and [precss](https://www.npmjs.com/package/precss) to your project for this to work. The technique is discussed in detail in the *Autoprefixing* chapter.

为了让其生效，您需要在项目中安装[autoprefixer](https://www.npmjs.com/package/autoprefixer) 和 [precss](https://www.npmjs.com/package/precss)。这个技术在*Autoprefixing*章节将被详细讨论。

T> PostCSS supports *postcss.config.js* based configuration. It relies on [cosmiconfig](https://www.npmjs.com/package/cosmiconfig) internally for other formats.

T> PostCSS还支持基于*postcss.config.js*的配置。它依赖于[cosmiconfig](https://www.npmjs.com/package/cosmiconfig)内部的其它格式。

### cssnext

[cssnext](http://cssnext.io/) is a PostCSS plugin that allows to experience the future now with certain restrictions. You can use it through [postcss-cssnext](https://www.npmjs.com/package/postcss-cssnext) and enable it as follows:

[cssnext](http://cssnext.io/)是一个PostCSS插件，允许我们现在就可以体验未来的特性。它虽然有一些限制，因为它不可能适应每个功能，但它仍然值得尝试。您可以通过[postcss-cssnext](https://www.npmjs.com/package/postcss-cssnext)来使用它，您可以按如下所示启用它：

```javascript
{
  loader: 'postcss-loader',
  options: {
    plugins: () => ([
      require('cssnext'),
    ]),
  },
},
```

See [the usage documentation](http://cssnext.io/usage/) for available options.

有关可用选项，请参见[使用文档](http://cssnext.io/usage/)。

T> cssnext includes *autoprefixer*! You don't have to configure autoprefixing separately for it to work in this case.

T> 注意cssnext包含自动前缀功能！ 在这种情况下，您不必单独配置自动前缀。

## Understanding Lookups
## 理解查找

To get most out of *css-loader*, you should understand how it performs its lookups. Even though *css-loader* handles relative imports by default, it doesn't touch absolute imports (`url("/static/img/demo.png")`). If you rely on these kind of imports, you have to copy the files to your project.

要充分利用*css-loader*，您应该了解它如何执行查找。 *css-loader*默认处理相对导入，它不会执行绝对导入（`url（“/static/img/demo.png”`）。 如果您需要这样的导入，您必须将文件复制到您的项目中。

[copy-webpack-plugin](https://www.npmjs.com/package/copy-webpack-plugin) works for this purpose, but you can also copy the files outside of webpack. The benefit of the former approach is that webpack-dev-server can pick that up.

[copy-webpack-plugin](https://www.npmjs.com/package/copy-webpack-plugin) 可实现这个目的，但您也可以将文件复制到webpack外部。前一种方法的好处是，webpack-dev-server可以选择它。

T> [resolve-url-loader](https://www.npmjs.com/package/resolve-url-loader) comes in handy if you use Sass or Less. It adds support for relative imports to the environments.

T> [resolve-url-loader](https://www.npmjs.com/package/resolve-url-loader)将会在您使用Sass、Less时派上用场。它增加了对各种环境中（不同环境的路径规则不一样）相对导入的支持。

### Processing *css-loader* Imports
### *css-loader*导入处理

If you want to process *css-loader* imports in a specific way, you should set up `importLoaders` option to a number that tells the loader how many loaders after the *css-loader* should be executed against the imports found. If you import other CSS files from your CSS through the `@import` statement and want to process the imports through specific loaders, this technique is essential.

如果您想以特定的方式处理*css-loader*的引入，您应该设置`importLoaders`选项为一个数字。告知在*css-loader*加载器之后，还有多少个加载器，需要对找到的内容执行操作。如果通过`@import`语句从CSS文件中导入其他CSS文件，并且希望通过特定加载器来处理导入，这是特别有用的。

{pagebreak}

Consider the following import from a CSS file:

考虑如下的情景，从CSS文件中引入另外一个Sass文件：

```css
@import "./variables.sass";
```

To process the Sass file, you would have to write configuration:

为了对Sass文件进行处理，您需要如下配置：

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

如果您在加载器链中添加了更多的加载器，例如*postcss-loader*，您必须相应地调整`importLoaders`选项。

### Loading from *node_modules* Directory
### 从*node_modules*文件夹加载

You can load files directly from your node_modules directory. Consider Bootstrap and its usage for example:

您可以从*node_modules*文件夹加载文件。对于*Bootstrap*，用法如下：

```less
@import "~bootstrap/less/bootstrap";
```

The tilde character (`~`) tells webpack that it's not a relative import as by default. If tilde is included, it performs a lookup against `node_modules` (default setting) although this is configurable through the [resolve.modules](https://webpack.js.org/configuration/resolve/#resolve-modules) field.

波浪符（`〜`）告诉webpack它不是默认的相对导入。 如果包括波浪符号（`〜`），它将对`node_modules`（默认设置）执行查找，这也可以通过[resolve.modules](https://webpack.js.org/configuration/resolve/#resolve-modules)字段配置 。

W> If you are using *postcss-loader*, you can skip using `~` as discussed in [postcss-loader issue tracker](https://github.com/postcss/postcss-loader/issues/166). *postcss-loader* can resolve the imports without a tilde.

W> 如果您使用*postcss-loader*，您可以不用使用`〜`，如[postcss-loader issue tracker](https://github.com/postcss/postcss-loader/issues/166)中所讨论的。 *postcss-loader*不使用波浪号，就可以解析导入。

## Enabling Source Maps
## 启用Source Maps

If you want to enable source maps for CSS, you should enable `sourceMap` option for *css-loader* and set `output.publicPath` to an absolute url pointing to your development server. If you have multiple loaders in a chain, you have to enable source maps separately for each. *css-loader* [issue 29](https://github.com/webpack/css-loader/issues/29) discusses this problem further.

如果要为CSS启用source maps，需要设置*css-loader*启用`sourceMap`选项，并将`output.publicPath`设置为指向开发服务器的绝对url。如果加载器链中有多个加载器，则必须为每个加载器分别启用source maps。*css-loader* [issue 29](https://github.com/webpack/css-loader/issues/29)进一步讨论了这个问题。

## Converting CSS to Strings
## 将CSS转换为字符串

Especially with Angular 2, it can be convenient if you can get CSS in a string format that can be pushed to components. [css-to-string-loader](https://www.npmjs.com/package/css-to-string-loader) achieves exactly this.

如果可以获得CSS的字符串格式，进而可以推送到组件，特别是对于Angular2，这是很有用的。[css-to-string-loader](https://www.npmjs.com/package/css-to-string-loader)实现了这一点。

## Using Bootstrap
## 使用Bootstrap

There are a couple of ways to use [Bootstrap](https://getbootstrap.com/) through webpack. One option is to point to the [npm version](https://www.npmjs.com/package/bootstrap) and perform loader configuration as above.

有多种方法可以通过webpack使用[Bootstrap](https://getbootstrap.com/)。 一个选项是指向[npm版本](https://www.npmjs.com/package/bootstrap)，并执行如上所述的加载程序配置。

The [Sass version](https://www.npmjs.com/package/bootstrap-sass) is another option. In this case, you should set `precision` option of *sass-loader* to at least 8. This is [a known issue](https://www.npmjs.com/package/bootstrap-sass#sass-number-precision) explained at *bootstrap-sass*.

[Sass版本](https://www.npmjs.com/package/bootstrap-sass)是另一个选项。 在这种情况下，您应该将*sass-loader*的'precision'选项至少设置为8.这是[已知的问题](https://www.npmjs.com/package/bootstrap-sass#sass-number-precision),在*bootstrap-sass*有详细解释。

The third option is to go through [bootstrap-loader](https://www.npmjs.com/package/bootstrap-loader). It does a lot more but allows customization.

第三个选项是通过[bootstrap-loader](https://www.npmjs.com/package/bootstrap-loader)。它可以实现更多并且支持定制。

## Conclusion
## 小结

Webpack can load a variety of style formats. It even supports advanced specifications like [CSS Modules](https://github.com/css-modules/webpack-demo). The approaches covered here inline the styling by default.

通过webpack可以加载各种格式的样式文件。它甚至支持更高级的规范，如[CSS模块](https://github.com/css-modules/webpack-demo)。这里介绍方法默认都将样式内联到JavaScript中了。

To recap:

回顾：

* *css-loader* evaluates the `@import` and `url()` definitions of your styling. *style-loader* converts it to JavaScript and implements webpack's Hot Module Replacement interface.
* *css-loader*执行并处理了样式文件中定义的`@import` 和 `url()`。*style-loader*将其转换为JavaScript，并实现了webpack的热加载接口。

* *css-loader* supports the CSS Modules specification. CSS Modules allow you maintain CSS in a local scope by default solving the biggest issue of CSS.
* *css-loader*支持CSS模块化定义。CSS模块允许您将CSS维护在局部中，进而解决了CSS中最大的问题（－全局污染）。

* Webpack supports a large variety of formats compiling to CSS through loaders. These include Sass, Less, and Stylus.
* Webpack支持通过加载器将各种格式的样式文件编译为CSS。包括Sass, Less, and Stylus。

* PostCSS allows you to inject functionality to CSS in through its plugin system. cssnext is an example of a collection of plugins for PostCSS that implements future features of CSS.
* PostCSS允许您通过其插件系统向CSS中注入功能。cssnext是这个插件集合中的一个例子，实现了可以让我们使用用CSS一些未来特性的功能。

* *css-loader* doesn't touch absolute imports by default. It allows customization of loading behavior through the `importLoaders` option. You can perform lookups against *node_modules* by prefixing your imports with a tilde (`~`) character.
* *css-loader*默认情况下，将不会处理绝对路径的引入。可以通过`importLoaders`配置项来实现加载行为的自定义。您可以通过在路径中增加波浪符号（`~`）来处理对*node_modules*目录的查找和引入问题。

* To use source maps, you have to enable `sourceMap` boolean through each style loader you are using except for *style-loader*. You should also set `output.publicPath` to an absolute url that points to your development server.
* 为了能够使用source maps，您需要对除了*style-loader*之外的，每一个样式加载器启用`sourceMap`。您还应该将`output.publicPath`设置为指向开发服务器的绝对路径。

* Using Bootstrap with webpack requires special care. You can either go through generic loaders or a bootstrap specific loader for more customization options.
* 用webpack使用Bootstrap时需要特别小心。您可以通过通用加载器，或特定于bootstrap的加载器来获得更多自定义选项。

Although the loading approach covered here is enough for development purposes, it's not ideal for production as it inlines the styling to the JavaScript bundles. You'll learn to solve this problem in the next chapter by separating CSS from the source.

虽然这里描述的加载方法用于开发目的已经足够了，但它对于生产来说并不理想，因为它在JavaScript中内联了样式代码。我们将在下一章学习分离CSS时，讨论这个问题。
