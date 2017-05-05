# 开始(Getting Started)

Before getting started, make sure you are using a recent version of [Node](http://nodejs.org/). You should use at least the most current LTS (long-term support) version. The configuration of the book has been written Node 6 features in mind. You should have `node` and `npm` commands available at your terminal. [Yarn](https://yarnpkg.com/) is a good alternative to npm and works for the tutorial as well.

在开始前，请确认你目前使用的[Node](http://nodejs.org/)是最新版本。至少，目前使用的也应该是lts最新版本。在命令行中，你应该可以使用`node`和`npm`命令。对于本书的教程，[Yarn](https://yarnpkg.com/)也是一个好的选择。

It's possible to get a more controlled environment by using a solution such as [Docker](https://www.docker.com/), [Vagrant](https://www.vagrantup.com/) or [nvm](https://www.npmjs.com/package/nvm). Vagrant comes with a performance penalty as it relies on a virtual machine. Vagrant is valuable in a team: each developer can have the same environment that is usually close to production.

通过使用[Docker](https://www.docker.com/),[Vagrant](https://www.vagrantup.com/)，[nvm](https://www.npmjs.com/package/nvm)等工具，我们可以构建更加灵活的环境。由于基于虚拟机的原因，Vagrant有一定的性能劣势。但是某种意义上(每一个开发者可以拥有线上开发环境的副本)，Vagrant还是有价值的。


T> The completed configuration is available at [GitHub](https://github.com/survivejs-demos/webpack-demo).

T> 完整的配置文件可以在[Github](https://github.com/survivejs-demos/webpack-demo)上找到

W> If you are using an older version than Node 6, you have to adapt the code or process your webpack configuration through Babel as discussed in the *Loading JavaScript* chapter.

W> 如果你使用的node版本低于6.x，你必须修改你的代码或者使用Babel对配置文件进行处理，详情可看 *Loading JavaScript* 章节。

{pagebreak}

## 搭建项目(Setting Up the Project)

To get a starting point, you should create a directory for the project and set up a *package.json* there. npm uses that to manage project dependencies. Here are the basic commands:

开始之前，你应该为该项目创建一个文件夹并且在里面放入*package.json*文件.npm使用该文件进行项目依赖管理.这里介绍一些基本的命令:

```bash
mkdir webpack-demo
cd webpack-demo
npm init -y # -y generates *package.json*, skip for more control
```

You can tweak the generated *package.json* manually to make further changes to it even though a part of the operations modify the file automatically for you. The official documentation explains [package.json options](https://docs.npmjs.com/files/package.json) in more detail.

即使npm已经帮你对package.json做了一些配置，你仍然可以对自动生成的配置项进行修改。官网对[package.json配置](https://docs.npmjs.com/files/package.json)进行了详细的阐述。

T> You can set those `npm init` defaults at *~/.npmrc*.

T> 在*~/.npmrc*文件中，你可以默认加入`npm init`

T> This is a good place to set up version control using [Git](https://git-scm.com/). You can create a commit per step and tag per chapter, so it's easier to move back and forth if you want.

T> 使用版本管理工具[Git](https://git-scm.com/)是个好习惯。每一章节你都一个创建一个commit和tag，方便你在不同版本间跳转。

## 安装Webpack(Installing Webpack)

Even though webpack can be installed globally (`npm install webpack -g`), it's a good idea to maintain it as a dependency of your project to avoid issues, as then you have control over the exact version you are running. The approach works nicely in **Continuous Integration** (CI) setups as well. A CI system can install your local dependencies, compile your project using them, and then push the result to a server.

虽然Webpack可以全局安装(`npm install webpack -g`)，但最佳实践告诉我们本地安装webpack作为一种好的方法可以避免很多问题，因为我们可以精确的控制我们想要的版本。在**Continuous Integration**章节中，使用该步骤将会非常方便。一个自动化集成系统可以安装你本地依赖，编译你的项目，并把最终编译包推送到服务器上。

{pagebreak}

To add webpack to the project, execute:
给一个项目添加webpack,执行:

```bash
npm install webpack --save-dev # -D if you want to save typing
```

You should see webpack at your *package.json* `devDependencies` section after this. In addition to installing the package locally below the *node_modules* directory, npm also generates an entry for the executable.

执行完上述命令后，你将会在*package.json*文件中的`devDependencies`选项中看到webpack。除了在本地的*node_modules*文件夹中安装了webpack之外，npm也给webpack创建了一个执行入口。

## 执行Webpack(Executing Webpack)

You can display the exact path of the executables using `npm bin`. Most likely it points at *./node_modules/.bin*. Try running webpack from there through the terminal using `node_modules/.bin/webpack` or a similar command.

你可以通过`npm bin`来显示精确的执行路径。通常，会在*./node_modules/.bin*目录下。尝试通过在命令行使用`node_modules/.bin/webpack`去运行webpack。

After running, you should see a version, a link to the command line interface guide and an extensive list of options. Most aren't used in this project, but it's good to know that this tool is packed with functionality if nothing else.

在运行`webpack`后，命令行中将出现版本号、帮助文档的链接以及一些命令选项。它们中大部分在本书中不使用，即便如此，了解一下总是好的。

```bash
webpack-demo $ node_modules/.bin/webpack
No configuration file found and no output filename configured via CLI option.
A configuration file could be named 'webpack.config.js' in the current directory.
Use --help to display the CLI options.
```

To get a quick idea of webpack output, try this:

想迫不及待的看下webpack的输出，尝试下如下步骤：

1. Set up *app/index.js* so that it contains `console.log('Hello world');`.
2. Execute `node_modules/.bin/webpack app/index.js build/index.js`.
3. Examine *build/index.js*. You should see webpack bootstrap code that begins executing the code. Below the bootstrap you should find something familiar.

1. 在app文件夹下新建一个包含`console.log('Hello world')`的index.js文件.
2. 执行`node_modules/.bin/webpack app/index.js build/index.js`.
3. 查看*build/index.js*.

T> You can use `--save` and `--save-dev` to separate application and development dependencies. The former installs and writes to *package.json* `dependencies` field whereas the latter writes to `devDependencies` instead.

T> 你可以使用`--save`和`--save-dev`对线上和开发环境的依赖进行区分。前者把依赖包写入`dependencies`字段，而后者写入到`devDependencies`字段中。

## 目录结构(Directory Structure)

To move further, you can implement a site that loads JavaScript, which you then build using webpack. After you progress a bit, you end up with a directory structure below:

更进一步，你可以尝试自己用webpack去构建一个网站，项目的目录结构大致如下:

* app/
  * index.js
  * component.js
* build/
* package.json
* webpack.config.js

The idea is that you transform *app/* to a bundle below *build/*. To make this possible, you should set up the assets needed and configure webpack through *webpack.config.js*.

如果想把*app*下的文件打包到*build/*文件夹下，我们可以在*webpack.config.js*文件中进行相关配置。

## 开始Coding(Setting Up Assets)

As you never get tired of `Hello world`, you will model a variant of that. Set up a component:

如果你还没有厌烦`Hello world`的话，你可以用它声明一个变量。构建一个组件:

**app/component.js**

```javascript
export default (text = 'Hello world') => {
  const element = document.createElement('div');

  element.innerHTML = text;

  return element;
};
```

{pagebreak}

Next, you are going to need an entry point for the application. It uses `require` against the component and renders it through the DOM:

接下来，你需要为程序添加一个入口文件。它使用require去加载组件，并用dom方法把渲染在页面上:

**app/index.js**

```javascript
import component from './component';

document.body.appendChild(component());
```

## 配置Webpack(Setting Up Webpack Configuration)

You need to tell webpack how to deal with the assets that were set up. For this purpose, you have to develop a *webpack.config.js* file. Webpack and its development server are able to discover this file through a convention.

你需要告诉webpack如何对刚写的代码进行处理。为此，我们需要新建一个*webpack.config.js*文件。按照规范，webpack与webpack server都可以找到这个文件。

To keep things convenient to maintain, you can use your first plugin: [html-webpack-plugin](https://www.npmjs.com/package/html-webpack-plugin). `HtmlWebpackPlugin` generates an *index.html* for the application and adds a `script` tag to load the generated bundle. Install it:

为了便于维护，你可以使用plugin:[html-webpack-plugin](https://www.npmjs.com/package/html-webpack-plugin).`HtmlWebpackPlugin`将新建一个*index.html*文件，并通过script标签去加载打包后的文件。安装它:

```bash
npm install html-webpack-plugin --save-dev
```

At a minimum, it's nice to have at least `entry` and `output` fields in your configuration. Often you see a lot more as you specify how webpack deals with different file types and how it resolves them.

在webpack的配置文件中，最少应该有`entry`和`output`这两个字段。通常，你会看到很多关于webpack针对不同文件类型的处理解析方式。

Entries tell webpack where to start parsing the application. In multi-page applications, you have an entry per page. Or you could have a configuration per entry as discussed later in this chapter.

入口`entry`会告诉webpack从哪里开始做解析。在多页面的程序中，针对每一个页面都会有一个入口文件。或者你可以采用本章节接下来讨论的，给每一个页面做相应的配置。

All output related paths you see in the configuration are resolved against the `output.path` field. If you had an output relation option somewhere and wrote `styles/[name].css`, that would be expanded so that you get `<output.path> + <specific path>`. Example: *~/webpack-demo/build/styles/main.css*.

通过`output.path`字段，你可以对输出文件的路径进行配置。如果你配置了改选项，并在一些地方写了`styles/[name].css`，解析后的路径应为`<output.path>+<specific path>`。例子：*~/webpack-demo/build/styles/main.css*.

{pagebreak}

To illustrate how to connect `entry` and `output` with `HtmlWebpackPlugin`, consider the code below:

为了阐述如何把`entry`、`output`与`HtmlWebpackPlugin`这几个字段关联起来，分析如下代码：

**webpack.config.js**

```javascript
const path = require('path');
const HtmlWebpackPlugin = require('html-webpack-plugin');

const PATHS = {
  app: path.join(__dirname, 'app'),
  build: path.join(__dirname, 'build'),
};

module.exports = {
  // Entries have to resolve to files! They rely on Node
  // convention by default so if a directory contains *index.js*,
  // it resolves to that.
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
};
```

The `entry` path could be given as a relative one using the [context](https://webpack.js.org/configuration/entry-context/#context) field used to configure that lookup. However, given plenty of places expect absolute paths, preferring them over relative paths everywhere avoids confusion.

在使用[context](https://webpack.js.org/configuration/entry-context/#context)后，`entry`字段可以采用相对路径。考虑到大部分场景下都采用绝对路径，为避免混淆，还是采用绝对路径对`entry`字段进行配置。

T> **Trailing commas** are used in the book examples on purpose as it gives cleaner diffs for the code examples. You'll learn to enforce this rule in the *Linting JavaScript* chapter.

T> **Trailing commas** 在本书中的例子中大量使用，主要目的是在做代码对比时可以更清晰。在*Linting JavaScript*章节中，你将会被告诉强制使用这个规则。

T> `[name]` is a placeholder. Placeholders are discussed in detail in the *Adding Hashes to Filenames* chapter, but they are effectively tokens that will be replaced when the string is evaluated. In this case `[name]` will be replaced by the name of the entry - 'app'.

T> `[name]`是占位符。在*Adding Hashes to Filenames*章节中，会对占位符进行详细讨论。在这个例子中，`[name]`将被入口文件名`app`给取代。

If you execute `node_modules/.bin/webpack`, you should see output:

如果你执行`node_modules/.bin/webpack`,你将看到如下输出：

```bash
Hash: 3f76ae042ff0f2d98f35
Version: webpack 2.2.1
Time: 376ms
     Asset       Size  Chunks             Chunk Names
    app.js    3.13 kB       0  [emitted]  app
index.html  180 bytes          [emitted]
   [0] ./app/component.js 148 bytes {0} [built]
   [1] ./app/index.js 78 bytes {0} [built]
Child html-webpack-plugin for "index.html":
       [0] ./~/lodash/lodash.js 540 kB {0} [built]
       [1] (webpack)/buildin/global.js 509 bytes {0} [built]
       [2] (webpack)/buildin/module.js 517 bytes {0} [built]
       [3] ./~/html-webpack-plugin/lib/loader.js!./~/html-webpack-plugin/default_index.ejs 540 bytes {0} [built]
```

The output tells a lot:

输出内容包含很多信息：

* `Hash: 3f76ae042ff0f2d98f35` - The hash of the build. You can use this to invalidate assets through `[hash]` placeholder. Hashing is discussed in detail in the *Adding Hashes to Filenames* chapter.

* `Hash: 3f76ae042ff0f2d98f35` - 输出内容的哈希值。通过`[hash]`占位符，你可以利用这个值使资源文件失效。在随后的*Adding Hashes to Filenames*章节中，会对哈希值进行更加详细的讨论。

* `Version: webpack 2.2.1` - Webpack version.

* `Version: webpack 2.2.1` - Webpack版本号

* `Time: 377ms` - Time it took to execute the build.

* `Time: 377ms` - 执行构建过程所花费的时间。

* `app.js    3.13 kB       0  [emitted]  app` - Name of the generated asset, size, the IDs of the **chunks** into which it's related, status information telling how it was generated, the name of the chunk.

* `app.js    3.13 kB       0  [emitted]  app` - 输出文件的名字，大小，关联chunks文件的id，创建过程的状态信息，chunk的名字。

* `index.html  180 bytes          [emitted]` - Another generated asset that was emitted by the process.

* `index.html  180 bytes          [emitted]` - 被webpack给创建的另外一个资源文件信息。

* `[0] ./app/component.js 148 bytes {0} [built]` - The ID of the entry asset, name, size, entry chunk ID, the way it was generated.

* `[0] ./app/component.js 148 bytes {0} [built]` - 入口资源的ID,名字，大小，入口chunk ID，被创建的方式。

* `Child html-webpack-plugin for "index.html":` - This is plugin-related output. In this case *html-webpack-plugin* is doing the output of its own.

* `Child html-webpack-plugin for "index.html":` - 相关插件输出。在该例子中，展示了*html-webpack-plugin*这个插件的输出。

Examine the output below `build/`. If you look closely, you can see the same IDs within the source. To see the application running, open the `build/index.html` file directly through a browser. On macOS `open ./build/index.html` works.

T> If you want webpack to stop execution on the first error, set `bail: true` option. Setting it kills the entire webpack process. The behavior is desirable if you are building in a CI environment.

T> In addition to a configuration object, webpack accepts an array of configurations. You can also return a `Promise` and eventually `resolve` to a configuration.

## Adding a Build Shortcut

Given executing `node_modules/.bin/webpack` is verbose, you should do something about it. This is where npm and *package.json* can be used for running tasks.

{pagebreak}

Adjust the file as follows:

**package.json**

```json
"scripts": {
  "build": "webpack"
},
```

Run `npm run build` to see the same output as before. This works because npm adds *node_modules/.bin* temporarily to the path. As a result, rather than having to write `"build": "node_modules/.bin/webpack"`, you can do `"build": "webpack"`.

You can execute this kind of scripts through *npm run* and you can use *npm run* anywhere within your project. If you run the command as is, it gives you the listing of available scripts.

T> There are shortcuts like *npm start* and *npm test*. You can run these directly without *npm run* although that works too. For those in a hurry, you can use *npm t* to run your tests.

## `HtmlWebpackPlugin` Extensions

Although you can replace `HtmlWebpackPlugin` template with your own, there are premade ones like [html-webpack-template](https://www.npmjs.com/package/html-webpack-template) or [html-webpack-template-pug](https://www.npmjs.com/package/html-webpack-template-pug).

There are also specific plugins that extend `HtmlWebpackPlugin`'s functionality:

下面列举一些继承了`HtmlWebpackPlugin`插件功能的插件

* [favicons-webpack-plugin](https://www.npmjs.com/package/favicons-webpack-plugin) is able to generate favicons.

* [favicons-webpack-plugin](https://www.npmjs.com/package/favicons-webpack-plugin)还可以创建收藏图标。

* [script-ext-html-webpack-plugin](https://www.npmjs.com/package/script-ext-html-webpack-plugin) gives you more control over script tags and allows you to tune script loading further.

* [script-ext-html-webpack-plugin](https://www.npmjs.com/package/script-ext-html-webpack-plugin)给予你对script标签更多的控制，并允许你调节脚本的加载。

* [multipage-webpack-plugin](https://www.npmjs.com/package/multipage-webpack-plugin) builds on top of *html-webpack-plugin* and makes it easier to manage multi-page configurations.

* [multipage-webpack-plugin](https://www.npmjs.com/package/multipage-webpack-plugin)基于*html-webpack-plugin*构建，在处理多页面配置会比较方便。

* [resource-hints-webpack-plugin](https://www.npmjs.com/package/resource-hints-webpack-plugin) adds [resource hints](https://www.w3.org/TR/resource-hints/) to your HTML files to speed up loading time.

* [resource-hints-webpack-plugin](https://www.npmjs.com/package/resource-hints-webpack-plugin)在页面中加入[resource hints](https://www.w3.org/TR/resource-hints/)，加速加载时间。

* [preload-webpack-plugin](https://www.npmjs.com/package/preload-webpack-plugin) enables `rel=preload` capabilities for scripts. This helps with lazy loading and it combines well with techniques discussed in the *Building* part of this book.

## 结论(Conclusion)

Even though you have managed to get webpack up and running, it does not do that much yet. Developing against it would be painful. Each time you wanted to check out the application, you would have to build it manually using `npm run build` and then refresh the browser. That's where webpack's more advanced features come in.

到目前，我们已经可以利用webpack简单的构建一个应用程序，但在实际开发中，远远不够。每次，在运行程序的时候，你都必须手动执行`npm run build`对程序进行打包，然后刷新浏览器。后面，我们会介绍webpack的其它功能，帮助我们提升开发效率。

To recap:

概述：

* It's a good idea to use a locally installed version of webpack over a globally installed one. This way you can be sure of what version you are using. The local dependency works also in a Continuous Integration environment.

* 本地安装指定版本的webpack是值得推荐的，因为你可以安装你想要的版本。并且，本地依赖包也能很好的在持续继承环境中工作。

* Webpack provides a command line interface. You can use it even without configuration, but then you are limited by the options it provides.

* Webpack 提供了命令行接口。虽然可以不依赖配置文件，但是命令行所提供的功能是有限的。

* To write more complicated setups, you most likely have to write a separate *webpack.config.js* file.

* 为了实现更多的功能，最好在将webpack的配置写入单独的*webpack.config.js*文件。

* `HtmlWebpackPlugin` can be used to generate an HTML entry point to your application. Later in the book, you see how to generate multiple separate pages using. The *Multiple Pages* chapter covers that.

* 利用`HtmlWebpackPlugin`，可以为应用程序创建一个入口html文件。在本书后面的*Multiple Pages*章节，将介绍如何创建多html文件。

* It's handy to use npm *package.json* scripts to manage webpack. You can use it as a light task runner and use system features outside of webpack.

* 使用npm的*package.json*文件管理webpack是非常方便的。可以把它当做轻量的任务管理器，并且可以使用一些webpack之外的系统功能。

In the next chapter you will learn how to improve the developer experience by enabling automatic browser refresh.

下一章，你将会学到如何利用浏览器自动刷新来提升开发效率。
