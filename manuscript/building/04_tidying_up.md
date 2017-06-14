# Tidying Up
# 整理

The current setup doesn't clean the *build* directory between builds. As a result, it keeps on accumulating files as the project changes. Given this can get annoying, you should clean it up in between.

当前的设置并不能在每次构建时清除*build*目录，随着项目的改变，它一直在积累文件。这是在很烦人，你应该在每次构建时进行清除。

Another nice touch would be to include information about the build itself to the generated bundles as a small comment at the top of each file including version information at least.

另外一个很好的设置就是将构建自身的信息以一个很小的注释包含到生成的代码包中的每个文件的顶端，至少要包含版本信息。

## Cleaning the Build Directory
## 清理Build文件夹

This issue can be resolved either by using a webpack plugin or solving it outside of it. You could trigger `rm -rf ./build && webpack` or `rimraf ./build && webpack` in an npm script to keep it cross-platform. A task runner could work for this purpose as well.

该问题可以通过使用一个webpack插件，也可以在其外部解决它。你可以触发`rm -rf ./build && webpack`或者使用npm脚本`rimraf ./build && webpack`来使其支持跨平台。一个任务运行器也可以解决这个问题。

### Setting Up `CleanWebpackPlugin`
### 设置`CleanWebpackPlugin`

Install the [clean-webpack-plugin](https://www.npmjs.com/package/clean-webpack-plugin) first:
首先安装[clean-webpack-plugin](https://www.npmjs.com/package/clean-webpack-plugin)：

```bash
npm install clean-webpack-plugin --save-dev
```

{pagebreak}

Next, you need to define a function to wrap the basic idea. You could use the plugin directly, but this feels like something that could be used across projects, so it makes sense to push it to the library:

接下来，你需要定义一个函数来包含这个基本的想法。你可以直接使用这个插件，但是这些东西感觉是可以跨项目使用的，所以将其放到一个库中也是有意义的：

**webpack.parts.js**

```javascript
...
leanpub-start-insert
const CleanWebpackPlugin = require('clean-webpack-plugin');
leanpub-end-insert

...

leanpub-start-insert
exports.clean = (path) => ({
  plugins: [
    new CleanWebpackPlugin([path]),
  ],
});
leanpub-end-insert
```

Connect it with the project:

**webpack.config.js**

```javascript
const productionConfig = merge([
leanpub-start-insert
  parts.clean(PATHS.build),
leanpub-end-insert
  ...
]);
```

After this change, the `build` directory should remain nice and tidy while building. You can verify this by building the project and making sure no old files remained in the output directory.

如此更改之后，`build`目录将会在构建时保持整洁和整齐。你可以通过构建项目来验证，确保在输出的目录中没有老的文件。

{pagebreak}

## Attaching a Revision to the Build
## 添加版本信息到构建中

Attaching information related to the current build revision to the build files themselves can be used for debugging. [webpack.BannerPlugin](https://webpack.js.org/plugins/banner-plugin/) allows you to achieve this. It can be used in combination with [git-revision-webpack-plugin](https://www.npmjs.com/package/git-revision-webpack-plugin) to generate a small comment at the beginning of the generated files.


将当前版本构建相关的信息添加到构建文件自身可用于调试。[webpack.BannerPlugin](https://webpack.js.org/plugins/banner-plugin/)可以让你实现这个。它可以和[git-revision-webpack-plugin](https://www.npmjs.com/package/git-revision-webpack-plugin)结合起来使用，在生成文件的靠头生成一个小的注释。

### Setting Up `BannerPlugin` and `GitRevisionPlugin`
### 设置`BannerPlugin`和`GitRevisionPlugin`

To get started, install the revision plugin:

首先，安装这个版本插件：

```bash
npm install git-revision-webpack-plugin --save-dev
```

Then define a part to wrap the idea:

然后如下定义一个包含该想法的方法：

**webpack.parts.js**

```javascript
...
leanpub-start-insert
const GitRevisionPlugin = require('git-revision-webpack-plugin');
leanpub-end-insert

...

leanpub-start-insert
exports.attachRevision = () => ({
  plugins: [
    new webpack.BannerPlugin({
      banner: new GitRevisionPlugin().version(),
    }),
  ],
});
leanpub-end-insert
```

{pagebreak}

And connect it to the main configuration:

在将其应用到主配置中：

**webpack.config.js**

```javascript
const productionConfig = merge([
  ...
leanpub-start-insert
  parts.attachRevision(),
leanpub-end-insert
]);
```

If you build the project (`npm run build`), you should notice the built files contain comments like `/*! 0b5bb05 */` or `/*! v1.7.0-9-g5f82fe8 */` in the beginning.

如果你现在构建应用(`npm run build`)，你将注意到构建的文件开头包含了评论，如`/*! 0b5bb05 */` or `/*! v1.7.0-9-g5f82fe8 */`

The output can be customized further by adjusting the banner. You can also pass revision information to the application using `webpack.DefinePlugin`. This technique is discussed in detail in the *Environment Variables* chapter.

通过调整这个标示可以进一步自定义输出。你也可以使用`webpack.DefinePlugin`将版本信息传给应用。该技术在*Environment Variables*相关的章节将被详细讨论。

W> The code expects you run it within a Git repository! Otherwise, you get a `fatal: Not a git repository (or any of the parent directories): .git` error. If you are not using Git, you can replace the banner with other data.

W> 当前的代码希望你将其放到一个Git仓库中！否则，你将得到`fatal: Not a git repository (or any of the parent directories): .git`错误。如果你没有使用Git，你可以用其他数据来替换这个标示。

## Copying Files
## 复制文件

Copying files is another common operation you can handle with webpack. [copy-webpack-plugin](https://www.npmjs.com/package/copy-webpack-plugin) can be handy if you need to bring external files to your build without having webpack pointing at them directly.

复制文件是另外一个可以通过webpack执行的普通的操作。如果你需要将外部文件引入到你的构建中，而不希望webpack直接引用它们，[copy-webpack-plugin](https://www.npmjs.com/package/copy-webpack-plugin)将非常有用。

[cpy-cli](https://www.npmjs.com/package/cpy-cli) is a good option if you want to copy outside of webpack in a cross-platform way. Plugins should be cross-platforms by definition.

如果你希望在webpack之外卷平台复制文件，[cpy-cli](https://www.npmjs.com/package/cpy-cli)是一个很好的选择。插件应该要跨平台的定义。

## Conclusion

## 小结

Often, you work with webpack by identifying a problem and then finding a plugin to tackle it. It's entirely acceptable to solve these types of issues outside of webpack, but webpack can often handle them as well.

通常，您通过确定问题，然后找到一个插件来处理它的方式，与webpack协同工作。解决webpack之外的这些类型的问题是完全可以接受的，但通常Webpack也可以解决这些问题。

To recap:

回顾:

* You can find many small plugins that work as tasks and push webpack closer to a task runner.
* 你可以找到很多小的像任务运行器一样工作的插件，来使得webpack更接近一个任务运行器。

* These tasks include cleaning the build and deployment. The *Deploying Applications* chapter discusses the latter topic in detail.
* 这些任务运行器包括清理构建和部署。*Deploying Applications*章节详细讲述了后者。

* It can be a good idea to include small comments to the production build to tell what version has been deployed. This way you can debug potential issues faster.
* 在生产环境的构建中包含小的注释来告知当前部署的版本是一个很好的注意。如此，你可以很快调试潜在的问题。

* Secondary tasks like these can be performed outside of webpack. If you are using a multi-page setup as discussed in the *Multiple Pages* chapter, this becomes a necessity.
* 其次，像这样的插件可以在webpack之外执行。如果你像*Multiple Pages*章节讨论的那样使用多页设置，这将是必要的。
