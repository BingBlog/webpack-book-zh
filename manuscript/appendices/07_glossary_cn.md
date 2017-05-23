# 术语（Glossary）

Given webpack comes with specific nomenclature, the main terms and their explanations have been gathered below based on the book part where they are discussed.

考虑到webpack引入了特定的术语，基于本书所讨论的主要术语及它们的解释都罗列如下：

## 介绍（Introduction）

* **Static analysis** - When a tool performs static analysis, it examines the code without running it. This is how tools like ESLint or webpack operate. Statically analyzable standards, like ES6 module definition, enable features like **tree shaking**.
* **Resolving** is the process that happens when webpack encounters a module or a loader. When that happens, it tries to resolve it based on the given resolution rules.

* **静态分析** - 当工具进行静态分析，其会审查没有运行的代码。这些工具如ESLint或webpack如何处理。静态分析的标准，如ES6的模块定义，支持如**tree shaking**特性。
* **分析** 是webpack加载模块或加载器时所产生的一个处理过程。当在分析阶段，其会尝试基于所提供的规则来分析当前资源。

## 开发（Developing）

* **Entry** refers to a file used by webpack as a starting point for bundling. An application can have multiple entries and depending on configuration, each entry can result in multiple bundles. Entries are defined in webpack's `entry` configuration. Entries are **modules** at the beginning of the dependency graph.
* **Module** is a general term to describe a piece of the application. In webpack, it can refer to JavaScript, a style sheet, an image or something else. **Loaders** allows webpack to support different file types and therefore different types of module. If you point to the same module from multiple places of a code base, webpack will generate a single module in the output. This enables the singleton pattern on module level.
* **Plugins** connect to webpack's event system and can inject functionality into it. They allow webpack to be extended and can be combined with loaders for maximum control. Whereas a loader works on a single file, a plugin has much broader access and is capable of more global control.
* **Hot Module Replacement (HMR)** refers to a technique where code running in the browser is patched on the fly without requiring a full page refresh. When an application contains complex state, restoring it can be difficult without HMR or a similar solution.
* **Linting** relates to the process in which code is statically analyzed for a series of user-defined issues. These issues can range from discovering syntax errors to enforcing code-style. Whilst linting is by definition limited in its capabilities, a linter is invaluable for helping with early error discovery and enforcing code consistency.

* **入口** 指的是webpack用于打包文件的入口。应用可以根据配置拥有多个入口，每个入口可以生成多个代码库。入口定义在webpack的`entry`配置。入口是**模块**依赖的开始。
* **模块**用于描述应用的某一部分的常用术语。在webpack中，其可以指JavaScript，样式，图片或者其他。**加载器**允许webpack支持不同的文件类型并因而有不同类型的模块。若从代码的不同的位置指向同一个模块，webpack将在输出时生成独立模块。其在模块级别支持单例模式。
* **插件**连接webpack的事件系统并注入功能。他们能扩展webpack同时能够和加载器相结合以获得最大控制权。然而，加载器可以在单一文件下工作，而插件具有更宽泛的访问权更全面的控制权。
* **热加载替换（HMR）** 指的是运行在浏览器中的代码能够在全站不刷新的情况下被替换的一种技术。当应用包含复杂状态时候，若没有HMR或类似解决方案，重建会变得比较复杂。
* **校验（Linting）**指的是用一系列用户定义的结果来对代码进行静态分析的过程。这些结果包含从语法错误到强制代码格式。尽管通过定义的校验限制了其功能，但校验器（linter）对于早期发现错误和执行代码一致性是无价的。

* **入口**

## 加载（Loading）

* **Loader** performs a transformation that accepts a source and returns transformed source. It can also skip processing and perform a check against the input instead. Through configuration, a loader targets a subset of modules, often based on the module type or location. A loader only acts on a single module at a time whereas a plugin can act on mulitple files.
* **Asset** is a general term for the media and source files of a project that are the raw material used by webpack in building a bundle.

