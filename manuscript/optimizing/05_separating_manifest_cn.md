# 分离清单（eparating a Manifest）

When webpack writes bundles, it maintains a **manifest** as well. You can find it in the generated *vendor* bundle in this project. The manifest describes what files webpack should load. It's possible to extract it and start loading the files of the project faster instead of having to wait for the *vendor* bundle to be loaded.

当webpack在写代码包，其也维护了一个**清单（manifest）**。你可以在这项目中的生成的*vendor*代码包中找到。该清单描述了webpack应该加载什么样的文件。能够提取和更快地加载项目文件而不是必须先等*依赖库（vendor）*先加载。

If the hashes webpack generates change, then the manifest changes as well. As a result, the contents of the vendor bundle change, and become invalidated. The problem can be eliminated by extracting the manifest to a file of its own or by writing it inline to the *index.html* of the project.

如果webpack生成的哈希值发生了改变，而后该清单也会发生改变。因此，代码依赖包的内容发生改变并失效。这一问题可以通过将该清单提取到一个单独的文件中或把其写入该项目的*index.html*里来解决。

T> To understand how a manifest is generated in detail, [read the technical explanation at Stack Overflow](https://stackoverflow.com/questions/39548175/can-someone-explain-webpacks-commonschunkplugin/39600793).

T> 要了解生成清单的细节，[可以阅读Stack Overflow上的技术解释](https://stackoverflow.com/questions/39548175/can-someone-explain-webpacks-commonschunkplugin/39600793)。

{pagebreak}

## 提取清单（Extracting a Manifest）

Most of the work was done already when `extractBundles` was set up in the *Bundle Splitting* chapter. To extract the manifest, a single change is required to capture the remaining code which contains webpack bootstrap:

大部分的工作已经在*代码包分割*章节中的配置`extractBundles`完成了。为提取清单，一个单独的改变是要求捕获涵盖在webpack启动程序的剩余代码：

**webpack.config.js**

```javascript
const commonConfig = merge([
  ...
  parts.extractBundles([
      {
        ...
      },
leanpub-start-insert
      {
        name: 'manifest',
        minChunks: Infinity,
      },
leanpub-end-insert
  ]),
]);
```

The name `manifest` is used by convention. You can use any other name and it will still work. It's important that the definition is after others, though, as it has to capture what has not been extracted yet. `minChunks` is optional in this case and passing `Infinity` tells webpack **not** to move any modules to the resulting bundle.

名字`manifest`通常是使用惯例。你可以使用其他名字且同样有效。重要的是其定义是在其他选项之后，但是还是不得不捕获未被提取的内容。`minChunks`在这一例子中是个可选项，同时传入`Infinity`告诉webpack**不要**移动任何模块到构建完的代码包中。

{pagebreak}

If you build the project now (`npm run build`), you should see something:

如果你现在构建项目（`npm run build`），你应该看到些内容：

```bash
Hash: 73f8c0d53361c3a81ea6
Version: webpack 2.2.1
Time: 4071ms
                   Asset       Size  Chunks             Chunk Names
         app.801b7672.js  865 bytes    2, 3  [emitted]  app
    ...font.912ec66d.svg     444 kB          [emitted]
    ...font.674f50d2.eot     166 kB          [emitted]
   ...font.fee66e71.woff      98 kB          [emitted]
  ...font.af7ae505.woff2    77.2 kB          [emitted]
       logo.85011118.png      77 kB          [emitted]
           0.e7c9bce9.js  432 bytes    0, 3  [emitted]
      vendor.c4ac6d53.js    23.4 kB    1, 3  [emitted]  vendor
    ...font.b06871f2.ttf     166 kB          [emitted]
leanpub-start-insert
    manifest.95266dc7.js    1.51 kB       3  [emitted]  manifest
leanpub-end-insert
        app.bf4d156d.css    2.54 kB    2, 3  [emitted]  app
       0.e7c9bce9.js.map    2.08 kB    0, 3  [emitted]
  vendor.c4ac6d53.js.map     129 kB    1, 3  [emitted]  vendor
     app.801b7672.js.map    2.34 kB    2, 3  [emitted]  app
    app.bf4d156d.css.map   93 bytes    2, 3  [emitted]  app
leanpub-start-insert
manifest.95266dc7.js.map    5.77 kB       3  [emitted]  manifest
leanpub-end-insert
              index.html  368 bytes          [emitted]
[1Q41] ./app/main.css 41 bytes {2} [built]
[2twT] ./app/index.js 557 bytes {2} [built]
[5W1q] ./~/font-awesome/css/font-awesome.css 41 bytes {2} [built]
...
```

This change gave a separate file that contains the manifest. In the output above it has been marked with `manifest` chunk name. Because the setup is using `HtmlWebpackPlugin`, there is no need to worry about loading the manifest ourselves as the plugin adds a reference to *index.html*.

这一改变独立成为一个涵盖了该清单的文件。在之前的输出，其已被标记上了`manifest`的名字。由于已经在配置中使用了`HtmlWebpackPlugin`，无需担心清单会再自己载入，插件已经将引用添加进了*index.html*。

Plugins, such as [inline-manifest-webpack-plugin](https://www.npmjs.com/package/inline-manifest-webpack-plugin) and [html-webpack-inline-chunk-plugin](https://www.npmjs.com/package/html-webpack-inline-chunk-plugin), [assets-webpack-plugin](https://www.npmjs.com/package/assets-webpack-plugin), work with `HtmlWebpackPlugin` and allow you to write the manifest within *index.html* to avoid a request.

插件，如[inline-manifest-webpack-plugin](https://www.npmjs.com/package/inline-manifest-webpack-plugin) 何 [html-webpack-inline-chunk-plugin](https://www.npmjs.com/package/html-webpack-inline-chunk-plugin), [assets-webpack-plugin](https://www.npmjs.com/package/assets-webpack-plugin)，能够与`HtmlWebpackPlugin`配合并允许你将manifest写进*index.html*以避免一次请求

T> To get a better idea of the manifest contents, comment out `parts.minify()` and examine the resulting manifest. You should see something familiar there.

T> 为了对清单内容有更好的了解，将`parts.minify()`注释并检查结果清单。你应该在这里看到些熟悉的东西。

Try adjusting *app/index.js* and see how the hashes change. This time around it should **not** invalidate the vendor bundle, and only the manifest and app bundle names should become different.

尝试调整*app/index.js*并看看哈希值如何发生改变。这次代码库**不**应该失效了，且仅有清单和应用代码的文件名发生了些不同。

T> To integrate with asset pipelines, you can consider using plugins like [chunk-manifest-webpack-plugin](https://www.npmjs.com/package/chunk-manifest-webpack-plugin), [webpack-manifest-plugin](https://www.npmjs.com/package/webpack-manifest-plugin), [webpack-assets-manifest](https://www.npmjs.com/package/webpack-assets-manifest), or [webpack-rails-manifest-plugin](https://www.npmjs.com/package/webpack-rails-manifest-plugin). These solutions emit JSON that maps the original asset path to the new one.

T> 为了能将资源进行管道化集成，你可以考虑使用插件如[chunk-manifest-webpack-plugin](https://www.npmjs.com/package/chunk-manifest-webpack-plugin)，[webpack-manifest-plugin](https://www.npmjs.com/package/webpack-manifest-plugin)，[webpack-assets-manifest](https://www.npmjs.com/package/webpack-assets-manifest)，或 [webpack-rails-manifest-plugin](https://www.npmjs.com/package/webpack-rails-manifest-plugin)。这些解决方案会发出JSON，将原资源路径映射到新路径上。

T> The build can be improved further by loading popular dependencies, such as React, through a CDN. That would decrease the size of the vendor bundle even further while adding an external dependency on the project. The idea is that if the user has hit the CDN earlier, caching can kick in like here.

T> 通过CDN来加载普遍使用的依赖，如React，能够让构建环节更进一步地提高。这将更进一步地减少了代码库的体积然而在项目中增加了外部依赖。这一理念旨在若用户之前已经访问了CDN，缓存如这里的则可以生效了。

{pagebreak}

## 使用记录（Using Records）

As mentioned in the *Bundle Splitting* chapter, plugins such as `AggressiveSplittingPlugin` use **records** to implement caching. The approaches discussed above are still valid, but records go one step further.

如之前在**分割代码包**章节提到的，类似`AggressiveSplittingPlugin`的插件使用**记录（records）**来实现缓存。之前讨论的方法仍然有效，但是记录则前进得更远。

Records are used for storing module IDs across separate builds. The problem is that you need to store this file. If you build locally, one option is to include it to your version control.

记录被用来存储横单独构建的模块ID。问题是你需要存储该文件。如果你仅需要进行本地构建，一个可行方式是将其包含在你的版本控制中。

To generate a *records.json* file, adjust the configuration as follows:

为生成*records.json*文件，将配置调整如下：

**webpack.config.js**

```javascript
const productionConfig = merge([
  {
    ...
leanpub-start-insert
    recordsPath: path.join(__dirname, 'records.json'),
leanpub-end-insert
  },
  ...
]);
```

If you build the project (`npm run build`), you should see a new file, *records.json*, at the project root. The next time webpack builds, it picks up the information and rewrites the file if it has changed.

如果你现在构建项目（`npm run build`），你应该能看到一个新文件，*records.json*，在项目的根路径。在下次webpack构建时，其将从该文件读取信息和重写如果发生了改变。

Records are particularly valuable if you have a complicated setup with code splitting and want to make sure the split parts gain correct caching behavior. The biggest problem is maintaining the record file.

如果你有一个涵盖了代码分割的复杂配置并想要确保分割部分能够得到正确的缓存那么记录尤其具有价值。最大的问题是维护记录文件。

T> `recordsInputPath` and `recordsOutputPath` give more granular control over input and output, but often setting only `recordsPath` is enough.

T> `recordsInputPath`和`recordsOutputPath`提供了更细颗粒度的输出和输入控制，但通常情况仅设置`recordsPath`就足够了。

W> If you change the way webpack handles module IDs (i.e., remove `HashedModuleIdsPlugin`), possible existing records are still taken into account! If you want to use the new module ID scheme, you have to remove your records file as well.

W> 如果你改变了webpack处理模块ID（如，移除`HashedModuleIdsPlugin`），可能现有的记录仍会被考虑在内！如果你想要使用新的ID体制，你也不得不移除记录文件。

## 结论（Conclusion）

The project has basic caching behavior now. If you try to modify *app.js* or *component.js*, the vendor bundle should remain the same.

这一项目现在有了基本的缓存行为。如果你尝试修改*app.js*或者*component.js*，代码基础库仍然与之前保持一致。

To recap:

* Webpack maintains a **manifest** containing information needed to run the application.
* If the manifest changes, the change invalidates the containing bundle.
* To overcome this problem, the manifest can be extracted to a bundle of its own using the `CommonsChunkPlugin`.
* Certain plugins allow you to write the manifest to the generated *index.html*. It's also possible to extract the information to a JSON file. The JSON comes in handy with *Server Side Rendering*.
* **Records** allow you to store module IDs across builds. The approach becomes essential if you rely on code splitting approaches. As a downside you have to track the records file somehow.

本章概要：

* Webpack维护了一个涵盖需要运行改应用的**清单**。
* 如果清单改变了，改变会使整个代码包都失效。
* 为了解决这一问题，使用`CommonsChunkPlugin`能够将该清单提取到一个单独的文件中。
* 某些插件能够让你将清单写进生成的*index.html*文件中。也可以将该信息提取到一个JSON文件。该JSON文件迟早会在*服务端渲染*中使用。
* **记录**允许你存储模块ID贯穿整个构建环节。如果你依赖于代码分割这一方案那么该方法将至关重要。缺陷则是你必须以某种方式跟踪记录文件。

You'll learn to analyze the build statistics in the next chapter. This analysis is essential for figuring out how to improve the build result.

你将会在下一章节了解分析构建静态资源。这种分析对于了解如何改进构建结果至关重要。
