# 测试（Testing）

Testing is a vital part of development. Even though techniques, such as linting, can help to spot and solve issues, they have their limitations. Testing can be applied against the code and an application on many different levels.

测试是开发中的一个重要部分。即使如代码格式检查工具（linting）技术能够帮助发现和解决问题，但他们仍然有自身的限制。测试能够在不同程度应用到代码和某一应用中。

You can **unit test** specific piece of code, or you can look at the application from the user's point of view through **acceptance testing**. **Integration testing** fits between these ends of the spectrum and is concerned about how separate units of code operate together.

你可以对代码的某一具体部分进行**单元测试**，或者你可以通过**验收测试**从用户的角度来检验应用程序。**集成测试**介于这两者间并关心每个独立的代码单元如何一起工作。

You can find a lot of testing tools for JavaScript. The most popular options work with webpack after you configure it right. Even though test runners work without webpack, running them through it allows you to process code the test runners do not understand while having control over the way modules are resolved. You can also use webpack's watch mode instead of relying on one provided by a test runner.

你可以找到很多基于JavaScript的测试工具。最普遍的一种做法是配合webpack一起使用在正确配置后。即使没有webpack测试驱动器也能正常工作，但通过webpack来运行他们能够让你处理测试驱动器所不能理解的代码，同时也能解决模块的控制加载。你也可以使用webpack的监听模式而不依赖于测试驱动器来提供。

## Mocha

