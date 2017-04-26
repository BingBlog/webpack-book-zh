# 给文件增加Hash值（Adding Hashes to Filenames）

Even though the build generates fine now, the naming it uses is problematic. It doesn't allow to leverage client level cache effectively as there's no way tell whether or not a file has changed. Cache invalidation can be achieved by including a hash to filenames.

即使目前构建文件已经生成好了，其使用的名字仍然存在问题。由于没有一种有效方式能够知道文件是否失效使得缓存效率无法提高。缓存失效可以通过给文件名增加哈希值来实现。


## 占位符（Placeholders）

Webpack provides **placeholders** for this purpose. These strings are used to attach specific information to webpack output. The most valuable ones are:

Webpack基于这一目的提供了**占位符**。这些字符串被用于为webpack的输出提供具体信息。有价值的占位符：

* `[path]` - Returns the file path.
* `[name]` - Returns the file name.
* `[ext]` - Returns the extension. `[ext]` works for most available fields. `ExtractTextPlugin` is a notable exception to this rule.
* `[hash]` - Returns the build hash. If any portion of the build changes, this changes as well.
* `[chunkhash]` - Returns an entry chunk-specific hash. Each `entry` defined at the configuration receives a hash of own. If any portion of the entry changes, the hash changes as well. `[chunkhash]` is more granular than `[hash]` by definition.
* `[contenthash]` - Returns a hash specific to content. `[contenthash]` is available for `ExtractTextPlugin` only and is the most specific option available.

* `[path]` - 返回文件路径
* `[name]` - 返回文件名
* `[ext]` - 返回文件扩展类型。`[ext]`适用于大部分情况。而`ExtractTextPlugin`则是该规则的一个明显的例外。
* `[hash]` - 返回构建结果的哈希值。如果构建结果的任何部分发生改变，那个该值也会改变。
* `[chunkhash]` - 返回入口具体的哈希值。每一个定义在配置的`entry`都会有一个自己的哈希值。如果入口的任何部分发生改变，哈希值也随之改变。`[chunkhash]`比`[hash]`能够提供更精细的定义。
* `[contenthash]` - 返回具体内容的哈希值。`[contenthash]`是唯一可用于`ExtraTextPlugin`且最为具体的可用选项。

It's preferable to use particularly `hash` and `chunkhash` only for production purposes as hashing doesn't do much good during development.

更倾向于在生产环境中使用`hash`和`chunkhash`原因在于哈希在开发环境中的效果并不是很好。

T> It's possible to slice `hash` and `chunkhash` using specific syntax: `[chunkhash:8]`. Instead of a hash like `8c4cbfdb91ff93f3f3c5` this would yield `8c4cbfdb`.

T> 使用语法：`[chunkhash:8]`能够对`hash`和`chunkhash`进行切片操作。这一操作会生成`8c4cbfdb`而不是类似`8c4cbfdb91ff93f3f3c5`的哈希值。

