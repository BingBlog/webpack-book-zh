# 动态加载（Dynamic Loading）

Even though you can get far with webpack's code splitting features covered in the *Code Splitting* chapter, there's more to it. Webpack provides more dynamic ways to deal with code through `require.context`.

即使你对*代码分割*章节中所涉及到的webpack的代码分割特性不是很了解，在这里我们也会更多谈论到他。Webpack提供更多动态的方式通过`require.context`来处理代码。

## `require.context`处理动态加载（Dynamic Loading with `require.context`）

[require.context](https://webpack.js.org/configuration/entry-context/#context) provides a general form of code splitting. Let's say you are writing a static site generator on top of webpack. You could model your site contents within a directory structure by having a `./pages/` directory which would contain the Markdown files.

[require.context](https://webpack.js.org/configuration/entry-context/#context) 提供了一种通用格式以用于代码分割。这也就是说你在webpack的顶端写了一个静态页面生成器。你可以通过`./pages/`的一个包含Markdown文件的目录结构来组织页面内容。

Each of these files would have a YAML frontmatter for their metadata. The url of each page could be determined based on the filename and mapped as a site. To model the idea using `require.context`, you could end up with code as below:

每一个这类文件有一个YAML文件用于存储自身元数据。每个页面的url根据文件名来确定并映射为一个站点。为使用`require.context`来实现这一理念， 你可以使用如下代码：

```javascript
// Process pages through `yaml-frontmatter-loader` and `json-loader`.
// The first one extracts the frontmatter and the body and the latter
// converts it into a JSON structure to use later. Markdown
// hasn't been processed yet.
const req = require.context(
  'json-loader!yaml-frontmatter-loader!./pages',
  true, // Load files recursively. Pass false to skip recursion.
  /^\.\/.*\.md$/ // Match files ending with .md.
);
```

T> The loader definition could be pushed to webpack configuration. The inline form is used to keep the example minimal.

T> 加载器定义能够被推入webpack配置。行内格式用于保证其例子的最小化。

`require.context` returns a function to `require` against. It also knows its module `id` and it provides a `keys()` method for figuring out the contents of the context. To give you a better example, consider the code below:

`require.context`返回一个函数给`require`。其也知道自身的模块`id`并提供了一个`keys()`方法用于找出上下文中的内容。为了能够更好地示范，思考如下代码：

```javascript
req.keys(); // ['./demo.md', './another-demo.md']

req.id; // 42

// {title: 'Demo', body: '# Demo page\nDemo content\n\n'}
const demoPage = req('./demo.md');
```

The technique can be valuable for other purposes, such as testing or adding files for webpack to watch. In that case, you would set up a `require.context` within a file which you then point to through a webpack `entry`.

这一技术对于其他技术来说是有一定价值的，如测试或增加文件以让webpack进行监听。在上一例子中，你将`require.context`设置为webpack`entry`指向的文件。

T> The information is enough for generating an entire site. This has been done with [Antwar](https://github.com/antwarjs/antwar).

T> 这一信息足够生成整个站点。其已被[Antwar](https://github.com/antwarjs/antwar)所处理。

{pagebreak}

## 多`require.context`结合 （Combining Multiple `require.context`s）

Multiple separate `require.context`s can be combined into one by wrapping them behind a function:

多个独立的`require.context`可以通过将其包含在一个函数内：

```javascript
const { concat, uniq } from 'lodash';

const combineContexts = (...contexts) => {
  function webpackContext(req) {
    // Find the first match and execute
    const matches = contexts.map(
      context => context.keys().indexOf(req) >= 0 && context
    ).filter(a => a);

    return matches[0] && matches[0](req);
  }
  webpackContext.keys = () => uniq(
    concat.apply(
      null,
      contexts.map(context => context.keys())
    )
  );

  return webpackContext;
}
```

{pagebreak}

## 动态路径及动态`import`（Dynamic Paths with a Dynamic `import`）

The same idea works with dynamic `import`. Instead of passing a complete path, you can pass a partial one. Webpack sets up a context internally. Here's a brief example:

动态`import`工作理念相同。你可以传一部分路径，而不用传完整路径。Webpack会在内部设置上下文。这有个简单的例子：

```javascript
// Set up a target or derive this somehow
const target = 'fi';

// Elsewhere in code
import(`translations/${target}.json`).then(...).catch(...);
```

The same idea works with `require` as long as webpack can analyze the situation statically.

其工作原理同`require`只要webpack能够静态分析资源。

T> Any time you are using dynamic imports, it's a good idea to specify file extension in the path as that helps with performance by keeping the context smaller than otherwise.

T> 任何时候在使用动态引入，一个理想方式是在路径中指定文件插件能够让整提内容体积保持比其他的体积更小。

## 处理动态路径（Dealing with Dynamic Paths）

Given the approaches discussed here rely on static analysis and webpack has to find the files in question, it doesn't work for every possible case. If the files you need are on another server or have to be accessed through a particular end-point, then webpack isn't enough.

考虑到这里所讨论的方式依赖于静态分析且webpack必须考虑找到文件，其并不适用所有案例。如果你所需要的文件在另外的服务上或通过特别的终端才能访问，那webpack将无能为力。

Consider using browser-side loaders like [$script.js](https://www.npmjs.com/package/scriptjs) or [little-loader](https://www.npmjs.com/package/little-loader) on top of webpack in this case.

在这一例子中，考虑使用浏览器端加载器如 [$script.js](https://www.npmjs.com/package/scriptjs) 或在webpack顶层使用 [little-loader](https://www.npmjs.com/package/little-loader)。

{pagebreak}

## 结论（Conclusion）

Even though `require.context` is a niche feature, it's good to be aware of it. It becomes valuable if you have to perform lookups against multiple files available within the file system. If your lookup is more complex than that, you have to resort to other alternatives that allow you to perform loading runtime.

尽管`require.context`是个合适的特性，最好能够很好的认识它。如果你能够在文件系统中很方便地查找文件，那么将会具有价值。如果文件查找要复杂得多，那你不得不求助其他能够允许你在运行时加载的方式。

To recap:

* `require.context` is an advanced feature that's often hidden behind the scenes. If you have to perform a lookup against a large amount of files, use it.
* If you write a dynamic `import` in a certain form, webpack generates a `require.context` call. The code reads slightly better in this case.
* The techniques work only against the file system. If you have to operate against urls, you should look into client-side solutions.

本章回顾：

* `require.context`是一个先进的特性，且其常常被隐藏在幕后。如果你不得不查找一大堆文件，那么则可以使用它。
* 如果你在确定地形式中编写动态`import`, webpack则会生成一个`require.context`调用。在这一例中，代码会稍微易读。
* 这一技术仅适用于文件系统。如果你想处理url，你需要看看客户端的解决方案。

The next chapter shows how to use web workers with webpack.

下一章节将展示如何在webpack中使用web workers。
