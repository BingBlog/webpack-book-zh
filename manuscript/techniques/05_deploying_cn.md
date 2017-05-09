# 部署应用（Deploying Applications）

A project built with webpack can be deployed to a variety of environments. A public project that doesn't rely on a backend can be pushed to GitHub Pages using the *gh-pages* package. Also, there are a variety of webpack plugins that can target other environments, such as S3.

用webpack构建的项目可以被部署到各类环境中。一个无需依赖后端的公开项目可以通过*gh-pages*包推送到GitHub页面（GitHub Pages）上。此外，还有定位于其他环境的各类wepback插件，如S3。

## 部署*gh-pages*（Deploying with *gh-pages*）

[gh-pages](https://www.npmjs.com/package/gh-pages) allows you to host stand-alone applications on GitHub Pages easily. It has to be pointed to a build directory first. It picks up the contents and pushes them to the `gh-pages` branch.

[gh-pages](https://www.npmjs.com/package/gh-pages) 允许你方便地在GitHub页面上架设独立应用。其首先必须指向构建目录。其将获取里面的内容并将它们推送到`gh-pages`分支。

Despite its name, the package works with other services that support hosting from a Git repository as well. But given GitHub is so popular, it can be used to demonstrate the idea. In practice, you would likely have more complicated setup in place that would push the result to other service through a Continuous Integration system.

除了其名字，该包也能够同其他支持从Git存储托管的服务一同使用。但考虑到GitHub是如此流行， 其可用于展示这一概念。在实际中，你可能会有更为复杂的配置以通过持续集成系统来将构建结果推送到其他服务。

### 配置*gh-pages*（Setting Up *gh-pages*）

To get started, execute

首先，执行如下命令

```bash
npm install gh-pages --save-dev
```

{pagebreak}

You are also going to need a script in *package.json*:

你还需要在  *package.json* 添加一段脚本：

**package.json**

```json
"scripts": {
leanpub-start-insert
  "deploy": "gh-pages -d build",
leanpub-end-insert
  ...
},
```

To make the asset paths work on GitHub Pages, `output.publicPath` field has to be adjusted. Otherwise, the asset paths end up pointing at the root, and that doesn't work unless you are hosting behind a domain root (say `survivejs.com`) directly.

为使资源路径能在GitHub页面上生效，`output.publicPath`字段需要调整。否则，资源路径最终将被指向根路径， 而这并不会生效，除非你直接托管到根域名（如`survivejs.com`）。

`publicPath` gives control over the resulting urls you see at *index.html* for instance. If you are hosting your assets on a CDN, this would be the place to tweak.

`publicPath`可以控制你在*index.html*中看到的url。如果你将你的资源托管在CDN上，那么这将需要进行调整。

In this case, it's enough to set it to point the GitHub project as below:

在这一例子中，如下将其指向GitHub项目就足够了：

**webpack.config.js**

```javascript
const productionConfig = merge([
  {
    ...
    output: {
      ...
leanpub-start-insert
      // Tweak this to match your GitHub project name
      publicPath: '/webpack-demo/',
leanpub-end-insert
    },
  },
  ...
]);
```

After building (`npm run build`) and deploying (`npm run deploy`), you should have your application from the `build/` directory hosted through GitHub Pages. You should find it at `https://<name>.github.io/<project>` assuming everything went fine.

在构建（`npm run build`）和部署（`npm run deploy`）后，你应该从`build`目录中看到你通过GitHub页面托管的应用。若一切正常，你应该能够访问`https://<name>.github.io/<project>`。

T> If you need a more elaborate setup, use the Node API that *gh-pages* provides. The default command line tool it provides is enough for basic purposes, though.

T> 如果你需要更多详细配置，使用*gh-pages*提供的Node API。尽管其所提供的默认命令行工具已能够达到基本的目的。

T> GitHub Pages allows you to choose the branch where you deploy. It's possible to use the `master` branch even as it's enough for minimal sites that don't need bundling. You can also point below the *./docs* directory within your `master` branch and maintain your site.

T> GitHub页面允许你选择你要部署的分支。可以考虑使用`master`分支，即使对于那些不需要进行打包的小网站也是如此。你也可以指向`master`分支下的`./docs`目录并维护你的站点。

### 归档旧版本（Archiving Old Versions）

*gh-pages* provides an `add` option for archival purposes. The idea goes as follows:

1. Copy the old version of the site in a temporary directory and remove *archive* directory from it. You can name the archival directory as you want.
2. Clean and build the project.
3. Copy the old version below *build/archive/<version>*
4. Set up a script to call *gh-pages* through Node as below and capture possible errors in the callback:

*gh-pages*提供`add`选项用于归档。其原理基于如下几点：

1. 复制旧版的站点到一个临时目录并从项目中移除*archive*目录。你可以根据需要命名归档目录。
2. 清理并构建项目。
3. 复制就版本到*build/archive/<version>*下。
4. 如下通过Node配置脚本用于调起*gh-pages*并捕获回调中可能出现的错误。

```javascript
ghpages.publish(path.join(__dirname, 'build'), { add: true }, cb);
```

## 部署到其他环境（Deploying to Other Environments）

Even though you can push the problem of deployment outside of webpack, there are a couple of webpack specific utilities that come in handy:

* [webpack-deploy](https://www.npmjs.com/package/webpack-deploy) is a collection of deployment utilities and works even outside of webpack.
* [webpack-s3-sync-plugin](https://www.npmjs.com/package/webpack-s3-sync-plugin) and [webpack-s3-plugin](https://www.npmjs.com/package/webpack-s3-plugin) sync the assets to Amazon S3.
* [ssh-webpack-plugin](https://www.npmjs.com/package/ssh-webpack-plugin) has been designed for deployments over SSH.

尽管你可以将部署的问题推向webpack之外，但webpack仍然提供了一系列方便的完备的插件：

* [webpack-deploy](https://www.npmjs.com/package/webpack-deploy)是一系列部署插件的合集，其甚至能脱离webpack工作。
* [webpack-s3-sync-plugin](https://www.npmjs.com/package/webpack-s3-sync-plugin) 和 [webpack-s3-plugin](https://www.npmjs.com/package/webpack-s3-plugin) 同步资源到Amazon S3服务上。
* [ssh-webpack-plugin](https://www.npmjs.com/package/ssh-webpack-plugin) 被设计通过SSH来部署。

{pagebreak}

## 动态解决`output.publicPath`（Resolving `output.publicPath` Dynamically）

If you don't know `publicPath` beforehand, it's possible to resolve it based on the environment by following these steps:

1. Set `__webpack_public_path__ = window.myDynamicPublicPath;` in the application entry point and resolve it as you see fit.
2. Remove `output.publicPath` setting from your webpack configuration.
3. If you are using ESLint, set it to ignore the global.

如果你之前不了解`publicPath`，那么最好基于当前环境通过以下三个步骤来解决：

1. 在应用入口设置`__webpack_public_path__ = window.myDynamicPublicPath;`并在你觉得合适的时候删掉。
2. 从webpack配置中移除`output.publicPath`。
3. 如果使用ESLint，配置其忽略该全局变量。

**.eslintrc.js**

```javascript
module.exports = {
  ...
  globals: {
    __webpack_public_path__: true
  },
  ...
};
```

When you compile, webpack picks up `__webpack_public_path__` and rewrites it so that it points to webpack logic.

当在编译时，webpack读取`__webpack_public_path__`并重写该值以让他指向webpack的逻辑路径。

## 结论（Conclusion）

Even though webpack isn't a deployment tool, you can find plugins for it.

即使webpack不是一个部署工具，你依然可以寻找插件来解决。

To recap:

* It's possible to handle the problem of deployment outside of webpack. You can achieve this in an npm script for example.
* You can configure webpack's `output.publicPath` dynamically. This technique is valuable if you don't know it compile-time and want to decide it later. This is possible through the `__webpack_public_path__ ` global.

本章回顾：

* 可以将部署问题放在webpack之外。例如你可以通过一段npm脚本来实现。
* 你可以动态地配置webpack的`output.publicPath`。如果你在编译时间并不知道且想在之后才确定其路径，那么这一技术将十分具有价值。这个可以通过`__webpack_public_path__`全局变量处理。
