# Webpack-SurviveJS

@(Webpack-book)

---------------------------

[TOC]

# 总结（Conclusion）

As this book has demonstrated, webpack is a versatile tool. To make it easier to recap the content and techniques, go through the checklists below.

如本书所示，webpack是个多样化的工具。为更好回顾起内容和技术，概括了如下的清单。

## 普通清单（General Checklist）

* **Source maps** allow you to debug your code in the browser during development. They can also give better quality stack traces during production usage if you capture the output. The *Source Maps* chapter delves into the topic.
* To keep your builds fast, consider optimizing. The *Performance* chapter discusses a variety of strategies you can use to achieve this.
* to keep your configuration maintainable, consider composing it. As webpack configuration is JavaScript code, it can be arranged in many ways. Find a way you are comfortable with and apply it. The *Composing Configuration* chapter discusses the topic.
* The way webpack consumes packages can be customized. The *Package Consuming Techniques* chapter covers specific techniques related to this.
* Sometimes you have to extend webpack. The *Extending with Loaders* and *Extending with Plugins* chapters show how to achieve this. You can also work on top of webpack’s configuration definition and implement an abstraction of your own for it to suit your purposes.

* **Source maps**允许你在开发期间在浏览器里调试代码。若你捕获输出，他们还能在生产环境提供更高质量的堆栈追踪信息。*Source Maps*一章探究了这一话题。
* 为能快速构建，需要考虑优化。*性能*一章讨论了大量可实现的优化策略。
* 为让配置可维护，考虑使用组合的方式。由于webpack配置是JavaScript代码，其可以被许多方式组织。找到一种你认为合适的方式并应用。*组合配置*一章讨论了这一话题。
* webpack使用包的方式可以被自定义。*包使用技术*一章涵盖了相关的特定技术。
* 有时你必须扩展webpack。*扩展加载器*和*扩展插件*一章演示了如何实现此。你也可以在webpack的顶层配置定义和实现一个能够满足你自己目的的抽象实现。

{pagebreak}

## 开发清单（Development Checklist）

