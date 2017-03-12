# Linting CSS
# 检测CSS语法

As discussed earlier in the *Linting JavaScript* chapter, linting is a technique that allows us to avoid certain categories of mistakes. Automation is good, as it can save effort. In addition to JavaScript, it’s possible to lint CSS.
正如之前在*Linting JavaScript*章节所讨论的，语法检测是一个可以让我们避免某些类型的错误的技术。自动化可以让开发者省力。除了JavaScript，我们也可以检测CSS.

[stylelint](http://stylelint.io/) is a tool that allows linting. It can be used with webpack through [postcss-loader](https://www.npmjs.com/package/postcss-loader).
[stylelint](http://stylelint.io/)是一个可以用来实现语法检测的工具。它可以通过[postcss-loader](https://www.npmjs.com/package/postcss-loader)加载器结合webpack使用。


## Connecting stylelint with *package.json*

To get started, install stylelint as a development dependency:
首先，请将stylelint作为开发依赖关系安装：

```bash
npm install stylelint --save-dev
```

To connect stylelint with npm and make it find our CSS files, adjust as follows:
连接stylelint和npm，并让它找到你的CSS文件，如下调整：

**package.json**

```json
...
"scripts": {
leanpub-start-insert
  "lint:style": "stylelint app/**/*.css",
leanpub-end-insert
  ...
},
...
```

To have something to test with, we should define a dummy rule:
为了有一些东西测试，我们要定义一个虚拟规则：

**.stylelintrc**

```json
{
  "rules": {
    "color-hex-case": "lower"
  }
}
```

If you break the rule at *app/main.css* and run `npm run lint:style`, you should see something like this:
如果你在*app/main.css*违反了规则，并运行`npm run lint：style`，你应该看到这样：

```bash
...

app/main.css
 2:15  Expected "#FFF" to be "#fff"   color-hex-case

...
```

To get less verbose output on error, use either `npm run lint:style --silent` or `npm run lint:style -s`.
为了避免在错误时得到冗长的输出，使用`npm run lint：style --silent`或`npm run lint：style -s`。

The same rules can be connected with webpack.
相同的规则可以同webpack关联起来。

## Connecting Stylelint with Webpack

To get started, install *postcss-loader* unless you have it set up already:
首先，请安装* postcss-loader *，如果您已经设置它，请忽略：

```bash
npm install postcss-loader --save-dev
```

Next, to integrate with configuration, set up a part first:
接下来，要与配置集成，首先设置parts文件：

**webpack.parts.js**

```javascript
...

exports.lintCSS = function({ include, exclude }) {
  return {
    module: {
      rules: [
        {
          test: /\.css$/,
          include,
          exclude,
          enforce: 'pre',

          loader: 'postcss-loader',
          options: {
            plugins: () => ([
              require('stylelint')({
                // Ignore node_modules CSS
                ignoreFiles: 'node_modules/**/*.css',
              }),
            ]),
          },
        },
      ],
    },
  };
};
```

Then add it to the common configuration:
在common configuration如下添加：

**webpack.config.js**

```javascript
...

const commonConfig = merge([
  ...
leanpub-start-insert
  parts.lintCSS({ include: PATHS.app }),
leanpub-end-insert
]);

...
```

If you define a CSS rule, such as `background-color: #EFEFEF;` at *main.css* now, you should see a warning like this at your terminal when you run the build (`npm start` or `npm run build`):
如果你现在在*main.css*定义一个CSS规则，如`background-color：#EFEFEF;`，当你运行构建时，你应该在终端上看到类似这样的警告（`npm start`或`npm run build`）：


```bash
WARNING in ./~/css-loader!./~/postcss-loader!./app/main.css
stylelint: /webpack-demo/app/main.css:2:21: Expected "#EFEFEF" to be "#efefef" (color-hex-case)
 @ ./app/main.css 4:14-117
 @ ./app/index.js
```

See the stylelint documentation for a full list of rules. npm lists [possible stylelint rulesets](https://www.npmjs.com/search?q=stylelint-config) you can enable through configuration.
有关规则的完整列表，请参阅stylelint文档。 npm列表[可用的stylelint规则集](https://www.npmjs.com/search?q=stylelint-config)，您可以通过配置启用。

T> [stylelint-scss](https://www.npmjs.com/package/stylelint-scss) provides a collection of SCSS specific linting rules.
T> [stylelint-scss]（https://www.npmjs.com/package/stylelint-scss）提供了SCSS特定检测规则的集合。

T> The `enforce` idea is discussed in detail in the *Loader Definitions* chapter.
T> “加载器定义”一章中详细讨论了`enforce`构想。

W> If you get `Module build failed: Error: No configuration provided for ...` kind of error, check your *.stylelintrc*.
W> 如果你得到“模块构建失败：错误：没有提供...配置...”种错误，检查你的*.stylelintrc*。

## *stylelint-webpack-plugin*

[stylelint-webpack-plugin](https://www.npmjs.com/package/stylelint-webpack-plugin) is an alternate way to achieve the same result. Its greatest advantage over the setup above is that it will follow possible `@import` statements you might have in your styling.
[stylelint-webpack-plugin](https://www.npmjs.com/package/stylelint-webpack-plugin)是实现相同结果的替代方法。 它的最大优点是，it will follow possible `@import` statements you might have in your styling.

T> [stylelint-bare-webpack-plugin](https://www.npmjs.com/package/stylelint-bare-webpack-plugin) is a variant of *stylelint-webpack-plugin* that allows you to control the version of stylelint you are using.
T> [stylelint-bare-webpack-plugin](https://www.npmjs.com/package/stylelint-bare-webpack-plugin)是*stylelint-webpack-plugin*的一个变体，它允许您控制正在使用的stylelint的版本。

## csslint

[csslint](http://csslint.net/) is another option to *stylelint*. It can be used through [csslint-loader](https://www.npmjs.com/package/csslint-loader) and follows a normal loader setup.
[csslint](http://csslint.net/)是*stylelint*的另一个选项。它可以通过[csslint-loader](https://www.npmjs.com/package/csslint-loader) 使用，并遵循正常的加载器设置。

## Conclusion
## 小结

After these changes, we have linting for our styling in place. Now you can catch CSS-related problems.
在这些变化之后，我们已经可以为我们的样式进行检测。 现在你可以捕获CSS相关的问题。

To recap:
回顾：

* It is possible to lint CSS through *stylelint*
* 通过*stylelint*可以对CSS进行语法检测

* Linting CSS allows you to capture common CSS-related problems and disallow problematic patterns.
* Linting CSS允许您捕获常见的CSS相关的问题和禁止在代码中出现这些类型的问题。

* *stylelint* can be treated as a PostCSS plugin, but it can also be used through *stylelint-webpack-plugin*.
* *stylelint*可以看作是一个PostCSS插件，但它也可以通过*stylelint-webpack-plugin*使用。

* *csslint* is an option to *stylelint*. It possible [the projects will merge](https://github.com/CSSLint/csslint/issues/668), though.
* *csslint*是*stylelint*的一个选项。 它可能会[项目将合并](https://github.com/CSSLint/csslint/issues/668)。