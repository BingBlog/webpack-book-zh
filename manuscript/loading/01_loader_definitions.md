# Loader Definitions
# 加载器定义

Webpack provides multiple ways to set up module loaders. Webpack 2 simplified the situation by introducing the `use` field. The legacy options (`loader` and `loaders`) still work, though. You see all the options for completeness, as they exist in older configurations online.

Webpack提供了多种方式来设置模块加载器。Webpack2通过引入`use`字段简化了形式。遗留下来的选项（`loader` 和 `loaders`）仍然有效。由于它们仍存在于先前的配置中，为了确保完整性你将看到所有的选项（都被讲解）。

It can be a good idea to prefer absolute paths here as they allow you to move configuration without breaking assumptions. The other option is to set `context` field as this gives a similar effect and affects the way entry points and loaders are resolved. It doesn't have an impact on the output, though, and you still need to use an absolute path or `/` there.

使用绝对路径是一个好主意，可以让你在不破坏假设的情况下移动配置文件。另一个选项就是设置`context`字段，也能实现类似的效果，但只能影响入口文件和加载器被处理的方式。对输出不会有任何影响，所以，仍然需要在设置输出时使用绝对路径或者`/`符号。

Assuming you set an `include` or `exclude` rule, packages loaded from *node_modules* still work as the assumption is that they have been compiled in such way that they work out of the box. If they don't, then you have to apply techniques covered in the *Package Consuming Techniques* chapter.

假设你设置了一个`include`或者`exclude`规则，从*node_modules*加载的代码包仍然会按照一个假设来处理，这个假设是它们已经被编译了，可以直接使用。如果它们不可以直接使用，你就需要应用到涵盖在*Package Consuming Techniques*章节的技术了。

