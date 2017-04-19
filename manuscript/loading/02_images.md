# Loading Images
# 加载图片

HTTP/1 application can be made slow by loading a lot of small assets. Each request comes with an overhead. HTTP/2 helps in this regard and changes the situation somewhat drastically. Till then you are stuck with different approaches. Webpack allows a couple of these. They are particularly relevant for loading images.

基于HTTP/1的应用，在加载很多小资源后将变会变得很慢。每一次请求都伴随着开销。HTTP/2在这方面有一些帮助，并有一些显著的变化。在使用HTTP/2之前，你仍需要继续坚持使用一些不同的方式。Webpack允许你使用其中的一些方法。它们与图片加载尤其相关。

Webpack allows you to inline assets by using [url-loader](https://www.npmjs.com/package/url-loader). It emits your images as base64 strings within your JavaScript bundles. The process decreases the number of requests needed while growing the bundle size. It's enough to use *url-loader* during development. You want to consider other alternatives for the production build, though.

Webpack允许你通过[url-loader](https://www.npmjs.com/package/url-loader)将资源内联到JavaScript中。它将你的图片生成基于64位的字符串，并注入到你的JavaScript代码包中。这个处理在增加构建包的同时减少了请求的数量。在开发环境中使用*url-loader*就足以满足需要。在生产环境中，你需要考虑其它的方法。

Webpack gives control over the inlining process and can defer loading to [file-loader](https://www.npmjs.com/package/file-loader). *file-loader* outputs image files and returns paths to them instead of inlining. This technique works with other assets types, such as fonts, as you see in the later chapters.

## Setting Up *url-loader*
## 设置*url-loader*

*url-loader* is a good starting point and it's the perfect option for development purposes, as you don't have to care about the size of the resulting bundle. It comes with a *limit* option that can be used to defer image generation to *file-loader* after a certain limit's reached. This way you can inline small files to your JavaScript bundles while generating separate files for the bigger ones.

*url-loader*是一个很好的开始，它是开发环境中的最佳选择，因为你不必担心最终的代码包的体积。它提供了一个*limit*选项，当一个特定的限制达到后可以使图片生成延迟并交给*file-loader*。这样，你就可以将较小的图片内联到你的JavaScript代码包中而将大的图片生成为单独的图片文件。

If you use the limit option, you need to install both *url-loader* and *file-loader* to your project. Assuming you have configured your styles correctly, webpack resolves any `url()` statements your styling contains. You can point to the image assets through your JavaScript code as well.

如果你使用了限制项，你需要在项目中同时安装*url-loader* 和 *file-loader*。假设你已经正确的配置了样式，webpack将会处理你的样式代码中包含的任何`url()`语句。你也通过JavaScript代码来引用图片资源。

In case the `limit` option is used, *url-loader* passes possible additional options to *file-loader* making it possible to configure its behavior further.

如果`limit`选项被使用了，*url-loader*将可能的附加选项传递给*file-loader*，可以进一步配置其行为。

To load *.jpg*, *.png*, and *.svg* files while inlining files below 25kB, you would have to set up a loader:

要内联加载小于25kB的*.jpg*，*.png*和*.svg*文件，你可以如下设置一个加载器：

```javascript
{
  test: /\.(jpg|png|svg)$/,
  loader: 'url-loader',
  options: {
    limit: 25000,
  },
},
```

## Setting Up *file-loader*
## 设置*file-loader*

If you want to skip inlining altogether, you can use *file-loader* directly. The following setup customizes the resulting filename. By default, *file-loader* returns the MD5 hash of the file's contents with the original extension:

如果你不希望使用内联的形式，你可以直接使用*file-loader*。下面的设置对最终的文件名进行了自定义。默认情况下，*file-loader*将返回文件内容的MD5哈希值和原始的文件后缀组成的文件名。

```javascript
{
  test: /\.(jpg|png|svg)$/,
  loader: 'file-loader',
  options: {
    name: '[path][name].[hash].[ext]',
  },
},
```

T> If you want to output your images below a particular directory, set it up with `name: './images/[hash].[ext]'`.

T> 如果你希望将你的图片输出到特定的目录下，可以这样设置：`name: './images/[hash].[ext]'`。

W> Be careful not to apply both loaders on images at the same time! Use the `include` field for further control if *url-loader* `limit` isn't enough.

W> 要注意，不要同时对图片使用两个加载器！如果`limit`选项不足以满足需要，可以使用`include`字段来进一步控制。

## Integrating Images to the Project
## 集成图片到项目中

The ideas above can be wrapped in a small helper that can be incorporated into the book project. To get started, install the dependencies:

上面的想法可以被报过到一个小小助手中，然后并本书的项目中。首依赖：

```bash
npm install file-loader url-loader --save-dev
```

Set up a function as below:

构建一个如下的函数：

**webpack.parts.js**

```javascript
exports.loadImages = ({ include, exclude, options } = {}) => ({
  module: {
    rules: [
      {
        test: /\.(png|jpg|svg)$/,
        include,
        exclude,

        use: {
          loader: 'url-loader',
          options,
        },
      },
    ],
  },
});
```

To attach it to the configuration, adjust as follows. The configuration defaults to *url-loader* during development and uses both *url-loader* and *file-loader* in production to maintain smaller bundle sizes. *url-loader* uses *file-loader* implicitly when `limit` is set and both have to be installed for the setup to work.

要将其连接到项目中，如下进行调整。下面的配置默认在开发环境中使用*url-loader*，而在生产环境中同时使用*url-loader*和*file-loader*来维持一个较小代码包体积。

{pagebreak}

**webpack.config.js**

```javascript
const productionConfig = merge([
  ...
leanpub-start-insert
  parts.loadImages({
    options: {
      limit: 15000,
      name: '[name].[ext]',
    },
  }),
leanpub-end-insert
]);

const developmentConfig = merge([
  ...
leanpub-start-insert
  parts.loadImages(),
leanpub-end-insert
]);
```

To test that the setup works, download an image or generate it (`convert -size 100x100 gradient:blue logo.png`) and refer to it from the project:

为了验证上述构建的有效性，下载一个图片或者生成一个图片(`convert -size 100x100 gradient:blue logo.png`)，并在项目中引用这个图片。

**app/main.css**

```css
body {
  background: cornsilk;
leanpub-start-insert
  background-image: url('./logo.png');
  background-repeat: no-repeat;
  background-position: center;
leanpub-end-insert
  display: flex;
}
```

The behavior changes depending on the `limit` you set. Below the limit, it should inline the image while above it should emit a separate asset and a path to it. The CSS lookup works because of *css-loader*. You can also try importing the image from JavaScript code and see what happens.

行为的改变取决于你设置的`limit`配置项。小于这个限制，图片将被内联，而上面的例子，将生成单独的图片资源以及资源的路径。*css-loader*使得CSS可以查找到这个图片。你也可以在JavaScript代码中引用这个图片看看将会发生什么。

## Loading SVGs
## 加载SVGs

Webpack allows a [couple ways](https://github.com/webpack/webpack/issues/595) to load SVGs. However, the easiest way is through *file-loader* as follows:

Webpack允许使用[多种方式](https://github.com/webpack/webpack/issues/595)来加载SVGs。然而，最简单的方式就是如下使用*file-loader*:

```javascript
{
  test: /\.svg$/,
  use: 'file-loader',
},
```

Assuming you have set up your styling correctly, you can refer to your SVG files as below. The example SVG path below is relative to the CSS file:

假设你已经正确的设置了你的样式部分，你可以的如下引用你的SVG文件。例子中的SVG路径是相对于CSS文件的。

```css
.icon {
   background-image: url('../assets/icon.svg');
}
```

If you want the raw SVG content, you can use the [raw-loader](https://www.npmjs.com/package/raw-loader) for this purpose. [svg-inline-loader](https://www.npmjs.com/package/svg-inline-loader) goes a step further and eliminates unnecessary markup from your SVGs. These loaders can be valuable if you want to inject the SVG content to directly to JavaScript or HTML markup.

如果你想得到原生的SVG内容，可以使用[raw-loader](https://www.npmjs.com/package/raw-loader)来实现。[svg-inline-loader](https://www.npmjs.com/package/svg-inline-loader)更进一步并为你去除了不必要的SVGs标记。如果你想将SVG的内容直接插入到JavaScript或者HTML标签中，这些加载器将非常有用。

[svg-sprite-loader](https://www.npmjs.com/package/svg-sprite-loader) can merge separate SVG files into a single sprite, making it potentially more efficient to load as you avoid request overhead. It supports raster images (*.jpg*, *.png*) as well.

[svg-sprite-loader](https://www.npmjs.com/package/svg-sprite-loader)可以将单独的SVG文件合并到一个图片中，潜在的使其可以避免请求的开销而更高效的加载。它也可以支持光栅图片(*.jpg*, *.png*)。

Translate> 光栅图片相对于矢量图片。

[react-svg-loader](https://www.npmjs.com/package/react-svg-loader) emits SVGs as React components meaning you could end up with code like `<Image width={50} height={50}/>` to render a SVG in your code after importing it.

[react-svg-loader](https://www.npmjs.com/package/react-svg-loader)将SVGs转化为React组件，让你可以在代码中将其引入后，使用`<Image width={50} height={50}/>`这样的代码来渲染SVG。

T> You can still use *url-loader* and the tips above with SVGs too.

T> 你仍然可以使用*url-loader*和上述的提示来加载SVGs。

## Optimizing Images
## 优化图片

In case you want to compress your images, use [image-webpack-loader](https://www.npmjs.com/package/image-webpack-loader), [svgo-loader](https://www.npmjs.com/package/svgo-loader) (SVG specific), or [imagemin-webpack-plugin](https://www.npmjs.com/package/imagemin-webpack-plugin). This type of loader should be applied first to the data, so remember to place it as the last within `use` listing.

如果你想对图片进行压缩，可以使用[image-webpack-loader](https://www.npmjs.com/package/image-webpack-loader)，[svgo-loader](https://www.npmjs.com/package/svgo-loader) (仅限于SVG), 或者 [imagemin-webpack-plugin](https://www.npmjs.com/package/imagemin-webpack-plugin)。这些类型的加载器应该首先被应用于数据，所以请将其置于`use`字段的最后。

Compression is particularly valuable for production builds as it decreases the amount of bandwidth required to download your image assets and speed up your site or application as a result.

压缩在生产环境尤为重要，因为可以大大减少下载图片资源占用的带宽，进而提高站点或应用的速度。

## Utilizing `srcset`
## 使用`srcset`

[resize-image-loader](https://www.npmjs.com/package/resize-image-loader) and [responsive-loader](https://www.npmjs.com/package/responsive-loader) allow you to generate `srcset` compatible collections of images for modern browsers. `srcset` gives more control to the browsers over what images to load and when resulting in higher performance.

[resize-image-loader](https://www.npmjs.com/package/resize-image-loader)和[responsive-loader](https://www.npmjs.com/package/responsive-loader)让你可以为现代浏览器生成`srcset`兼容的图像集合。`srcset`属性让浏览器可以更好的控制应该加载什么样的图片，同时提升了性能。

## Loading Images Dynamically
## 动态加载图片

Webpack allows you to load images dynamically based on a condition. The techniques covered in the *Code Splitting* chapter are enough for this purpose. Doing this can save bandwidth and load images only when you need them or preload them while you have time.

Webpack让你可以根据条件动态的加载图片。涵盖在*Code Splitting*章节中的技术就可以满足这个需求。

## Getting Image Dimensions
## 获得图片的尺寸

Sometimes getting the only reference to an image isn't enough. [image-size-loader](https://www.npmjs.com/package/image-size-loader) emits image dimensions, type, and size in addition to the reference to the image itself.

有时仅仅获得图片的引用还不够。[image-size-loader](https://www.npmjs.com/package/image-size-loader)可以在图片的引用中额外生成图片的尺寸、类型、大小信息。

## Referencing to Images
## 图片的引用

Webpack can pick up images from style sheets through `@import` and `url()` assuming *css-loader* has been configured. You can also refer to your images within code. In this case, you have to import the files explicitly:

如果*css-loader*已经被配置了，Webpack可以通过`@import`和`url()`语法从样式表中查找图片。你也可以在代码中直接引用图片。如此，你需要明确的引入图片文件：

```javascript
const src = require('./avatar.png');

// Use the image in your code somehow now
const Profile = () => (
  <img src={src} />
);
```

If you are using React, then you use [babel-plugin-transform-react-jsx-img-import](https://www.npmjs.com/package/babel-plugin-transform-react-jsx-img-import) to generate the `require` automatically. In that case, you would end up with code:

如果你使用React，你可以使用[babel-plugin-transform-react-jsx-img-import](https://www.npmjs.com/package/babel-plugin-transform-react-jsx-img-import)来自动生成`require`语句。如此，你的代码将如下呈现：

```javascript
const Profile = () => (
  <img src="avatar.png" />
);
```

It's also possible to set up dynamic imports as discussed in the *Code Splitting* chapter. Here's a small example:

如*Code Splitting*章节所讨论的那样，动态导入图片也是可行的。如下是一个简单的例子：

```javascript
// The name of the avatar is received from somewhere
const src = require(`./avatars/${avatar}`);

...
```

{pagebreak}

## Loading Sprites
## 加载Sprites

**Spriting** technique allows you to combine multiple smaller images into a single image. It has been used for games to describe animations and it's valuable for web development as well as you avoid request overhead.

**Spriting**技术让你可以将多个小图片组合到一个单独的图片中。它已经在游戏中被应用于描述动画，在web的开发中该技术也很有意义，因为它可以避免请求的开销。

[webpack-spritesmith](https://www.npmjs.com/package/webpack-spritesmith) converts provided images into a sprite sheet and Sass/Less/Stylus mixins. You have to set up a `SpritesmithPlugin`, point it to target images, and set the name of the generated mixin. After that, your styling can pick it up:

[webpack-spritesmith](https://www.npmjs.com/package/webpack-spritesmith)将所提供的图片转换为sprite表并和Sass/Less/Stylus结合起来。你需要设置`SpritesmithPlugin`插件，指向目标图片，并设置生成的混合文件。然后，你的样式文件就可以查找到这些图片了：


```sass
@import '~sprite.sass';

.close-button {
  sprite($close);
}

.open-button {
  sprite($open);
}
```

## Images and *css-loader* Source Map Gotcha
## 图片和*css-loader*的Source Map的疑惑

If you are using images and *css-loader* with the `sourceMap` option enabled, it's important that you set `output.publicPath` to an absolute value pointing to your development server. Otherwise, images aren't going to work. See [the relevant webpack issue](https://github.com/webpack/style-loader/issues/55) for further explanation.

如果你将图片和*css-loader*一起使用，并开启了*css-loader*的`sourceMap`选项，那么将`output.publicPath`设置为一个指向你的开发服务器的绝对路径就很重要了。否则，图片将不会生效。详细解释请看这个[已知的问题](https://github.com/webpack/style-loader/issues/55)。

{pagebreak}

## Conclusion
##  小结

Webpack allows you to inline images within your bundles when needed. Figuring out proper inlining limits for your images requires experimentation. You have to balance between bundle sizes and the number of requests.

Webpack可以让你在需要的时候将图片内联到代码包中。总结出一个合适的将图片内联的限制值需要一些尝试。你需要权衡代码包的体积和请求的数量。

To recap:

回顾：

* *url-loader* inlines the assets within JavaScript. It comes with a `limit` option that allows you to defer assets above it to *file-loader*.
* *url-loader*将资源内联到JavaScript代码中。它提供了一个`limit`选线，可以让你将上述的资源延迟交给*file-loader*处理。

* *file-loader* emits image assets and returns paths to them to the code. It allows hashing the asset names.
* *file-loader* 生成图片资源并想代码返回资源的路径。它还可以像资源名称中增加哈希值。

* You can find image optimization related loaders and plugins that allow you to tune their size further.
* 你可以找到图片优化相关的加载器和插件，他们使你可以进一步调节图片的大小。

* It's possible to generate **sprite sheets** out of smaller images to combine them into a single request.
* 可以将很多小图片组合成一个单独的图片来生成**sprite sheets**

* Webpack allows you to load images dynamically based on a given condition.
* Webpack使你可以基于给定的条件来动态的加载图片

* If you are using source maps, you should remember to set `output.publicPath` to an absolute value for the images to show up.
* 如果你使用了source maps，你要记住将`output.publicPath`设置为一个绝对路径以保证图片可以显示。

You'll learn to load fonts using webpack in the next chapter.

你将在下一章节学习如何使用webpack来加载字体。
