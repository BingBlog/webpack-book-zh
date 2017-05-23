# Code Splitting
# 代码拆分

Web applications have the tendency to grow big as features are developed. The longer it takes for your application to load, the more frustrating it's to the user. This problem is amplified in a mobile environment where the connections can be slow.

随着不断开发出的新特性，web应用的体积将会变得越来越大。应用加载时花费的时间越长，用户也将越失望。这个问题在网络很慢的移动环境中将被放大。

Even though splitting bundles can help a notch, they are not the only solution, and you can still end up having to download a lot of data. Fortunately, it's possible to do better thanks to **code splitting**. It allows to load code lazily as you need it.

尽管代码拆分可以帮助解决一些问题，但其不是唯一的方法，你仍然会得一个需要下载很多代码的应用。幸运的是，由于**code splitting**的特性，这个问题可以被更好的处理。它允许根据你的需要来对代码进行懒加载。

You can load more code as the user enters a new view of the application. You can also tie loading to a specific action like scrolling or clicking a button. You could also try to predict what the user is trying to do next and load code based on your guess. This way the functionality would be already there as the user tries to access it.

当用户进入到应用的一个新的视图时，你可以加载更多代码。你也可以将加载同特定的行为绑定在一起，如滚动或者点击事件。你也可以去揣测用户下一步将要干什么，并基于你的猜测来加载代码。如此，当用户尝试访问这个功能时，它已经加载好了。