* **加载器** 执行接收的资源文件并返回转换后的资源文件的一个转换。其可以跳过这一环节而仅检查输入。通过配置，加载器定位于模块的子集，通常基于模块类型或位置。一个加载器一次仅能处理一个模块而插件一次可以处理多个文件。
* **资源** 对于项目多媒体文件和源文件是一个常见术语，其是webpack构建代码库的原材料。

## 构建（Building）

* **Source maps** describe the mapping between the original source code and the generated code, allowing browsers to provide a better debugging experience. For example, running ES6 code through Babel generates completely new ES5 code. Without a source map, a developer would lose the link from where something happens in the generated code and where it happens in the source code. The same is true for style sheets when they run through a pre or post-processor.
* **Bundle** is the result of bundling. Bundling involves processing the source material of the application into a final bundle that is ready to use. A bundler can generate more than one bundle.
* **Bundle splitting** offers one way of optimising a build, allowing webpack to generate multiple bundles for a single application. As a result, each bundle can be isolated from changes effecting others, reducing the amount of code that needs to be republished and therefore re-downloaded by the client and taking advantage of browser caching.
* **Code splitting** produces more granular bundles than bundle splitting. To use it, the developer has to enable it through specific calls in the source code. Using a dynamic `import()` is one way.
* **Chunk** is a webpack-specific term that is used internally to manage the bundling process. Webpack composes bundles out of chunks, and there are several types of those.

* **source maps** 描述了源代码和生成代码间的映射关系，以让浏览器提供更好的调试体验。例如，运行基于Babel生成全新的ES 5的代码的ES 6代码。没有source map，开发者将会对于生成代码发生了什么以及其对应的源码位置感到迷茫。同样的问题会出现在样式中，当对它们进行预处理或后期处理。
* **代码库**是打包的结果。打包将所有将要使用到的应用中原材料包含到最终的代码库的一个处理环节。打包器（bundler）可以生成多个代码库。
* **分割代码库**提供了一种优化构建的方式，允许webpack为单一应用生成多个代码包。每个代码库可以和影响其他的代码库分离开，减少重新发布的代码数量并因此客户端重下载数量，有利于浏览器缓存。
* **代码分割**是比代码库分割更为精细化的过程。为使用它，开发者必须在源码中使用特定调用来支持。使用动态`import()`是一种方式。
* **文本块** 是webpack特定术语，通常用于内部管理打包流程。Webpack组合大量文本块，并存在几种类型。

## 优化（Optimizing）

* **Minifying**, or minification, is an optimization technique in which code is written in a more compact form without losing meaning. Certain destructive transformations break code if you are not careful.
* **Tree shaking** is the process of dropping unused code based on static analysis. A good ES6 module definition allows this process as it's possible to analyze in this particular manner.
* **Hashing** refers to the process of generating a hash that is attached to the asset/bundle path to invalidate it on the client. Example of a hashed bundle name: *app.f6f78b2fd2c38e8200d.js*.

* **压缩**或缩小， 是一种将代码写得更为紧凑而不丢失语义的一种优化技术。若不留意，某些特定转换会损坏代码。
* **Tree shaking** 基于静态分析删除未使用代码的环节。好的ES 6模块定义允许这一环节，由于其能够以这一特殊的方式进行分析。
* **哈希**指的是为资源、代码库生成哈希值以让客户端失效的一个处理过程。一个带有哈希值代码库的名称的例子：*app.f6f78b2fd2c38e8200d.js*。

## 输出（Output）

* **Output** refers to files emitted by webpack. More specifically, webpack emits **bundles** and **assets** based on the output settings.
* **Target** options of webpack allow you to override the default web target. You can use webpack to develop code for specific JavaScript platforms.

* **输出** 指的是webpack生成的文件。更具体地说，webpack根据输出配置生成**代码包**和**资源**。
* webpack的**目标**参数允许你重写默认的web目标。你可以使用wepback开发特定的JavaScript平台代码。
