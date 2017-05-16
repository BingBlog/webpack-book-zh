# What is Webpack
# Webpack是什么？

Webpack is a module bundler. You can use a separate task runner while leaving it to take care of bundling, however this line has become blurred as the community has developed plugins for it. Sometimes these plugins are used to perform tasks that are usually done outside of webpack, for example cleaning the build directory or deploying the build.

Webpack是一个模块打包工具。你可以使用一个单独的任务运行器，同时将打包的事情交给Webpack。随着社区开发出很多Webpack的插件，这条（分界）线也越来越模糊。有时这些插件被用来执行一些通常是webpack打包之外的任务，例如：清除build目录或者部署代码。

Webpack became particularly popular with React due to **Hot Module Replacement** (HMR) which helped greatly to popularize webpack and lead to its usage in other environments, such as [Ruby on Rails](https://github.com/rails/webpacker). Despite its name, web pack is not limited to web alone. It can bundle for other targets as well as discussed in the *Build Targets* chapter.

由于**Hot Module Replacement**
(HMR)－热加载的功能在React中的使用，Webpack变得很受欢迎。很大程度的使webpack得以普及并导致其在其它环境中被使用，如[Ruby on Rails](https://github.com/rails/webpacker)。虽然它的名字叫"webpack"，但是web pack并不表示仅限于web。它也可以用来构建其它目标，就像在*Build Targets*章节所讨论的一样。

T> If you want to understand build tools and their history in a better detail, check out the *Comparison of Build Tools* appendix.

T> 如果你想更详细的了解构建工具和它们的历史，请查看*Comparison of Build Tools*附录。


## Webpack Relies on Modules
## webpack依赖于模块（Modules）

The smallest project you could bundle with webpack consists of **input** and **output**. The bundling process begins from user defined **entries**. Entries themselves are **modules** and can point to other modules through **imports**.

如果你想象一下webpack打包的最简单的项目，那么（这个项目）就只有**输入(input)**和**输出(output)**。对webpack来说，打包过程从用户定义的**入口文件(entries)**开始。入口文件本身也是**模块**并且可以通过**imports**引入其它模块。

When you bundle a project through webpack, it traverses through imports, constructing a **dependency graph** of the project and then generating the **output** based on the configuration. It's possible to define **split points** generating separate bundles within the project code itself.

当你使用Webpack来打包一个项目时，它将查找所有的imports，此时，webpack会构建一个**依赖关系图(dependency graph)**，然后基于配置文件生成**输出文件(output)**。并可以通过定义**拆分规则(split points)**将项目代码生成单独的代码包。

Webpack supports ES6, CommonJS, and AMD module formats out of the box. The loader mechanism works for CSS as well, and `@import` and `url()` are supported through *css-loader*. You can also find plugins for specific tasks, such as minification, internationalization, HMR, and so on.

Webpack支持ES6，CommonJS，和AMD模块格式的开箱即用。加载器（loader）机制也适用于CSS，并且通过css-loader也可支持`@import`和`url()`。你也可以为不同的任务找到各种各样的插件，如代码压缩，国际化，HMR等。

T> A dependency graph describes is a directed graph that describes how nodes relate to each other. In this case the graph definition is defined through references (`require`, `import`) between files. Webpack traverses this information in a static manner without executing the source to generate the graph it needs to create bundles.

T> 关系依赖图描述的是一个有向图，这个有向图描绘了各个节点的相互关系。在此情况下，这个图是通过定义文件之间的引用（`require`，`import`）来定义的。Webpack在不执行源代码的情况下，通过静态方式遍历这些信息，然后生成这张图，用以创建代码包。

## Webpack's Execution Process
## Webpack执行过程

![Webpack's execution process](https://raw.githubusercontent.com/survivejs-translations/webpack-book-zh/master/manuscript/images/webpack-process.png)

Webpack begins its work from **entries**. Often these are JavaScript modules where webpack begins its traversal process. During this process webpack evaluates the matches against **loader** configuration that tells how to transform the files.

Webpack从**入口(entries)**开始工作。通常它们是JavaScript模块，webpack从这里开始遍历进程。在此过程中对匹配的**加载器(loader)**配置进行执行，这些配置告知了webpack如何进行文件转化。

{pagebreak}

### Resolution Process
### 解析过程

An entry itself is a module and when webpack encounters one, it tries to match it against the file system using its `resolve` configuration. You can tell webpack to perform the lookup against specific directories in addition to *node_modules*. It's also possible to adjust the way it matches against file extensions and you can define specific aliases against directories. The *Package Consuming Techniques* chapter covers these ideas in greater detail.

一个入口本身就是一个模块，当webpack遇到一个入口，它将会用其`resolve`配置来与文件系统进行匹配。除了*node_modules*目录目录以外，你可以告知webpack对特定文件夹执行查找。你也可以调整匹配文件扩展名的方式并为一些目录设置特定的别名。*Package Consuming Techniques*章节更加详细地涵盖了这些想法。

If the resolution pass failed, webpack gives a runtime error. If webpack managed to resolve a file correctly, webpack performs processing over the matched file based on the loader definition. Each loader applies a specific transformation against the module contents.

如果解析失败，webpack将会给出一个运行时错误。如果webpack正确的解析了这个文件，webpack将根据加载器定义，对匹配的文件执行处理。每一个加载器都会对模块的内容执行特定的转化。

The way a loader gets matched against a resolved file can be configured in multiple ways including file type and its location in the file system. Webpack provides even more flexible ways to achieve this as you can apply specific transformation against a file based on *where* it was imported to the project.

加载器和要被处理的文件的匹配方式，可以以多种形式进行配置，（这些形式）包括文件类型以及其在文件系统中的位置。Webpack提供了更加灵活的方法来实现这一点，您可以在文件被导入到项目中的**地方**，来对文件应用特定的转换。


The same resolution process is performed against webpack's loaders. It allows you to apply similar logic there while it figures out which loader it should use. Loaders have resolve configuration of their own for this reason. If webpack fails to perform a loader lookup, you will get a runtime error.

对webpack加载器进行同样过程的解析。它允许你使用类似的逻辑，同时确定使用何种加载器。因此加载器有自己的解析配置。如果webpack执行加载器查找失败，你将会得到一个运行时错误。

### Webpack Resolves Against Any File Type
### Webpack解析各种类型的文件

Webpack will resolve against each module it encounters while constructing the dependency graph. If an entry contains dependencies, the process will be performed against each until the traversal has completed. Webpack performs this process against any file type unlike a specialized tool like Babel or Sass compiler.

Webpack将会对所遇到的每个模块进行解析，同时构造出依赖关系图。如果一个入口包含依赖，该过程将会对每一个（依赖）执行直到遍历结束。Webpack对所有类型的文件都会执行这个过程，而不像特定的工具，如Babel或者Sass编译器。

Webpack gives you control over how to treat different assets it encounters. You can decide to **inline** assets to your JavaScript bundles to avoid requests for example. Webpack also allows you to use techniques, like CSS Modules, to couple styling with components and to avoid issues of standard CSS styling. It's this flexibility which makes webpack valuable.

Webpack让你来控制如何对待所遇到的不同的资源。例如你可以决定以**内联**的形式将资源放到到你的JavaScript代码包中来避免（更多的）请求。Webpack也允许你使用其它技术，比如CSS模块（CSS Modules），可以将样式和组件配对，来避免标准的CSS样式的问题（全局污染）。所以说，是上述的这些灵活性使webpack变得更有价值。

Although webpack is used mainly to bundle JavaScript, it can capture assets like images or fonts and emit separate files for them. Entries are only a starting point of the bundling process. What webpack emits depends entirely on the way you configure it.

尽管webpack主要被用来打包JavaScript，它也可以抓取其他资源，如图片、字体，并可将这些资源分发到单独的文件中。入口只是打包过程的开始。webpack所分发出的内容完全取决于你如何配置它。

### Evaluation Process
### 执行过程

Assuming all loaders were found, webpack evaluates the matched loaders from bottom to top and right to left (`styleLoader(cssLoader('./main.css'))`), running the module through each loader in turn. As a result you get output which webpack will inject in the resulting **bundle**. The *Loader Definitions* chapter covers the topic in detail.

假设所有的加载器都被找到了，webpack会按照从下往上，从右到左的顺序(`styleLoader(cssLoader('./main.css'))`)执行匹配的加载器，在每个加载器中运行模块。最后，你将得到的是webpack注入到最终的**代码包**的输出。*Loader Definitions*章节更详细的涵盖了该话题。

If all loader evaluation completed without a runtime error, webpack includes the source in the last bundle. **Plugins** allow you to intercept **runtime events** at different stages of the bundling process.

如果所有的加载器在运行时都没有报错，webpack会将资源包裹到最终的代码包。**Plugins**允许你在打包过程的不同阶段插入**运行时事件(runtime events)**

Although loaders can do a lot, they don’t provide enough power for advanced tasks by themselves. Plugins intercept **runtime events** provided by webpack. A good example is bundle extraction performed by `ExtractTextPlugin` which, working in tandem with a loader, extracts CSS files out of the bundle and into a file of its own.

尽管加载器可以实现很多，但是它们本身并没有为更高级的任务提供足够强大的处理能力。插件（却）可以插入webpack支持的**运行时事件(runtime events)**。一个很好的例子就是`ExtractTextPlugin`执行的代码包提取，它同一个加载器串起来工作，从代码包中提取CSS文件并输出到其独自的文件。

Without this step, CSS would end up in the resulting JavaScript as webpack treats all code as JavaScript by default. The extraction idea is discussed in the *Separating CSS* chapter.

没有这个步骤，CSS将会（内联）在最终的JavaScript代码中，因为webpack默认将所有的代码都视为JavaScript。提取的想法在*Separating CSS*章节将被讨论。

### Finishing
### 结束

After every module has been evaluated, webpack writes **output**. The output includes a bootstrap script with a manifest that describes how to begin executing the result in the browser. The manifest can be extracted to a file of its own as discussed later in the book. The output differs based on the build target you are using and targeting web is not the only option.

当每个模块都被执行后，webpack会写入到**输出**中。输出中包含了带有清单的引导脚本，这个脚本描绘了如何在浏览器中执行这些结果。清单可以如本书后面讲到的那样被提取到其独自的文件中。输出也会根据你的构建的目的而不同，web（构建的）目的并不是唯一的选项。

That’s not all there is to the bundling process. For example, you can define specific **split points** where webpack generates separate bundles that are loaded based on application logic. The idea is discussed in the *Code Splitting* chapter.

上述也并不是所有的打包过程。例如，你可以定义特定的**拆分规则(split points(**，进而使webpack根据应用的逻辑生成单独的代码包。该想法将在*代码拆分(Code Splitting)*章节被讨论。

{pagebreak}

## Webpack Is Configuration Driven
## Webpack由配置驱动 

At its core webpack relies on configuration. Here is a sample adapted from [the official webpack tutorial](https://webpack.js.org/get-started/) that shows how the main ideas go together:

webpack在核心上依赖于配置。如下是一个从[the official webpack tutorial](https://webpack.js.org/get-started/)摘取的应用实例，将主要的思想一起进行了展示：

**webpack.config.js**

```javascript
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
        exclude: /node_modules/,
      },
    ],
  },

  // What extra processing to perform
  plugins: [
    new webpack.optimize.UglifyJsPlugin(),
  ],

  // Adjust module resolution algorithm
  resolve: {
    alias: { ... },
  },
};
```

Webpack's configuration model can feel a bit opaque at times as the configuration file can appear monolithic. It can be difficult to understand what it's doing unless you understand the ideas behind it. Providing means to tame configuration is one of the main purposes why this book exists.

Webpack的配置模式有时会变得有些不那么透明，因为配置文件可以以单一形式展现。除非你了解了背后的原理，否则就很难理解它到底在做些什么。提供驾驭配置文件的方法也是本书存在的主要目的之一。

T> To understand webpack on source code level, check out [the artsy webpack tour](https://github.com/TheLarkInn/artsy-webpack-tour).

T> 如果想从源代码的层面来理解webpack，可查阅[the artsy webpack tour](https://github.com/TheLarkInn/artsy-webpack-tour)。

## Hot Module Replacement
## 热加载

You are likely familiar with tools, such as [LiveReload](http://livereload.com/) or [BrowserSync](http://www.browsersync.io/), already. These tools refresh the browser automatically as you make changes. HMR takes things one step further. In the case of React, it allows the application to maintain its state without forcing a refresh. While this does not sound that special, it makes a big difference in practice.

你可能已经很熟悉像LiveReload或者BrowerSync这样的工具。这些工具在你更改文件后自动刷新浏览器。HWR更进一步。在React中，它可以使应用保持状态，而不用真正的刷新。尽管听起来没有什么大不了的，但是它在实践上做出了很大的改变。


HMR is available in Browserify via [livereactload](https://github.com/milankinen/livereactload), so it’s not a feature that’s exclusive to webpack.

HMR可以通过[livereactload](https://github.com/milankinen/livereactload)在Browserify中使用，因此它不是Webpack专有的功能。

## Code Splitting
## 代码拆分

Aside from the HMR feature, webpack’s bundling capabilities are extensive. Webpack allows you to split code in various ways. You can even load code dynamically as your application gets executed. This sort of lazy loading comes in handy, especially for larger applications. You can load dependencies as you need them.

除了热加载这个特性外，webpack的打包能力是可扩展的。Webpack允许你用不同的方式进行代码拆分。你甚至可以在应用执行时动态地加载代码。尤其是在大型项目中，懒加载非常方便。你可以在需要时再加载依赖。

Even small applications can benefit from code splitting, as it allows the users to get something useable in their hands faster. Performance is a feature, after all. Knowing the basic techniques is worthwhile.

即使对于轻量级应用来说，也能够受益于代码拆分，因为它可以使用户更快的得到有用的东西。毕竟，性能很重要。知晓基本的技术总是值得的。

## Asset Hashing

With webpack, you can inject a hash to each bundle name (e.g., *app.d587bbd6.js*) to invalidate bundles on the client side as changes are made. Bundle-splitting allows the client to reload only a small part of the data in the ideal case.

使用webpack，你可以向每个文件名中注入一个类似于（e.g.,app.d587bbd6.js）的哈希值，来表明客户端代码包发生了更改。构建包的拆分让客户在理想情况下仅需重新加载一小部分数据。

## Conclusion
## 小结

Webpack comes with a significant learning curve. However it’s a tool worth learning, given it saves so much time and effort over the long term. To get a better idea how it compares to other tools, check out [the official comparison](https://webpack.js.org/get-started/why-webpack/#comparison).

webpack的学习曲线很陡。但从长远来看，它可以节省很多时间和工作量，所以仍然是一个值得学习的好工具。为了更好的将其与其它工具进行对比，请查看[official comparison](https://webpack.js.org/get-started/why-webpack/#comparison)。

Webpack won’t solve everything, however, it does solve the problem of bundling. That’s one less worry during development. Using *package.json* and webpack alone can take you far.

Webpack并不能解决所有问题。但是，它的确解决了打包的问题。开发中再也不用担心这个问题。仅使用pakage.json和webpack就可以让你前进很多步。

{pagebreak}

To summarize:
总结一下：

* Webpack is a **module bundler**, but you can also use it for tasks as well.
* Webpack是一个**模块构建工具**，但你也可以用它来执行任务。

* Webpack relies on a **dependency graph** underneath. Webpack traverses through the source to construct the graph and it uses this information and configuration to generate bundles.
* Webpack的底层基于**依赖关系图**。webpack通过遍历源代码构建出这个图，并使用图信息和配置来生成代码包。

* Webpack relies on **loaders** and **plugins**. Loaders operate on module level while plugins rely on hooks provided by webpack and have the best access to its execution process.
* Webpack基于**加载器**和**插件**。加载器在模块层面上运行，而插件依赖于webpack提供的钩子，可以最好地访问其执行过程。

* Webpack’s **configuration** describes how to transform assets of the graphs and what kind of output it should generate. A part of this information can be included in the source itself if features like **code splitting** are used.
* Webpack的**配置**描述了如何对依赖系图中的资源进行转换以及要生成什么样的输出。如果使用了**代码拆分**这样的功能，有些信息就会被包含在源代码本身中。

* **Hot Module Replacement** (HMR) helped to popularize webpack. It's a feature that can enhance development experience by updating code in the browser without a full refresh. 
* 热加载帮助webpack流行了起来。它是一个可以提高用户体验的特性，可以在不刷新浏览器的情况下更新代码。

* Webpack can generate **hashes** for filenames allowing you to invalidate bundles as their contents change.
* Webpack可以为文件名生成**哈希值(hashes)**来表明代码包的内容被更改

In the next part of the book you'll learn to construct a development configuration using webpack while learning more about its basic concepts.

接下来的章节中，在你学习如何构建一个基本的可用于开发环境webpack配置的同时，也将更加学习一些更高级的概念。
