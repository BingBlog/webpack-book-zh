# 分析构建数据（Analyzing Build Statistics）

Analyzing build statistics is a good step towards understanding webpack better. Visualizing them helps you to understand the composition of your bundles.

分析构建数据对于更好理解webpack来说是很好的方法。将他们可视化能够帮助你了解你代码包的构成。

## 配置Webpack（Configuring Webpack）

To get suitable output, you need to do a couple of tweaks to the configuration. Most importantly, you have to enable two flags:

为了得到合适的输出，你需要调整一系列配置。最重要的是，你必须支持两个标志：

* `--profile` to capture timing-related information. This is optional, but good to set.
* `--json` to make webpack output statistics.

* `--profile`用于捕获时间相关的信息。可选项，但最好还是设置。
* `--json`设置webpack输出数据。

Here's the line of code you need to pipe the output to a file:

这行代码需要将其输出到一个文件上：

**package.json**

```json
"scripts": {
leanpub-start-insert
  "stats": "webpack --env production --profile --json > stats.json",
leanpub-end-insert
  ...
},
```

The above is the basic setup you need, regardless of your webpack configuration. Execute `npm run stats` now. After a while you should find *stats.json* at your project root. This file can be pushed through a variety of tools to understand better what's going on.

无论的你webpack配置如何，上面的代码是一个你需要的基础配置。现在执行`npm run stats`。过一会你应该会在你的项目根路径发现*stats.json*文件。这一文件可以通过一系列工具来推行以更好了解发生了什么。

W> Given you piggyback on the production target in the current setup, this process cleans the build directory! If you want to avoid that, set up a separate target where you don't clean.

W> 考虑到当前配置在生产环境上，这一方式会清理构建目录。如果你想要避免那样，单独配置一个不需要进行清理的目标路径。

### Node API

Stats can be captured through Node. Since stats can contain errors, so it's a good idea to handle that case separately:

统计数据能够通过Node来捕获。由于数据当中包含错误，因此最好将其单独进行处理：

```javascript
const webpack = require('webpack');
const config = require('./webpack.config.js')('production');

webpack(config, (err, stats) => {
  if (err) {
    return console.error(err);
  }

  if (stats.hasErrors()) {
    return console.error(stats.toString('errors-only'));
  }

  console.log(stats);
});
```

This technique can be valuable if you want to do further processing on stats although often the other solutions are enough.

如果想要更进一步处理数据，那这一方案十分具有价值。尽管其他方案已经能够满足要求。

### `StatsWebpackPlugin` and `WebpackStatsPlugin`