![Mocha](https://raw.githubusercontent.com/survivejs-translations/webpack-book-zh/master/manuscript/images/mocha.png)

[Mocha](https://mochajs.org/) is a popular test framework for Node. While Mocha provides test infrastructure, you have to bring your asserts to it. Even though [Node assert](https://nodejs.org/api/assert.html) can be enough, there are good alternatives such as [power-assert](https://www.npmjs.com/package/power-assert), [Chai](http://chaijs.com/), or [Unexpected](http://unexpected.js.org/).

[Mocha](https://mochajs.org/)是一款流行的基于Node的测试框架。但Mocha仅提供测试基础框架，你需要将自己的断言库添加进去。尽管[Node assert](https://nodejs.org/api/assert.html) 已经足够。但仍有些不错的选择如[power-assert](https://www.npmjs.com/package/power-assert)， [Chai](http://chaijs.com/)，或[Unexpected](http://unexpected.js.org/)。

[mocha-loader](https://www.npmjs.com/package/mocha-loader) allows running Mocha tests through webpack. [mocha-webpack](https://www.npmjs.com/package/mocha-webpack) is another option that aims to provide more functionality. You'll learn the basic *mocha-loader* setup next.

[mocha-loader](https://www.npmjs.com/package/mocha-loader)能够通过webpack运行Mocha测试。[mocha-webpack](https://www.npmjs.com/package/mocha-webpack) 则是另外致力于提供更多功能的选项。在下一部分你将会了解基本的*mocha-loader*配置。

{pagebreak}

### 配置*mocha-loader*和Webpack（onfiguring *mocha-loader* with Webpack）

To get started, include Mocha and *mocha-loader* to your project:

首先，将Mocha和*mocha-loader*引入项目：

```bash
npm install mocha mocha-loader --save-dev
```

To make ESLint aware of Mocha globals, tweak as follows:

为了让ESLint能够监听到Mocha全局对象，调整如下：

**eslintrc.js**

```javascript
module.exports = {
  env: {
    ...
leanpub-start-insert
    mocha: true,
leanpub-end-insert
  },
  ...
};
```

### 配置测试代码（Setting Up Code to Test）

To have something to test, set up a function:

为了能有代码用于测试，编写一个函数：

**tests/add.js**

```javascript
module.exports = (a, b) => a + b;
```

{pagebreak}

Then, to test that, set up a small test suite:

而后，为了测试那套代码，编写一个小的测试套件：

**tests/add.test.js**

```javascript
const assert = require('assert');
const add = require('./add');

describe('Demo', () => {
  it('should add correctly', () => {
    assert.equal(add(1, 1), 2);
  });
});
```

### 配置Mocha（Configuring Mocha）

To run Mocha against the test, add a script:

为了能让Mocha运行该测试，添加一段脚本：

**package.json**

```json
"scripts": {
  "test:mocha": "mocha tests",
  ...
},
```

If you execute `npm run test:mocha` now, you should see output:

如果你现在执行`npm run test:mocha`，你应该看到输出：

```
Demo
  should add correctly


1 passing (10ms)
```

Mocha also provides a watch mode which you can activate through `npm run test:mocha -- --watch`. It runs the test suite as you modify the code.

Mocha也提供了监听模式，可以通过`npm run test:mocha --watch`激活。在你修改代码时会自动运行测试套件。

T> `--grep <pattern>` can be used for constraining the behavior if you want to focus only on a particular set of tests.

T> `--grep <pattern>` 能够被用来进行约束，如果你仅想集中处理某一套特别的测试。

### 配置Webpack（Configuring Webpack）

Webpack can provide similar functionality through a web interface. The hard parts of the problem have been solved earlier in this book, what remain is combining those solutions together through configuration.

Webpack能够通过web界面提供相同功能。困难的部分之前已在本书中解决了，剩下的是将这些方案集成到配置中。

To tell webpack which tests to run, they need to be imported somehow. The *Dynamic Loading* chapter discussed `require.context` that allows to aggregate files based on a rule. It's ideal here. Set up an entry point as follows:

为告诉webpack哪一部分的测试要被执行，他们需要以一定方式引入。在*动态加载*一章中所讨论的`require.context`则允许基于某一规则来合并文件。其非常贴合该场景。如下配置入口：

**tests/index.js**

```javascript
// Skip execution in Node
if (module.hot) {
  const context = require.context(
    'mocha-loader!./', // Process through mocha-loader
    false, // Skip recursive processing
    /\.test.js$/ // Pick only files ending with .test.js
  );

  // Execute each test suite
  context.keys().forEach(context);
}
```

{pagebreak}

To allow webpack to pick this up, a small amount of configuration is required:

为了能够让webpack处理这一部分，一小部分的配置是需要的：

**webpack.mocha.js**

```javascript
const path = require('path');
const merge = require('webpack-merge');

const parts = require('./webpack.parts');

module.exports = merge([
  parts.devServer(),
  parts.page({
    title: 'Mocha demo',
    entry: {
      tests: path.join(__dirname, 'tests'),
    },
  }),
]);
```

T> See the *Composing Configuration* chapter for the full `devServer` setup. The page setup is explained in the *Multiple Pages* chapter.

T> 完整的`devServer`配置查看*组装配置*一章。页面配置在*多页面配置*章节中有详细解释。

Add a helper script to make it convenient to run:

添加一个辅助脚本方便配置运行：

**package.json**

```json
"scripts": {
  "test:mocha:watch": "webpack-dev-server --hot --config webpack.mocha.js",
  ...
},
```

T> If you want to understand what `--hot` does better, see the *Hot Module Replacement* appendix.

T> 若想要更好地了解`--hot`，查看附录中的*Hot Module Replacement*。

If you execute the server now and navigate to `http://localhost:8080/`, you should see the test:

如果你执行该代码并访问`http://localhost:8080`，你应该看到测试结果：

![Mocha in browser](https://raw.githubusercontent.com/survivejs-translations/webpack-book-zh/master/manuscript/images/mocha-browser.png)

Adjusting either the test or the code should lead to a change in the browser. You can grow your specification or refactor the code while seeing the status of the tests.

调整测试或代码将会使得浏览器里的结果发生改变。你可以添加测试规范或重构代码并查看测试状态。

Compared to the vanilla Mocha setup, configuring Mocha through webpack comes with a couple of advantages:

* It's possible to adjust module resolution. Webpack aliasing and other techniques work now, but this would also tie the code to webpack.
* You can use webpack's processing to compile your code as you wish. With vanilla Mocha that would imply more setup outside of it.

相较于配置原生的Mocha配置，通过webpack来配置Mocha具有以下优点：

* 能够调整模块的颗粒度。Webpack别名和其他技术现在可以使用了，但这会将代码和webpack耦合在一起。
* 你可以使用webpack来编译处理你想要的代码。而通过原生的Mocha配置则需要更多的第三方支持。

On the downside, now you need a browser to examine the tests. *mocha-loader* is at its best as a development helper. The problem can be solved by running the tests through a headless browser.

而缺点则是，现在你需要一个浏览器来执行测试。*mocha-loader*无疑是最好的一个开发帮手。而这问题可以通过将测试运行在一个headless的浏览器来解决。

## Karma and Mocha

![Karma](https://raw.githubusercontent.com/survivejs-translations/webpack-book-zh/master/manuscript/images/karma.png)

[Karma](https://karma-runner.github.io/) is a test runner that allows you to run tests against real devices and [PhantomJS](http://phantomjs.org/), a headless browser. [karma-webpack](https://www.npmjs.com/package/karma-webpack) is a Karma preprocessor that allows you to connect Karma with webpack. The same benefits as before apply still. This time around, however, there is more control over the test environment.

[Karma](https://karma-runner.github.io/)是一款允许你在真实环境和[PhantomJS](http://phantomjs.org/)，一款headless浏览器，运行测试的测试工具。 [karma-webpack](https://www.npmjs.com/package/karma-webpack)是一款将Karma和webpack连接起来的Karma预处理器。其优点和之前提到类似。然而，这一次，对测试环境有更多控制。

To get started, install Karma, Mocha, *karma-mocha* reporter, and *karma-webpack*:

首先，安装 Karma, Mocha, *karma-mocha*以及*karma-webpack*：

```
npm install karma mocha karma-mocha karma-webpack --save-dev
```

{pagebreak}

Like webpack, Karma relies on the configuration as well. Set up a file as follows to make it pick up the tests:

类似webpack，Karma也需要进行配置。进行如下配置以让Karma能识别测试文件：

**karma.conf.js**

```javascript
module.exports = (config) => {
  const tests = 'tests/*.test.js';

  config.set({
    frameworks: ['mocha'],

    files: [
      {
        pattern: tests,
      },
    ],

    // Preprocess through webpack
    preprocessors: {
      [tests]: ['webpack'],
    },

    singleRun: true,
  });
};
```

W> The file has to be named exactly as *karma.conf.js*. Otherwise Karma doesn't pick it up automatically.

W> 该文件被准确地命名为*karma.conf.js*。否则Karma无法自动识别。

W> The setup generates a bundle per each test. If you have a large amount of tests and want to improve performance, set up `require.context` as for Mocha above. See [karma-webpack issue 23](https://github.com/webpack-contrib/karma-webpack/issues/23) for more details.

该配置在每一次测试生成一个测试代码包。如果你有大量的测试并想要提高性能，类似之前的Mocha配置`require.context`。更多细节可查看[karma-webpack issue 23](https://github.com/webpack-contrib/karma-webpack/issues/23)

{pagebreak}

Add an npm shortcut:

添加npm快捷方式：

```json
...
"scripts": {
  "test:karma": "karma start",
  ...
},
...
```

If you execute `npm run test:karma` now, you should see terminal output:

若你现在执行`npm run test:karam`， 你应该能在终端看到输出：

```
...
webpack: Compiled successfully.
...:INFO [karma]: Karma v1.5.0 server started at http://0.0.0.0:9876/
```

This means Karma is in a waiting state and you have to visit that url to run the tests. As per configuration (`singleRun: true`), Karma terminates execution after that:

这意味着karma处于等待状态，你需要访问那个连接来运行测试。根据配置（`singleRun: true`），Karma将在之后终止执行。

```
...
...:INFO [karma]: Karma v1.5.0 server started at http://0.0.0.0:9876/
...:INFO [Chrome 57...]: Connected on socket D...A with id manual-73
Chrome 57...): Executed 1 of 1 SUCCESS (0.003 secs / 0 secs)
```

Given running tests this way can become annoying, it's a good idea to configure alternative ways. Using PhantomJS is one option.

通过这一方式来运行测试十分麻烦。配置替代方案是一个理想方式。使用PhantomJS是一个可选项。

T> You can point Karma to specific browsers through the `browsers` field. Example: `browsers: ['Chrome']`.

T> 你可以通过`browsers`字段来指定具体的浏览器。例如：`browsers['Chrome']`。

{pagebreak}

### 通过PhantomJS运行测试（Running Tests Through PhantomJS）

Running tests through PhantomJS requires a couple of dependencies:

通过PhantomJS运行测试需要一系列的依赖：

```
npm install karma-phantomjs-launcher phantomjs-prebuilt --save-dev
```

To make Karma run tests through Phantom, adjust its configuration as follows:

为了让Karma通过Phantom执行测试，将配置做如下调整：

**karma.conf.js**

```javascript
module.exports = (config) => {
  ...

  config.set({
    ...

leanpub-start-insert
    browsers: ['PhantomJS'],
leanpub-end-insert
  });
};
```

If you execute the tests again (`npm run test:karma`), you should get output without having to visit an url:

如果你再执行一次测试（`npm run test:karma`），你应该看到一个不要求访问url的输出结果：

```
...
webpack: Compiled successfully.
...:INFO [karma]: Karma v1.5.0 server started at http://0.0.0.0:9876/
...:INFO [launcher]: Launching browser PhantomJS with unlimited concurrency
...:INFO [launcher]: Starting browser PhantomJS
...:INFO [PhantomJS ...]: Connected on socket 7...A with id 123
PhantomJS ...: Executed 1 of 1 SUCCESS (0.005 secs / 0.001 secs)
```

Given running tests after the change can get boring after a while, Karma provides a watch mode.

考虑到在发生改变之后再重新运行一次测试十分枯燥，Karma提供了监听模式。

W> PhantomJS does not support ES6 features yet so you have to preprocess the code for tests using them. The webpack setup is done later in this chapter. ES6 support is planned for PhantomJS 2.5.

W> PhantomJS目前不支持ES 6语法，因此你需要对测试中用到的代码进行预处理。Webpack配置将在本章稍后部分完成。ES6计划在PhantomJS 2.5中得到支持。

### Karma监听模式（Watch Mode with Karma）

Accessing Karma's watch mode is possible as follows:

可通过如下方式使用Karma的监听模式：

**package.json**

```json
"scripts": {
leanpub-start-insert
  "test:karma:watch": "karma start --auto-watch --no-single-run",
leanpub-end-insert
  ...
},
```

If you execute `npm run test:karma:watch` now, you should see watch behavior.

如果你现在执行`npm run test:karam:watch`，你应该能看到监听行为。

### 生成覆盖率报告（Generating Coverage Reports）

To know how much of the code the tests cover, it can be a good idea to generate coverage reports. Doing this requires code-level instrumentation. Also, the added information has to be reported. This can be done through HTML and LCOV reports.

为了了解测试覆盖了多少代码，理想的方式是生成一份测试报告。处理这一功能要求对代码层面的工具。此外，这些增加的信息必须被报告。这一工作可以通过HTML和LCOV来完成。

T> LCOV integrates well with visualization services. You can send coverage information to an external service through a continuous integration environment and track the status in one place.

T> LCOV很好地集成了可视化服务。你可以将覆盖率信息通过持续集成环境发送到一个额外的服务并在某一地方检查该状态。

[isparta](https://www.npmjs.com/package/isparta) is a popular, ES6 compatible code coverage tool. Connecting it with Karma requires configuration. Most importantly the code has to be instrumented through [babel-plugin-istanbul](https://www.npmjs.com/package/babel-plugin-istanbul). Doing this requires a small amount of webpack configuration as well due to the setup. [karma-coverage](https://www.npmjs.com/package/karma-coverage) is required for the reporting portion of the problem.

[isparta](https://www.npmjs.com/package/isparta) 是一款受欢迎的、能兼容ES6的代码覆盖率工具。将其和Karma结合起来需要进行配置。最重要的是代码已经通过 [babel-plugin-istanbul](https://www.npmjs.com/package/babel-plugin-istanbul)进行处理。由于配置的原因，处理这一工作也需要对webpack的配置进行调整。[karma-coverage](https://www.npmjs.com/package/karma-coverage)是报告所必备的。

{pagebreak}

Install the dependencies first:

首先安装依赖：

```
npm install babel-plugin-istanbul karma-coverage --save-dev
```

Connect the Babel plugin so that the instrumentation happens when Karma is run:

连接Babel插件，这样当Karma运行时instrumentation测试才会进行。

**.babelrc**

```json
...
leanpub-start-insert
"env": {
  "karma": {
    "plugins": [
      [
        "istanbul",
        { "exclude": ["tests/*.test.js"] }
      ]
    ]
  }
}
leanpub-end-insert
```

Make sure to set Babel environment, so it picks up the plugin:

确认设置了Babel环境，以让其能够识别该插件：

**karma.conf.js**

```javascript
module.exports = (config) => {
  ...

leanpub-start-insert
  process.env.BABEL_ENV = 'karma';
leanpub-end-insert

  config.set({
    ...
  });
};
```

T> If you want to understand the `env` idea, see the *Loading JavaScript* chapter.

T> 若想要了解`env`概念，查看*加载JavaScript*一章。

On Karma side, reporting has to be set up and Karma configuration has to be connected with webpack. *karma-webpack* provides two fields for this purpose: `webpack` and `webpackMiddleware`. You should use the former in this case to make sure the code gets processed through Babel.

在Karma的角度，代码覆盖报告需要配置，同时Karma配置需要挂载到webpack。*Karma-webpack*基于这一目的提供了两个字段：`webpack`和`webpackMiddleware`。在这一例子中你应该使用前者以确保代码能够通过Babel被处理。

**karma.conf.js**

```javascript
leanpub-start-insert
const path = require('path');
leanpub-end-insert

...

module.exports = (config) => {
  ...

  config.set({
    ...

leanpub-start-insert
    webpack: require('./webpack.parts').loadJavaScript({
      include: path.join(__dirname, 'tests'),
    }),

    reporters: ['coverage'],

    coverageReporter: {
      dir: 'build',
      reporters: [
        { type: 'html' },
        { type: 'lcov' },
      ],
    },
leanpub-end-insert
  });
};
```

T> If you want to emit the reports to specific directories below `dir`, set `subdir` per each report.

T> 如果你想要让覆盖率报告在`dir`配置路径下针对具体目录，为每份报告设置`subdir`。

If you execute karma now (`npm run test:karma`), you should see a new directory containing coverage reports. The HTML report can be examined through the browser.

若现在运行karma（`num run test:karma`）。你应该看见一个新的包含代码覆盖率报告的目录。HTML报告可以通过浏览器进行查看。

![Coverage in browser](https://raw.githubusercontent.com/survivejs-translations/webpack-book-zh/master/manuscript/images/coverage.png)

LCOV requires specific tooling to work. You can find editor plugins such as [lcov-info](https://atom.io/packages/lcov-info) for Atom. A properly configured plugin can give you coverage information while you are developing using the watch mode.

LCOV要求特殊工具才能进行工作。你可以寻找编辑器插件，如Atom的[lcov-info](https://atom.io/packages/lcov-info)。一个合适的插件配置能够在开发期间通过监听模式提供代码覆盖率信息。

## Jest

![Jest](https://raw.githubusercontent.com/survivejs-translations/webpack-book-zh/master/manuscript/images/jest.png)

Facebook's [Jest](https://facebook.github.io/jest/) is an opinionated alternative that encapsulates functionality, including coverage and mocking, with minimal setup. It can capture snapshots of data making it valuable for projects where you have the behavior you would like to record and retain.

Facebook的[Jest](https://facebook.github.io/jest/)特立独行的选项。其在最小化配置情况下封装了包括代码覆盖率、mocking等功能。其能够捕获数据快照，使得其对于想要记录和保留行为的项目来说变得十分有价值。

Jest tests follow [Jasmine](https://www.npmjs.com/package/jasmine) test framework semantics, and it supports Jasmine-style assertions out of the box. Especially the suite definition is close enough to Mocha so that the current test should work without any adjustments to the test code itself. Jest provides [jest-codemods](https://www.npmjs.com/package/jest-codemods) for migrating more complex projects to Jest semantics.

Jest 测试遵循 [Jasmine](https://www.npmjs.com/package/jasmine)测试框架的语义，其支持Jasmine风格的断言。尤其是套件定义紧贴Mocha，这样一来当前的测试再不做任何测试代码的调整下应该能够工作。Jest提供[jest-codemods](https://www.npmjs.com/package/jest-codemods)来迁移更为复杂的项目到Jest语法下。

Install Jest first:

首先安装Jest：

```
npm install jest --save-dev
```

Jest captures tests through *package.json* [configuration](https://facebook.github.io/jest/docs/configuration.html). It detects tests within a *__tests__* directory it also happens to capture the naming pattern the project is using by default:

Jest 捕获测试通过*package.json* [configuration](https://facebook.github.io/jest/docs/configuration.html)。其在一个名为*__tests__*文件中检测测试。默认情况，其也捕获项目正在使用的命名模式。

**package.json**

```json
"scripts": {
leanpub-start-insert
  "test:jest:watch": "jest --watch",
  "test:jest": "jest",
leanpub-end-insert
  ...
},
```

Now you have two new commands: one to run tests once and other to run them in a watch mode. To capture coverage information, you have to set `"collectCoverage": true` at `"jest"` settings in *package.json* or pass `--coverage` flag to Jest. It emits the coverage reports below *coverage* directory by default.

现在你有两个新命令：一个仅执行一侧测试，另一个则在监听模式下运行测试。为捕获代码覆盖信息，你需要在*package.json*文件中的`"jest"`向配置`"collectCoverage": true`或给Jest传`--coverage`参数。默认情况下，其会在*coverage*目录下生成代码覆盖率报告。

Given generating coverage reports comes with a performance overhead, enabling the behavior through the flag can be a good idea. This way you can control exactly when to capture the information.

考虑到生成代码覆盖率报告会产生一定的性能损耗，通过标识符支持这一行为是一个理想的情况。这一方法能让你准确地控制当要捕获代码覆盖率信息时。

Porting a webpack setup to Jest requires more effort especially if you rely on webpack specific features. [The official guide](https://facebook.github.io/jest/docs/webpack.html) covers quite a few of the common problems. You can also configure Jest to use Babel through [babel-jest](https://www.npmjs.com/package/babel-jest) as it allows you to use Babel plugins like [babel-plugin-module-resolver](https://www.npmjs.com/package/babel-plugin-module-resolver) to match webpack's functionality.

移植webpack配置到Jest要求更多的精力，尤其是若你依赖于webpack的特定的特性时。[官方文档](https://facebook.github.io/jest/docs/webpack.html) 提供了一部分常见问题。你也可以使用Babel并通过[babel-jest](https://www.npmjs.com/package/babel-jest)来配置Jest，其允许你使用如 [babel-plugin-module-resolver](https://www.npmjs.com/package/babel-plugin-module-resolver) Babel插件来满足webpack的功能。

## AVA

![AVA](https://raw.githubusercontent.com/survivejs-translations/webpack-book-zh/master/manuscript/images/ava.png)

[AVA](https://www.npmjs.com/package/ava) is a test runner that has been designed to take advantage of parallel execution. It comes with a test suite definition of its own. [webpack-ava-recipe](https://github.com/greyepoxy/webpack-ava-recipe) covers how to connect it with webpack.

[AVA](https://www.npmjs.com/package/ava) 是一款被设计为用来并发执行的测试框架。其由自己的定义的一套测试套件。[webpack-ava-recipe](https://github.com/greyepoxy/webpack-ava-recipe)包含了如何将其和webpack连接起来。

The main idea is to run both webpack and AVA in watch mode to push the problem of processing code to webpack while allowing AVA to consume the processed code. The `require.context` idea discussed with Mocha comes in handy here as you have to capture tests for webpack to process somehow.

其主要理念是让webpack和AVA在监听模式下同时运行并将处理代码的问题推给webpack，同时允许AVA使用已处理过的代码。之前和Mocha一起探讨的`require.context`的理念在这非常方便，因此你不得不以某种方式处理以为webpack捕获测试。

## Mocking

Mocking is a technique that allows you to replace test objects. Consider the solutions below:

* [Sinon](https://www.npmjs.com/package/sinon) provides mocks, stubs, and spies. It works well with webpack since version 2.0.
* [inject-loader](https://www.npmjs.com/package/inject-loader) allows you to inject code to modules through their dependencies making it valuable for mocking.
* [rewire-webpack](https://www.npmjs.com/package/rewire-webpack) allows mocking and overriding module globals. [babel-plugin-rewire](https://www.npmjs.com/package/babel-plugin-rewire) implements [rewire](https://www.npmjs.com/package/rewire) for Babel.

Mocking是一种允许你替换测试对象的技术。考虑如下解决方案：

* [Sinon](https://www.npmjs.com/package/sinon)提供mocks，stubs和spies。其从2.0版本开始能够和webpack很好地协作。
* [inject-loader](https://www.npmjs.com/package/inject-loader) 能够通过他们的依赖将代码插入模块中，使得mocking具有价值。
* [rewire-webpack](https://www.npmjs.com/package/rewire-webpack)能够mocking以及重载全局模块。[babel-plugin-rewire](https://www.npmjs.com/package/babel-plugin-rewire) 是 [rewire](https://www.npmjs.com/package/rewire) 为Babel的实现方案。

## 结论（Conclusion）

Webpack can be configured to work with a large variety of testing tools. Each tool has its sweet spots, but they also have quite a bit of common ground.

Webpack可以配置成和各类测试工具协作。每个工具都有其自身的优势，但他们也有相当一部分的共性。

To recap:

* Running testing tools allows you to benefit from webpack's module resolution mechanism.
* Sometimes the test setup can be quite involved. Tools like Jest remove most of the boilerplate and allow you to develop tests with minimal setup.
* You can find multiple mocking tools for webpack. They allow you to shape test environment. Sometimes you can avoid mocking through design, though.

本章概括：

* 运行测试工具能够让你从webpack的分离机制中获利。
* 有时测试配置会相当复杂。例如Jest移除了大部分的样板文件并能够以最小化配置进行测试。
* 你可以发现许多基于webpack的mocking工具。他们允许你模拟测试环境。尽管有时你可以从设计层面避免mocking。

You'll learn to deploy applications using webpack in the next chapter.

你将在下一章节了解使用webpack来部署应用。
