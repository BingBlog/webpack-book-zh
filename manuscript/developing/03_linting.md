# Linting JavaScript
# 检查JavaScript代码

Linting is one of those techniques that can help you make fewer mistakes while coding JavaScript. You can spot issues before they become actual problems. Modern editors and IDEs offer strong support for popular tools allowing you to detect possible issues as you are developing.

检查代码这些技术中一种可以帮助你在编写JavaScript时少犯错误的技术。你可以在其造成为真正的问题之前就注意到它。现代的编辑器和集成开发环境都为广受欢迎的工具提供了强有力的支持，让你可以在开发时检查可能的问题。

Despite this, it's a good idea to set them up with webpack or at least in a separate task that gets run regularly. That allows you to cancel a production build that is not up to your standards.

初次之外，在webpack中设置这个功能，或者至少在一个单独的任务中定期运行，都是一个不错的主意。这可以让你避免生成一个不符合你的标准的生产构建。

## Brief Introduction to ESLint
## ESlint简介

The linter that started it all for JavaScript is Douglas Crockford's [JSLint](http://www.jslint.com/). JSLint is known to be opinionated like Douglas himself. The next step in evolution was [JSHint](http://jshint.com/), which took the opinionated edge out of JSLint and allowed for more customization. [ESLint](http://eslint.org/) is the newest tool in vogue, and it goes even further.

