# Composing Configuration
# 对配置进行组合

Even though not a lot has been done with webpack yet, the amount of configuration is starting to feel substantial. Now you have to be careful about the way you compose it as you have separate production and development targets in the project. The situation can only get worse as you want to add more functionality to the project.

尽管还没有使用webpack做很多事情，就已经开始感觉到庞大的配置数量。现在你必须小心处理组合它们的方式，因为在本项目中有生产环境和开发环境不同的目标。当你希望在项目中加入更多功能时，处境加将会越来越糟糕。

Using a single monolithic configuration file impacts comprehension and removes any potential for reusablity. As the needs of your project grow, you have to figure out the means to manage webpack configuration more effectively.

使用一个单独的单片配置文件，不仅影响对配置内容的理解，还丢失了重复利用的可能性。随着项目需求的增长，你需要寻找能够更加高效管理webpack配置的方法。

## Possible Ways to Manage Configuration
## 可行的配置管理方式

You can manage webpack configuration in the following ways:

你可以用下面的几种方式来管理webpack配置:

* Maintain configuration in multiple files for each environment and point webpack to each through the `--config` parameter, sharing configuration through module imports. You can see this approach in action at [webpack/react-starter](https://github.com/webpack/react-starter).
* 对于不同的环境，将配置维护到不同的文件中，并通过`--config`参数将webpack指向它们，并通过模块来导入共享的配置。你可以在[webpack/react-starter](https://github.com/webpack/react-starter)中见证这种方式。

* Push configuration to a library, which you then consume. Example: [HenrikJoreteg/hjs-webpack](https://www.npmjs.com/package/hjs-webpack).
* 将之后要用的配置放到一个库中。例如：[HenrikJoreteg/hjs-webpack](https://www.npmjs.com/package/hjs-webpack)。

* Maintain all configuration within a single file and branch there and by relying on the `--env` parameter.
* 将所有配置维护在一个文件中，并在其中基于`--env`参数创建分支。

These approaches can be combined to create a higher level configuration that is then composed of smaller parts. Those parts could then be added to a library which you then use through npm making it possible to consume the same configuration across multiple projects.

这些方式被结合起来可以创建一个较高水平的配置，它们是较小零件的组合。这些零件可以被添加到一个库中，你可以利用npm来实现在多个项目中使用相同的配置。

{pagebreak}

## Composing Configuration by Merging
## 通过合并来组合配置

If the configuration file is broken into separate pieces, they have to be combined together again somehow. Normally this means merging objects and arrays. To eliminate the problem of dealing with `Object.assign` and `Array.concat`, [webpack-merge](https://www.npmjs.org/package/webpack-merge) was developed.

如果配置文件被拆分成单独的碎片，它们还需要再结合起来。通常情况下，这表明需要合并对象和数组。为了避免使用`Object.assign`和`Array.concat`带来的问题，[webpack-merge](https://www.npmjs.org/package/webpack-merge)被开发了。

*webpack-merge* does two things: it concatenates arrays and merges objects instead of overriding them. Even though a basic idea, this allows you to compose configuration and gives a degree of abstraction.

*webpack-merge*做了两件事：它连接数组、合并对象而不是重写它们。尽管只是一个简单的注意，但是它允许你组合配置，进行一定程度的抽象化。

The example below shows the behavior in detail:

下面的例子详细展示了其行为：

```bash
> merge = require('webpack-merge')
...
> merge(
... { a: [1], b: 5, c: 20 },
... { a: [2], b: 10, d: 421 }
... )
{ a: [ 1, 2 ], b: 10, c: 20, d: 421 }
```

*webpack-merge* provides even more control through strategies that enable you to control its behavior per field. Strategies allow you to force it to append, prepend, or replace content.

*webpack-merge*通过提供策略提供了更多控制，该策略可以让你控制每个部分的行为。策略允许你强制它进行附加，预处理或替换内容等操作。

Even though *webpack-merge* was designed for this book, it has proven to be an invaluable tool beyond it. You can consider it as a learning tool and pick it up in your work if you find it handy.

尽管*webpack-merge*是为本书而设计的，它也被证明是一个超越其自身的非常有用的工具。你可以将其视为一个学习工具，如果觉得它有用，也可以在你的工作中使用它。

T> [webpack-chain](https://www.npmjs.com/package/webpack-chain) provides a fluent API for configuring webpack allowing you to avoid configuration shape-related problems while enabling composition.

T> [webpack-chain](https://www.npmjs.com/package/webpack-chain)为配置webpack提供了清晰的API，使你可以在组合时避免配置中出现形状相关的问题。（猜测是缩进相关的语法问题）

{pagebreak}

## Setting Up *webpack-merge*
## 设置*webpack-merge*

To get started, add *webpack-merge* to the project:

首先，将*webpack-merge*添加到项目中：

```bash
npm install webpack-merge --save-dev
```

To give a degree of abstraction, you can define *webpack.config.js* for higher level configuration and *webpack.parts.js* for configuration parts to consume. Here are the parts with small function-based interfaces extracted from the existing code:

为了进行一定程度的抽象化，你可以为更高水平的配置生成一个*webpack.config.js*文件，而为之后被使用的配置零件生成一个*webpack.parts.js*文件。如下就是从已有的代码中提取出来的，基于函数的接口，所组成的零件的一部分。

**webpack.parts.js**

```javascript
exports.devServer = ({ host, port } = {}) => ({
  devServer: {
    historyApiFallback: true,
    stats: 'errors-only',
    host, // Defaults to `localhost`
    port, // Defaults to 8080
    overlay: {
      errors: true,
      warnings: true,
    },
  },
});

exports.lintJavaScript = ({ include, exclude, options }) => ({
  module: {
    rules: [
      {
        test: /\.js$/,
        include,
        exclude,
        enforce: 'pre',

        loader: 'eslint-loader',
        options,
      },
    ],
  },
});
```

T> The same `stats` idea works for production configuration as well. See [the official documentation](https://webpack.js.org/configuration/stats/) for all the available options.

T> 同样的`stats`想法对生产环境配置也有用。可查看[the official documentation](https://webpack.js.org/configuration/stats/)来获取所有的选项。

To connect these configuration parts, set up *webpack.config.js* as in the code example below:

为了使用这些配置零件，如下设置*webpack.config.js*：

**webpack.config.js**

```javascript
const path = require('path');
const HtmlWebpackPlugin = require('html-webpack-plugin');
const merge = require('webpack-merge');

const parts = require('./webpack.parts');

const PATHS = {
  app: path.join(__dirname, 'app'),
  build: path.join(__dirname, 'build'),
};

const commonConfig = merge([
  {
    entry: {
      app: PATHS.app,
    },
    output: {
      path: PATHS.build,
      filename: '[name].js',
    },
    plugins: [
      new HtmlWebpackPlugin({
        title: 'Webpack demo',
      }),
    ],
  },
  parts.lintJavaScript({ include: PATHS.app }),
]);

const productionConfig = merge([
]);

const developmentConfig = merge([
  parts.devServer({
    // Customize host/port here if needed
    host: process.env.HOST,
    port: process.env.PORT,
  }),
]);

module.exports = (env) => {
  if (env === 'production') {
    return merge(commonConfig, productionConfig);
  }

  return merge(commonConfig, developmentConfig);
};
```

After this change, the build should behave the same way as before. This time, however, you have room to expand, and you don't have to worry about how to combine different parts of the configuration.

如此修改后，构建应该和之前表现的一样。但是，此时，你拥有了扩展的空间，而且你不必担心如何将不同的配置零件结合起来。

You can add more targets by expanding the *package.json* definition and branching at *webpack.config.js* based on the need. *webpack.parts.js* grows to contain specific techniques you can then use to compose the configuration.

你可以通过扩展*package.json*文件的定义来增加更多的构建目标，并在*webpack.config.js*文件中基于需要创建分支。*webpack.parts.js*也会增长来包含特定的技术，你可以用它们来组装配置。

T> Webpack 2 validates the configuration by default. If you make a mistake like a typo, it lets you know.

T> Webpack2默认会对配置进行验证。如果你做错了什么，例如错字，它会提示你。

## Benefits of Composing Configuration
## 从组合配置中收益

Splitting configuration allows you to keep on expanding the setup. The biggest win is the fact that you can extract commonalities between different targets. You can also identify smaller configuration parts to compose. These configuration parts can be pushed to packages of their own to consume across projects.

将配置拆分让你可以持续扩展该设置。最大的好处就是你可以从不同的构建目标的配置中提取共性。你也可以使用较小的配置零件来进行组合。这些配置零件可以被推送到它们自己的代码包中，以便于跨项目使用。

Instead of duplicating similar configuration across multiple projects, you can manage configuration as a dependency now. As you figure out better ways to perform tasks, all your projects receive the improvements.

相比于跨项目复制相同的配置，你可以将配置视为一个依赖来进行管理。当你找到一个更好的方式来执行任务，所有的项目都能获得这个提升。

Each approach comes with its pros and cons. Composition-based approach myself is a good starting point. In addition to composition, it gives you a limited amount of code to scan through, but it's a good idea to check out how other people do it too. You can find something that works the best based on your tastes.

每个方法有其利弊。我的这个基于组合的方式是一个很好的开始。除了组合之外，它还提供的有限数量的代码来阅览，但是查看其他人如何处理也是一个很好的注意。你可以基于你的测试，找出一些最好的东西。

Perhaps the biggest problem is that with composition you need to know what you are doing, and it's possible you aren't going to get the composition right the first time around. But that's a software engineering problem that goes beyond webpack.

也许使用组合的最大问题就是你需要明确你在做什么，而且很有可能在开始阶段不能找到正确的组合。但是那也是超越webpack的软件工程问题。

You can always iterate on the interfaces and find better ones. By passing in a configuration object instead of multiple arguments, you can change the behavior of a part without effecting its API. You can expose API as you need it.

你可以对接口进行迭代，并找到更好地方法。通过传入一个配置对象而不是多个参数，你可以更改配置零件的行为，而不影响其API。你可以根据需要，来暴露API。

T> If you have to support both webpack 1 and 2, you can perform branching based on version using `require('webpack/package.json').version` to detect it. After that, you have to set specific branches for each and merge.

T> 如果你需要同时支持webpack1和webpack2，你可以基于版本来创建分支，并使用`require('webpack/package.json').version`来进行检测。然后，你需要为不同的版本设置特定的分支并进行合并操作。

{pagebreak}

## Configuration Layouts
## 配置的布局

In the book project, you push all of the configuration into two files: *webpack.config.js* and *webpack.parts.js*. The former contains higher level configuration while the latter lower level and isolates you from webpack specifics. The chosen approach allows more layouts, and you can evolve it further.

在本书的项目中，你将所有的配置都放入到了两个文件中：*webpack.config.js*和*webpack.parts*。前置包含了更高层面的配置，而后者是更低层面的并将你与webpack的细节隔离。所选的方式允许更多的布局，而你可以进一步对其扩展。

### Split per Configuration Target
### 按配置目标拆分

If you split the configuration per target, you could end up with a file structure as below:

如果你按配置的目标进行拆分，你可以得到如下的文件结构：

```bash
.
└── config
    ├── webpack.common.js
    ├── webpack.development.js
    ├── webpack.parts.js
    └── webpack.production.js
```

In this case, you would point to the targets through webpack `--config` parameter and `merge` common configuration through `module.exports = merge(common, config);`.

在此情况下，你将通过webpack的`--config`参数来制定配置目标，并通过`module.exports = merge(common, config);`语法引入公共的配置并对其进行`merge`。

{pagebreak}

### Split Parts per Purpose
### 按照目的来拆分零件集

To add hierarchy to the way configuration parts are managed, you could decompose *webpack.parts.js* per category:

如果希望将零件集分开管理，你可以将*webpack.parts.js*按照分类来拆分：

```bash
.
└── config
    ├── parts
    │   ├── devserver.js
    ...
    │   ├── index.js
    │   └── javascript.js
    └── ...
```

This arrangement would make it faster to find configuration related to a category. A good option would be to arrange the parts within a single file and use comments to split it up.

这个安排使得很容易找到一个分类相关的配置。一个很好的选项就是将所有的配置零件放在一个单独的文件中，然后使用注释将其分隔开。

### Pushing Parts to Packages
### 将配置零件放到代码包中

Given all configuration is JavaScript, nothing prevents you from consuming it as a package or packages. It would be possible to package the shared configuration so that you can consume it across multiple projects. See the *Package Authoring Techniques* chapter for further information on how to achieve this.

由于所有的配置都是使用JavaScript编写的，谁都不能阻止你将其作为一个代码包来使用。将共享的配置打包，并用于多个项目也是可行的。查看*Package Authoring Techniques*章节了解更多关于如何实现这个目标的信息。

{pagebreak}

## Conclusion
## 小结

Even though the configuration is technically the same as before, now you have room to grow it.

尽管当前的配置在技术上和之前并没有区别，但是现在你拥有了扩展它的空间。

To recap:

回顾：

* Given webpack configuration is JavaScript code underneath, there are many ways to manage it.
* 由于webpack的配置是用JavaScript代码的编写的，所以有多种管理的方式。

* You should choose a method to compose configuration that makes the most sense to you. [webpack-merge](https://www.npmjs.com/package/webpack-merge) was developed to provide a light approach for composition, but you can find many other options in the wild.
* 你可以选择一种对你来说最有意思的方法，来组合配置。[webpack-merge](https://www.npmjs.com/package/webpack-merge)被开发出来，以提供一个轻便的组合方式，但你也可以找到一些其它的选项。

* Composition can enable configuration sharing. Instead of having to maintain a custom configuration per repository, you can share it across repositories this way. Using npm packages enables this. Developing configuration is close to developing any other code. This time, however, you codify your practices as packages.
* 组合的方案更利于配置的共享。相比于在每个仓库中维护一个自定义的配置，你可以用这种方式在多个仓库中共享它。使用npm包就是一实现这个。开发一个配置和开发其他代码没有太多差别。然而，这一次，你将自己的实践编码成了包。

The next parts of this book cover different techniques, and *webpack.parts.js* sees a lot of action as a result. The changes to *webpack.config.js* fortunately remain minimal.

本书接下来的部分包含了不同的技术，并最终在*webpack.parts.js*中见证了效果。而*webpack.config.js*文件有幸保持较小的规模。
