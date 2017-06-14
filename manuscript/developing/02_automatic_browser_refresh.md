# Automatic Browser Refresh
# 自动刷新浏览器

Tools, such as [LiveReload](http://livereload.com/) or [Browsersync](http://www.browsersync.io/), allow to refresh the browser as you develop the application and avoid a refresh for CSS changes. It's possible to setup Browsersync to work with webpack through [browser-sync-webpack-plugin](https://www.npmjs.com/package/browser-sync-webpack-plugin), but webpack has more tricks in store.

诸如[LiveReload](http://livereload.com/)或者[Browsersync](http://www.browsersync.io/)这样的工具，让我们可以在开发应用时自动刷新浏览器，甚至可以在更改CSS不用后刷新浏览器就实现更新。通过[browser-sync-webpack-plugin](https://www.npmjs.com/package/browser-sync-webpack-plugin)来，将Browsersync与webpack一起使用也是可行的，但是webpack有更多的特技。

## Webpack `watch` Mode and *webpack-dev-server*
## Webpack `watch`模式和*webpack-dev-server*

A good first step towards a better development environment is to use webpack in its **watch** mode. You can activate it through `webpack --watch`. Once enabled, it detects changes made to your files and recompiles automatically. *webpack-dev-server* (WDS) builds on top of the watch mode and goes even further.

如果想实现一个更好地开发环境，在**watch**模式下使用webpack将是一个很好的起步。你可以通过`webpack --watch`命令来激活这个模式。一旦启用，它将会监听文件的所有改动并重新进行编译。

WDS is a development server running **in-memory**, meaning the bundle contents aren't written out to files, but stored in memory. This is an important distinction when trying to debug code and styles.

WDS是一个运行在内存中的开发服务器，这意味着代码包的内容不会写入到文件中，而是保存在内存中。在尝试调试代码和样式时，这是一个很重要的改进。

By default WDS refreshes content automatically in the browser while you develop your application so you don't have to do it yourself. However it also supports an advanced webpack feature, **Hot Module Replacement** (HMR).

在你开发你的应用时，WDS默认将会自动在浏览器中刷新内容，因此你不必自己实现它。然而，它还支持webpack更高级的特性，**Hot Module Replacement** (HMR)。

HMR allows patching the browser state without a full refresh making it particularly handy with libraries like React where a refresh blows away the application state. The *Hot Module Replacement* appendix covers the feature in detail.

HWR可以在不用正真刷新浏览器的情况下以打补丁方式更新状态，这使得其在使用诸如React这样的库时特别便利，因为一个刷新将丢失应用的状态。附录*Hot Module Replacement*更详细的涵盖了这个特性。

WDS provides an interface that makes it possible to patch code on the fly, however for this to work effectively, you have to implement this interface for the client-side code. It's trivial for something like CSS because it's stateless, but the problem is harder with JavaScript frameworks and libraries.

WDS提供了一个接口，使得其可以在运行中来修补代码，然而为了使其更高效的工作，你需要为客户端代码实现这个接口。这对其它的如CSS来说并不重要，因为它是无状态的，但是在JavaScript的库和框架中，问题就变得严重了。

## Emitting Files from WDS
## 从WDS中生成文件

Even though it's good that WDS operates in-memory by default for performance reasons, sometimes it can be good to emit files to the file system. This applies in particular, if you are integrating with another server that expects to find the files. [webpack-disk-plugin](https://www.npmjs.com/package/webpack-disk-plugin), [write-file-webpack-plugin](https://www.npmjs.com/package/write-file-webpack-plugin), and more specifically [html-webpack-harddisk-plugin](https://www.npmjs.com/package/html-webpack-harddisk-plugin) can achieve this.

尽管为了性能的原因，WDS默认运行在内存中是很好的，有时，生成文件却更好。如果你集成了其它服务，这些服务期望能够找到这些文件，那么该方法就特别有用了。

W> You should use WDS strictly for development. If you want to host your application, consider other standard solutions, such as Apache or Nginx.

W> 你应该严格遵守仅在开发环境使用WDS。如果你希望部署你的应用，请考虑其他的标准方案，如Apache或者Nginx。

## Getting Started with WDS
## 开始设置WDS

To get started with WDS, install it first:

要使用WDS，首先需要安装它：

```bash
npm install webpack-dev-server --save-dev
```

As before, this command generates a command below the `npm bin` directory and you could run *webpack-dev-server* from there. After running the WDS, you have a development server running at `http://localhost:8080`. Automatic browser refresh is in place now, although at a basic level.

<<<<<<< HEAD
和之前一样，该命令在`npm bin`目录下生成了一个命令，你可以从那里运行*webpack-dev-server*。在启动WDS后，你就有了一个开发服务器，运行在`http://localhost:8080`上。尽管只是一个基础版本，自动刷新现在已经OK了。

W> If you are using an IDE, consider enabling **save write** from its settings. This way WDS is able to detect changes made to the files correctly.
=======
{pagebreak}
>>>>>>> d199bf6f6ac7a9196a6595dc074425c87047c3e0

W> 如果你使用的是集成开发环境，考虑在设置中启用**save write(自动保存)**。如此，WDS就可以正确的监听文件的更改。

## Attaching WDS to the Project
## 在项目中使用WDS

To integrate WDS to the project, define an npm script for launching it. To follow npm conventions, call it as *start*. To tell the targets apart, pass information about the environment to webpack configuration so you can specialize as needed:

为了将WDS集成到项目中，你应该定义一个npm脚本来启动它。为了遵循npm的约定，你可以称之为*start*。为了区别不够的构建目标，你应该想webpack配置传入环境的信息。这可以让你根据需要进行专门的配置：

**package.json**

```json
"scripts": {
leanpub-start-insert
  "start": "webpack-dev-server --env development",
  "build": "webpack --env production"
leanpub-end-insert
leanpub-start-delete
  "build": "webpack"
leanpub-end-delete
},
```

T> WDS picks up configuration like webpack itself. The same rules apply.

T> WDS可以向webpack那样查找配置。运用的是同样的规则。

If you execute either *npm run start* or *npm start* now, you should see something in the terminal:

如果你现在执行*npm run start*或者*npm start*，你可以在终端中看到如下信息：

```bash
> webpack-dev-server --env development

Project is running at http://localhost:8080/
webpack output is served from /
Hash: c1b0a0508a91f4b1ac74
Version: webpack 2.2.1
Time: 757ms
     Asset       Size  Chunks                    Chunk Names
    app.js     314 kB       0  [emitted]  [big]  app
index.html  180 bytes          [emitted]
chunk    {0} app.js (app) 300 kB [entry] [rendered]
...
webpack: bundle is now VALID.
```

{pagebreak}

The server is running and if you open `http://localhost:8080/` at your browser, you should see something familiar:

服务已经运行了，如果你在浏览器中访问`http://localhost:8080/`，你可以看到如下熟悉的东西：

![Hello world](images/hello_01.png)

If you try modifying the code, you should see output in your terminal. The browser should also perform a hard refresh on change.

如果你试图修改这个代码，你应该会在终端中看到输出。浏览器也会对更改执行一次刷新。

WDS tries to run in another port in case the default one is being used. The terminal output tells you where it ends up running. You can debug the situation with a command like `netstat -na | grep 8080`. If something is running on the port 8080, it should display a message on Unix.

如果默认的端口被占用了，WDS是尝试在其它端口运行。终端的输出将会告诉你它最终运行的端口。你可以通过如`netstat -na | grep 8080`的命令来调试这个问题。如果有其它程序运行在该端口上，在Unix系统中终端将会展示出信息。

## Verifying that `--env` Works
## 验证`--env`的效果

Webpack configuration is able to receive the value of `--env` if the configuration is defined within a function. To check that the correct environment is passed, adjust the configuration as follows:

如果将webpack配置定义在一个函数中，就可以接受`--env`的值。为了能够正确的检测所传入的环境参数，如下调整配置：

**webpack.config.js**

```javascript
leanpub-start-delete
module.exports = {
  // Entries have to resolve to files! It relies on Node.js
  // convention by default so if a directory contains *index.js*,
  // it resolves to that.
leanpub-end-delete
leanpub-start-insert
const commonConfig = {
leanpub-end-insert
  ...
};

leanpub-start-insert
module.exports = (env) => {
  console.log('env', env);

  return commonConfig;
};
leanpub-end-insert
```

If you run the npm commands now, you should see a different terminal output depending on which one you trigger:

如果你现在执行如下的npm命令，你将在终端看到不同的输出，而这取决于你触发的是哪一个：

```bash
> webpack-dev-server --env development
env development
...
```

T> The result could be verified also by using the `DefinePlugin` to write it to the client code. The *Environment Variables* chapter discusses how to achieve this.

T> 通过使用`DefinePlugin`将其写入客户端代码，也可以验证该结果。*Environment Variables*章节讨论了如何实现这个。

### Understanding `--env`
### 理解`--env`

Even though `--env` allows to pass strings to the configuration, it can do a bit more. Consider the following example:

`--env`不仅可以将字符串传给配置，它还可以实现更多。思考如下的例子：

**package.json**

```json
"scripts": {
  "start": "webpack-dev-server --env development",
  "build": "webpack --env.target production"
},
```

Instead of a string, you should receive an object `{ target: 'production' }` at configuration now. You could pass more key-value pairs, and they would go to the `env` object. If you set `--env foo` while setting `--env.target`, the string overrides the object.

相比于一个字符创，你也可以在配置中接受一个对象`{ target: 'production' }`。你可以传入更多键值对，他们都会并入到`env`对象中。如果你已经设置了`--env.target`，再设置`--env foo`，字符串将会覆盖这个对象。

T> Webpack relies on yargs underneath. To understand the dot notation in greater detail, see [yargs documentation](http://yargs.js.org/docs/#parsing-tricks-dot-notation).

T> webpack在底层依赖yarg。要详细理解这个点语法，可以查看[yargs documentation](http://yargs.js.org/docs/#parsing-tricks-dot-notation)

W> Webpack 2 changed argument behavior compared to webpack 1. You are not allowed to pass custom parameters through the CLI anymore. Instead, it's better to go through the `--env` mechanism if you need to do this.

W> 相比于webpack1，webpack2改变了参数的行为。你不允许通过CLI传入自定义的参数。如果你需要这样做，最好是通过`--env`的机制来实现。

## Configuring WDS Through Webpack Configuration
## 通过webpack的配置来配置WDS

To customize WDS functionality it's possible to define a `devServer` field at webpack configuration. You can set most of these options through the CLI as well, but managing them through webpack is a good approach.

可以通过在webpack的配置中定义一个`devServer`字段来自定义WDS的功能。你也可以通过CLI来设置大多数的选项，但是通过webpack来管理它们是一个较好的办法。

Enable additional functionality as below:

如下启用额外的功能：

**webpack.config.js**

```javascript
...

leanpub-start-insert
const productionConfig = () => commonConfig;

const developmentConfig = () => {
  const config = {
    devServer: {
      // Enable history API fallback so HTML5 History API based
      // routing works. Good for complex setups.
      historyApiFallback: true,

      // Display only errors to reduce the amount of output.
      stats: 'errors-only',

      // Parse host and port from env to allow customization.
      //
      // If you use Docker, Vagrant or Cloud9, set
      // host: options.host || '0.0.0.0';
      //
      // 0.0.0.0 is available to all network devices
      // unlike default `localhost`.
      host: process.env.HOST, // Defaults to `localhost`
      port: process.env.PORT, // Defaults to 8080
    },
  };

  return Object.assign(
    {},
    commonConfig,
    config
  );
};
leanpub-end-insert

module.exports = (env) => {
leanpub-start-delete
  console.log('env', env);

  return commonConfig;
leanpub-end-delete
leanpub-start-insert
  if (env === 'production') {
    return productionConfig();
  }

  return developmentConfig();
leanpub-end-insert
};
```

After this change, you can configure the server host and port options through environment parameters. The merging portion of the code (`Object.assign`) is beginning to look knotty and it will be fixed in the *Splitting Configuration* chapter.

如此更改之后，你可以你可以通过环境变量参数来配置服务的主机和端口。在合并的时候使用的代码(`Object.assign`)看起不是很好理解，这个将在*Splitting Configuration*被解决。

If you access through `http://localhost:8080/webpack-dev-server/`, WDS provides status information at the top. If your application relies on WebSockets and you use WDS proxying, you need to use this particular url as otherwise WDS logic interferes.

如果你现在访问`http://localhost:8080/webpack-dev-server/`，WDS在顶端提供了状态信息。如果你的应用基于WebSocket，或者你使用了WDS代理，你需要使用这个特定的URL以防止WDS其它逻辑的干扰。

![Status information](images/status-information.png)

T> [dotenv](https://www.npmjs.com/package/dotenv) allows you to define environment variables through a *.env* file. *dotenv* allows you to control the host and port setting of the setup quickly.

T> [dotenv](https://www.npmjs.com/package/dotenv)让你可以通过一个*.env*文件来定义环境变量。 *dotenv*可以让你快速的控制主机和端口的设置。

## Enabling Hot Module Replacement
## 启用Hot Module Replacement

Hot Module Replacement is one of those features that sets webpack apart. Implementing it requires additional effort on both server and client-side. The *Hot Module Replacement* appendix discusses the topic in greater detail. If you want to integrate HMR to your project, give it a look. It won't be needed to complete the tutorial, though.

Hot Module Replacement是将webpack同其它工具分开的一个特性。实现它，需要在服务端和客户端都做一些额外的工作。*Hot Module Replacement*附录详细讨论了这个问题。如果你希望将HWR集成到项目中，可以看一下。尽管，在本书的项目中，他并不是必须的。

## Accessing the Development Server from Network
## 通过网络来访问开发服务器

It's possible to customize host and port settings through the environment in the setup (i.e., `export PORT=3000` on Unix or `SET PORT=3000` on Windows). The default settings are enough on most platforms.

可以通过在环境中如此设置(i.e., `export PORT=3000` on Unix or `SET PORT=3000` on Windows)，来对主机和端口进行自定义的设置。默认的设置对大多数平台来说已经足够了。

To access your server, you need to figure out the ip of your machine. On Unix, this can be achieved using `ifconfig | grep inet`. On Windows, `ipconfig` can be utilized. An npm package, such as [node-ip](https://www.npmjs.com/package/node-ip) come in handy as well. Especially on Windows, you need to set your `HOST` to match your ip to make it accessible.

为了能够访问服务，你需要查找机器的IP。在Unix上，可以使用`ifconfig | grep inet`。在Window上，可以使用`ipconfig`。有一些npm包，如[node-ip](https://www.npmjs.com/package/node-ip)特别好用。特别是在Window上，你需要将你的`HOST`设置与ip匹配，才能够访问。

## Making It Faster to Develop Configuration
## 更快的开发配置

WDS will handle restarting the server when you change a bundled file, but what about when you edit the webpack config? Restarting the development server each time you make a change tends to get boring after a while. This can be automated as [discussed in GitHub](https://github.com/webpack/webpack-dev-server/issues/440#issuecomment-205757892) by using [nodemon](https://www.npmjs.com/package/nodemon) monitoring tool.

当你更改了一个已经被打包过的文件后，WDS将会重新启动服务，但是当你编辑webpack配置文件呢？每次更改配置文件后都需要重新启动，会让你感到烦恼。这个问题可以通过[nodemon](https://www.npmjs.com/package/nodemon)管理工具如在[GitHub上讨论的那样](https://github.com/webpack/webpack-dev-server/issues/440#issuecomment-205757892)解决。

To get it to work, you have to install it first through `npm install nodemon --save-dev`. After that, you can make it watch webpack config and restart WDS on change. Here's the script if you want to give it a go:

为了使其能够失效，首先需要安装它：`npm install nodemon --save-dev`。然后，你可以用它来监听webpack的配置，并在更改后重启WDS。如下是一段可以使用的代码：

**package.json**

```json
"scripts": {
  "start": "nodemon --watch webpack.config.js --exec \"webpack-dev-server --env development\"",
  "build": "webpack --env production"
},
```

It's possible WDS [will support the functionality](https://github.com/webpack/webpack/issues/3153) itself in the future. If you want to make it reload itself on change, you should implement this workaround for now.

WDS在未来可能会[支持这个特性](https://github.com/webpack/webpack/issues/3153)。如果你现在就希望在更改后能够自重启，你需要自己实现这个。

{pagebreak}

## Polling Instead of Watching Files
## 轮询而不是监听文件

Sometimes the file watching setup provided by WDS won't work on your system. It can be problematic on older versions of Windows, Ubuntu, Vagrant, and Docker. Enabling polling is a good option then:

有时候，WDS提供的文件监听的设置在你的系统中不能生效。这在老版本的Window，Ubuntu，Vagrant，和Docker上会特别麻烦。这时使用轮询也是一个好办法：

**webpack.config.js**

```javascript
const developmentConfig = merge([
leanpub-start-insert
  {
    devServer: {
      watchOptions: {
        // Delay the rebuild after the first change
        aggregateTimeout: 300,

        // Poll using interval (in ms, accepts boolean too)
        poll: 1000,
      },
    },
    plugins: [
      // Ignore node_modules so CPU usage with poll
      // watching drops significantly.
      new webpack.WatchIgnorePlugin([
        path.join(__dirname, 'node_modules')
      ]),
    ]
leanpub-end-insert
  },
  ...
]);
```

The setup is more resource intensive than the default but it's worth trying out.

这个设置比默认的更消耗资源，但也值得尝试。

T> There are more details in *webpack-dev-server* issue [#155](https://github.com/webpack/webpack-dev-server/issues/155).

T> 更多关于*webpack-dev-server*的问题信息可见[#155](https://github.com/webpack/webpack-dev-server/issues/155)。

## Alternate Ways to Use *webpack-dev-server*
## 其他使用*webpack-dev-server*的方法

You could have passed the WDS options through a terminal. It's clearer to manage the options within webpack configuration as that helps to keep *package.json* nice and tidy. It's also easier to understand what's going on as you don't need to dig out the answers from the webpack source.

你可以通过终端向WDS传入选项。在webpack配置中来管理选项更加清晰，而且有助于保持*package.json*文件的整洁。也能够更容易的理解所发生的事情，而不必从webpack的源码中寻找答案。

Alternately, you could have set up an Express server and use a middleware. There are a couple of options:

其它的可选方案，你可以设置一个Express服务并使用一个中间件。下面是一个选择：

* [The official WDS middleware](https://webpack.js.org/guides/development/#webpack-dev-middleware)
* [webpack-hot-middleware](https://www.npmjs.com/package/webpack-hot-middleware)
* [webpack-universal-middleware](https://www.npmjs.com/package/webpack-universal-middleware)

There's also a [Node.js API](https://webpack.js.org/configuration/dev-server/) if you want more control and flexibility.

如果你希望更多灵活性和可控性，还有一个[Node.js API](https://webpack.js.org/configuration/dev-server/)。

W> There are [slight differences](https://github.com/webpack/webpack-dev-server/issues/106) between the CLI and the Node API.

W> 在CLI和Node API之间存在着[细微的差异](https://github.com/webpack/webpack-dev-server/issues/106)

## Other Features of *webpack-dev-server*
## *webpack-dev-server*的其他特性

WDS provides functionality beyond what was covered above. There are a couple of relevant fields that you should be aware of:

WDS提供了不止上述的功能。下面是一些你需要知晓的相关知识：

* `devServer.contentBase` - Assuming you don't generate *index.html* dynamically and prefer to maintain it yourself in a specific directory, you need to point WDS to it. `contentBase` accepts either a path (e.g., `'build'`) or an array of paths (e.g., `['build', 'images']`). This defaults to the project root.
* `devServer.contentBase` - 假设你没有动态生成*index.html*文件，而更喜欢自己将其维护在一个特定的目录中，你需要向WDS指明它。`contentBase`可以接受一个路径(e.g., `'build'`)或者一个路径数组(e.g., `['build', 'images']`)。默认为项目的根目录。

* `devServer.proxy` - If you are using multiple servers, you have to proxy WDS to them. The proxy setting accepts an object of proxy mappings (e.g., `{ '/api': 'http://localhost:3000/api' }`) that resolve matching queries to another server. Proxy settings are disabled by default.
* `devServer.proxy` -  如果你使用多个服务器，你需要为他们代理WDS。代理的设置接受一个代码映射的对象，如`{ '/api': 'http://localhost:3000/api' }`将匹配的查询处理为另外一个服务。代码默认是禁用的。

* `devServer.headers` - Attach custom headers to your requests here.
* `devServer.headers` - 向请求中添加额外的头信息

T> [The official documentation](https://webpack.js.org/configuration/dev-server/) covers more options.
T> [官方文档](https://webpack.js.org/configuration/dev-server/)涵盖了更多信息。

## Development Plugins
## 开发可用的插件

The webpack plugin ecosystem is diverse and there are a lot of plugins that can help specifically with development:
webpack插件生态系统是多样的，有很多插件可以帮助开发人员：

* [case-sensitive-paths-webpack-plugin](https://www.npmjs.com/package/case-sensitive-paths-webpack-plugin) can be handy when you are developing on a case-insensitive environments like macOS or Windows but using case-sensitive environment like Linux for production.
* [case-sensitive-paths-webpack-plugin](https://www.npmjs.com/package/case-sensitive-paths-webpack-plugin)当你在一个大小写不敏感的环境开发时，如macOS或者Windows，但却使用了大小写敏感的生产环境时，如Linux，这个插件将很有用。

* [npm-install-webpack-plugin](https://www.npmjs.com/package/npm-install-webpack-plugin) allows webpack to install and wire the installed packages with your *package.json* as you import new packages to your project.
* [npm-install-webpack-plugin](https://www.npmjs.com/package/npm-install-webpack-plugin)可以让webpack在你引入一个新的代码包时，安装并将已经安装的代码包信息写入到你的*package.json*文件中。

* [system-bell-webpack-plugin](https://www.npmjs.com/package/system-bell-webpack-plugin) rings the system bell on failure instead of letting webpack fail silently.
* [system-bell-webpack-plugin](https://www.npmjs.com/package/system-bell-webpack-plugin)让webpack在失败时发出响铃，而不是保持沉默。

* [friendly-errors-webpack-plugin](https://www.npmjs.com/package/friendly-errors-webpack-plugin) improves on error reporting of webpack. It captures common errors and displays them in a friendlier manner.
<<<<<<< HEAD
* [friendly-errors-webpack-plugin](https://www.npmjs.com/package/friendly-errors-webpack-plugin)改进了webpack的错误报告。它捕获普通的错误，并以有好的方式展示出来。

* [nyan-progress-webpack-plugin](https://www.npmjs.com/package/nyan-progress-webpack-plugin) can be used to get tidier output during the build process. Take care if you are using Continuous Integration (CI) systems like Travis, though, as they can clobber the output. Webpack provides `ProgressPlugin` for the same purpose. No nyan there, though.

=======
* [nyan-progress-webpack-plugin](https://www.npmjs.com/package/nyan-progress-webpack-plugin) can be used to get tidier output during the build process. Take care if you are using Continuous Integration (CI) systems like Travis as they can clobber the output. Webpack provides `ProgressPlugin` for the same purpose. No nyan there, though.
>>>>>>> d199bf6f6ac7a9196a6595dc074425c87047c3e0
* [react-dev-utils](https://www.npmjs.com/package/react-dev-utils) contains webpack utilities developed for [Create React App](https://www.npmjs.com/package/create-react-app). Despite its name, they can find use beyond React.
* [react-dev-utils](https://www.npmjs.com/package/react-dev-utils)包含有助于[Create React App](https://www.npmjs.com/package/create-react-app)开发的webpack实用程序。尽管它的名字中有React，它们在React之外也大有用处。

* [webpack-dashboard](https://www.npmjs.com/package/webpack-dashboard) gives an entire terminal based dashboard over the standard webpack output. If you prefer clear visual output, this one comes in handy.
* [webpack-dashboard](https://www.npmjs.com/package/webpack-dashboard)通过标准的webpack输出提供整个基于终端的仪表板。 如果你喜欢清晰的视觉输出，这个可以派上用场。

In addition to plugins like these, it can be worth your while to set up linting to enforce coding standards. The *Linting JavaScript* chapter digs into that topic in detail.

除了像这些插件外，花费你的一点儿时间来设置一个语法检测，进而强制代码的标准化也是很值得的。*Linting JavaScript*章节详细的探讨了这个问题。

## Conclusion
## 小结

WDS complements webpack and makes it more friendly by developers by providing development oriented functionality.

WDS补充了Webpack，并通过提供面向开发的功能使其对开发人员更加友好。

To recap:

回顾：

* Webpack's `watch` mode is the first step towards a better development experience. You can have webpack compile bundles as you edit your source.
* 使用webpack的`watch`模式是迈向一个更好地开发体验的第一步。你可以使webpack在你编辑代码后就重新编译代码包。

* Webpack's `--env` parameter allows you to control configuration target through terminal. You receive the passed `env` through a function interface.
* webpack的`--env`参数让你可以通过终端来控制配置的构建目标。你可以通过一个函数接口来接受这个传入的`env`参数。

* WDS can refresh the browser on change. It also implements **Hot Module Replacement**.
* WDS可以在更改后自动刷新浏览器。它还实现了**Hot Module Replacement**。

* The default WDS setup can be problematic on certain systems. For this reason, more resource intensive polling is an alternative.
* 在特定的系统中，默认的WDS设置可能比较麻烦。因此，更多资源消耗的轮询是一种替代方案。

* WDS can be integrated to an existing Node server using a middleware. This gives you more control than relying on the command line interface.
* WDS可以使用一个中间件被集成到一个已存在的Node服务中。相比于命令行接口，给你提供了更多可控性。

* WDS does far more than refreshing and HMR. For example proxying allows you to connect it with other servers.
* WDS可以做的远不止自动刷新和HMR。例如，代理可以让你将其同其它服务连接起来。

In the next chapter, you make it harder to make mistakes by introducing JavaScript linting to the project.

下一章节，你通过在项目中引入JavaScript语法检测，从而更难以犯错。
