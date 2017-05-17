## 服务端渲染（Server Side Rendering）

**Server Side Rendering** (SSR) is a technique that allows you to serve an initial payload with HTML, JavaScript, CSS, and even application state. You serve a fully rendered HTML page that would make sense even without JavaScript enabled. In addition to providing potential performance benefits, this can help with Search Engine Optimization (SEO).

**服务端渲染**（SSR）是一项允许你在服务端初始化HTML、JavaScript、CSS甚至是应用状态的一项技术。你能够完整地渲染HTML页面，即使在无JavaScript支持下其也是有意义的。除了提供潜在的性能优势，这项技术有助于搜索引擎优化（SEO）。

Even though the idea does not sound that special, there is a technical cost involved, and you can find sharp corners. The approach was popularized by React. Since then frameworks encapsulating the tricky bits, such as [Next.js](https://www.npmjs.com/package/next), have appeared. [isomorphic-webpack](https://www.npmjs.com/package/isomorphic-webpack) is a good example of a solution designed on top of webpack.

即使这一理念听起来并不特别，但其仍然具有一定的技术复杂度，同时你会发现一些棘手问题。该方式由React推广。至此框架封装大部分的棘手问题，如 [Next.js](https://www.npmjs.com/package/next)的出现。 [isomorphic-webpack](https://www.npmjs.com/package/isomorphic-webpack) 是一个在webpack顶部设计的一个很好的解决方案的例子。

To demonstrate SSR, you can use webpack to compile a client-side build that then gets picked up by a server that renders it using React following the principle. This is enough to understand how it works and also where the problems begin.

为演示SSR，你可以使用webpack来编译客户端构建以让服务器根据React规则来进行渲染。这已足够理解服务端渲染是如何工作和问题所在。

### 配置Babel和React（Setting Up Babel with React）

The *Loading JavaScript* chapter covers the essentials of using Babel with webpack. There's setup that is particular to React you should perform, though. Given most of React projects rely on [JSX](https://facebook.github.io/jsx/) format, you have to enable it through Babel.

在*加载JavaScript*一章涉及了使用Babel和webpack的基本要素。但你应该执行针对React的配置。考虑到大多数React项目依赖于 [JSX](https://facebook.github.io/jsx/) 格式，你必须通过Babel来支持它。

To get React, and particularly JSX, work with Babel, install the preset first:

为了能够获得React，尤其是JSX能够和Babel协作，首先安装预先插件：

```bash
npm install babel-preset-react --save-dev
```

{pagebreak}

Connect the preset with Babel configuration as follows:

将Babel和预先插件进行如下拼接：

**.babelrc**

```json
{
  ...
  "presets": [
leanpub-start-insert
    "react",
leanpub-end-insert
    ...
  ]
}
```

### 配置React和ESLint（Configuring React with ESLint）

Using React with ESLint and JSX requires extra work as well. [eslint-plugin-react](https://www.npmjs.com/package/eslint-plugin-react) does a part of the work, but also ESLint configuration is needed.

使用React和ESLint和JSX还要求一些额外的工作。[eslint-plugin-react](https://www.npmjs.com/package/eslint-plugin-react) 处理了一部分工作，但ESLint配置仍是需要。

Install *eslint-plugin-react* to get started:

首先安装*eslint-plugin-react*：

```bash
npm install eslint-plugin-react --save-dev
```

The suggested minimum configuration is as follows:

**.eslintrc.js**

```javascript
module.exports = {
  ...
leanpub-start-delete
  extends: 'eslint:recommended',
leanpub-end-delete
leanpub-start-insert
  extends: ['eslint:recommended', 'plugin:react/recommended'],
leanpub-end-insert
  parser: 'babel-eslint',
  parserOptions: {
    sourceType: 'module',
    allowImportExportEverywhere: true,

leanpub-start-insert
    // Enable JSX
    ecmaFeatures: {
      jsx: true,
    },
leanpub-end-insert
  },
leanpub-start-insert
  plugins: [
    'react',
  ],
leanpub-end-insert
  ...
};
```

`plugin:react/recommended` gives a good starting point. It's important to remember to enable JSX at the `parserOptions` as otherwise it will fail to parse JSX syntax.

`plugin:react/recommended` 提供了一个好的开始。记得在`parserOptions`支持JSX十分重要，否则其将无法解析JSX语法。

### 配置React Demo （Setting Up a React Demo）

To make sure the project has the dependencies in place, install React and [react-dom](https://www.npmjs.com/package/react-dom). The latter package is needed to render the application to the DOM.

为确保项目有具备依赖关系，安装React和[react-dom](https://www.npmjs.com/package/react-dom)。后者是需要将应用渲染为DOM节点。

```bash
npm install react react-dom --save
```

Next, the React code needs a small entry point. If you are on the browser side, you should mount `Hello world` `div` to the document. To prove it works, clicking it should give a dialog with a "hello" message. On server-side the React component is returned so the server can pick it up.

下一步，React代码需要一个小的入口。如果你是浏览器端，你应该渲染`Hello world``div`到文档上。为证明其生效，点击它应该弹出一个带“hello”的弹框。在服务端返回React组件以让服务端能够识别。

{pagebreak}

Adjust as follows:

调整如下：

**app/ssr.js**

```javascript
const React = require('react');
const ReactDOM = require('react-dom');

const SSR = <div onClick={() => alert('hello')}>Hello world</div>;

// Render only in the browser, export otherwise
if (typeof document === 'undefined') {
  module.exports = SSR;
} else {
  ReactDOM.render(SSR, document.getElementById('app'));
}
```

You are still missing webpack configuration to turn this file into something the server can pick up.

你还缺失了webpack的配置以将该文件的一些内容转换成服务器可识别的。

W> Given ES6 style imports and CommonJS exports cannot be mixed, the entry point was written in CommonJS style.

W> 考虑到ES 6风格的引入机制（imports）和CommonJS的导出机制（exports）无法混用，因此入口用CommonJS风格进行书写。

{pagebreak}

### 配置Webpack（Configuring Webpack）

To keep things nice and tidy, it's possible to push the demo configuration to a file of its own. A lot of the work has been done already. Given you have to consume the same output from multiple environments, using UMD as the library target makes sense:

为保值一致和整洁，最好将配置放入其自身文件中。大部分的工作已经完成。考虑到你不得不使用来自不同环境的相同的输出，使用UMD作为库的目标规范将会很有意义：

**webpack.ssr.js**

```javascript
const path = require('path');
const merge = require('webpack-merge');

const parts = require('./webpack.parts');

const PATHS = {
  build: path.join(__dirname, 'static'),
  ssrDemo: path.join(__dirname, 'app', 'ssr.js'),
};

module.exports = merge([
  {
    entry: {
      index: PATHS.ssrDemo,
    },
    output: {
      path: PATHS.build,
      filename: '[name].js',
      libraryTarget: 'umd',
    },
  },
  parts.loadJavaScript({ include: PATHS.ssrDemo }),
]);
```

{pagebreak}

To make it convenient to generate a build, add a helper script:

为方便生成构建文件，添加一个帮助脚本：

**package.json**

```json
"scripts": {
leanpub-start-insert
  "build:ssr": "webpack --config webpack.ssr.js",
leanpub-end-insert
  ...
},
```

If you build the SSR demo (`npm run build:ssr`), you should see a new file at *./static/index.js*. The next step is to set up a server to render it.

如果构建SSR的demo（`npm run build:ssr`），你应该看到一个新文件在**./static/index.js**。下一步则是配置服务器进行渲染。

### 配置服务器（Setting Up a Server）

To keep things clear to understand, you can set up a standalone Express server that picks up the generated bundle and renders it following the SSR principle. Install Express first:

为保持整洁和易于理解，你可以单独安装Express服务以让其生成代码包并根据服务端渲染规则进行渲染。首先安装Express：

```bash
npm install express --save-dev
```

Then, to get something running, implement a server as follows:

而后，为能够运行，服务实现如下：

**server.js**

```javascript
const express = require('express');
const { renderToString } = require('react-dom/server');

const SSR = require('./static');

server(process.env.PORT || 8080);

function server(port) {
  const app = express();

  app.use(express.static('static'));
  app.get('/', (req, res) => res.status(200).send(
    renderMarkup(renderToString(SSR))
  ));

  app.listen(port);
}

function renderMarkup(html) {
  return `<!DOCTYPE html>
<html>
  <head>
    <title>Webpack SSR Demo</title>
    <meta charset="utf-8" />
  </head>
  <body>
    <div id="app">${html}</div>
    <script src="./index.js"></script>
  </body>
</html>`;
}
```

Run the server now (`node ./server.js`) and go below `http://localhost:8080`, you should see something familiar:

现在运行服务（`node ./server.js`）并访问如下地址`http://localhost:8080`，你应该会看到一些熟悉的东西：

![Hello world](https://raw.githubusercontent.com/survivejs-translations/webpack-book-zh/master/manuscript/images/hello_01.png)

{pagebreak}

Even though there is a basic React application running now, it's difficult to develop. If you try to modify the code, nothing happens. This can be solved running webpack in a multi-compiler mode as earlier in this book. Another option is to run webpack in **watch mode** against the current configuration and set up a watcher for the server. You'll learn the latter setup next.

即使基础的React应用现在能运行，但仍很喊进行开发。如果你想要修改代码，啥也不会发生。该问题可以如本书之前提到的多编译模式下运行webpack来解决。另一个选择是在**监听模式**下运行webpack。针对当前配置并为服务端设置一个监听者。你将会在稍后了解配置。

T> If you want to debug output from the server, set `export DEBUG=express:application`.

T> 如果你想要从服务端输出调试信息，设置export DEBUG=express:application`。

T> The references to the assets generated by webpack could be written automatically to the server side template if you wrote a manifest as discussed in the *Separating a Manifest* chapter.

T> 引用webpack生成的资源可以被自动写入服务端模板中，如果你编写了一个之前在*分离清单*一章中所讨论的清单。

W> If you get a linting warning like `warning  'React' is defined but never used  no-unused-vars`, make sure the ESLint React plugin has been enabled and its default preset is in use.

W> 如果你得到了一个格式（linting）警告如`warning  'React' is defined but never used  no-unused-vars`，请确保ESLint React插件已被支持且其默认的预先设置已在使用。

### 监听服务端渲染改变并刷新浏览器（Watching SSR Changes and Refreshing the Browser）

The first portion of the problem is fast to solve. Run `npm run build:ssr -- --watch` in a terminal. That forces webpack to run in a watch mode. It would be possible to wrap this idea within an npm script for convenience, but this is enough for this demo.

第一部分需要解决的问题是速度问题。在终端运行`npm run build:ssr -- --watch`。其让webpack强制运行在监听模式下。将其包装在npm脚本代码中仅是出于方便，但这对于演示来说足够了。

The remaining part is harder than what was done so far. How to make the server aware of the changes and how to communicate the changes to the browser?

余下部分将会比之前所做的难得多。如何让服务器意识到变动以及如何将改变通知浏览器？

[browser-refresh](https://www.npmjs.com/package/browser-refresh) can come in handy as it solves both of the problems. Install it first:

[browser-refresh](https://www.npmjs.com/package/browser-refresh)可以非常方便地解决这两个问题。首先安装它：

```bash
npm install browser-refresh --save-dev
```

The client portion requires two small changes to the server code:

客户端需要对服务器代码进行两处小的调整：

**server.js**

```javascript
server(process.env.PORT || 8080);

function server(port) {
  ...

leanpub-start-delete
  app.listen(port);
leanpub-end-delete
leanpub-start-insert
  app.listen(port, () => process.send && process.send('online'));
leanpub-end-insert
}

function renderMarkup(html) {
  return `<!DOCTYPE html>
<html>
  ...
  <body>
    ...
leanpub-start-insert
    <script src="${process.env.BROWSER_REFRESH_URL}"></script>
leanpub-end-insert
  </body>
</html>`;
}
```

The first change tells the client that the application is online and ready to go. The latter change attaches the client script to the output. *browser-refresh* manages the environment variable in question.

第一处调整告诉客户端应用已上线并已准备好。后面的调整则绑定了客户端的脚本输出。*browser-refresh* 管理有问题的环境变量。

Run `node_modules/.bin/browser-refresh ./server.js` in another terminal and open the browser at `http://localhost:8080` as earlier to test the setup. Remember to have webpack running in the watch mode at another terminal. If everything went right, any change you make to the demo client script (*app/ssr.js*) should show up in the browser or cause a failure at the server.

在另一个终端里运行 `node_modules/.bin/browser-refresh ./server.js`并打开浏览器访问`http://localhost:8080`如之前测试的配置。记住webpack之前以监听模式运行在另一终端。若一切正常，你对客户端的样例脚本（app/srr.js**）的任何修改都将会展示在浏览器中或造成服务端错误。

If the server crashes, it loses the WebSocket connection. You have to force a refresh in the browser in this case. If the server was managed through webpack as well, the problem could have been avoided.

若服务端宕机，其将会丢失WebSocket连接。在这例子中，你必须强制重新刷新浏览器。若服务也是通过webpack进行管理，那该问题则可被避免。

To prove that SSR works, check out the browser inspector. You should see something familiar there:

为证明SSR已生效，打开浏览器的开发工具。你应该看到熟悉的东西：

![SSR output](https://raw.githubusercontent.com/survivejs-translations/webpack-book-zh/master/manuscript/images/ssr.png)

Instead of a `div` where to mount an application, you can see all related HTML there. It's not much in this particular case, but it's enough to showcase the approach.

而非将一个`div`渲染到应用，你可以查看所有相关的HTML。在这一特殊例子中并没有很多，但这足以展现这种做法。

T> The implementation could be refined further by implementing a production mode for the server that would skip injecting the browser refresh script at a minimum. The server could inject initial data payload to the generated HTML. Doing this would avoid queries on the client-side.

T> 通过为服务器忽略实现最少插入浏览器刷新脚本的生产模式来进一步改进实现。服务器可以为生成好的HTML插入初始化数据。这么处理可以避免客户端请求。

### 公开问题（Open Questions）

Even though the demo illustrates the basic idea of SSR, it still leaves open questions:

* How to deal with styles? Node doesn't understand CSS related imports.
* How to deal with anything else than JavaScript? If the server side is processed through webpack, this is less of an issue as you can patch it at webpack.
* How to run the server through something else than Node? One option would be to wrap the Node instance in a service you then run through your host environment. Ideally, the results would be cached, and you can find more specific solutions for this particular per platform.

即使演示阐述了服务端渲染的基本理念，其仍然遗留了些公开问题：

* 如何处理样式？Node并不了解CSS的相关引入机制。
* 如何处理除了JavaScript之外的其他？如果服务端通过webpack进行处理，这将会少了些问题同时你也可以在wepback上进行修补。
* 如何通过除了Node之外的其他来运行服务？一个选择是在一个服务里包装Node实例而后通过宿主环境进行运行。理想情况下，结果将会被缓存，你可以发现针对每个特定平台更多具体的解决方案。

Questions like these are the reason why solutions such as [isomorphic-webpack](https://www.npmjs.com/package/isomorphic-webpack) or [Next.js](https://github.com/zeit/next.js) exist. They have been designed to solve SSR-specific problems like these.

类似这些问题就是诸如[isomorphic-webpack](https://www.npmjs.com/package/isomorphic-webpack) 或[Next.js](https://github.com/zeit/next.js) 存在的理由。它们被设计用于解决类似于这样的服务端渲染的特定问题。

T> Routing is a big problem of its own solved by frameworks like Next.js. Patrick Hund [discusses how to solve it with React and React Router 4](https://ebaytech.berlin/universal-web-apps-with-react-router-4-15002bb30ccb).

T> 路由是被如Next.js之类框架解决的其中一个大问题。Patrick Hund[讨论如何通过React和React Router 4解决路由问题](https://ebaytech.berlin/universal-web-apps-with-react-router-4-15002bb30ccb).

### 结论（Conclusion）

SSR comes with a technical challenge and for this reason specific solutions have appeared around it. Webpack is a good fit for SSR setups.

服务端渲染带来的一系列技术挑战以及基于这一缘由围绕其的特定解决方案也应运而生。Webpack很适合服务端渲染配置。

To recap:

* **Server Side Rendering** can provide more for the browser to render initially. Instead of waiting for the JavaScript to load, you can display markup instantly.
* Server Side Rendering also allows you to pass initial payload of data to the client to avoid unnecessary queries to the server.
* Webpack can manage the client side portion of the problem. It can be used to generate the server as well if more integrated solution is required. Abstractions, such as Next.js, hide these details.
* Server Side Rendering does not come without a cost and it leads to new problems as you need better approaches for dealing with aspects, such as styling or routing. The server and the client environment differ in important manners, so code has to be written so that it does not rely on platform-specific features too much.

本章回顾：

* **服务端渲染**可以为浏览器提供更多地初始化渲染。你可以立即展示DOM元素而不用等到JavaScript加载。
* 服务端渲染还允许你传递初始化数据给客户端以避免不必要的服务端请求。
* Webpack能够管理客户端的问题。其也可以被用于处理服务端，若要求更多的集成解决方案。抽象，如Next.js隐藏了这些细节。
* 服务端渲染并不会带来开销，同时其带来新的问题需要更好的方式来处理如样式和路由等方面。服务端和客户端环境具有不同的重要性，因此代码必须被写以让其不过于依赖特定平台特性。