T> Incidentally, it's possible to implement Google's [PRPL pattern](https://developers.google.com/web/fundamentals/performance/prpl-pattern/) using webpack's lazy loading. PRPL (Push, Render, Pre-cache, Lazy-load) has been designed with mobile web in mind.

T> 顺便说一下，可以使用webpack的懒加载来实现Google的[PRPL pattern](https://developers.google.com/web/fundamentals/performance/prpl-pattern/)。PRPL (Push, Render, Pre-cache, Lazy-load) has been designed with mobile web in mind.

## Code Splitting Formats
## 代码拆分格式化

Code splitting can be done in two primary ways in webpack: through a dynamic `import` or `require.ensure` syntax. The former is used in this project.

在webpack中进行代码拆分可以通过两种基本的方式：通过动态的`import`或者`require.ensure`语法。前者将被用于本项目。

The goal is to end up with a split point that gets loaded on demand. There can be splits inside splits, and you can structure an entire application based on splits. The advantage of doing this is that then the initial payload of your application can be smaller than it would be otherwise.

目标是最终需要得到按需加载的拆分点。可以在拆分之间再进行拆分，您可以根据拆分构建整个应用程序。 这样做的好处是，应用程序的初始有效负载可能会小于原来的有效负载。

![Code splitting](images/dynamic.png)

### Dynamic `import`
### 动态的`import`

The [dynamic `import` syntax](https://github.com/tc39/proposal-dynamic-import) isn't in the official language specification yet. To use it, minor tweaks are needed especially at ESLint and Babel.

The [dynamic `import` syntax](https://github.com/tc39/proposal-dynamic-import)目前还没有成为官方的规范。为了使用它，需要进行一些小的调整，特别是在对于ESLint和Babel

Dynamic imports are defined as `Promise`s:

动态import被定义为`Promise`：

```javascript
import(/* webpackChunkName: "optional-name" */ './module').then(
  module => {...}
).catch(
  error => {...}
);
```

{pagebreak}

The optional name allows you to pull multiple split points into a single bundle. As long as they have the same name, they will be grouped together. Each split point generates a separate bundle by default.

可选的名称让你可以将多个拆分点拉入到一个单独的代码包中。只要它们有同样的名称，它们就将被组合在一起。每一个拆分都会默认都会生成一个单独的代码包。

The interface allows composition, and you could load multiple resources in parallel:

该接口允许进行组合，你可以串联加载多个资源：

```javascript
Promise.all([
  import('lunr'),
  import('../search_index.json'),
]).then(([lunr, search]) => {
  return {
    index: lunr.Index.load(search.index),
    lines: search.lines,
  };
});
```

This creates separate bundles to request. If you wanted only one, you would have to use naming or define an intermediate module to `import`.

这将为每个请求创建单独的代码包。如果你只希望一个代码包，你可以使用命名或者定义一个中间模块来`import`。

T> Webpack provided support for `System.import` in the early versions of webpack 2 and it still does. The functionality has been deprecated and gets removed in webpack 3. Until then, you can use the functionality interchangeably.

T> Webpack在早期的webpack2版本中为`System.import`提供了支持，目前仍然可用。该功能将在webpack3中被移除。在此之前，你都可以继续使用这个功能。

W> The syntax works only with JavaScript after configured the right way. If you use another environment, you have to use alternatives covered in the following sections.

W> 该语法仅在正确配置之后对JavaScript有效。如果你使用的是其他的环境，你需要使用下面讲到的其他方式。

{pagebreak}

### `require.ensure`

[require.ensure](https://webpack.js.org/guides/code-splitting-require/#require-ensure-) provides an alternate way:

[require.ensure](https://webpack.js.org/guides/code-splitting-require/#require-ensure-)提供了另外的方式：

```javascript
require.ensure(
  // Modules to load, but not execute yet
  ['./load-earlier'],
  () => {
    const loadEarlier = require('./load-earlier');

    // Load later on demand and include to the same chunk
    const module1 = require('./module1');
    const module2 = require('./module2');

    ...
  },
  err => console.error(err),
  'optional-name'
);
```

Often you can achieve what you want through a dynamic `import`, but it's good to know this alternate form exists as well. `require.ensure` supports naming as well and [the official example](https://github.com/webpack/webpack/tree/master/examples/named-chunks) shows the output in detail.

通常你可以通过动态的`import`来实现你所想的，但是了解这个已有的可选的方法也是不错的。`require.ensure`也支持命名，[官方实例](https://github.com/webpack/webpack/tree/master/examples/named-chunks)详细展示了输出结果。

W> `require.ensure` relies on `Promise`s internally. If you use `require.ensure` with older browsers, remember to shim `Promise` using a polyfill such as [es6-promise](https://www.npmjs.com/package/es6-promise).

W> `require.ensure`基于内部基于`Promise`。如果你在老的浏览器中使用`require.ensure`，记得使用一个`Promise`的的polyfill，如[es6-promise](https://www.npmjs.com/package/es6-promise)。

{pagebreak}

### `require.include`

The example above could be rewritten using webpack particular `require.include`:

如上的例子可以使用webpack特定的`require.include`来改写：

```javascript
require.ensure(
  [],
  () => {
    require.include('./load-earlier');

    const loadEarlier = require('./load-earlier');

    // Load later on demand and include to the same chunk
    const module1 = require('./module1');
    const module2 = require('./module2');

    ...
  }
);
```

If you had nested `require.ensure` definitions, you could pull a module to the parent chunk using either syntax. It's a similar idea as you saw in the *Bundle Splitting* chapter.

如果你已经嵌套了`require.ensure`的定义，你可以使用先前的语法拉取一个模块到父一级的代码包。这和你在*Bundle Splitting* 章节看到的是一样的道理。

T> The formats respect `output.publicPath` option. You can also use `output.chunkFilename` to shape where they output. Example: `chunkFilename: '[name].js'`.

T> 该格式遵守`output.publicPath`选项。你可以使用`output.chunkFilename`来设置输出的位置。例如：`chunkFilename: '[name].js'`。

{pagebreak}

## Setting Up Code Splitting
## 设置代码拆分

To demonstrate the idea of code splitting, you can use dynamic `import`. Both ESLint and Babel setup of the project needs additions to make the syntax work.

为了演示代码拆分的想法，你可以使用动态的`import`。项目中的ESLint和Babel设置都需要额外的调整来使该语法有效。

### Configuring ESLint

### 配置ESLint

Given ESLint supports only standard ES6 out of the box, it requires tweaking to work with dynamic `import`. Install *babel-eslint* parser first:

由于ESLint默认仅支持标准的ES6语法，需要一些调整才能正常处理动态`import`。首先安装*babel-eslint*解析器：

```bash
npm install babel-eslint --save-dev
```

Tweak ESLint configuration as follows:

如下调整ESLint配置:

**.eslintrc.js**

```javascript
module.exports = {
  ...
leanpub-start-insert
  parser: 'babel-eslint',
leanpub-end-insert
  parserOptions: {
    sourceType: 'module',
leanpub-start-insert
    allowImportExportEverywhere: true,
leanpub-end-insert
  },
  ...
}
```

After these changes, ESLint doesn't complain if you write `import` in the middle of the code.

更改之后，如果你在代码中间使用了`import`，ESLint将不会再报错。
{pagebreak}

### Configuring Babel
### 配置Babel

Given Babel doesn't support the dynamic `import` syntax out of the box, it needs [babel-plugin-syntax-dynamic-import](https://www.npmjs.com/package/babel-plugin-syntax-dynamic-import) to work. Install it first:

由于Babel默认并不支持动态的`import`语法，需要[babel-plugin-syntax-dynamic-import](https://www.npmjs.com/package/babel-plugin-syntax-dynamic-import)来使其生效。先安装它：

```bash
npm install babel-plugin-syntax-dynamic-import --save-dev
```

To connect it with the project, adjust the configuration as follows:

为了在项目汇总使用来，如下调整配置：

**.babelrc**

```json
{
leanpub-start-insert
  "plugins": ["syntax-dynamic-import"],
leanpub-end-insert
  ...
}
```

### Defining a Split Point Using a Dynamic `import`
### 使用动态的`import`来定义拆分点

The idea can be demonstrated by setting up a module that contains a string that replaces the text of the demo button:

该想法可以通过例子来证明。设置一个模块，该模块包含替换示例中按钮文本的字符串：

**app/lazy.js**

```javascript
export default 'Hello from lazy';
```

{pagebreak}

You also need to point the application to this file, so the application knows to load it. This can be done by binding the loading process to click. Whenever the user happens to click the button, you trigger the loading process and replace the content:

你还需要在应用中引用该文件，所以应用指导要去加载它。这可以通过将加载程序同点击事件进行绑定来实现。不论何时用户点击了这个按钮，都将触发加载程序并替换掉内容：

**app/component.js**

```javascript
export default () => {
  const element = document.createElement('div');

  element.className = 'fa fa-hand-spock-o fa-1g';
  element.innerHTML = 'Hello world';
leanpub-start-insert
  element.onclick = () => {
    import('./lazy').then((lazy) => {
      element.textContent = lazy.default;
    }).catch((err) => {
      console.error(err);
    });
  };
leanpub-end-insert

  return element;
};
```

If you open up the application (`npm start`) and click the button, you should see the new text in the button.

如果你打开该应用(`npm start`)，并点击按钮，你将看到按钮中将会出现新的文本内容。

![Lazy loaded content](images/lazy.png)

{pagebreak}

If you run `npm run build`, you should see something:

如果你执行`npm run build`，你可以看到如下信息：

```bash
Hash: e61343b53de634da8aac
Version: webpack 2.2.1
Time: 2890ms
        Asset       Size  Chunks                    Chunk Names
       app.js     2.4 kB       1  [emitted]         app
  ...font.eot     166 kB          [emitted]
...font.woff2    77.2 kB          [emitted]
 ...font.woff      98 kB          [emitted]
  ...font.svg     444 kB          [emitted]  [big]
     logo.png      77 kB          [emitted]
leanpub-start-insert
         0.js  313 bytes       0  [emitted]
leanpub-end-insert
  ...font.ttf     166 kB          [emitted]
    vendor.js     150 kB       2  [emitted]         vendor
      app.css    3.89 kB       1  [emitted]         app
leanpub-start-insert
     0.js.map  233 bytes       0  [emitted]
leanpub-end-insert
   app.js.map    2.13 kB       1  [emitted]         app
  app.css.map   84 bytes       1  [emitted]         app
vendor.js.map     178 kB       2  [emitted]         vendor
   index.html  274 bytes          [emitted]
   [0] ./~/process/browser.js 5.3 kB {2} [built]
   [3] ./~/react/lib/ReactElement.js 11.2 kB {2} [built]
  [18] ./app/component.js 461 bytes {1} [built]
...
```

That *0.js* is your split point. Examining the file reveals that webpack has wrapped the code in a `webpackJsonp` block and processed the code bit.

那个*0.js*就是你的拆分点。检查该文件，揭示了webpack将代码包裹到了一个`webpackJsonp`块中，并对代码进行了一些处理。

{pagebreak}

### Lazy Loading Styles
###  样式的懒加载

Lazy loading can be applied to styling as well. Expand the definition:

懒加载也可以被应用于样式。如下扩展定义：

**app/lazy.js**

```javascript
leanpub-start-insert
import './lazy.css';
leanpub-end-insert

export default 'Hello from lazy';
```

And to have a style definition to load, set up a rule:

**app/lazy.css**

```css
body {
  color: blue;
}
```

The idea is that after *lazy.js* gets loaded, *lazy.css* is applied as well. You can confirm this by running the application (`npm start`). The same behavior is visible if you build the application (`npm run build`) and examine the output (`0.js`). This is due to the `ExtractTextPlugin` definition.

该想法就是，在*lazy.js*被加载完后，*lazy.css*将会被应用。你可以通过运行应用(`npm start`)来确认。相同的行为也是可见的，如果你构建应用(`npm run build`)并检查输出(`0.js`)。这是应为`ExtractTextPlugin`的定义。

![Lazy styled content](images/lazy-styled.png)

{pagebreak}

### Defining a Split Point Using `require.ensure`
### 使用`require.ensure`定义拆分点

It's possible to achieve the same with `require.ensure`. Consider the full example below:

也可以使用`require.ensure`来实现相同的效果。思考如下的完整的例子：

```javascript
export default () => {
  const element = document.createElement('div');

  element.className = 'pure-button';
  element.innerHTML = 'Hello world';
  element.onclick = () => {
    require.ensure([], (require) => {
      element.textContent = require('./lazy').default;
    });
  };

  return element;
};
```

You could name the split point as outlined above. If you add another split point and give it the same name, the splits should end up in the same bundle.

你可以如概述中那样对拆分点进行命名。如果你增加了另外一个拆分点，并以相同的名称命名，该拆分将会在同一个代码包中。

T> [bundle-loader](https://www.npmjs.com/package/bundle-loader) gives similar results, but through a loader interface. It supports bundle naming through its `name` option.

T> [bundle-loader](https://www.npmjs.com/package/bundle-loader)提供了相同的结果，但是通过一个加载器的接口。它通过其`name`选项来支持代码包命名。

T> The *Dynamic Loading* chapter covers other techniques that come in handy when you have to deal with more complicated splits.

T> 如果你需要处理更复杂的拆分，*Dynamic Loading*章节包含了其它有用的技术

{pagebreak}

## Code Splitting in React

## React中的代码拆分

The splitting pattern can be wrapped into a React component. Airbnb uses the following solution [as described by Joe Lencioni](https://gist.github.com/lencioni/643a78712337d255f5c031bfc81ca4cf):

拆分的模式可以被包裹到一个React的组件中。Airbnb使用如下的场景[as described by Joe Lencioni](https://gist.github.com/lencioni/643a78712337d255f5c031bfc81ca4cf):

```javascript
import React from 'react';

...

// Somewhere in code
<AsyncComponent loader={() => import('./SomeComponent')} />

...

// React wrapper for loading
class AsyncComponent extends React.Component {
  constructor(props) {
    super(props);

    this.state = {
      Component: null,
    };
  }

  componentDidMount() {
    // Load the component now
    this.props.loader().then(Component => {
      this.setState({ Component });
    });
  }

  render() {
    const { Component } = this.state;
    const { Placeholder } = this.props;

    if (Component) {
      return <Component {...this.props} />;
    }

    return <Placeholder>
  }
}

AsyncComponent.propTypes = {
  // A loader is a function that should return a Promise.
  loader: PropTypes.func.isRequired,

  // A placeholder to render while waiting completion.
  Placeholder: PropTypes.node.isRequired
};
```

T> [react-async-component](https://www.npmjs.com/package/react-async-component) wraps the pattern in a `createAsyncComponent` call and provides server side rendering specific functionality. [react-loadable](https://www.npmjs.com/package/react-loadable) is another option.

T> [react-async-component](https://www.npmjs.com/package/react-async-component)在一个叫做`createAsyncComponent`包含了改模式，并提供了特定于服务端渲染的功能。[react-loadable](https://www.npmjs.com/package/react-loadable)是另外一个选择。

{pagebreak}

## Conclusion
## 小结

Code splitting is one of those features that allows you to push your application a notch further. You can load code when you need it to gain faster initial load times and improved user experience especially in a mobile context where bandwidth is limited.

代码拆分是允许您将应用程序进一步推进的功能之一。你可以在需要的时候来加载代码，并提升初始化加载速度，提高了用户的体验，特别是带宽有限的移动环境下。

To recap:

回顾：

* **Code splitting** comes with extra effort as you have to decide what to split and where. Often, you find good split points within a router. Or you notice that specific functionality is required only when a particular feature is used. Charting is a good example of this.
* **Code splitting** 带来了额外的工作，你需要决定如何拆分并在何处拆分代码。通常，你将在路由中找到一个很好的拆分点。或者你将发现特定的功能只有在特定的特性被使用时才会需要。制图就是一个很好的例子。

* To use dynamic `import` syntax, both Babel and ESLint require careful tweaks. Webpack supports the syntax ouf of the box.
* 为了使用动态的`import`语法，Babel和ESLint都需要经过仔细的调整。Webpack默认。。。。。。。

* Use naming to pull separate split points into the same bundles.
* 使用命名将单独的拆分点拉入到同一个代码包中

* The techniques can be used within modern frameworks and libraries like React. You can wrap related logic to a specific component that handles the loading process in a user-friendly manner.
* 该技术可以同现代的框架和类库一起使用，如React。你可以将相关的逻辑包含在一个特定的组件中，以用户友好的方式来处理加载过程。

You'll learn to tidy up the build in the next chapter.

你将在下一章节学习如何整理构建。

T> The *Searching with React* appendix contains a complete example of code splitting. It shows how to set up a static site index that's loaded when the user searches information.

T> *Searching with React*附录包含了一个完整的代码拆分例子。它显示了如何设置用户搜索信息时加载的静态网站索引。

T> [webpack-pwa](https://github.com/webpack/webpack-pwa) illustrates the idea on a larger scale and discusses different shell based approaches. You get back to this topic in the *Multiple Pages* chapter.

T> [webpack-pwa](https://github.com/webpack/webpack-pwa)描绘了更大规模下的想法，并讨论了不同的基于壳的方法。 你可以在*Multiple Pages*章节回到这个主题。