JavaScript的检测器始于Douglas Crockford的[JSLint](http://www.jslint.com/)。JSLint像Douglas被人以严苛闻名。接下来的革新是[JSHint](http://jshint.com/)，它从JSLint中脱颖而出，并允许更多的自定义。[ESLint](http://eslint.org/)是时下最新的一个，它的进步更大。

### ESLint Is Customizable
### ESLint是可自定义的

ESLint goes to the next level as it allows you to implement custom rules, parsers, and reporters. ESLint works with JSX syntax making it a good fit for React projects. You have to use a Babel specific parser with custom language features, although ESLint supports ES6 out of the box.

ESLint因其可以让你实现自定义的规则、解析、报告而变得更高级。将其同JSX一起使用，非常适用于React项目。你必须使用具有自定义语言功能的Babel解析器，尽管ESLint支持ES6的开箱即用。

ESLint rules have been documented well, and you have full control over their severity. These features alone make it a powerful tool. Better yet, there is a significant number of rules and presets beyond the core as the community has built on top of it.

ESLint的规则被很好地编写成了文档，你完全可以自己控制他们的重要性。仅该特性就使其成为了一个很强大的工具。更完美的是，社区已经基于它创建了大量的除了其核心之外的规则和预设器。

T> It's telling that a competing project, JSCS, [decided to merge its efforts with ESLint](http://eslint.org/blog/2016/04/welcoming-jscs-to-eslint). JSCS reached the end of its life with its 3.0.0 release, and the core team joined with ESLint.

T> 该消息告知了一个与其竞争的项目，JSCS， [决定将其合并到ESLint中](http://eslint.org/blog/2016/04/welcoming-jscs-to-eslint)。JSCS在3.0.0版本发布后结束了其生命，核心的团队都加入到了ESLint团队中。

### eslint-config-airbnb

[eslint-config-airbnb](https://www.npmjs.com/package/eslint-config-airbnb) is a good example of a popular preset. Often it's enough to find a preset you like, tweak it to your liking with local rules or by deriving a preset of your own based on it, and then using that. This way you don't have to worry so much about all the available functionality.

[eslint-config-airbnb](https://www.npmjs.com/package/eslint-config-airbnb)就是广受欢迎的预设项的一个好例子。通常情况下，你可以找到一个你喜欢的预设项，用一个本地的规则来调整它，或者基于此开发一个你自己的预设，然后再使用。如此，你不必担心所有可用的功能了。

T> [eslint-config-cleanjs](https://www.npmjs.com/package/eslint-config-cleanjs) is a good example of how you can use ESLint to restrict JavaScript to a purely functional subset.

T> [eslint-config-cleanjs](https://www.npmjs.com/package/eslint-config-cleanjs)是一个很好的例子，阐释了如何使用ESLint来限制JavaScript为纯函数式的风格。

## Linting Is about More than Catching Issues
## 检查的意义远不仅在于发现问题

Besides linting for issues, it can be valuable to manage the code style. Nothing is more annoying than having to work with source code that has mixed tabs and spaces. Stylistically consistent code reads better. When integrated with an Editor or IDE linting also points out your mistakes as you make them, making it easier to adjust your technique and avoid making more of the same errors.

除了来检查问题外，更大的意义在于管理代码风格。没有什么比工作时，看到的全是Tab和空格混淆的源码，更烦人的了。风格一致的代码更容易阅读。当与一个编辑器或者一个集成开发环境集成时，代码检查能在你犯错时指出来，是你更容易调整技术，避免造成更多相同的错误。

Establishing strong linting can be beneficial, especially in a context where you need to collaborate with others. Even when working alone you benefit from linting as it can catch issues you could otherwise neglect. JavaScript as a language allows usage which, while valid, is not be the clearest to understand or may even be incorrect.

建立健全的代码检查将受益匪浅，特别是在你需要和其他人合作的时候。即使你独自工作，你也可以从代码检查中收益，因为它可以帮你发现那些被你忽视的问题。JavaScript作为一门语言，在使用时，有时虽然有效，但不一定是最明的，甚至不一定是正确的。

Linting does **not** replace proper testing, but it can complement testing approaches. It allows you to harden a codebase and make it more difficult to break. As the size of your project grows, and it becomes more challenging to manage, this becomes particularly important.

代码检查并**不**能代替合理的测试，但是它完善了测试的方式。它让你强化了代码库，使其更难以出现问题。随着项目体积的增长，管理变得更具挑战，代码检测也变得愈加重要。

## Setting Up ESLint
## 设置ESlint

![ESLint](images/eslint.png)

[ESLint](http://eslint.org/) is the most versatile linting solution for JavaScript. It builds on top of ideas presented by JSLint and JSHint. More importantly, it allows you to develop custom rules.

[ESLint](http://eslint.org/)是JavaScript代码检查解决方案的集大成者。它基于JSLint和JSHint提出的思想之上。更重要的是，它允许你开发自定义的规则。

T> Since *v1.4.0* ESLint supports [autofixing](http://eslint.org/blog/2015/09/eslint-v1.4.0-released/). It allows you to perform certain rule fixes automatically. To activate it, pass the flag `--fix` to the tool. It's also possible to use this feature with webpack, although you should be careful with it. [js-beautify](https://www.npmjs.com/package/js-beautify) can perform a similar operation.

T> 从*v1.4.0*开始，ESLint支持[自动修补](http://eslint.org/blog/2015/09/eslint-v1.4.0-released/)。它让你可以执行特定规则的自动修正。为了实现这个，向该工具传入`--fix`标示。也可以将其与webpack一起使用，但是你需要小心处理它。[js-beautify](https://www.npmjs.com/package/js-beautify)可以执行相同的操作。

### Connecting ESlint with *package.json*
### 通过*package.json*连接ESlint

To get started, install ESLint as a development dependency:
首先，装装ESLint开发依赖：

```bash
npm install eslint --save-dev
```

Next, you have to write configuration so you can run ESLint smoothly through npm. The `lint` namespace can be used to signify it's a linting related task. Caching should be enabled to improve performance on subsequent runs.

接下来，你需要编写配置，使你可以通过npm来顺畅的运行ESLint。命名空间`lint`可以用来表示这是一个代码检查相关的任务。应启用缓存可以提高后续运行的性能。

{pagebreak}

Add the following to lint the application files and configuration:

在应用的配置文件中添加如下检查语句：

**package.json**

```json
"scripts": {
leanpub-start-insert
  "lint:js": "eslint app/ webpack.*.js --cache",
leanpub-end-insert
  ...
},
```

The potential problem with using an include based approach is that you forget to lint source code. Using excludes solves this but then you have to be careful to update the exclude list as your project grows to avoid linting too much.

使用如上这样的方式的一个问题就是你会忘记检查源代码。使用忽略的方式可以解决这个问题，但是随着项目的增长，你需要仔细的更新要忽略的列表。

The exclusion approach can be achieved by pointing ESLint to the project root through `eslint .` and setting up a *.eslintignore* file for the excludes like for *.gitignore*. You can point to your Git ignores using `--ignore-path .gitignore` for maximum reuse. Individual patterns are supported through `--ignore-pattern <pattern>`.

是用忽略的方式可以通过将ESLint用`eslint .`来指向项目，并为要排除的部分设置一个*.eslintignore*文件，就是为Git设置*.gitignore*文件一样。为了最大化利用，你可以使用`--ignore-path .gitignore`指向你的Git忽略文件。单独的模式可通过`--ignore-pattern <pattern>`来支持。

### Defining Linting Rules
### 定义检查的规则

Given ESLint expects configuration to work, you need to define rules to describe what to lint and how to react if the rules aren't obeyed. The severity of an individual rule is defined by a number as follows:

由于ESLint需要根据配置才可以工作，你需要定义规则来描述需要检查什么，以及在违反规则时要如何反应。一个单独规则的严重程度可以通过如下的数字来定义：

* 0 - The rule has been disabled.
* 1 - The rule emits a warning.
* 2 - The rule emits an error.

* 0 - 禁用该规则
* 1 - 触发一个警告
* 2 - 触发一个错误

Rules, such as `quotes`, accept an array instead allowing you to pass extra parameters to them. Refer to the [ESLint rules documentation](http://eslint.org/docs/rules/) for specifics.

规则，像`quotes`这样的，接受一个数组而不是让你传递一个额外的参数。具体的内容可查看[ESLint规则文档](http://eslint.org/docs/rules/)

{pagebreak}

Here's a starting point that works with the project:

如下是该项目使用的一个入门例子：

**.eslintrc.js**

```javascript
module.exports = {
  env: {
    browser: true,
    commonjs: true,
    es6: true,
    node: true,
  },
  extends: 'eslint:recommended',
  parserOptions: {
    sourceType: 'module',
  },
  rules: {
    'comma-dangle': ['error', 'always-multiline'],
    indent: ['error', 2],
    'linebreak-style': ['error', 'unix'],
    quotes: ['error', 'single'],
    semi: ['error', 'always'],
    'no-unused-vars': ['warn'],
    'no-console': 0,
  },
};
```

If you invoke `npm run lint:js` now, it should execute without any warnings or errors. If you see either, this is a good time to try ESLint autofixing. You can run it through `npm run lint:js -- --fix`. Running an npm script this way allows you to pass extra parameters to it.

如果你现在执行`npm run lint:js`，它运行时应该不会有任何警告和错误。

Another alternative would be to push it behind a *package.json* script. Autofix is not able to repair each error, but it can fix a lot. And as time goes by and ESLint improves, it can perform more work.

Beyond vanilla JSON, ESLint supports other formats, such as JavaScript or YAML. I.e., *.eslintrc.yaml* would expect YAML. See the [documentation](http://eslint.org/docs/user-guide/configuring#configuration-file-formats) for further details.

T> When ESLint gives errors, npm shows a long `ELIFECYCLE error` error block of its own. It's possible to disable that using the `silent` flag: `npm run lint:js --silent` or a shortcut `npm run lint:js -s`.

### Connecting ESLint with Webpack

You can make webpack emit ESLint messages by using [eslint-loader](https://www.npmjs.com/package/eslint-loader). As the first step execute

```bash
npm install eslint-loader --save-dev
```

W> *eslint-loader* uses a globally installed version of ESLint unless you have one included in the project itself. Make sure you have ESLint as a development dependency to avoid the strange behavior.

The loader needs wiring to work. Loaders are discussed in detail in the *Loading* part of this book, but the basic idea is fast to understand. A loader is connected to webpack through a rule that contains preconditions related to it and a reference to the loader itself.

In this case, you want to ensure that ESLint gets executed before anything else using the `enforce` field. It allows to guarantee that linting happens before any other processing. The idea is discussed in detail in the *Loader Definitions* chapter.

{pagebreak}

To add linting to the project, adjust the configuration as follows:

**webpack.config.js**

```javascript
const developmentConfig = () => {
  const config = {
    ...
leanpub-start-insert
    module: {
      rules: [
        {
          test: /\.js$/,
          enforce: 'pre',

          loader: 'eslint-loader',
          options: {
            emitWarning: true,
          },
        },
      ],
    },
leanpub-end-insert
  };

  ...
};
```

If you execute `npm start` now and break a linting rule while developing, you should see that in the terminal output.

W> Webpack configuration lints only the application code you refer. If you want to lint webpack configuration itself, execute `npm run lint:js` separately.

T> Attaching the linting process to Git through a prepush hook allows you to catch problems earlier. [husky](https://www.npmjs.com/package/husky) allows you to achieve this quickly. Doing this allows you to rebase your commits and fix possible problems early.

### Enabling Error Overlay

WDS provides an overlay for capturing warnings and errors:

**webpack.config.js**

```javascript
const developmentConfig = () => {
  const config = {
    devServer: {
      ...
leanpub-start-insert
      // overlay: true captures only errors
      overlay: {
        errors: true,
        warnings: true,
      },
leanpub-end-insert
    },
  };

  ...
};
```

Run the server now (`npm start`) and break the code to see an overlay in the browser:

![Error overlay](images/error-overlay.png)

{pagebreak}

## ESLint Tips

The great thing about ESLint is that you can shape it to your purposes. The community around it's active, and you can find good integration in other tooling as well. Consider the tips below.

### Usability Tips

* It can make sense to rely on an existing preset or set up custom configuration. That's where `--init` can come in handy. You can run it from `npm bin` and end up with a call like `node_modules/.bin/eslint --init`
* ESLint supports custom formatters through `--format` parameter. [eslint-friendly-formatter](https://www.npmjs.com/package/eslint-friendly-formatter) is an example of a formatter that provides terminal-friendly output. This way you can jump conveniently straight to the warnings and errors from there.

### Performance Tips

* Especially on bigger projects it's beneficial to run ESLint outside of webpack. That keeps code compilation fast while still giving the advantage of linting. Solutions like [lint-staged](https://www.npmjs.com/package/lint-staged) and [fastlint](https://www.npmjs.com/package/fastlint) can make this even more quickly.
* You can get more performance out of ESLint by running it through a daemon, such as [eslint_d](https://www.npmjs.com/package/eslint_d). Using it brings down the overhead, and it can bring down linting times considerably.

### Extension Tips

* ESLint supports ES6 features through configuration. You have to specify the features to use through the [ecmaFeatures](http://eslint.org/docs/user-guide/configuring.html#specifying-language-options) property.
* Plugins, such as [eslint-plugin-node](https://www.npmjs.com/package/eslint-plugin-node), [eslint-plugin-promise](https://www.npmjs.com/package/eslint-plugin-promise), [eslint-plugin-compat](https://www.npmjs.com/package/eslint-plugin-compat), and [eslint-plugin-import](https://www.npmjs.com/package/eslint-plugin-import), are worth studying.
* Most IDEs and editors have good linter integration so you can spot issues as you develop.
* To learn about ESLint customizations options and how to write an ESLint plugin, check out the *Customizing ESLint* appendix.

### Configuring ESLint Further

Since webpack 2, the configuration schema of webpack has become stricter, and it doesn't allow arbitrary fields at configuration root level anymore. To overcome this issue and to access all functionality of the *eslint-loader*, you have to use `LoaderOptionsPlugin` as below:

```javascript
{
  plugins: [
    new webpack.LoaderOptionsPlugin({
      options: {
        eslint: {
          // Fail only on errors
          failOnWarning: false,
          failOnError: true,

          // Toggle autofix
          fix: false,

          // Output to Jenkins compatible XML
          outputReport: {
            filePath: 'checkstyle.xml',
            formatter: require('eslint/lib/formatters/checkstyle'),
          },
        },
      },
    }),
  ],
},
```

There are more options, and [eslint-loader](https://www.npmjs.com/package/eslint-loader) documentation covers those in detail.

## Webpack and JSHint

No JSLint particular loader exists for webpack yet. Fortunately, there's one for JSHint. You could set it up on a legacy project quickly. The key is in configuring [jshint-loader](https://www.npmjs.com/package/jshint-loader).

JSHint looks into specific rules to apply from `.jshintrc`. You can also define custom settings within a `jshint` object at your webpack configuration. Exact configuration options have been covered by [the JSHint documentation](http://jshint.com/docs/) in detail.

## Prettier

[Prettier](https://www.npmjs.com/package/prettier) goes one step further and can format your code automatically according to your coding style. If you want to use Prettier with ESLint, you should use [eslint-config-prettier](https://www.npmjs.com/package/eslint-config-prettier) to disable ESLint rules that conflict with Prettier. Be warned that Prettier is highly opinionated in how it thinks code should look and offers few formatting options.

{pagebreak}

## Danger

[Danger](https://www.npmjs.com/package/danger) operates on a higher level than other tools discussed. For example, it can check that the project change log was updated before a release is pushed to the public. You can also force pull requests of your project to comply specific standards.

## EditorConfig

[EditorConfig](http://editorconfig.org/) allows you to maintain a consistent coding style across different IDEs and editors. Also, you need to set up a file:

**.editorconfig**

```yaml
root = true

# General settings for whole project
[*]
indent_style = space
indent_size = 4
end_of_line = lf
charset = utf-8
trim_trailing_whitespace = true
insert_final_newline = true

# Format specific overrides
[app/**/*.js]
indent_style = space
indent_size = 2
```

{pagebreak}

## Conclusion

Linting is one of those techniques that yields benefits over the long term. You can fix possible problems before they become actual issues.

To recap:

* ESLint is the most versatile of the current options. You can expand it to fit your exact use case.
* ESLint can be run through webpack. It can terminate your build and even prevent it from getting deployed if your build does not pass the linting rules.
* To get a better development experience, consider enabling WDS `overlay` to capture errors and warnings emitted by webpack.
* EditorConfig complements ESLint by allowing you to define a project-level coding style. Editors integrate with EditorConfig making it easier to keep a project consistent regardless of the development platform.
* Prettier is a complimentary solution that can format your code automatically whilst Danger operates on repository level and can perform higher level tasks related to the development process.

Given the webpack configuration of the project is starting to get messy, it's a good time to discuss how to compose configuration and improve the situation.