T> There are more options available, and you can even modify the hashing and digest type as discussed at [loader-utils](https://www.npmjs.com/package/loader-utils#interpolatename) documentation.

T> 更多的选项可供使用，甚至可用修改哈希值也可以在[loader-utils](https://www.npmjs.com/package/loader-utils#interpolatename) 文档中了解更多之前讨论的类型。

### 占位符示例（Example Placeholders）

Assuming you have the following configuration:

假设配置如下：

```javascript
{
  output: {
    path: PATHS.build,
    filename: '[name].[chunkhash].js',
  },
},
```

Webpack would generate filenames like these:

Webpack将会生成类似的文件名：

```bash
app.d587bbd6e38337f5accd.js
vendor.dc746a5db4ed650296e1.js
```

If the file contents related to a chunk are different, the hash changes as well, thus invalidating the cache. More accurately, the browser sends a new request for the new file. If only `app` bundle gets updated, only that file needs to be requested again.

如果文件内容相关的内容块不同，哈希值也会改变。因而缓存失效。更准确地说，浏览器将会发送一个新请求来获取新文件。如果只有`app`包更新，那么仅有该文件需要再进行请求。

The same result can be achieved by generating static filenames and invalidating the cache through a querystring (i.e., `app.js?d587bbd6e38337f5accd`). The part behind the question mark invalidates the cache. According to [Steve Souders](http://www.stevesouders.com/blog/2008/08/23/revving-filenames-dont-use-querystring/), attaching the hash to the filename is the more performant.

通过查询字符串（如`app.js?d587bbd6e38337f5accd`），同样的结果也可被用于处理静态文件名和缓存失效。问号后边的部分能够让缓存失效。根据[Steve Souders](http://www.stevesouders.com/blog/2008/08/23/revving-filenames-dont-use-querystring/)的文章, 将哈希值附在文件名中更为高效。

{pagebreak}

## 设置哈希值（Setting Up Hashing）

The build needs tweaking to generate proper hashes. Images and fonts should receive `hash` while chunks should use `chunkhash` in their names to invalidate them correctly:

构建结果需要稍作调整以生成合适的哈希值。图片和字体接收`hash`而块文件（chunk）则应该在他们的名字中使用`chunkhash`以验证他们是否正确失效。

如果你也使用`chunkhash`来处理打包好的CSS，这将会产生问题，即指向CSS的代码通过JavaScript将其引至相同的入口。那意味着如果应用代码或者CSS发生改变，二者都将失效。因此，应该使用`contenthash`其基于提取的内容来生成哈希值，而不是使用`chunkhash`。

**webpack.config.js**

```javascript
const commonConfig = {
  ...
  parts.loadFonts({
    options: {
leanpub-start-delete
      name: '[name].[ext]',
leanpub-end-delete
leanpub-start-insert
      name: '[name].[hash:8].[ext]',
leanpub-end-insert
    },
  }),
  ...
};

const productionConfig = merge([
  {
    ...
leanpub-start-insert
    output: {
      chunkFilename: '[name].[chunkhash:8].js',
      filename: '[name].[chunkhash:8].js',
    },
leanpub-end-insert
  },
  ...
  parts.loadImages({
    options: {
      limit: 15000,
leanpub-start-delete
      name: '[name].[ext]',
leanpub-end-delete
leanpub-start-insert
      name: '[name].[hash:8].[ext]',
leanpub-end-insert
    },
  }),
  ...
]);
```

If you used `chunkhash` for the extracted CSS as well, this would lead to problems as the code points to the CSS through JavaScript bringing it to the same entry. That means if the application code or CSS changed, it would invalidate both. Therefore, instead of `chunkhash`, you can use `contenthash` that's generated based on the extracted content:

如果你也使用`chunkhash`来处理打包好的CSS，这将会产生问题，即指向CSS的代码通过JavaScript将其引至相同的入口。那意味着如果应用代码或者CSS发生改变，二者都将失效。因此，应该使用`contenthash`其基于提取的内容来生成哈希值，而不是使用`chunkhash`。

**webpack.parts.js**

```javascript
exports.extractCSS = ({ include, exclude, use }) => {
  // Output extracted CSS to a file
  const plugin = new ExtractTextPlugin({
leanpub-start-delete
    filename: '[name].css',
leanpub-end-delete
leanpub-start-insert
    filename: '[name].[contenthash:8].css',
leanpub-end-insert
  });

  ...
};
```

W> The hashes have been sliced to make the output fit better in the book. In practice, you can skip slicing them.

W> 哈希值经过切片操作使得输出结果能更好适配本书。实践中，你可以忽略切片。

{pagebreak}

If you generate a build now (`npm run build`), you should see something:

如果现在你生成构建（`npm run build`），你应该看到：

```bash
Hash: 16b92fddd41e579e77ba
Version: webpack 2.2.1
Time: 4258ms
                 Asset       Size  Chunks             Chunk Names
       app.e0f59512.js  811 bytes       1  [emitted]  app
  ...font.674f50d2.eot     166 kB          [emitted]
...font.af7ae505.woff2    77.2 kB          [emitted]
 ...font.fee66e71.woff      98 kB          [emitted]
  ...font.912ec66d.svg     444 kB          [emitted]
     logo.85011118.png      77 kB          [emitted]
         0.470796d5.js  408 bytes       0  [emitted]
  ...font.b06871f2.ttf     166 kB          [emitted]
    vendor.f897ca59.js    24.4 kB       2  [emitted]  vendor
      app.bf4d156d.css    2.54 kB       1  [emitted]  app
     0.470796d5.js.map    2.08 kB       0  [emitted]
   app.e0f59512.js.map    2.33 kB       1  [emitted]  app
  app.bf4d156d.css.map   93 bytes       1  [emitted]  app
vendor.f897ca59.js.map     135 kB       2  [emitted]  vendor
            index.html  301 bytes          [emitted]
   [4] ./~/object-assign/index.js 2.11 kB {2} [built]
  [14] ./app/component.js 461 bytes {1} [built]
  [15] ./app/shake.js 138 bytes {1} [built]
...
```

The files have neat hashes now. To prove that it works for styling, you could try altering *app/main.css* and see what happens to the hashes when you rebuild.

这些文件现在都有了一个整洁的哈希值。为了证明其对样式文件能够生效，你可以尝试变更*app/main.css*文件并在重新构建后看看哈希值是否发生变化。

There's one problem, though. If you change the application code, it invalidates the vendor file as well! Solving this requires extracting a **manifest**, but before that, you can improve the way the production build handles module IDs.

但仍然存在一个问题。如果你改变了应用的代码，基础库的也缓存失效了。解决这一问题需要提取**manifest**文件，但在那之前，你可以通过处理模块ID来提高生产版本的构建质量。

{pagebreak}

## 支持`HashedModuleIdsPlugin`（Enabling `HashedModuleIdsPlugin`）

Webpack uses number based IDs for the module code it generates. The problem is that they are difficult to work with and can lead to difficult to debug issues, particularly with hashing. This is why webpack provides two plugins. `NamedModulesPlugin` replaces module IDs with paths to the modules making it ideal for development. `HashedModuleIdsPlugin` does the same except it hashes the result and hides the path information.

Webpack使用数字来作为其生成的模块代码ID。这一问题在于其难以使用且难以进行问题处理。这也就是为什么webpack提供两个插件。`NamedModulesPlugin`在理想开发环境下通过模块路径来代替模块ID。`HashedModuleIdsPlugin`做法相同但是其哈希值隐藏了路径的相关信息。

The process keeps module IDs stable as they aren't derived based on order. You sacrifice a couple of bytes for a cleaner setup, but the trade-off is well worth it.

这一环节保证了模块ID的稳定由于其并不是基于顺序得到的。虽然使用一些字节来换取更为整洁的构建环节，但这一权衡是值得的。

Tweak the configuration as follows:

调整配置如下：

**webpack.config.js**

```javascript
const productionConfig = merge([
  {
    ...
leanpub-start-insert
    plugins: [
      new webpack.HashedModuleIdsPlugin(),
    ],
leanpub-end-insert
  },
  ...
]);
```

{pagebreak}

As you can see in the build output, the difference is negligible:

当你查看构建输出时，二者差异是微不足道的：

```bash
Hash: 11891d736f3749fb9f8f
Version: webpack 2.2.1
Time: 4115ms
                 Asset       Size  Chunks             Chunk Names
       app.4330d101.js  863 bytes       1  [emitted]  app
  ...font.912ec66d.svg     444 kB          [emitted]
  ...font.674f50d2.eot     166 kB          [emitted]
 ...font.fee66e71.woff      98 kB          [emitted]
...font.af7ae505.woff2    77.2 kB          [emitted]
     logo.85011118.png      77 kB          [emitted]
         0.b2a1fec0.js  430 bytes       0  [emitted]
  ...font.b06871f2.ttf     166 kB          [emitted]
    vendor.3c78d233.js    24.8 kB       2  [emitted]  vendor
      app.bf4d156d.css    2.54 kB       1  [emitted]  app
     0.b2a1fec0.js.map    2.08 kB       0  [emitted]
   app.4330d101.js.map    2.34 kB       1  [emitted]  app
  app.bf4d156d.css.map   93 bytes       1  [emitted]  app
vendor.3c78d233.js.map     135 kB       2  [emitted]  vendor
            index.html  301 bytes          [emitted]
[1Q41] ./app/main.css 41 bytes {1} [built]
[2twT] ./app/index.js 557 bytes {1} [built]
[5W1q] ./~/font-awesome/css/font-awesome.css 41 bytes {1} [built]
...
```

Note how the output has changed, though. Instead of numbers, you can see hashes. But this is expected given the change you made.

尽管无论输出如何改变。你也可以发现哈希值没变而不是数字没变。但这一改变正是所期望的。

T> The *Hot Module Replacement* appendix shows how to set up `NamedModulesPlugin` as it can be used for debugging HMR.

T>  *Hot Module Replacement* 附录展示了如何设置`NamedModulesPlugin`使其可以被用于调试HMR。

{pagebreak}

## 结论（Conclusion）

Including hashes related to the file contents to their names allows to invalidate them on the client side. If a hash has changed, the client is forced to download the asset again.

将文件内容相关的哈希值添加其文件名中使得客户端能够对他们进行验证。如果哈希值发生改变，则客户端强制再下载一次资源。

To recap:

* Webpack's **placeholders** allow you to shape filenames and enable you to include hashes to them.
* The most valuable placeholders are `[name]`, `[chunkhash]`, and `[ext]`. A chunk hash is derived based on the entry in which the asset belongs.
* If you are using `ExtractTextPlugin`, you should use `[contenthash]`. This way the generated assets get invalidated only if their content changes.
* `HashedModuleIdsPlugin` generates module IDs based on module paths. This is more stable than relying on the default order based numeric module IDs.

本章概括：

* Webpack**占位符**能够改变文件名并支持为其添加哈希值。
* 使用最多的占位符是`[name]`、`[chunkhash]`以及`[ext]`。块哈希值基于资源的入口文件得出。
* 如果你正在使用`ExtractTextPlugin`，你应该使用`[contenthash]`。当且仅当内容发生改变则这一方式构建的静态资源将失效。
* `HashedModuleIdsPlugin`基于模块路径来生成模块ID。这比依赖于默认的数字化模块ID顺序更稳定。

Even though the project generates hashes now, the output isn't flawless. The problem is that if the application changes, it invalidates the vendor bundle as well. The next chapter digs deeper into the topic and shows you how to extract a **manifest** to resolve the issue.

即使项目现在能够生成哈希值，但其输出仍不是完美的。问题在于若应用代码发生改变，其依赖库也会失效。下一章节将对该话题进行更深入挖掘并展示如何通过提取**manifest**来解决这一问题。