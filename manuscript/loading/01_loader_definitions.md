# Loader Definitions
# 加载器定义

Webpack provides multiple ways to set up module loaders. Webpack 2 simplified the situation by introducing the `use` field. The legacy options (`loader` and `loaders`) still work, though. I’ll discuss all the options for completeness, as you may see them in existing configurations.

Webpack提供了多种方式来设置模块化加载器。Webpack2通过引入`use`字段来简化了形式。遗留下来的选项（`loader` 和 `loaders`）仍然有效。由于你将在很多已经存在的配置中看到它们，为了确保完整性我将讨论所有的选项。

It can be a good idea to prefer absolute paths here as they allow you to move configuration without breaking assumptions. The other option is to set `context` field as this gives a similar effect and affects the way entry points and loaders are resolved. It won’t have an impact on the output, though, and you still need to use an absolute path or `/` there.

使用绝对路径是一个好主意，可以让你在不破坏假设的情况下移动配置文件。另外一个选项就是设置`context`字段，可以实现类似的效果，并影响入口文件和加载器被处理的方式。但是对输出不会有任何影响，所以，仍然需要在那里使用绝对路径或者`/`。

Assuming you set an `include` or `exclude` rule, packages loaded from *node_modules* will still work as the assumption is that they have been compiled in such way that they work out of the box. Sometimes you may come upon a poorly packaged one, but often you can work around these by tweaking your loader configuration or setting up a `resolve.alias` against an asset that is included in the offending package.

假设你设置了一个`include`或者`exclude`规则，从*node_modules*加载的代码包仍然会按照一个假设来处理，这个假设是它们已经被编译了，可以直接使用。有时你可能会遇到一个没有很好打包的代码，但是你可以通过调整加载器配置或者对包含在这个违反规则的代码包中的资源设置`resolve.alias`来解决这个问题。