If you want to manage stats through a plugin, check out [stats-webpack-plugin](https://www.npmjs.com/package/stats-webpack-plugin). It gives you a bit more control over the output. You can use it to exclude certain dependencies from the output.

如果想通过插件来管理数据，查看[stats-webpack-plugin](https://www.npmjs.com/package/stats-webpack-plugin)。其可以更好控制输出。可以通过它来将具体的依赖从输出总剔除。

[webpack-stats-plugin](https://www.npmjs.com/package/webpack-stats-plugin) is another option. It allows you to transform the data before outputting it.

[webpack-stats-plugin](https://www.npmjs.com/package/webpack-stats-plugin) 是另一个可选项。其允许你在输出前转化数据。

## 可用分析工具（Available Analysis Tools）

Even though having a look at the file itself gives you idea of what's going on, often it's preferable to use a particular tool for that. Consider the following.

即使查看文件能够让你了解发生了什么，通常更倾向于使用特殊工具来处理。考虑如下工具。

### 官方分析工具（The Official Analyse Tool）

![The Official Analyse Tool](https://raw.githubusercontent.com/survivejs-translations/webpack-book-zh/master/manuscript/images/analyse.png)

[The official analyse tool](https://github.com/webpack/analyse) gives you recommendations and a good idea of your application's dependency graph. It can be run locally as well.

[官方分析工具](https://github.com/webpack/analyse) 提供给你一些建议以及很好的应用程序依赖图。其也能够在本地运行。

### Webpack Visualizer

![Webpack Visualizer](https://raw.githubusercontent.com/survivejs-translations/webpack-book-zh/master/manuscript/images/webpack-visualizer.png)

[Webpack Visualizer](https://chrisbateman.github.io/webpack-visualizer/) provides a pie chart showing your bundle composition allowing to understand which dependencies contribute to the size of the overall result.

[Webpack Visualizer](https://chrisbateman.github.io/webpack-visualizer/) 提供了一个显示你代码包构成的饼图以让你了解哪些依赖影响了整体的体积大小。

### `DuplicatePackageCheckerPlugin`

[duplicate-package-checker-webpack-plugin](https://www.npmjs.com/package/duplicate-package-checker-webpack-plugin) warns you if it finds single package multiple times in your build. This situation can be hard to spot otherwise.

[duplicate-package-checker-webpack-plugin](https://www.npmjs.com/package/duplicate-package-checker-webpack-plugin) 将会发出警告，若其在你构建中多次
发现某一独立的包。否则这一问题很难被发现。

### Webpack Chart

![Webpack Chart](https://raw.githubusercontent.com/survivejs-translations/webpack-book-zh/master/manuscript/images/webpack-chart.png)

[Webpack Chart](https://alexkuz.github.io/webpack-chart/) is another similar visualization.

[Webpack Chart](https://alexkuz.github.io/webpack-chart/) 是另一个相似的可视化工具。

### webpack-unused

[webpack-unused](https://www.npmjs.com/package/webpack-unused) prints out unused files and can be used to understand which assets are no longer used and can be removed from the project.

[webpack-unused](https://www.npmjs.com/package/webpack-unused) 指出未被使用的文件，并能够了解哪些资源不再被使用并将其从项目中移除。

### Stellar Webpack

![Stellar Webpack](https://raw.githubusercontent.com/survivejs-translations/webpack-book-zh/master/manuscript/images/stellar-webpack.jpg)

[Stellar Webpack](https://alexkuz.github.io/stellar-webpack/) gives a universe based visualization and allows you to examine your application in a 3D form.

[Stellar Webpack](https://alexkuz.github.io/stellar-webpack/) 提供了一个基于宇宙般的可视化图形，其能够让你在3D模型下检查你的应用。

### webpack-bundle-tracker

[webpack-bundle-tracker](https://www.npmjs.com/package/webpack-bundle-tracker) can capture data while webpack is compiling. It uses JSON for this purpose.

[webpack-bundle-tracker](https://www.npmjs.com/package/webpack-bundle-tracker) 能够在webpack编译时捕捉数据。基于这一目的其使用JSON格式。

### webpack-bundle-analyzer

![webpack-bundle-analyzer](https://raw.githubusercontent.com/survivejs-translations/webpack-book-zh/master/manuscript/images/webpack-bundle-analyzer.jpg)

[webpack-bundle-analyzer](https://www.npmjs.com/package/webpack-bundle-analyzer) provides a zoomable treemap.

[webpack-bundle-analyzer](https://www.npmjs.com/package/webpack-bundle-analyzer) 提供了一个可缩放的树形结构图。

{pagebreak}

### webpack-bundle-size-analyzer

[webpack-bundle-size-analyzer](https://www.npmjs.com/package/webpack-bundle-size-analyzer) gives a text based composition.

[webpack-bundle-size-analyzer](https://www.npmjs.com/package/webpack-bundle-size-analyzer) 提供了一个基于文本的组合。

```bash
$ webpack-bundle-size-analyzer stats.json
react: 93.99 KB (74.9%)
purecss: 15.56 KB (12.4%)
style-loader: 6.99 KB (5.57%)
fbjs: 5.02 KB (4.00%)
object-assign: 1.95 KB (1.55%)
css-loader: 1.47 KB (1.17%)
<self>: 572 B (0.445%)
```

### inspectpack

[inspectpack](https://www.npmjs.com/package/inspectpack) can be used for figuring out specific places of code to improve.

[inspectpack](https://www.npmjs.com/package/inspectpack) 被用于指出代码中具体需要提高的地方。

```bash
$ inspectpack --action=duplicates --bundle=bundle.js
## Summary

* Bundle:
    * Path:                /PATH/TO/bundle.js
    * Bytes (min):         1678533
* Missed Duplicates:
    * Num Unique Files:    116
    * Num Extra Files:     131
    * Extra Bytes (min):   253955
    * Pct of Bundle Size:  15 %
```

{pagebreak}

## 独立工具（Independent Tools）

In addition to tools that work with webpack output, there are a couple that are webpack agnostic and worth a mention.

除了和wepback的输出搭配使用的工具，还有一系列webpack不知道的并值得一提的。

### source-map-explorer

[source-map-explorer](https://www.npmjs.com/package/source-map-explorer) is a tool independent from webpack. It allows you to get insight into your build by using source maps. It gives a treemap based visualization showing what code contributes to the result.

[source-map-explorer](https://www.npmjs.com/package/source-map-explorer) 
是一款独立于webpack的工具。其通过使用source map能够对你的构建结构有深入了解。
其提供了基于树状结构图的可视化以展示构建结果中包含了哪些代码。

### madge

![madge](https://raw.githubusercontent.com/survivejs-translations/webpack-book-zh/master/manuscript/images/madge.png)

[madge](https://www.npmjs.com/package/madge) is another independent tool that can output a graph based on module input. The graph output allows you to understand the dependencies of your project in greater detail.

{pagebreak}

## 结论（Conclusion）

When you are optimizing the size of your bundle output, these tools are invaluable. The official tool has the most functionality, but even a basic visualization can reveal problem spots. You can use the same technique with old projects to understand their composition.

当你在优化你构建包的体积时，这些工具并没有什么价值。官方工具提供最多的功能，但哪怕基本的可视化工具就能将问题暴露。你可以在旧项目中使用相同的技术以了解他们的构成。

To recap:

* Webpack allows you to extract a JSON file containing information about the build. The information can include the build composition and timing.
* The generated information can be analyzed using various tools that give insight into aspects such as the bundle composition.
* Understanding the bundles is the key to insights on how to optimize the overall size, what to load and when. It can also reveal bigger issues, such as redundant data.
* You can find third party tools that don't depend on webpack but are still valuable for analysis.

本章回顾：

* webpack能够将包含构建的相关信息提取到一个JSON文件中。这些信息涵盖了构建结果的组成和时间。
* 可以使用各类工具来分析生成信息，这些工具可以提供深入了解诸如代码包构成等方面。
* 了解代码包的关键在于对如何优化整体体积的深刻理解，加载什么及何时加载。其也能暴露出更大问题，如重复数据。
* 你可以了解那些不依赖于webpack但对分析仍有重要价值的第三方工具。

You'll learn to tune webpack performance in the next chapter.

你将在下一章中了解调整webpack性能。
