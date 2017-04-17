# 环境变量（nvironment Variables）

Sometimes a part of your code should execute only during development. Or you could have experimental features in your build that are not ready for production yet. This is where controlling **environment variables** becomes valuable as you can toggle functionality using them.

有时你的一部分代码只在开发环境下执行。或者你可以在生产版本还没准备妥当的情况下尝试下实验的特性。

Since JavaScript minifiers can remove dead code (`if (false)`), you can build on top of this idea and write code that gets transformed into this form. Webpack's `DefinePlugin` enables replacing **free variables** so that you can convert `if (process.env.NODE_ENV === 'development')` kind of code to `if (true)` or `if (false)` depending on the environment.

自从JavaScript压缩器能够移除无用代码（`if (false)`），你可以在这一理念之上来组织，也可以编写代码来转换成为这一形式。Webpack的`DefinePlugin`支持替换无用变量，因此你可以将`if (process.env.NODE_ENV === 'development')`类型的代码根据不同的环境转换为`if (true)`或者`if (false)`。

You can find packages that rely on this behavior. React is perhaps the most known example of an early adopter of the technique. Using `DefinePlugin` can bring down the size of your React production build somewhat as a result, and you can see a similar effect with other packages as well.

你可以发现依赖这一行为的NPM包。React可能是一个众所周知的在早期采用该技术的一个例子。使用`DefinePlugin`在构建你的React生产版本一定程度上能够减少包的体积，你也可以在其他的包上看到类似的效果。

## `DefinePlugin`的基本概念（The Basic Idea of `DefinePlugin`）

To understand the idea of `DefinePlugin` better, consider the example below:

为了更好理解`DefinePlugin`的概念，思考如下代码：

```javascript
var foo;

// Not free due to "foo" above, not ok to replace
if (foo === 'bar') {
  console.log('bar');
}

// Free since you don't refer to "bar", ok to replace
if (bar === 'bar') {
  console.log('bar');
}
```

If you replaced `bar` with a string like `'foobar'`, then you would end up with code as below:

如果你用字符串如`foobar`去替换`bar`，结果你会得到如下代码：

```javascript
var foo;

// Not free due to "foo" above, not ok to replace
if (foo === 'bar') {
  console.log('bar');
}

// Free since you don't refer to "bar", ok to replace
if ('foobar' === 'bar') {
  console.log('bar');
}
```

Further analysis shows that `'foobar' === 'bar'` equals `false` so a minifier gives the following:

进一步分析可知`'foobar' === 'bar'`等同于`false`，因此压缩器给出如下代码：

```javascript
var foo;

// Not free due to "foo" above, not ok to replace
if (foo === 'bar') {
  console.log('bar');
}

// Free since you don't refer to "bar", ok to replace
if (false) {
  console.log('bar');
}
```

{pagebreak}

A minifier eliminates the `if` statement as it has become dead code:

压缩器能够移除`if`表达式当它成为无用代码时：

```javascript
var foo;

// Not free, not ok to replace
if (foo === 'bar') {
  console.log('bar');
}

// if (false) means the block can be dropped entirely
```

Elimination is the core idea of `DefinePlugin` and it allows toggling. A minifier performs analysis and toggles entire portions of the code.

移除是`DefinePlugin`的一个核心理念，同时其允许转换。压缩器能够分析和转换全部代码。

## 设置`process.env.NODE_ENV` (Setting `process.env.NODE_ENV`)

Given you are using React in the project and it happens to use the technique, you can try to enable `DefinePlugin` and see what it does to the production build.

若你当前的项目正在使用且碰巧运用了这一技术，你可以试试`DefinePlugin`并看看如何处理生产版本的构建结果。

As before, encapsulate this idea to a function. Due to the way webpack replaces the free variable, you should push it through `JSON.stringify`. You end up with a string like `'"demo"'` and then webpack inserts that into the slots it finds:

与之前类似，将该想法包装为一个函数。由于wepack能够替换无用变量，需要将其通过`JSON.stringify`进行包装。你可以将其包装成字符串形如`'"demo"'`而后webpack将其插入对应的插槽（slots）中：

**webpack.parts.js**

```javascript
exports.setFreeVariable = (key, value) => {
  const env = {};
  env[key] = JSON.stringify(value);

  return {
    plugins: [
      new webpack.DefinePlugin(env),
    ],
  };
};
```

You can connect this with the configuration:

你可以将其合并至配置中：

**webpack.config.js**

```javascript
const productionConfig = merge([
  ...
leanpub-start-insert
  parts.setFreeVariable(
    'process.env.NODE_ENV',
    'production'
  ),
leanpub-end-insert
]);
```

