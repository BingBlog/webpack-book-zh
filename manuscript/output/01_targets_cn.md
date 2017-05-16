# 构建目标（Build Targets）

Even though webpack is used most commonly for bundling web applications, it can do more. You can use it to target Node or desktop environments, such as Electron. Webpack can also bundle as a library while writing an appropriate output wrapper making it possible to consume the library.

webpack除了被用于构建大部分web应用，其还能具备更多功能。你可以使用它来定位Node或桌面环境，如Electron。Webpack可以生成一个库，并通过写入合适的输出包装器以能够使用库。

Webpack's output target is controlled by the `target` field. You'll learn about the main targets next and dig into library specific options after that.

Webpack的输出目标通过`target`字段控制。你将会在下一部分了解到主要目标并在之后对库的具体选项进行深挖。

## Web目标（Web Targets）

Webpack uses the *web* target by default. This is ideal for a web application like the one you have developed in this book. Webpack bootstraps the application and load its modules. The initial list of modules to load is maintained in a manifest, and then the modules can load each other as defined.

Webpack默认使用*web*目标。这对于如之前你在本书中所开发的web应用来说是很理想的。Webpack启动应用并加载自身模块。初始化加载列表通过一个清单来维护，而后模块能够如定义那样互相加载。

### Web Workers

The *webworker* target wraps your application as a [web worker](https://developer.mozilla.org/en-US/docs/Web/API/Web_Workers_API). Using web workers is valuable if you want to execute computation outside of the main thread of the application without slowing down the user interface. There are a couple of limitations you should be aware of:

* You cannot use webpack's hashing features when the *webworker* target is used.
* You cannot manipulate the DOM from a web worker. If you wrapped the book project as a worker, it would not display anything.

*Webworker*目标将应用包装成一个[web worker](https://developer.mozilla.org/en-US/docs/Web/API/Web_Workers_API)。若你想要在应用的主线程之外执行计算且不会拖慢用户界面，那使用web workers将十分具有价值。但这其中存在一些你应当了解的限制：

* 当*webworker*被使用作为目标时，无法使用webpack的哈希特性。
* 你无法从web worker中操作DOM，如果你将本书的项目包装成一个worker，其将不会展示任何东西。

T> Web workers and their usage are discussed in detail in the *Web Workers* chapter.

T> Web workers及其用法已在*Web Workers*章节详细讨论。

## Node宿主（Node Targets）

Webpack provides two Node-specific targets: `node` and `async-node`. It uses standard Node `require` to load chunks unless async mode is used. In that case, it wraps modules so that they are loaded asynchronously through Node `fs` and `vm` modules.

Webpack提供了两个Node相关d的宿主目标值：`node`和`async-node`。其使用标准的Node`require`API加载文本，除非使用异步模式。在那一案例中，其将模块进行包装以使得他们能够通过Node的`fs`和`vm`模块来异步加载。

The main use case for using the Node target is *Server Side Rendering* (SSR). The idea is discussed in the *Server Side Rendering* chapter.

使用Node宿主的主要用例是*服务端渲染*（SSR）。这一概念将在*服务端渲染*章节进行讨论。

## 桌面宿主（Desktop Targets）

There are desktop shells, such as [NW.js](https://nwjs.io/) (previously *node-webkit*) and [Electron](http://electron.atom.io/) (previously *Atom*). Webpack can target these as follows:

* `node-webkit` - Targets NW.js while considered experimental.
* `atom`, `electron`, `electron-main` - Targets [Electron main process](https://github.com/electron/electron/blob/master/docs/tutorial/quick-start.md).
* `electron-renderer` - Targets Electron renderer process.

有一些桌面客户端壳子，如[NW.js](https://nwjs.io/) (之前为 *node-webkit*)和[Electron](http://electron.atom.io/) (之前为 *Atom*)。Webpack可以通过如下方式进行定位：

* `node-webkit` - 在考虑实验的情况时，针对NW.js。
* `atom`、`electtron`、`electron-main` - 针对[Electron主程](https://github.com/electron/electron/blob/master/docs/tutorial/quick-start.md)。
* `electron-renderer` - 针对Electron渲染环节。 

[electron-react-boilerplate](https://github.com/chentsulin/electron-react-boilerplate) is a good starting point if you want hot loading webpack setup for Electron and React based development. [electron-compile](https://github.com/electron/electron-compile) skips webpack entirely and can be a lighter alternative for compiling JavaScript and CSS for Electron. Using [the official quick start for Electron](https://github.com/electron/electron-quick-start) is one way.

若想基于开发环境为Electron和React设置webpack的热加载，那么[electron-react-boilerplate](https://github.com/chentsulin/electron-react-boilerplate) 是一个好的出发点。[electron-compile](https://github.com/electron/electron-compile)完全忽略了webpack，且是一个为Electron编译JavaScript和CSS更轻量的选择。
{pagebreak}

## 结论（Conclusion）

Webpack supports targets beyond the web. Based on this you can say name webpack is an understatement considering its capabilities.

Webpack支持除了web外的其他宿主目标。基于此，你可以说webpack的命名并不能完整地表述其功能

To recap:

* Webpack's output target can be controlled through the `target` field. It defaults to `web`, but accepts other options too.
* Webpack can target the desktop, Node, and web workers in addition to its web target.
* The Node targets come in handy if especially in Server Side Rendering setups.

本章回顾：

* Webpack的输出目标可以通过`target`字段进行控制。其默认被设为`web`，但也接受其他选项。
* 除了web宿主外，Webpack还可将宿主环境设置为桌面，Node和web workers。
* Node宿主十分便利由于是在服务端渲染配置。

You'll learn how to bundle libraries in the next chapter.

你将会在下章了解如何打包库。