Translate> 作者这个地方表达的意思是，webpack默认是会遍历和处理*node_modules*中的文件的，但是如果你设置了`include`或者`exclude`规则中，仅包含项目代码或者排除了*node_modules*目录。例如，在项目中，我们通常用这样的方式来屏蔽*node_modules*目录。这个时候，webpack就会按照从*node_modules*目录中加载的文件都可以立即执行的假设来处理。如果你遇到了一些没有打包好的代码，就需要你自己通过加载器设置或者设置`resolve.alias`字段来解决这个问题。关于这个问题，可以[查看](https://github.com/webpack/webpack/issues/2031)

T> `include`/`exclude` is handy with *node_modules* as webpack processes and traverses the installed packages by default when you import JavaScript files to your project. Therefore you need to configure it to avoid that behavior. Other file types don't suffer from this issue.

T> `include`/`exclude`对于*node_modules*目录来说特别有用，因为当你在项目中引入了JavaScript文件后，webpack会默认处理和遍历这些已经安装的代码包。因此，你需要通过配置来避免这个行为。其他的文件类型不会有这样的问题。

## Anatomy of a Loader
## 解剖加载器

Webpack supports a large variety of formats through *loaders*. Also, it supports a couple of JavaScript module formats out of the box. The idea is the same. You always set up a loader, or loaders, and connect those with your directory structure.

Webpack通过*加载器*可以支持各种各样的格式。它还支持好几种格式的JavaScript模块化，这些模块格式化方法可以被直接使用。基本思想都是相同的，总是需要设置加载器或者多个加载器，并将它们同你的目录结构联系起来。

{pagebreak}

Consider the example below where webpack is set to process JavaScript through Babel:

考虑如下的例子， 设置webpack使其可以通过Babel来处理JavaScript代码：

**webpack.config.js**

```javascript
module.exports = {
  ...
  module: {
    rules: [
      {
        // **Conditions**
        // Match files against RegExp or a function.
        test: /\.js$/,

        // **Restrictions**
        // Restrict matching to a directory. This
        // also accepts an array of paths or a function.
        // The same applies to `exclude`.
        include: path.join(__dirname, 'app'),
        exclude(path) {
          // You can perform more complicated checks
          // through functions if you want.
          return path.match(/node_modules/);
        },

        // **Actions**
        // Apply loaders the matched files.
        use: 'babel-loader',
      },
    ],
  },
};
```

T> If you are not sure how a particular RegExp matches, consider using an online tool, such as [regex101](https://regex101.com/), [RegExr](http://regexr.com/), or [Regexper](https://regexper.com).

T> 如果你不确定一个特定的RegExp是如何匹配的，考虑使用一个在线工具，如[regex101](https://regex101.com/) 或者 [RegExr](http://regexr.com/)。

## Loader Evaluation Order
## 加载器的执行顺序

It's good to keep in mind that webpack's loaders are always evaluated from right to left and from bottom to top (separate definitions). The right-to-left rule is easier to remember when you think about as functions. You can read definition `use: ['style-loader', 'css-loader']` as `style(css(input))` based on this rule.

需要牢记，webpack的加载器总是按照从右到左、从下到上（单独定义）的顺序被执行。对于从右到左的规则，当你将其视为函数时，更易于记忆。根据这条规则，你可以将定义`use: ['style-loader', 'css-loader']`读作`style(css(input))`。

To see the rule in action, consider the example below:

要实际观察这个规则，考虑下面的例子：

```javascript
{
  test: /\.css$/,
  use: ['style-loader', 'css-loader'],
},
```

Based on the right to left rule, the example can be split up while keeping it equivalent:

根据从右到左的规则，该例子可以被拆分成如下等同的形式：

```javascript
{
  test: /\.css$/,
  use: ['style-loader'],
},
{
  test: /\.css$/,
  use: ['css-loader'],
},
```

### Enforcing Order
### 强制顺序

Even though it would be possible to develop an arbitrary configuration using the rule above, it can be convenient to be able to force certain rules to be applied before or after regular ones. The `enforce` field can come in handy here. It can be set to either `pre` or `post` to push processing either before or after other loaders.

使用上述规则就可以开发出任何（满足需求的）配置，但如果能够在一般的规则之前或之后强制应用一些特定的规则，就更加方便了。`enforce`字段可以派上用场。可以将它设置为`pre` or `post`，以在其它加载器执行之前或之后，来执行特定的处理过程。

You used the idea earlier in the *Linting JavaScript* chapter. Linting is a good example as the build should fail before it does anything else. Using `enforce: 'post'` is rarer and it would imply you want to perform a check against the built source. Performing analysis against the built source is one potential example.

在前面的*Linting JavaScript*章节已经使用过这个主意。语法检测（Linting）是一个很好的例子。因为在构建时，需要在其执行其它任务之前就报出语法错误。`enforce: 'post'`的应用很少，它意味着你要对构建后的代码进行检查。对构建后的代码进行分析可能是一个例子。

The basic syntax goes as below:

基本的语法如下：

```javascript
{
  // Conditions
  test: /\.js$/,
  enforce: 'pre', // 'post' too

  // Actions
  loader: 'eslint-loader',
},
```

It would be possible to write the same configuration without `enforce` if you chained the declaration with other loaders related to the `test` carefully. Using `enforce` removes the necessity for that allows you to split loader execution into separate stages that are easier to compose.

如果你将这个加载器声明同其它和`test`字段相关的加载器配置小心地串到一起，也可以编写出相同的配置而不是用`enforce`。使用`enforce`免除了这个必要，让你可将加载器的执行拆分为不同的阶段，进而可以更容易的进行组合。

{pagebreak}

## Passing Parameters to a Loader
## 向加载器传递参数

There's a query format that allows passing parameters to loaders:

如下是一个查询格式，可以向加载器传递参数

```javascript
{
  // Conditions
  test: /\.js$/,
  include: PATHS.app,

  // Actions
  use: 'babel-loader?cacheDirectory,presets[]=es2015',
},
```

This style of configuration works in entries and source imports too as webpack picks it up. The format comes in handy in certain individual cases, but often you are better off using more readable alternatives.

因为webpack也可以识别，这种风格的配置在入口和资源引入时同样有效。这种格式在个别特定的情况下很有用，但通常情况下，你最好还是使用更易读的方式。

It's preferable to use the combination of `loader` and `options` fields:

最好的方式是将`loader` 和 `options`结合起来使用：

```javascript
{
  // Conditions
  test: /\.js$/,
  include: PATHS.app,

  // Actions
  loader: 'babel-loader',
  options: {
    cacheDirectory: true,
    presets: ['react', 'es2015'],
  },
},
```

{pagebreak}

Or you can also go through `use`:

也可以使用`use`：

```javascript
{
  // Conditions
  test: /\.js$/,
  include: PATHS.app,

  // Actions
  use: {
    loader: 'babel-loader',
    options: {
      cacheDirectory: true,
      presets: ['react', 'es2015'],
    },
  },
},
```

If you wanted to use more than one loader, you could pass an array to `use` and expand from there:

如果你想使用多个加载器，你可以向`use`传递一个数组并进行扩展：

```javascript
{
  test: /\.js$/,
  include: PATHS.app,

  use: [
    {
      loader: 'babel-loader',
      options: {
        cacheDirectory: true,
        presets: ['react', 'es2015'],
      },
    },
    // Add more loaders here
  ],
},
```

## Branching at `use` Using a Function
## 在`use`字段上使用函数创建分支

In the book setup, you compose configuration on a higher level. Another option to achieve similar results would be to branch at `use` as webpack's loader definitions accept functions that allow you to branch depending on the environment. Consider the example below:

在本书的构建中，你以很高的水平组合了配置。另外一个可以实现相同结果的选项是在`use`字段中创建分支，因为webpack的加载器定义可以接受函数，进而允许你根据环境创建分支。

```javascript
{
  test: /\.css$/,

  // resource refers to the resource path matched.
  // resourceQuery contains possible query passed to it (?sourceMap)
  // issuer tells about match context path
  use: ({ resource, resourceQuery, issuer }) => {
    // You have to return either something falsy,
    // string (i.e., 'style-loader'), or an object from here.
    //
    // Returning an array fails! To get around that,
    // it's possible to nest rules.
    if (env === 'development') {
      return {
        // Trigger css-loader first
        loader: 'css-loader',
        rules: [
          // And style-loader after it
          'style-loader',
        ],
      };
    }

    ...
  },
},
```

Carefully applied, this technique allows different means of composition.

小心应用，这个技术允许使用不同方式的组合。

## Inline Definitions
## 行内定义

Even though configuration level loader definitions are preferable, it's possible to write loader definitions inline:

尽管可配置水平的加载器定义更为可取，但是也可以直接将加载器定义以行内形式书写：

```javascript
// Process foo.png through url-loader and other
// possible matches.
import 'url-loader!./foo.png';

// Override possible higher level match completely
import '!!url-loader!./bar.png';
```

The problem with this approach is that it couples your source with webpack. But it's a good form to know still. Since configuration entries go through the same mechanism, the same forms work there as well:

这种方式的问题就是它将你的资源和webpack进行了挂钩。但仍然值得了解。由于入口配置以同样的机制运行，所以相同格式的配置也可以运用到入口配置:

```javascript
{
  entry: {
    app: 'babel-loader!./app',
  },
},
```

## Alternate Ways to Match Files
## 匹配文件的可选方式

`test` combined with `include` or `exclude` to constrain the match is the most common approach to match files. These accept the data types as listed below:

`test`同`include` 或 `exclude`结合起来对匹配进行限制是最常用的匹配文件的方式。他们接受的数据类型如下：

* `test` - Match against a RegExp, string, function, an object, or an array of conditions like these.
* `test` - 可匹配的格式包括：正则、字符串、函数、对象、或者一个包含它们的条件数组

* `include` - The same.
* `include` - 同上

* `exclude` - The same, except the output is the inverse of `include`.
* `exclude` - 同上，除了返回值是`include`的补集

* `resource: /inline/` - Match against a resource path including the query. Examples: `/path/foo.inline.js`, `/path/bar.png?inline`.
* `resource: /inline/` - 对包含参数的资源路径进行配置。例如：`/path/foo.inline.js`, `/path/bar.png?inline`。

* `issuer: /bar.js/` - Match against a resource requested from the match. Example: `/path/foo.png` would match if it was requested from `/path/bar.js`.
* `issuer: /bar.js/` - 匹配（匹配的文件）请求的资源进行。例如：`/path/foo.png`将会被匹配，如果它从`/path/bar.js`被请求

* `resourcePath: /inline/` - Match against a resource path without its query. Example: `/path/foo.inline.png`.
* `resourcePath: /inline/` - 匹配没有参数的资源路径。例如:`/path/foo.inline.png`。

* `resourceQuery: /inline/` - Match against a resource based on its query. Example: `/path/foo.png?inline`.
* `resourceQuery: /inline/` - 匹配基于参数的资源。例如：`/path/foo.png?inline`。

Boolean based fields can be used to constrain these matchers further:

基于布尔的字段可被用来进一步限制这些匹配：

* `not` - Do **not** match against a condition (see `test` for accepted values).
* `not` - **不**匹配一个条件（查看`test`了解可接受的值）

* `and` - Match against an array of conditions. All must match.
* `and` - 匹配一个条件数据。所有的都必须匹配

* `or` - Match against an array while any must match.
* `or` - 匹配一个数组，必须有可匹配的

## Loading Based on `resourceQuery`
## 基于`resourceQuery`的加载

`oneOf` field makes it possible to route webpack to a specific loader based on a resource related match:

`oneOf`字段可以基于资源相关的匹配，引导webpack执行特定的加载器

```javascript
{
  test: /\.css$/,

  oneOf: [
    {
      resourceQuery: /inline/,
      use: 'url-loader',
    },
    {
      resourceQuery: /external/,
      use: 'file-loader',
    },
  ],
},
```

If you wanted to embed the context information to the filename, the rule could use `resourcePath` over `resourceQuery`.

如果你想将上下文信息嵌入到文件名中，可以使用`resourcePath` 代替 `resourceQuery`.

## Loading Based on `issuer`
## 基于 `issuer`加载

`issuer` can be used to control behavior based on where a resource was imported. In the example below adapted from [css-loader issue 287](https://github.com/webpack-contrib/css-loader/pull/287#issuecomment-261269199), *style-loader* is applied only when webpack captures a CSS file from a JavaScript import:

`issuer`可以基于资源在哪里被加载的来控制行为。如下的例子被应用到了[css-loader issue 287](https://github.com/webpack-contrib/css-loader/pull/287#issuecomment-261269199)，当webpack捕获了一个从JavaScript引入的CSS文件后，*style-loader*才会被应用。

```javascript
{
  test: /\.css$/,

  rules: [
    {
      issuer: /\.js$/,
      use: 'style-loader',
    },
    {
      use: 'css-loader',
    },
  ],
},
```

## Understanding Loader Behavior
## 理解加载器的行为

Loader behavior can be understood in greater detail by inspecting them. [loader-runner](https://www.npmjs.com/package/loader-runner) allows you to run them in isolation without webpack. Webpack uses this package internally and *Extending with Loaders* chapter covers it in detail.

加载器的行为可以通过观察来更细致的理解。[loader-runner](https://www.npmjs.com/package/loader-runner)让你不用webpack就可以独立运行它们。webpack在内部使用这个代码包，*Extending with Loaders*章节更详细的涵盖了这个话题。

[inspect-loader](https://www.npmjs.com/package/inspect-loader) allows you to inspect what's being passed between loaders. Instead of having to insert `console.log`s within *node_modules*, you can attach this loader to your configuration and inspect the flow there.

[inspect-loader](https://www.npmjs.com/package/inspect-loader)可以让你检查在加载器之间传递的数据。你不需要在*node_modules*中插入`console.log`，可以将这个加载器附加到您的配置中，然后观察其中的流程。

{pagebreak}

## `LoaderOptionsPlugin`

Given webpack 2 forbids arbitrary root level configuration, you have to use `LoaderOptionsPlugin`. The plugin exists for legacy compatibility and disappears in a future release. Consider the example below:

由于webpack2禁止随意使用基于根目录的配置，你可能需要使用`LoaderOptionsPlugin`。这个插件是为了遗留的兼容性问题而存在的，在未来的发版中将会被去掉。考虑如下的例子:

```javascript
plugins: [
  new webpack.LoaderOptionsPlugin({
    sassLoader: {
      includePaths: [
        path.join(__dirname, 'style'),
      ],
    },
  }),
],
```

## Conclusion
## 小结

Webpack provides multiple ways to set up loaders but sticking with `use` is enough in webpack 2. Be careful with loader ordering, as it's a common source of problems.

Webpack提供了多种方式来设置加载器，但是在webpack2中仅使用`use`就足够了。需要注意是加载器的顺序，因为它是最常见问题的根源。

To recap:

回顾：

* **Loaders** allow you determine what should happen when webpack's module resolution mechanism encounters a file.
* **Loaders**允许你来决定当webpack的模块处理机制遇到一个文件后将会发生什么。

* A loader definition consists of **conditions** based on which to match and **actions** that should be performed when a match happens.
* 一个加载器定义包含什么将会被匹配的**条件**和当匹配发生后将会被执行的**行动**。

* Webpack2 introduced the `use` field. It combines the ideas of old `loader` and `loaders` fields into a single construct.
* Webpack2引入了`use`字段。它将先前的`loader` 和 `loaders`字段结合起来，成为了一个单独的结构。

* Webpack 2 provides multiple ways to match and alter loader behavior. You can, for example, match based on a **resource query** after a loader has been matched and route the loader to specific actions.
* Webpack2提供了多种方式来进行匹配和改变加载的行为。例如，你可以在一个加载器被匹配后，基于**resource query**的匹配，引导加载器执行不同的行动。

* `LoaderOptionsPlugin` exists for legacy purposes and allows you to get around the strict configuration schema of webpack 2.
* `LoaderOptionsPlugin`因遗留问题而存在，让你可以处理webpack2中严格的配置模式而导致的问题

In the next chapter, you'll learn to load images using webpack.

下一章节，你将会学习使用webpack来加载图片