Execute `npm run build` and you should see improved results:

执行`npm run build`而后你应该能看到提升后的结果：

```bash
Hash: fe11f4781275080dd01a
Version: webpack 2.2.1
Time: 4726ms
        Asset       Size  Chunks             Chunk Names
       app.js  802 bytes       1  [emitted]  app
  ...font.eot     166 kB          [emitted]
...font.woff2    77.2 kB          [emitted]
 ...font.woff      98 kB          [emitted]
  ...font.svg     444 kB          [emitted]
     logo.png      77 kB          [emitted]
         0.js  399 bytes       0  [emitted]
  ...font.ttf     166 kB          [emitted]
leanpub-start-insert
    vendor.js    24.3 kB       2  [emitted]  vendor
leanpub-end-insert
      app.css    2.48 kB       1  [emitted]  app
     0.js.map    2.07 kB       0  [emitted]
   app.js.map    2.32 kB       1  [emitted]  app
  app.css.map   84 bytes       1  [emitted]  app
vendor.js.map     135 kB       2  [emitted]  vendor
   index.html  274 bytes          [emitted]
   [4] ./~/object-assign/index.js 2.11 kB {2} [built]
  [14] ./app/component.js 461 bytes {1} [built]
  [15] ./app/shake.js 138 bytes {1} [built]
...
```

You went from 150 kB to 45 kB, and finally, to 24 kB. The final build is faster than the previous one as well.

代码体积从150 kB到45 kB，并最后到达24 kB。最终的构建速度也快于之前的构建速度。

Given the 24 kB can be served gzipped, it's somewhat reasonable. gzipping drops around another 40%, and it's well supported by browsers.

考虑到24 kB还能够被服务器gzip压缩，在一定程度上这体积是可靠的。gzip压缩能够减少额外的40%的体积，且已得到浏览器的良好支持。