* To get most out of webpack during development, use [webpack-dev-server](https://www.npmjs.com/package/webpack-dev-server) (WDS). You can also find middlewares which you can attach to your Node server during development. The *Automatic Browser Refresh* chapter covers WDS in greater detail.
* Webpack implements **Hot Module Replacement** (HMR). It allows you to replace modules without forcing a browser refresh while your application is running. The *Hot Module Replacement* appendix covers the topic in detail. If you are using React, read the *Hot Module Replacement with React* appendix.
* **Linting** is a technique that allows you to detect code related issue through static analysis. You can disallow certain kind of usage while enforcing coding style using linting tools. Particularly [ESLint](http://eslint.org/) is a popular option for JavaScript code. [Stylelint](https://www.npmjs.com/package/stylelint) can be used with CSS. The *Linting JavaScript* and *Linting CSS* chapters cover the topic in detail. See also the *Customizing ESLint* appendix to learn how to expand ESLint.

* 为在开发期间充分使用webpack，使用[webpack-dev-server](https://www.npmjs.com/package/webpack-dev-server) （WDS）。你也可以在开发环节使用绑定在Node服务上的中间件。*自动刷新浏览器*一章更为详细地涵盖了WDS的内容。
* Webpack实现了**热模块替换**（HMR）。其能够在应用运行期间替换模块而不用强制刷新浏览器。附录*热模块替换*详细的探讨了这一话题。若你使用React，阅读附件*React的热模块替换*。

## 生产清单（Production Checklist）

### 样式（Styling）

* Webpack inlines style definitions to JavaScript by default. To avoid this, separate CSS to a file of its own using `ExtractTextPlugin` or an equivalent solution. The *Separating CSS* chapter covers how to achieve this.
* To decrease the number of CSS rules to write, consider **autoprefixing** your rules. The *Autoprefixing* chapter shows how to do this.
* Unused CSS rules can be eliminated based on static analysis. The *Eliminating Unused CSS* chapter explains the basic idea of this technique.

* Webpack默认内嵌样式到JavaScript中。为避免于此，使用`ExtractTextPlugin`将CSS分离到自己独立的文件或类似解决方案。*分离CSS*一章涵盖了如何实现。
* 为减少CSS规则编写数量，考虑为规则使用**自动化前缀**。**自动化前缀**章节演示了如何实现。
* 基于静态分析未使用的CSS规则可以被移除。*移除未使用CSS*一章解释了这一技术的基本理念。

### 资源（Assets）

* When loading images through webpack, optimize them, so the users have less to download. The *Loading Images* chapter shows how to do this.
* Load only the fonts you need based on the browsers you have to support. The *Loading Fonts* chapter discusses the topic.
* Minify your source files to make sure the browser to decrease the payload the client has to download. The *Minifying* chapter shows how to achieve this.

* 当通过webpack加载图片时，优化他们，以让用户下载内容更少。*加载图片*一章展示了如何实现该方式。
* 仅基于你需要支持的浏览器加载所需字体。*字体加载*一章探路了这一主题。
* 压缩源文件以确保浏览器减少客户端必须下载的数据。*压缩*一章展示了如何实现该技术。

### 缓存（Caching）

* To benefit from client caching, split a vendor bundle out of your application. This way the client has less to download in the ideal case. The *Bundle Splitting* chapter discusses the topic. The *Adding Hashes to Filenames* chapter shows how to achieve cache invalidation on top of that.
* Use webpack’s **code splitting** functionality to load code on demand. The technique is handy if you don’t need all the code at once and instead can push it behind a logical trigger such as clicking a user interface element. The *Code Splitting* chapter covers the technique in detail. The *Dynamic Loading* chapter shows how to handle more advanced scenarios.
* Add hashes to filenames as covered in the *Adding Hashes to Filenames* chapter to benefit from caching and separate a manifest to improve the solution further as discussed in the *Separating Manifest* chapter.

* 为从客户端缓存获利，将代码库包从应用中分割出来。在理想情况下，该方式可以减少客户端的下载量。*代码库分割*一章谭库了这一话题。*为文件增加哈希名*一章演示了如何让缓存失效。
* 使用webpack的**代码分割**功能按需加载代码。这一技术非常方便若你并非立刻需要所有代码，而是放入一个逻辑触发器如点击用户界面元素。*代码分割*一章详细地涵盖了这一技术。*动态加载*一章显示如何处理更复杂的场景。
* 为文件增加哈希名在*给文件增加哈希值*一章中谈论到，以从缓存获利和在*分离清单*一章所谈论的，分离清单以进一步改进解决方案。

### 优化（Optimization）

* Use ES6 module definition to leverage **tree shaking**. It allows webpack to eliminate unused code paths through static analysis. See the *Tree Shaking* chapter for the idea.
* Set application specific environment variables to compile it production mode. You can implement feature flags this way. See the *Environment Variables* chapter to recap the technique.
* Analyze build statistics to learn what to improve. The *Analyzing Build Statistics* chapter shows how to do this against multiple available tools.
* Push a part of the computation to web workers. The *Web Workers* chapter covers how to achieve this.

* 使用ES 6模块定义以利用**tree shaking**。其允许wepback通过静态分析移除未使用的代码路径。查看*Tree Shaking*一章了解这一理念。
* 设置应用的具体环境变量用以在生产环境编译。可以实现特性标识符。查看*环境变量*一章来回顾相关技术。
* 分析构建数据以了解什么需要提升。*分析构建数据*一章展示如何使用多种方便工具来处理。
* 将一部分计算放入web workers。*Web Workers*一章涉及了如何实现。

### 输出（Output）

* Clean up and attach information about the build to the result. The *Tidying Up* chapter shows how to do this.
* If you are authoring a package, take care with how you bundle it. The *Authoring Packages* chapter covers the related considerations.

* 清理并将信息绑定到构建结果上。*清理*一章演示了如何处理。
* 若你是包作者，需要注意如何打包。*包开发*一章涵盖了相关的注意事项。

## 结论（Conclusion）

Webpack allows you to use a lot of different techniques to splice up your build. It supports multiple output formats as discussed in the *Output* part of the book. Despite its name, it’s not only for the web. That’s where most people use it, but the tool does far more than that.

Webpack允许你使用大量不同的技术来分割构建。其如在本书*输出*部分所讨论，支持多种输出格式。尽管其名字，其不仅适用于web。大部分web开发人员都使用它，但这工具远不仅于此。