Translate> 作者这个地方表达的意思是，webpack默认是会遍历和处理*node_modules*这个文件夹中的文件的，但是如果你设置了`include`或者`exclude`规则来仅包含项目代码或者排除了*node_modules*目录，在项目中，我们通常用这样的方式来屏蔽*node_modules*目录。这个时候，webpack就会按照从*node_modules*目录中加载的文件都可以立即执行的假设来处理。如果你遇到了一些没有很好的被打包的代码，就需要你自己通过更改配置文件的加载器设置或者设置`resolve.alias`字段来解决这个问题。关于这个问题，可以[查看](https://github.com/webpack/webpack/issues/2031)

T> The *Consuming Packages* chapter discusses the aliasing idea in further detail.

T> *Consuming Packages*章节更加详细的讨论了别名的思想

T> `include`/`exclude` is particularly useful with *node_modules* as webpack will process and traverse the installed packages by default when you import JavaScript files to your project. Therefore you need to configure it to avoid that behavior. Other file types don’t suffer from this issue.

T> `include`/`exclude`对于*node_modules*来说特别有用，因为当你在项目中引入了JavaScript文件后，webpack会默认处理和遍历这些已经安装的代码包。因此，你需要通过配置来避免这个行为。其他的文件类型不会有这样的问题。


## Anatomy of a Loader
## 解剖加载器

Webpack supports a large variety of formats through *loaders*. Also, it supports a couple of JavaScript module formats out of the box. The idea is the same. You always set up a loader, or loaders, and connect those with your directory structure.

Webpack支持各种类型的*加载器*。他还支持直接使用多种格式的JavaScript的模块。基本思想是相同的，设置加载器或者多个加载器，并将它们同你的目录结构联系起来。

Consider the example below where webpack is set to process JavaScript through Babel:

考虑如下的例子， 设置webpack通过Babel来处理JavaScript：

**webpack.config.js**

```javascript
...

module.exports = {
  ...
  module: {
    rules: [
      {
        // Conditions

        // Match files against RegExp. This accepts
        // a function too.
        test: /\.js$/,

        // Restrict matching to a directory. This
        // also accepts an array of paths.
        //
        // Although optional, I prefer to set this for
        // JavaScript source as it helps with
        // performance and keeps the configuration cleaner.
        //
        // This accepts an array or a function too. The
        // same applies to `exclude` as well.
        include: path.join(__dirname, 'app'),

        /*
        exclude(path) {
          // You can perform more complicated checks
          // through functions if you want.
          return path.match(/node_modules/);
        },
        */

        // Actions

        // Apply loaders the matched files. These need to
        // be installed separately. In this case our
        // project would need *babel-loader*. This
        // accepts an array of loaders as well and
        // more forms are possible as discussed below.
        use: 'babel-loader',
      },
    ],
  },
};
```

T> If you are not sure how a particular RegExp matches, consider using an online tool, such as [regex101](https://regex101.com/) or [RegExr](http://regexr.com/).

T> 如果你不确定一个特定的RegExp是如何匹配的，考虑使用一个在线工具，如[regex101](https://regex101.com/) 或者 [RegExr](http://regexr.com/)。

T> Babel is discussed in detail in the *Loading JavaScript* chapter. We’ll attach it to the book project there.

T> *Loading JavaScript*章节将会详细的讨论Babel。我们将会在那时将Babel添加到项目中。

## Loader Evaluation Order
## 加载器的执行顺序

It is good to keep in mind that webpack’s `loaders` are always evaluated from right to left and from bottom to top (separate definitions). The right-to-left rule is easier to remember when you think about as functions. You can read definition `use: ['style-loader', 'css-loader']` as `style(css(input))` based on this rule.

webpack的加载器总是按照从右到左、从下到上（单独定义）的顺序执行。对于从右到左的规则，当你将其视为函数时，更容易记住。根据这条规则，你可以将定义`use: ['style-loader', 'css-loader']`读作`style(css(input))`。

To see the rule in action, consider the example below:

要查看这条执行规则，考虑下面的示例：

```javascript
{
  test: /\.css$/,
  use: ['style-loader', 'css-loader'],
}
```

Based on the right to left rule, the example can be split up while keeping it equivalent:

根据从右到左的规则，实例可以被拆分等同的规则：

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



We used the idea earlier in the *Linting JavaScript* chapter. Linting is a good example as the build should fail before it does anything else. Using `enforce: 'post'` is rarer and it would imply you want to perform a check against the built source. Performing analysis against the built source is one potential example.

The basic syntax goes like this:

```javascript
{
  // Conditions
  test: /\.js$/,
  enforce: 'pre',

  // Actions
  loader: 'eslint-loader',
},
```

It would be possible to write the same configuration without `enforce` if you chained the declaration with other loaders related to the `test` carefully. Using `enforce` removes the necessity for that allows you to split loader execution into separate stages that are easier to compose.

## Passing Parameters to a Loader

There’s a query format that allows passing parameters to loaders:

```javascript
{
  // Conditions
  test: /\.js$/,
  include: PATHS.app,

  // Actions
  use: 'babel-loader?cacheDirectory,presets[]=es2015',
},
```

It is good to note that this style of configuration works in entries and source imports too as webpack will pick it up. The format may come in handy in certain individual cases, but often you are better off using more readable alternatives.

It is preferable to use the combination of `loader` and `options` fields either like this:

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

Or you can also go through `use` like this:

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

If we wanted to use more than one loader, we could pass an array to `use` and expand from there:

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

In the book setup, we compose configuration on a higher level. Another option to achieve similar results would be to branch at `use` as webpack’s loader definitions accept functions that allow you to branch depending on the environment. Consider the example below:

```javascript
{
  test: /\.css$/,

  // resource refers to the resource path matched.
  // resourceQuery contains possible query passed to it (?sourceMap)
  // issuer tells about match context path
  use: ({ resource, resourceQuery, issuer }) => {
    // You have to return either something falsy,
    // string (i.e., 'style-loader'), or an object
    // from here.
    //
    // Returning an array will fail! To get around that,
    // it is possible to nest rules.
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

    // Do something else in production environment now
    ...
  },
},
```

Carefully applied, this technique can be useful and allow different ways of composition.

## Inline Definitions

Even though configuration level loader definitions are preferable, it is possible to write loader definitions inline like this:

```javascript
// Process foo.png through url-loader and other
// possible matches.
import 'url-loader!./foo.png';

// Override possible higher level match completely
import '!!url-loader!./bar.png';
```

The problem with this approach is that it couples your source with webpack. But it is a good form to know still. Since configuration entries go through the same mechanism, the same forms work there as well:

```javascript
{
  entry: {
    app: 'babel-loader!./app',
  },
},
```

## Alternate Ways to Match Files

`test` combined with `include` or `exclude` to constrain the match is the most common way to match files. These accept the data types as listed below:

* `test` - Match against a RegExp, string, function, an object, or an array of conditions like these.
* `include` - The same.
* `exclude` - The same, except the output is the inverse of `include`.

There are a couple of boolean based fields that can be used to constrain the result further:

* `not` - Do **not** match against a condition like above.
* `and` - Match against an array of conditions. All must match.
* `or` - Match against an array while any must match.

Webpack can also match based on the resource path and related information:

* `resource: /inline/` - Match against a resource path including the query. Example matches: `/path/foo.inline.js`, `/path/bar.png?inline`.
* `issuer: /bar.js/` - Match against a resource requested from the match. Example: `/path/foo.png` would match if it was requested from `/path/bar.js`.
* `resourcePath: /inline/` - Match against a resource path without its query. Example match: `/path/foo.inline.png`.
* `resourceQuery: /inline/` - Match against a resource based on its query. Example match: `/path/foo.png?inline`.

The fields above can be combined to apply different loaders based on the context with the `oneOf` field. To handle images differently based on their query, we could apply a rule like this:

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
}
```

If you wanted to embed the context information to the filename, the rule could use `resourcePath` over `resourceQuery`.

## `LoaderOptionsPlugin`

Given webpack 2 forbids arbitrary root level configuration, you have to use `LoaderOptionsPlugin` to manage it. The plugin exists for legacy compatibility and may disappear in a future release. Consider the example below:

```javascript
{
  plugins: [
    new webpack.LoaderOptionsPlugin({
      sassLoader: {
        includePaths: [
          path.join(__dirname, 'style'),
        ],
      },
    }),
  ],
},
```

## Conclusion

Webpack provides multiple ways to set up loaders, but sticking with `use` is enough in webpack 2. You should be careful, especially with loader ordering, as this is a common source of problems.

To recap:

* **Loaders** allow you determine what should happen when webpack’s module resolution mechanism encounters a file.
* A loader definition consists of **conditions** based on which to match and **actions** that should be performed when a match happens.
* Webpack 2 introduced the `use` field. It combines the ideas of old `loader` and `loaders` fields into a single construct.
* Webpack 2 provides multiple ways to match and alter loader behavior. You can, for example, match based on a **resource query** after a loader has been matched and route the loader to specific actions.
* `LoaderOptionsPlugin` exists for legacy purposes and allows you to get around the strict configuration schema of webpack 2 to work with older plugins and loaders.

In the next chapter, I will show you how to load images using webpack.
