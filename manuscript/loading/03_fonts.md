# Loading Fonts
# 加载字体

Loading fonts is similar to loading images. It does come with special challenges, though. How to know what font formats to support? There can be up to four font formats to worry about if you want to provide first class support to each browser.

加载字体类似于加载图片。但是也伴随着挑战。如何知道需要提供那种字体？如果你希望所有浏览器都呈现出一流的效果，你需要考虑至少四种字体格式。

The problem can be solved by deciding a set of browsers and platforms that should receive first class service. The rest can use system fonts.

可以通过优先支持选定的浏览器和平台来解决这个问题，而让剩下的都使用系统字体。

You can approach the problem in several ways through webpack. You can still use *url-loader* and *file-loader* as with images. Font `test` patterns tend to be more complicated, though, and you have to worry about font file related lookups.

webpack可以让你用几种不同的方式来解决这个问题。你可以仍像处理图片那样使用*url-loader*和*file-loader*。字体的`test`规则将变得更加复杂，毕竟，你需要关心何如对字体文件进行查找方面的问题。

T> [canifont](https://www.npmjs.com/package/canifont) helps you to figure out which font formats you should support. It accepts a **browserslist** definition and then checks font support of each browser based on the definition.

T> [canifont](https://www.npmjs.com/package/canifont)可以帮助你分析出需要支持那种字体格式。它根据浏览器列表（**browserslist**）中的定义来确定需要为每个浏览器提供哪些字体。

## Choosing One Format
## 选择一个格式

If you exclude Opera Mini, all browsers support the *.woff* format. Its newer version, *.woff2*, is widely supported by modern browsers and can be a good alternative.

你过将Opera Mini排除，那么所有的浏览器都支持*.woff*格式。它的新版本*.woff2*，广泛的被现代浏览器支持，也可以作为一个选项。

{pagebreak}

Going with one format you can use a similar setup as for images and rely on both *file-loader* and *url-loader* while using the limit option:

如果仅使用一个格式，你可以基于*file-loader*和*url-loader*，设置是否内联的限制参数，使用和处理图片一样的设置：

```javascript
{
  test: /\.woff$/,
  loader: 'url-loader',
  options: {
    limit: 50000,
  },
},
```

A more elaborate approach to achieve a similar result that includes *.woff2* and others, would be to end up with code as below:

另一个更细致的方式也可以实现相同的效果，并且包含了*.woff2*和其它版本，代码如下：

```javascript
{
  // Match woff2 in addition to patterns like .woff?v=1.1.1.
  test: /\.(woff|woff2)(\?v=\d+\.\d+\.\d+)?$/,
  loader: 'url-loader',
  options: {
    // Limit at 50k. Above that it emits separate files
    limit: 50000,

    // url-loader sets mimetype if it's passed.
    // Without this it derives it from the file extension
    mimetype: 'application/font-woff',

    // Output below fonts directory
    name: './fonts/[name].[ext]',
  },
},
```

{pagebreak}

## Supporting Multiple Formats
## 支持多种字体格式

In case you want to make sure the site looks good on a maximum amount of browsers, you can use *file-loader* and forget about inlining. Again, it's a trade-off as you get extra requests, but perhaps it's the right move. Here you could end up with a loader configuration:

如果你想确保网站在最大程度上的所有浏览器中都看起来很棒，你可以是用*file-loader*并放弃内联的形式。这同样是一个有得有失的策略，因为你需要额外的请求，但也许这也是正确的选择。如下是加载器的配置：

```javascript
{
  test: /\.(ttf|eot|woff|woff2)$/,
  loader: 'file-loader',
  options: {
    name: 'fonts/[name].[ext]',
  },
},
```

The way you write your CSS definition matters. To make sure you are getting the benefit from the newer formats, they should become first in the definition. This way the browser picks them up.

书写CSS定义的方式也很重要。为了确保从优先使用较新的格式，应该将它们放在定义的前面。这样浏览器才能查找到它们。

```css
@font-face {
  font-family: 'myfontfamily';
  src: url('./fonts/myfontfile.woff2') format('woff2'),
    url('./fonts/myfontfile.woff') format('woff'),
    url('./fonts/myfontfile.eot') format('embedded-opentype'),
    url('./fonts/myfontfile.ttf') format('truetype');
    /* Add other formats as you see fit */
}
```

T> [MDN discusses the font-family rule](https://developer.mozilla.org/en/docs/Web/CSS/@font-face) in detail.

T> 可查看[MDN关于字体的详细规则](https://developer.mozilla.org/en/docs/Web/CSS/@font-face)。

{pagebreak}

## Manipulating *file-loader* Output Path and `publicPath`
## *file-loader*输出路径和`publicPath`配置项的操作

As discussed above and in [webpack issue tracker](https://github.com/webpack/file-loader/issues/32#issuecomment-250622904), *file-loader* allows shaping the output. This way you can output your fonts below `fonts/`, images below `images/`, and so on over using the root.

*file-loader*允许改变输出的形式，上面的例子中已经得以展示，在[webpack issue tracker](https://github.com/webpack/file-loader/issues/32#issuecomment-250622904)中也进行了讨论。这样你就可以将字体文件输出到`fonts/`，将图片文件输出到
`images/`目录中等等。

Furthermore, it's possible to manipulate `publicPath` and override the default per loader definition. The following example illustrates these techniques together:

还可以操作 `publicPath`选项并重载每个加载器的默认定义。下面的例子展示了综合展示了这个技术：

```javascript
{
  // Match woff2 in addition to patterns like .woff?v=1.1.1.
  test: /\.woff2?(\?v=\d+\.\d+\.\d+)?$/,
  loader: 'url-loader',
  options: {
    limit: 50000,
    mimetype: 'application/font-woff',

    // Output below the fonts directory
    name: './fonts/[name].[ext]',

    // Tweak publicPath to fix CSS lookups to take
    // the directory into account.
    publicPath: '../',
  },
},
```

## Generating Font Files Based on SVGs
## 生成基于SVGs的图片

If you prefer to use SVG based fonts, they can be bundled as a single font file by using [webfonts-loader](https://www.npmjs.com/package/webfonts-loader).

如果你更喜欢使用基于SVG的字体，可以通过[webfonts-loader](https://www.npmjs.com/package/webfonts-loader)将其打包成一个单独的字体文件。

W> Take care with SVGs if you have SVG specific image setup in place already. If you want to process font SVGs differently, set their definitions carefully. The *Loader Definitions* chapter covers alternatives.

W> 如果你已经设置了特定于SVG的图片加载，要用不同的方式来处理SVGs字体，就需要格外小心。*Loader Definitions*章节包含了可选的方法。

## Using Font Awesome
## 使用Font Awesome

The ideas above can be applied with [Font Awesome](https://www.npmjs.com/package/font-awesome). It's a collection of high-quality font icons you can refer to using CSS classes.

上面的想法可以结合[Font Awesome](https://www.npmjs.com/package/font-awesome)应用。它是一个可以用CSS类名来引用的高质量的图表字体，

### Integrating Font Awesome to the Project
### 在项目中集成Font Awesome

To integrate Font Awesome to the book project, install it first:

要在本书的项目中集成Font Awesome，首先要安装它:

```bash
npm install font-awesome --save
```

Given Font Awesome doesn't define a `main` field in its *package.json* file, you need to point to it through a direct path instead of package name alone.

由于Font Awesome并没有在其*package.json*文件中定义`main`字段，你需要通过直接使用路径来引用它，而不像其他类库一样可以用名称引用。

Refer to Font Awesome as follows:

可以下引用Font Awesome到项目中：

**app/index.js**

```javascript
leanpub-start-insert
import 'font-awesome/css/font-awesome.css';
leanpub-end-insert
...
```

Font Awesome includes Sass and Less versions as well, but given you have not set up either, this definition is enough.

Font Awesome也包含Sass和Less的版本，但是目前你都没有设置，所以上面的定义就足够了。

T> The `import` could be cleaned up as `import 'font-awesome'` by setting up a `resolve.alias`. The *Package Consuming Techniques* chapter discusses this idea in detail.

T> `import`语句可以通过设置一个`resolve.alias`简化为`import 'font-awesome'`。在*Package Consuming Techniques*章节详细的讨论的这个想法。

{pagebreak}

If you run the project now (`npm start`), webpack should give a long list of errors:

如果你现在运行项目(`npm start`)，webpack将会抛出一长串的错误：

```bash
ERROR in ./~/font-awesome/fonts/fontawesome-webfont.woff2?v=4.7.0
Module parse failed: .../node_modules/font-awesome/fonts/fontawesome-webfont.woff2?v=4.7.0 Unexpected character '' (1:4)
You may need an appropriate loader to handle this file type.
(Source code omitted for this binary file)
 @ ./~/css-loader!./~/font-awesome/css/font-awesome.css 6:479-532
 @ ./~/font-awesome/css/font-awesome.css
 @ ./app/index.js
 @ multi (webpack)-dev-server/client?http://localhost:8080 webpack/hot/only-dev-server react-hot-loader/patch ./app
```

### Implementing Webpack Configuration
### 集成到Webpack的配置中

The result is expected as you haven't configured loaders for any of Font Awesome fonts yet and webpack doesn't know what to do with the files in question. To match the files and map them through *file-loader*, attach the following snippet to the project:

上面的结果是因为你并没有为Font Awesome的字体配置任何加载器，webpack并不知道如何处理这些文件。要匹配这些文件并通过*file-loader*映射它们，请将以下代码段添加到项目中：

**webpack.parts.js**

```javascript
exports.loadFonts = ({ include, exclude, options } = {}) => ({
  module: {
    rules: [
      {
        // Capture eot, ttf, woff, and woff2
        test: /\.(eot|ttf|woff|woff2)(\?v=\d+\.\d+\.\d+)?$/,
        include,
        exclude,

        use: {
          loader: 'file-loader',
          options,
        },
      },
    ],
  },
});
```

The idea is the same as for loading images. This time around you match font files. If you wanted, you could refactor the commonality to a function to share between the two.

这个想法和加载图片类似。这是这一次你处理的是字体文件。如果你愿意，你也可以将其封装到一个函数中，让二者可以共享这个函数。

You still need to connect the above with the main configuration:

你还需要将上述的代码段关联到主配置文件中：

**webpack.config.js**

```javascript
const commonConfig = merge([
  ...
leanpub-start-insert
  parts.loadFonts({
    options: {
      name: '[name].[ext]',
    },
  }),
leanpub-end-insert
]);
```

The project should run (`npm start`) without any errors now.

再次运行(`npm start`)项目就不会报错了：

To see Font Awesome in action, adjust the application as follows:

为了实际查看Font Awesome的效果，如下调整应用：

**app/component.js**

```javascript
export default (text = 'Hello world') => {
  const element = document.createElement('div');

leanpub-start-delete
  element.className = 'pure-button';
leanpub-end-delete
leanpub-start-insert
  element.className = 'fa fa-hand-spock-o fa-1g';
leanpub-end-insert
  element.innerHTML = text;

  return element;
}
```

{pagebreak}

If you build the application (`npm run build`), you should see that it processed and Font Awesome assets were included.

如果你现在构建应用(`npm run build`)，你将会看到它已经处理并且包含了Font Awesome资源。

```bash
Hash: e379b2c5a9f46663f367
Version: webpack 2.2.1
Time: 2547ms
        Asset       Size  Chunks                    Chunk Names
  ...font.eot     166 kB          [emitted]
  ...font.ttf     166 kB          [emitted]
...font.woff2    77.2 kB          [emitted]
 ...font.woff      98 kB          [emitted]
  ...font.svg     444 kB          [emitted]  [big]
     logo.png      77 kB          [emitted]
       app.js    4.22 kB       0  [emitted]         app
      app.css    7.72 kB       0  [emitted]         app
   index.html  227 bytes          [emitted]
   [0] ./app/component.js 185 bytes {0} [built]
   [1] ./~/font-awesome/css/font-awesome.css 41 bytes {0} [built]
   [2] ./app/main.css 41 bytes {0} [built]
...
```

The SVG file included in Font Awesome has been marked as `[big]`. It's beyond the performance budget defaults set by webpack and the topic is discussed in detail in the *Minifying* chapter.

包含的SVG文件被标记为`[big]`。它超过了webpack设置的性能预设值，在*Minifying*章节将会详细的讨论这个问题。

T> [font-awesome-loader](https://www.npmjs.com/package/font-awesome-loader) allows more customization. Font Awesome 5 improves the situation further and make it easier to decide what fonts to consume. [Font Awesome wiki](https://github.com/FortAwesome/Font-Awesome/wiki/Customize-Font-Awesome) points to available online services that allow you to select specific fonts from Font Awesome collection.

T> [font-awesome-loader](https://www.npmjs.com/package/font-awesome-loader)允许更多的自定义设置。Font Awesome 5改善了现状并让决策使用哪种字体变得更加简单。[Font Awesome wiki](https://github.com/FortAwesome/Font-Awesome/wiki/Customize-Font-Awesome)提供了可靠的在线服务，让你可以从Font Awesome集合中选择特定的字体。

## Conclusion
## 小结

Loading fonts is similar to loading other assets. You have to consider the browsers you want to support and choose the loading strategy based on that.

加载字体类似于加载其他资源。你需要考虑的是要支持那些浏览器，并基于此来选择加载策略。

To recap:

回顾：

* When loading fonts, the same techniques as for images apply. You can choose to inline small fonts while bigger ones are served as separate assets.

* 当加载字体时，应用了和加载图片相似的技术。你可以选择内联较小的字体而将大的文件处理成单独的资源。

* If you decide to provide first class support to only modern browsers, you can select only a font format or two and let the older browsers to use system level fonts.

* 如果你决定只为现代浏览器提供一流的支持，你可以只选择一到两种字体，而让其它老的浏览器使用系统字体。

* Using larger font collections, such as Font Awesome, can be problematic especially if you want to avoid loading additional rules. The problem is dependent on the packages in question and can be solved with webpack to an extent.

* 要使用大的字体集合，如Font Awesome，比较麻烦，特别是当你要避免加载额外的规则时。这些问题取决于所讨论的依赖包并可以使用webpack在一定程度上解决问题。

In the next chapter, you'll learn to load JavaScript using Babel and webpack. Webpack loads JavaScript by default but there's more to the topic as you have to consider what browsers you want to support.

在下一章节，你将会学习如何使用Babel和webpack加载JavaScript。虽然webpack默认就可以加载JavaScript，但是有更多话题需要探讨，比如你也需要考虑支持那些浏览器。
