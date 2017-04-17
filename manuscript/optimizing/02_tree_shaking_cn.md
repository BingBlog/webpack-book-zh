# Tree Shaking

**Tree shaking** is a feature enabled by the ES6 module definition. The idea is that given it's possible to analyze the module definition in a static way without running it, webpack can tell which parts of the code are being used and which are not. It's possible to verify this behavior by expanding the application and adding code there that should be eliminated.

**Tree-shaking**是ES 6模块定义支持的特性。这一概念指在静态环境下，尽可能地分析代码的模块定义而不是运行它， webpack能够指出代码的哪个模块被使用哪个模块没有。可以通过扩大应用和增加一些应该被删除的代码来验证这一行为。

## Tree-Shaking演示 (Demonstrating Tree Shaking)

To shake code, you have to define a module and use only a part of its code. Set one up:

为了能够应用该特性，你应该需要先定义一个模块并使用该模块的一部分。配置如下：

**app/shake.js**

```javascript
const shake = () => console.log('shake');
const bake = () => console.log('bake');

export { shake, bake };
```

To make sure you use a part of the code, alter the application entry point:

为了确保你仅使用一部分代码，将其应用入口修改如下：

**app/index.js**

```javascript
...
leanpub-start-insert
import { bake } from './shake';

bake();
leanpub-end-insert

...
```

{pagebreak}

If you build the project again (`npm run build`) and examine the build (*build/app.js*), it should contain `console.log('bake')`, but miss `console.log('shake')`. That's tree shaking in action.

如果你再次进行项目构建（`npm run build`）并检查构建结果（*build/app.js*），它应该仅包含了`console.log('bake')`，而丢弃了`console.log('shake')`。这就是tree shaking在起作用。

To get a better idea of what webpack is using for tree shaking, run it through `npm run build -- --display-used-exports`. You should see additional output like `[no exports used]` or `[only some exports used: bake]` in the terminal.

为了对webpack是如何使用tree shaking有更好的了解，通过命令`npm run build --display-used-exports`。你应该能够在中断看到额外的输出如[`no exports used`]或者[`only some exports used: bake`]。

T> If you are using `UglifyJsPlugin`, enable warnings for a similar effect. In addition to other messages, you should see lines like `Dropping unused variable treeShakingDemo [./app/component.js:17,6]`.

T> 如果你正在使用`UglifyJsPlugin`，支持警告类似的效果。除了其他信息，你应该还能看到一行类似于`Dropping unsed variable treeShakingDemo [./app/component.js: 17,6]`。

T> There is a CSS Modules related tree shaking proof of concept at [dead-css-loader](https://github.com/simlrh/dead-css-loader).

T> [dead-css-loader](https://github.com/simlrh/dead-css-loader).是一个和tree shaking理念类似的CSS模块。

## 包级别的Tree Shaking (Tree Shaking on Package Level)

The same idea works with dependencies that use the ES6 module definition. Given the related packaging standards are still emerging, you have to be careful when consuming such packages. Webpack tries to resolve *package.json* `module` field for this reason.

这一想法同样适用于ES 6模块中的依赖。考虑到相关的模块标准仍在制定中，当在使用这些包时需要注意。Webpack基于这一原因尝试在*package.json*文件中增加`module`字段来解决这一问题。

For tools like webpack to allow tree shake npm packages, you should generate a build that has transpiled everything else except the ES6 module definitions and then point to it through *package.json* `module` field. In Babel terms, you have to let webpack to manage ES6 modules by setting `"modules": false`.

类似webpack的工具支持对npm包tree shake， 你应该生成一个将所有代码都进行转换了的构建结果，除了通过在*package.json*的`module`字段指定的ES 6模块。在Babel的配置中，你不得不通过设置`"modules": false`来管理ES 6模块。

{pagebreak}

To get most out of tree shaking with external packages, you have to use [babel-plugin-transform-imports](https://www.npmjs.com/package/babel-plugin-transform-imports) to rewrite imports so that they work with webpack's tree shaking logic. See [webpack issue #2867](https://github.com/webpack/webpack/issues/2867) for more information.

为了能够将tree shaking用于其他的扩展包，你需要用 [babel-plugin-transform-imports](https://www.npmjs.com/package/babel-plugin-transform-imports)来重写引入机制以让他们能够和webpack的tree shaking逻辑相吻合。查看[webpack issue #2867](https://github.com/webpack/webpack/issues/2867) 了解更多信息。

## 结论 (Conclusion)

Tree shaking is a potentially powerful technique. For the source to benefit from tree shaking, npm packages have to be implemented using the ES6 module syntax, and they have to expose the ES6 version through *package.json* `module` field tools like webpack can pick up.

Tree shaking是一个潜在地强有力的技术。对于源码来说，为了从tree shaking当中获益，npm包必须采用ES 6语法来实现，同时这些npm包还需要暴露通过*package.json*文件的`module`字段来暴露ES 6版本让类似于webpack的工具能够识别。

To recap:

* **Tree shaking** drops unused pieces of code based on static code analysis. Webpack performs this process for you as it traverses the dependency graph.
* To benefit from tree shaking, you have to use ES6 module definition.
* As a package author, you can provide a version of your package that contains ES6 modules, while the rest has been transpiled to ES5.

概要如下：

* Tree shaking通过静态代码分析将代码中未使用的部分删除。Webpack通过遍历整个依赖图来处理这一过程。
* 为了能够从tree shaking中获益，你不得不使用ES 6模块
* 作为一个包(pacakge)的作者，你可以提供一个包版本还该ES 6模块，而其余部分则被转换为ES5。

You'll learn how to manage environment variables using webpack in the next chapter.

在下个章节，你讲了解到如何通过webpack管理环境变量。