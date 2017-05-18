# 多页面（Multiple Pages）

Even though webpack is often used for bundling single page applications, it's possible to use it with multiple separate pages as well. The idea is similar to the way you generated multiple output files in the *Targets* chapter. This time, however, you have to generate separate pages. That's achievable through `HtmlWebpackPlugin` and a bit of configuration.

尽管webpack常用于打包单页应用，但其也能用于处理多个独立页面。原理与在*目标*一章中生成多输出文件类似。而这次，你需要生成独立页面。可以通过`HtmlWebpackPlugin`和一系列配置实现。

## 可行方法（Possible Approaches）

When generating multiple pages with webpack, you have a couple of possibilities:

* Go through the *multi-compiler mode* and return an array of configurations. The approach would work as long as the pages are separate and there is a minimal need for sharing code across them. The benefit of this approach is that you can process it through [parallel-webpack](https://www.npmjs.com/package/parallel-webpack) to improve build performance.
* Set up a single configuration and extract the commonalities. The way you do this can differ depending on how you chunk it up.
* If you follow the idea of [Progressive Web Applications](https://developers.google.com/web/progressive-web-apps/) (PWA), you can end up with either an **app shell** or a **page shell** and load portions of the application as it's used.

在通过webpack生成多页面时，你可以有一系列的选择：

* 通过*多编译器模式*运行并返回一个包含配置的数组。该方式仅当页面是完全独立的且很少需要共享的代码才可成立。这一方法的收益在于你可以通过[parallel-webpack](https://www.npmjs.com/package/parallel-webpack) 来处理以提高构建性能。
* 设置单一配置并提取公共部分。该方式会有不同，取决于如何进行分割。
* 若遵循 [渐进式Web应用](https://developers.google.com/web/progressive-web-apps/) (PWA)的理念，你可以使用**app shell**或**page shell**并加载应用所使用的部分。

In practice, you have more dimensions. For example, you have to generate i18n variants for pages. These ideas grow on top of the basic approaches.

实践中，你可以有更多的维度。你可以为页面生成国际化版本。这些理念源自些基本方法之上。

## 生成多页面（Generating Multiple Pages）

To generate multiple separate pages, they should be initialized somehow. You should also be able to return a configuration for each page, so webpack picks them up and process them through the multi-compiler mode.

为生成多个独立页面，他们需要被初始化。你应该能够为每个页面返回一个配置，以让webpack能够识别并在多编译器模式下处理。

### 抽象页面（Abstracting Pages）

To initialize a page, it should receive page title, output path, and an optional template at least. Each page should receive optional output path, and a template for customization. The idea can be modeled as a configuration part:

为初始化页面，其至少应该接受页面标题，输出路径和一个可选的模板。每个页面应接受可选的输出路径和自定义模板。这理念可以包装到配置的一部分：

**webpack.parts.js**

```javascript
...
leanpub-start-insert
const HtmlWebpackPlugin = require('html-webpack-plugin');
leanpub-end-insert

...

leanpub-start-insert
exports.page = ({
  path = '',
  template = require.resolve(
    'html-webpack-plugin/default_index.ejs'
  ),
  title,
} = {}) => ({
  plugins: [
    new HtmlWebpackPlugin({
      filename: `${path && path + '/'}index.html`,
      template,
      title,
    }),
  ],
});
leanpub-end-insert
```

{pagebreak}

### 集成配置（Integrating to Configuration）

To incorporate the idea to the configuration, the way it's composed has to change. Also, a page definition is required. To get started, let's reuse the same JavaScript logic for each page for now:

为将该理念集成到配置中，其集成方式需要做些调整。此外，一个页面定义也是必须的。首先，现在为每个页面重用相同的JavaScript逻辑：

**webpack.config.js**

```javascript
const path = require('path');
leanpub-start-delete
const HtmlWebpackPlugin = require('html-webpack-plugin');
leanpub-end-delete
...


const commonConfig = merge([
  {
    ...
leanpub-start-delete
    plugins: [
      new HtmlWebpackPlugin({
        title: 'Webpack demo',
      }),
    ],
leanpub-end-delete
  },
]);

...

module.exports = (env) => {
leanpub-start-delete
  if (env === 'production') {
    return merge(commonConfig, productionConfig);
  }

  return merge(commonConfig, developmentConfig);
leanpub-end-delete
leanpub-start-insert
  const pages = [
    parts.page({ title: 'Webpack demo' }),
    parts.page({ title: 'Another demo', path: 'another' }),
  ];
  const config = env === 'production' ?
    productionConfig :
    developmentConfig;

  return pages.map(page => merge(commonConfig, config, page));
leanpub-end-insert
};
```

After this change you should have two pages in the application: `/` and `/another`. It should be possible to navigate to both while seeing the same output.

此次调整后应用中应该有两个页面：`/` 和 `/another`。当看到相同输出时，应该可以看到同时指向两者。

### 每个页面插入不同脚本（Injecting Different Script per Page）

The question is, how to inject a different script per each page. In the current configuration, the same `entry` is shared by both. To solve the problem, you should move `entry` configuration to lower level and manage it per page. To have a script to test with, set up another entry point:

问题在于如何为每个页面插入不同的脚本。在目前的配置中，相同的`入口`是共享的。为解决这一问题，需要将入口配置移到更低的位置并为每个页面进行管理。为了有脚本对此进行测试，设置另一个入口：

**app/another.js**

```javascript
import './main.css';
import component from './component';

let demoComponent = component('Another');

document.body.appendChild(demoComponent);
```

{pagebreak}

The file could go to a directory of its own. Here the existing code is reused to get something to show up. Webpack configuration has to point to this file:

文件会到它的目录下。这里重用现存的代码以作为展示。Webpack配置必须指向这一文件：

**webpack.config.js**

```javascript
const commonConfig = merge([
  {
leanpub-start-delete
    entry: {
      app: PATHS.app,
    },
leanpub-end-delete
    ...
  },
  ...
]);

...

module.exports = (env) => {
leanpub-start-delete
  const pages = [
    parts.page({ title: 'Webpack demo' }),
    parts.page({ title: 'Another demo', path: 'another' }),
  ];
leanpub-end-delete
leanpub-start-insert
  const pages = [
    parts.page({
      title: 'Webpack demo',
      entry: {
        app: PATHS.app,
      },
    }),
    parts.page({
      title: 'Another demo',
      path: 'another',
      entry: {
        another: path.join(PATHS.app, 'another.js'),
      },
    }),
  ];
leanpub-end-insert
  const config = env === 'production' ?
    productionConfig :
    developmentConfig;

  return pages.map(page => merge(commonConfig, config, page));
};
```

The tweak also requires a change at the related part so that `entry` gets included in the configuration:

这一调整还要求相关部分进行改变以便`入口`被包含进配置中：

**webpack.parts.js**

```javascript
exports.page = ({
  ...
leanpub-start-insert
  entry,
leanpub-end-insert
} = {}) => ({
leanpub-start-insert
  entry,
leanpub-end-insert
  ...
});
```

After these changes `/another` should show something familiar:

在这些调整之后，`/another`应该显示类似：

![Another page shows up](https://raw.githubusercontent.com/survivejs-translations/webpack-book-zh/master/manuscript/images/another.png)

{pagebreak}

### 优劣（Pros and Cons）

If you build the application (`npm run build`), you should find *another/index.html*. Based on the generated code, you can make the following observations:

* It's clear how to add more pages to the setup.
* The generated assets are directly below the build root. The pages are an exception as those are handled by `HtmlWebpackPlugin`, but they still point to the assets below the root. It would be possible to add more abstraction in the form of *webpack.page.js* and manage the paths by exposing a function that accepts page configuration.
* Records should be written separately per each page in files of their own. Currently, the configuration that writes the last, wins. The above solution would allow solving this.
* Processes like linting and cleaning run twice currently. The *Targets* chapter discussed potential solutions to that problem.

如果构建应用（`npm run build`），你应该会发现*another/index.html*。基于生成的diam，你可以观察到如下内容：

* 如何为设置增加多页面是变得非常清晰。
* 生成的资源直接在构建目录下。那些通过`HtmlWebpackPlugin`处理的页面是个例外，但他们仍指向根路径下的资源。通过导出一个接受页面配置的函数可以以*webpack.page.js*的形式增加更多抽象和管理路径。
* 每个页面的记录应该写在其自己的文件中。目前，配置仅写入最后一个成功的记录。之前的方案已经解决了这一问题。
* 如清理和格式校验目前运行两次。在*目标*章节讨论的潜在方案针对的就是那一问题。

The approach can be pushed to another direction by dropping the multi-compiler mode. Even though it's slower to process this kind of build, it enables code sharing, and the implementation of shells. The first step towards a shell setup is to rework the configuration so that it picks up the code shared between the pages.

该方法通过移除多编译器模式可以被推入其他方向。即使其处理速度慢于这样的构建方式，但其支持代码共享和shell实现。完成shell配置的第一步是重构配置以让其能够在页面间共享代码。

## 生成多页面并共享代码（Generating Multiple Pages While Sharing Code）

The current configuration shares code by coincidence already due to the usage patterns. Only a small part of the code differs, and as a result only the page manifests, and the bundles mapping to their entries differ.

由于使用模式，目前的配置恰巧已实现共享代码。仅有一小部分代码不同，以及最后只有页面清单和映射到代码包的入口不同。

In a more complicated application, you should apply techniques covered in the *Bundle Splitting* chapter across the pages. Dropping the multi-compiler mode can be worthwhile then.

在更为负载的应用中，你应该会在页面上应用到*代码包分割*章节所涵盖的技术。移除多编译器模式是值得的。

{pagebreak}

### 调整配置（Adjusting Configuration）

To reach a code sharing setup, a minor adjustment is needed. Most of the code can remain the same. The way you expose it to webpack has to change so that it receives a single configuration object. As `HtmlWebpackPlugin` picks up all chunks by default, you have to adjust it to pick up only the chunks that are related to each page:

为能够达到代码共享配置，一些微小的调整是需要的。大部分代码仍保持一致。暴露给webpack的方式需要做些改变以便接受一个单一配置对象。默认情况下`HtmlWebpackPlugin`获取所有的文件，你需要将它调整为仅获取每个页面相关的文件：

**webpack.config.js**

```javascript
module.exports = (env) => {
  const pages = [
    parts.page({
      title: 'Webpack demo',
      entry: {
        app: PATHS.app,
      },
leanpub-start-insert
      chunks: ['app', 'manifest', 'vendor'],
leanpub-end-insert
    }),
    parts.page({
      title: 'Another demo',
      path: 'another',
      entry: {
        another: path.join(PATHS.app, 'another.js'),
      },
leanpub-start-insert
      chunks: ['another', 'manifest', 'vendor'],
leanpub-end-insert
    }),
  ];
  const config = env === 'production' ?
    productionConfig :
    developmentConfig;

leanpub-start-delete
  return pages.map(page => merge(commonConfig, config, page));
leanpub-end-delete
leanpub-start-insert
  return merge([commonConfig, config].concat(pages));
leanpub-end-insert
};
```

{pagebreak}

The page-specific configuration requires a small tweak as well:

针对页面的配置也需要一些微调：

**webpack.parts.js**

```javascript
exports.page = ({
  ...
leanpub-start-insert
  chunks,
leanpub-end-insert
} = {}) => ({
  entry,
  plugins: [
    new HtmlWebpackPlugin({
leanpub-start-insert
      chunks,
leanpub-end-insert
      ...
    }),
  ],
});
```

If you generate a build (`npm run build`), you should notice that something is different compared to the first multiple page build you did. Instead of two manifest files, you can find only one. If you examine it, you notice it contains references to all files that were generated.

若你生成一个构建（`npm run build`），你应该注意到相较于第一次多页面构建的结构有一些不同。你发现清单文件仅剩一个而非两个。如果检查它，你会注意到其包含所有生成文件的引用。

Studying the entry specific files in detail reveals more. You can see that they point to different parts of the manifest. The manifest runs different code depending on the entry. Multiple separate manifests are not needed.

详细了解具体的入口文件透露了更多信息。你可以看到他们指向清单的不同部分。该清单根据不同的入口执行不同的代码。多个独立清单不再需要了。

### 优劣（Pros and Cons）

Compared to the earlier approach, something was gained, but also lost:

* Given the configuration isn't in the multi-compiler form anymore, processing can be slower.
* Plugins such as `CleanWebpackPlugin` don't work without additional consideration now.
* Instead of multiple manifests, only one remains. The result is not a problem, though, as the entries use it differently based on their setup.
* `CommonsChunkPlugin` related setup required careful thought to avoid problems with styling. The earlier approach avoided this issue through isolation.

相较于早前的实现方式，得到了些东西，但也失去了些：

* 考虑到所有配置不再是在多编译器下，处理速度更慢。
* 类似于`CleanWebpackPlugin`的插件在没有额外条件下不再运行。
* 仅剩下一个清单而非多个。尽管，入口根据它们的配置使用不同部分，但这一结果并非是问题。

## 渐进式Web应用（Progressive Web Applications）

If you push the idea further by combining it with code splitting and smart routing, you'll end up with the idea of Progressive Web Applications (PWA). [webpack-pwa](https://github.com/webpack/webpack-pwa) example illustrates how to implement the approach using webpack either through an app shell or a page shell.

通过结合代码分割和智能路由，若将这一理念进一步推进，其将会被渐进式Web应用（PWA）所终结。 [webpack-pwa](https://github.com/webpack/webpack-pwa) 演示了如何通过webpakc和app shell或page shell来实现该方式。

App shell is loaded initially, and it manages the whole application including its routing. Page shells are more granular, and more are loaded as you use the application. The total size of the application is larger but conversely you can load initial content faster.

App shell初始化被加载，其管理涵盖路由的整个应用。Page shell更为精细，并能够所使用
的应用加载更多内容。其应用的整体体积更大，但相反其初始内容也被更快地加载。

PWA combines well with plugins like [offline-plugin](https://www.npmjs.com/package/offline-plugin) and [sw-precache-webpack-plugin](https://www.npmjs.com/package/sw-precache-webpack-plugin). Using [Service Workers](https://developer.mozilla.org/en/docs/Web/API/Service_Worker_API) and improves the offline experience.

PWA可以同类似[offline-plugin](https://www.npmjs.com/package/offline-plugin) 和 [sw-precache-webpack-plugin](https://www.npmjs.com/package/sw-precache-webpack-plugin) 插件很好的结合。使用 Using [Service Workers](https://developer.mozilla.org/en/docs/Web/API/Service_Worker_API)能够提升离线体验。

## 结论（Conclusion）

Webpack allows you to manage multiple page setups. The PWA approach allows the application to be loaded progressively as it's used and webpack allows implementing it.

Webpack允许你管理多页面配置。PWA方式允许应用渐进式加载并能够webpack来实现。

To recap:

* Webpack can be used to generate separate pages either through its multi-compiler mode or by including all the page configuration into one.
* The multi-compiler configuration can run in parallel using external solutions, but it's harder to apply techniques such as bundle splitting against it.
* A multi-page setup can lead to a **Progressive Web Application**. In this case you use various webpack techniques to come up with an application that is fast to load and that fetches functionality as required. Both two flavors of this technique have their own merits.


本章回顾：

* Webpack能够被用于生成多个独立页面通过多编译器模式或通过将所有页面配置包含到一起。
* 使用额外的解决方案，多编译器配置可并发运行，但其很难将技术应用如代码包分割上。
* 一个多页面配置可产生**渐进式Web应用**。在这一案例中你使用多种webpack技术来让应用能够快速地被加载以及满足要求的功能。这两类技术都有着自身的优点。

You'll learn to implement Server Side Rendering in the next chapter.

你将会在下一章节了解实现服务端渲染。