T> `webpack.EnvironmentPlugin(['NODE_ENV'])` is a shortcut that allows you to refer to environment variables. It uses `DefinePlugin` underneath and you can achieve the same effect by passing `process.env.NODE_ENV` to the custom function you made. The [documentation covers `EnvironmentPlugin`](https://webpack.js.org/plugins/environment-plugin/) in greater detail.

T> `webpack.EnvironmentPlugin(['NODE_ENV'])`是允许你设置环境变量的一个捷径。其在底层使用`DefinePlugin`，你可以通过给你创建的自定义函数传递`process.env.NODE_ENV`参数来实现同样的效果。更具体的细节可以参看[documentation covers `EnvironmentPlugin`](https://webpack.js.org/plugins/environment-plugin/) 。

## 通过Babel替换无用变量（Replacing Free Variables Through Babel）

[babel-plugin-transform-inline-environment-variables](https://www.npmjs.com/package/babel-plugin-transform-inline-environment-variables) Babel plugin can be used to achieve the same effect. See [the official documentation](https://babeljs.io/docs/plugins/transform-inline-environment-variables/) for details.

[babel-plugin-transform-inline-environment-variables](https://www.npmjs.com/package/babel-plugin-transform-inline-environment-variables) Babel插件可以被用来实现同样的效果。查看[官方文档](https://babeljs.io/docs/plugins/transform-inline-environment-variables/)了解更多细节。

[babel-plugin-transform-define](https://www.npmjs.com/package/babel-plugin-transform-define) and [babel-plugin-minify-replace](https://www.npmjs.com/package/babel-plugin-minify-replace) are other alternatives for Babel.

[babel-plugin-transform-define](https://www.npmjs.com/package/babel-plugin-transform-define) 和 [babel-plugin-minify-replace](https://www.npmjs.com/package/babel-plugin-minify-replace) 是Babel的其他可选项。

## 选择使用模块（Choosing Which Module to Use）

The techniques discussed in this chapter can be used to choose entire modules depending on the environment. As seen above, `DefinePlugin` based splitting allows you to choose which branch of code to use and which to discard. This idea can be used to implement branching on module level.

本篇章节所讨论的技术可以被用于根据不同环境使用整个不同模块。如前所见，`DefinePlugin`允许你选择使用哪一部分的代码并忽略哪一部分。这一理念也可以被应用于模块级别。

{pagebreak}

Consider the file structure below:

考虑如下文件结构：

```bash
.
└── store
    ├── index.js
    ├── store.dev.js
    └── store.prod.js
```

The idea is that you choose either `dev` or `prod` version of the store depending on the environment. It's that *index.js* which does the hard work:

这一想法可以让你根据不同环境来选择采用`dev`还是`prod`版本的存储。这一复杂的工作由*index.js*来处理：

```javascript
if (process.env.NODE_ENV === 'production') {
  module.exports = require('./store.prod');
} else {
  module.exports = require('./store.dev');
}
```

Webpack can pick the right code based on the `DefinePlugin` declaration and this code. You have to use CommonJS module definition style here as ES6 `import`s don't allow dynamic behavior by design.

Webpack 能够基于`DefinePlugin`的声明来选择正确的代码，同时该代码需要使用CommonJS规范来定义。由于ES 6的import机制在设计时就不允许动态加载的行为，因此在这里你必须使用CommonJS模块规范。

T> A related technique, **aliasing**, is discussed in the *Package Consuming Techniques* chapter.

T> 与此相关的技术，别名（aliasing），在NPM包使用技术章节中讨论。

## Webpack优化插件（Webpack Optimization Plugins）

Webpack includes a collection of optimization related plugins:

* [compression-webpack-plugin](https://www.npmjs.com/package/compression-webpack-plugin) allows you to push the problem of generating compressed files to webpack to potentially save processing time on the server.
* `webpack.optimize.UglifyJsPlugin` allows you to minify output using different heuristics. Certain of them break code unless you are careful.
* `webpack.optimize.AggressiveSplittingPlugin` allows you to split code into smaller bundles as discussed in the *Bundle Splitting* chapter. The result is ideal for a HTTP/2 environment.
* `webpack.optimize.CommonsChunkPlugin` makes it possible to extract common dependencies into bundles of their own.
* `webpack.DefinePlugin` allows you to use feature flags in your code and eliminate the redundant code as discussed in this chapter.
* [lodash-webpack-plugin](https://www.npmjs.com/package/lodash-webpack-plugin) creates smaller Lodash builds by replacing feature sets with smaller alternatives leading to more compact builds.

Webpack涵盖了一系列优化相关的插件：

* [ompression-webpack-plugin](https://www.npmjs.com/package/compression-webpack-plugin) 
* `webpack.optimize.UglifyJsPlugin` 能够让你使用不同的策略来压缩输出文件。很明显他们会中断代码运行，除非你非常小心。
* `webpack.optimize.AggressiveSplittingPlugin`能够让你将代码分割成更小的代码包，如之前在代码包分割章节所讨论的那样。这一策略适合在HTTP/2 环境下
* `webpack.optimize.CommonsChunkPlugin` 使其从不同的代码包中提取公共模块成为了可能。
* `webpack.DefinePlugin` 让你在代码中使用特别的标志，同时还能移除多余的代码如本章所讨论的那样。
* [lodash-webpack-plugin](https://www.npmjs.com/package/lodash-webpack-plugin) 创建一个更小的Lodash构建结果

## 结论（Conclusion）

Setting environment variables is a technique that allows you to control which paths of the source are included in the build.

环境变量设置是一种允许控制将哪一部分源码包含进构建文件的技术。

To recap:

* Webpack allows you to set **environment variables** through `DefinePlugin` and `EnvironmentPlugin`. Latter maps system level environment variables to the source.
* `DefinePlugin` operates based on **free variables** and it replaces them as webpack analyzes the source code. You can achieve similar results by using Babel plugins.
* Given minifiers eliminate dead code, using the plugins allows you to remove the code from the resulting build.
* The plugins enable module level patterns. By implementing a wrapper, you can choose which file webpack includes to the resulting build.
* In addition to these plugins, you can find other optimization related plugins that allow you to control the build result in many ways.

本章总结：
* Webpack允许你通过`DefinePlugin`和`EnvironmentPlugin`设置**环境变量**。而后将系统层级的环境变量映射到源码中。
* `DefinePlugin`基于**无用变量**进行处理并通过webpack对源码的分析将其替换掉。使用Babel插件也能实现相同的效果。
* 考虑到压缩器能够移除无用代码，使用插件能够从构建结果中移除代码。
* 该插件支持模块级别。通过封装，你可选择让webpack将哪个文件包含进构建结果中。
* 除了这些插件外，你还可以通过许多方式找到允许你控制构建结果其他相关插件。

To ensure the build has good cache invalidation behavior, you'll learn to include hashes to the generated filenames in the next chapter. This way the client notices if assets have changed and can fetch the updated versions.

为了确保构建结果缓存能够失效，你将要在下一章节了解给文件增加哈希值。这一方式会让客户端注意到若资源发生变更将去获取更新版本。