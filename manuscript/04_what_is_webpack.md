# What is Webpack
Webpack是一个模块儿打包工具。你可以将它和任务运行器结合起来使用，但将打包工作交给Webpack。随着社区开发出很多Webpack的插件，这条线也越来越模糊。有时这些插件被用来执行一些打包之外的任务。清除build目录或者部署代码是这些插件的代表。

由于Hot Module Replacement
(HMR)－热加载的功能在React中的使用，Webpack变得很受欢迎。很大程度的帮助了webpack得以普及并导致其在其它环境中被使用，如[Ruby on Rails](https://github.com/rails/webpacker)。虽然它的名字叫"webpack"，但是webpack并不表示仅限于web。它也可以用来构建其它目标，就像在*Build Targets*章节所讨论的一样。
> 如果你想更详细的了解构建工具和它们的历史，请查看*Comparison of Build Tools*附录。

## webpack依赖于模块（Modules）
如果你想象一下webpack打包的最简单的项目，那么就只有输入和输出。对webpack来说，打包处理从用户定义的**入口文件（entries）**开始。入口文件本身也是模块并且可以通过**imports**引入其它模块。

当你使用Webpack来打包一个项目时，它将查找所有的imports，此时，webpack会构建一个**依赖关系图（dependency graph）**，然后生成基于配置文件的**输出（输出）**。它默认将所有内容都输出到一个单个**包（bundle）**，但是，它也可以通过配置来输出多个文件。

Webpack支持ES6，CommonJS，和AMD模块格式开箱即用。加载器（loader）机制也适用于CSS，并且通过css-loader也可支持@import和url()。你也可以为不同的任务找到各种各样的插件，如代码压缩，国际化，HMR等。

>关系依赖图描述的是一个有向图，这个有向图描绘了各个节点的相互联系。在此情况下，这个图是通过定义文件之间的引用（require，import）来定义的。Webpack在不执行源代码的情况下，通过静态方法遍历这些信息，然后生成这张图，用以创建捆绑包。

## 加载器执行模块
当webpack碰到每一个**模块（module）**时，它将执行一下操作：
 1. webpack将会**解析（resolve）**这个模块。Webpack提供了配置来适配这种行为。我们在**包消费（Consuming Packages）**章节着重讨论了与此相关的问题。如果模块解析失败，webpack将给出一个错误信息。
 2. 假设一个模块被成功解析，webpack将尝试将它交给你配置的**加载器（loaders）**。每个加载器的定义包含了该加载器被执行的条件。如果一个加载器被匹配上了，webpack将尝试用类似于模块的方式来解析它。
 3. 如果所有的加载器都被找到，webpack将按照从下到上，从右到左的顺序来执行所匹配到的加载器。在**加载器的定义（Loader Definitions）**章节对该话题进行了详细的讲解。
 4. 如果加载器的执行都没有报错，webpack将把源代码包含到最后一个捆绑包中。插件（Plugins）可以组织这个行为并更改打包的方式。
 5. 当每个模块都被执行后，webpack将编写一个引导脚本，包括描述如何在浏览器中开始执行结果的清单。最后一步会根据你的构建目标而有所不同。

这并没有包含打包处理的所有步骤。例如，你可以定义特定的分割点（split points），使webpack生成单独的构建包，进而可以根据业务逻辑来进行加载。在代码拆分（Code Splitting）章节将会详细讨论。

## 通过插件进行额外控制
尽管加载器非常有用，但是它们并不支持更多先进的功能。**插件（plugins）**允许你拦截webpack的运行时时间。ExtractTextPlugin可以用来对构建包进行提取操作就是一个很好例子。
这个插件同加载器一起工作，从构建包中捕获并生成单独的文件。否则，CSS将被打包进JavaScript中。在*Separating CSS*章节对此进行了详细的讨论。

下面的图包含了上面讨论的主要思想，并展示了它们之间的关系：

![](leanote://file/getImage?fileId=58b8066122826822ec000002)

## Webpack是配置驱动的
Webpack的核心就是其依赖于配置。下面是根据[官方示例](https://webpack.js.org/get-started/)修改的一个简单例子，一起展示了所有主要的思想：
**webpack.config.js**
```
const webpack = require('webpack');
module.exports = {
    // Where to start bundling
    entry: {
        app: './entry.js',
    },
    // Where to output
    output: {
        // Output to the same directory
        path: __dirname,
        // Capture name from the entry using a pattern
        filename: '[name].js',
    },
    // How to resolve encountered imports
    module: {
        rules: [
            {
                test: /\.css$/,
                use: ['style-loader', 'css-loader'],
            },
            {
                test: /\.js$/,
                use: 'babel-loader',
            },
        ],
    },
    // What extra processing to perform
    plugins: [
        new webpack.optimize.UglifyJsPlugin(),
    ],
    // Adjust resolution algorithm
    resolve: {
        alias: { ... },
    },
};
```
由于这个配置是用JavaScript来写的，它是相当可塑的。这个模型可能会让你感觉webpack有点不是那么容易一眼看透，它将在复杂的条件下变得更加难以理解。这也是这本书产生的目的。

>如果想从源码层面上来了解webpack，请查看[artsy webpack tour](https://github.com/TheLarkInn/artsy-webpack-tour).

## 热加载（Hot Module Replacement）
你可能已经很熟悉像LiveReload，或者BrowerSync这样的工具。这些工具在你更改文件后自动刷新浏览器。HWR更进一步。在React中，它可以使应用保持状态，而不用硬刷新。尽管听起来没有什么大不了的，但是它在实践上做出了很大的改变。

## 代码拆分（code spliting）
除了热加载这个特性外，webpack的打包能力是可扩展的。Webpack允许你用不同的方式进行代码拆分。你甚至可以在应用执行时动态的加载代码。尤其是在大型项目中，懒加载非常方便，你可以在需要时再加载依赖。

即使对于轻应用来说，也能够受益于代码拆分，因为它可以使用户更快的得到有用的东西。毕竟，性能很重要。知道基本的技术总是值得的。

## 资源哈希值（Asset Hashing）
使用webpack，在客户端代码发生更改后，你可以向每个文件名中注入一个类似于（e.g.,app.d587bbd6.js）的哈希值。构建包的拆分让客户在理想情况下仅重新加载一小部分数据。

## 加载器和插件
所有这些小的特性加起来。让人非常惊喜，直接使用就可以处理很多事情。如果你却少什么，可以使用加载器和插件。

webpack的学习曲线很陡。从长远来看，它可以节省很多时间和工作量，所以仍然是一个值得学习的好工具。为了更好的将其与其它工具进行对比，请查看[official comparison](https://webpack.js.org/get-started/why-webpack/#comparison)。

## 小结
你可以将webpack同其它工具结合起来使用。它并不能解决所有问题。但是，它可以解决打包的问题。开发中再也不用担心这个问题。仅使用pakage.json，脚本和webpack就可以实现很多，很快你将见证。
总结：
 - webpack是一个模块构建工具，但你也可以用它来执行任务。
 - 热加载使webpack流行起来。它是一个可以提高用户体验的特性
 - webpack基于**依赖关系图**。webpack遍历源代码构建出这个图，并使用图信息和配置来生成构建包。
 - webpack的**配置**描述了如何**依赖转关系图**中的资源和要输出什么。如果使用了代码拆分这样的功能，有些信息就包含在源代码中。
 - webpack将为文件名生成**哈希（hashes）**来表明构建包的内容被更改
 - webpack的逻辑包含在**加载器（loaders）**和**插件（plugins）**中。它们通过配置文件被调用。
在接下来的章节中，在你学习如何编写出一个基本的开发环境和生产环境的webpack配置的同时，我们将更加详细的研究webpack。接下来的章节我们将继续深入并探讨更高级的话题。