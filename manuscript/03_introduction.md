# Introduction
# 简介

[Webpack](https://webpack.js.org/) simplifies web development by solving a fundamental problem: bundling. It takes in various assets, such as JavaScript, CSS, and HTML, and then transforms these assets into a format that’s convenient to consume through a browser. Doing this well takes away a significant amount of pain from web development.

对于Web开发来说，Webpack解决了一个最基本的问题－集成打包，进而简化了开发的过程。Webpack可以对各类资源进行处理，例如JavaScript,CSS和HTML，可以将它们转化成更方便由浏览器处理的格式。解决了这个问题，也就让Web开发不再那么痛苦。

It's not the easiest tool to learn due to its configuration-driven approach, but it's incredibly powerful. The purpose of this guide is to help you get started with webpack and then to go beyond the basics.

但是由于它复杂的配置驱动方式，Webpack并不是最容易掌握的工具，但是它强大到难以置信。本教程的目的就是帮助你开始使用Webpack，然后逐步超越基础！

## What Is Webpack
## Webpack到底是什么？

Web browsers are designed to consume HTML, CSS, and JavaScript. As a project grows, tracking and configuring all of these files grows too complex to manage without help. Webpack was designed to help with this problem.

Web浏览器可以处理HTML，CSS和JavaScript。随着项目的增长，跟踪、配置所有的文件也变得越来越复杂，以至于很难在没有（工具）帮助的情况下进行管理。Webpack就是设计出来解决这个问题的。

As an application develops, handling it becomes more difficult. Webpack was designed to counter this problem. Managing complexity is one of the fundamental issues of web development, and solving the problem well can help you a lot.

随着应用的开发，处理它的复杂性也随之增长。Webpack设计出来就是为了解决这个问题的。（项目）管理的复杂性是Web开发的基本问题之一。一旦很好的解决了这个问题，你将受益匪浅。

Webpack isn’t the only solution for this problem, and a collection of different tools have emerged. Task runners, such as Grunt and Gulp, are good examples of higher level tools. Often the problem is that need to write the workflows by hand. Pushing that issue to a bundler, such as webpack, is a step forward.

Webpack并不是解决这个问题的唯一选择，各种不同的工具也早已出现。任务运行管理工具，如Grunt和Gulp，是高级工具中更好的例子。但是常常遇到的问题是，需要你手动编写工作流。将这个问题交给一个打包工具，例如webpack，将是一个很大改进。

{pagebreak}

### How Does Webpack Change The Situation
### Webpack如何改变现状？

Webpack takes another route. It allows you to treat your project as a dependency graph. You could have an *index.js* in your project that pulls in the dependencies the project needs through standard `require` or `import` statements. You can refer to your style files and other assets the same way if you want.

Webpack采用了另外一种方式。它让你将你的项目视为一个依赖关系图。你可以在项目中的index.js文件中通过使用标准的`require`或者`import`语法来加载项目需要的所有依赖。你也可以用同样的方式引入样式文件和其它资源。

Webpack does all the preprocessing for you and gives you the bundles you specify through configuration and your code. This declarative approach is powerful, but it's difficult to learn. Webpack becomes an indispensable tool after you begin to understand how it works. This book has been designed to get through that initial learning curve and even go further.

Webpack基于你指定的配置和代码，完成了所有的预处理工作，然后生成了代码包。这些（配置的）声明方式很强大，但是学起来也不容易。不过，只要你理解了Webpack的工作原理，它便会成为一个不可却少的工具。这本书就是帮你你越过开始的那个小山坡，让你能够走的更远。


## What Will You Learn
## 你将学到什么？

This book has been designed to complement [the official documentation of webpack](https://webpack.js.org/). The official documentation goes deeper in many aspects, and this book can be considered a companion to it. This book is more like a quick walkthrough that eases the initial learning curve while giving food for thought to more advanced users.

这本书是对*Webpack的官方文档*的补充。官方文档在很多方面有更深入讲解，本书可以作为官方文档的伴侣。俗话说，“万事开头难”，这本书就像一个概览，可以让你在开始学习时不那么痛苦，但如果你已经是有一定基础的使用者，本书也会让你温故而知新。

The book teaches you to develop a composable webpack configuration for both development and production purposes. Advanced techniques covered by the book allow you to get the most out of webpack.

本书将会教你开发一个可组合的Webpack配置，可同时在开发和生产中使用。本书涵盖的先进的技术可以让您充分利用Webpack。

T> The book is based on webpack 2. If you want to apply its techniques to webpack 1, you should see [the official migration guide](https://webpack.js.org/guides/migrating/) as it covers the changes made between the major versions. There are also [codemods at the webpack-cli repository](https://github.com/webpack/webpack-cli) for migrating from webpack 1 to 2.

本书基于webpack2。如果你想使用webpack1，官方迁移文档中包含了所有主要版本的更改。webpack-cli的GitHub官方库中也有方法可以让你从webpack1迁移到webpack2。

{pagebreak}

## How Is The Book Organized
## 这本书如何组织的？

The book starts by explaining what webpack is. After that, you find multiple parts, each of which discusses webpack from a different direction. While going through those chapters, you develop your webpack configuration. The chapters also double as reference material.

本书一开始，讲解了何为webpack。然后，你将会看到很多部分，每部分从不同的方向讨论了webpack。在这些章节中，你将学习如何开发webpack配置。（在之后的实际开发中）它们也可以作为参考资料。 

The book has been split into the following parts:

本书被拆分为以下几个章节：

* **Developing** gets you up and running with webpack. This part goes through features such as automatic browser refresh and explains how to compose your configuration so that it remains maintainable.
* **Developing－开发**部分讲解如何初始化并启动webpack。本章介绍了诸如浏览器自动刷新的特性，并讲解如何撰写易于维护的配置。

* **Styling** puts heavy emphasis on styling related topics. You will learn how to load styles with webpack and how to introduce techniques such as autoprefixing to your setup.
* **Styling－样式**部分花了很大的篇幅来讲解样式相关的话题。你将学习如何通过加载样式文件，如何在设置中引入自动前缀等技术。

* **Loading** explains webpack’s loader definitions in detail and shows you how to load assets such as images, fonts, and JavaScript.
* **Loading部分**详细讲解了定义webpack的加载器，并向你介绍了如何加载诸如图片、字体、JavaScript等资源。

* **Building** introduces source maps and the ideas of bundle and code splitting. You will learn to tidy up your build.
* **Building－构建**部分介绍了source maps，打包的想法和代码拆分。你也将学习如何组织你自己的构建。 

* **Optimizing** pushes your build to production quality level and introduces many smaller tweaks to make it smaller. You will learn to tune webpack for performance.
* **Optimizing－优化**部分将你的构建放到最终的生产质量的层面上，向你介绍了许多小技巧，来使其变体积变得更小。同时，你也将学习如何调整webpack的性能。

* **Output** discusses webpack’s output options. Despite its name, it’s not only for the web. You see how to manage multiple page setups with webpack and pick up the basic idea of Server Side Rendering.
* **Output－输出**部分讨论了webpack的输出选项。虽然它的名字叫"webpack"，但它不仅限于web。你将会学习如何管理多页面配置并了解服务端渲染的基本概念。

* **Techniques** discusses several specific ideas including dynamic loading, web workers, internationalization, and deploying your applications.
* **Techniques－技术**部分讨论了几个特定的想法，包括动态加载，web workers，国际化和部署应用程序。

* **Packages** has a heavy focus on npm and webpack related techniques. You will learn both to consume and author npm packages in an efficient way.
* **Techniques－技术**部分讨论了几个特定的想法，包括动态加载，web workers，国际化和部署应用程序。

* **Extending** shows how to extend webpack with your loaders and plugins.
* **Extending－扩展**部分向你展示了如何使用你自己的loaders-加载器和plugins-插件来扩展webpack。


There's a short conclusion chapter after the main content that recaps the main points of the book. It contains checklists that allow you to go through your projects against the book techniques.

在你阅读完本书所有主要的内容后，最后一章，有一个简短的总结，重述了本书的要点。本章节还包含一个检查清单，让你可以根据自己的项目来回顾本书所讲的技术。

The appendices the end of the book cover secondary topics and sometimes dig deeper into the main ones. You can approach them in any order you want depending on your interest.

本书最后的附录包含了一些额外的话题以及对一些对主要内容更深入的探索。你可以根据自己的兴趣来有选择性的阅读。

最后一个附录是疑难解答，可以帮助你理解如何调试webpack错误。

Given eventually webpack will give you an error, the *Troubleshooting* appendix at the end covers what to do then. It covers a basic process on what to do and how to debug the problem. When in doubt, study the appendix. If you are unsure of a term and its meaning, see the *Glossary* at the end of the book.

由于webpack总是会给你报出一个错误，本书最后的*Troubleshooting*附录涵盖了如何如理这些报错。在有bug时，如何做以及怎么做，这个章节包含了一个基本的处理方法。


## Who Is The Book For

## 这书适合那些读者？

You should have basic knowledge of JavaScript, Node, and npm. If you know something about webpack, that’s great. By reading this book, you deepen your understanding of these tools.

你应该掌握了JavaScript、Node.js、npm的基本知识。如果你对Webpack有一些了解，那就太棒了。读完此书，将会加强你对这些工具的理解。

If you don’t know much about the topic, consider going carefully through the early parts. You can scan the rest to pick the bits you find worthwhile. If you know webpack already, skim and choose the techniques you find valuable.

如果你不太了解这个话题，可以考虑仔细阅读前面的部分。您可以浏览其余的，以选择您认为有价值的部分。如果您已经知道webpack，请浏览并选择您认为有价值的技术。

In case you know webpack well already, there is still something in the book for you. Skim through it and see if you can pick up new techniques. Read especially the summaries at the end of the chapters and the conclusion chapter.

如果你是Webpack老手，你也能学到一些东西。浏览本书，看看你能否学到一些新技术。尤其要阅读每章末尾的总结和结论章节。

## Book Versioning

## 本书的版本

Given this book receives a fair amount of maintenance and improvements due to the pace of innovation, there's a versioning scheme in place. Release notes for each new version are maintained at the [book blog](http://survivejs.com/blog/). You can also use GitHub *compare* tool for this purpose. Example:

由于本书不断创新的步伐，并经过很多修补和改进，所以有很多个版本。我通过本书[官方博客](http://survivejs.com/blog/)对新老版本信息进行发布，版本信息会告诉你各个版本的差异。通过GitHub仓库检测也是一个很好的方法。推荐使用Github对比工具。例如：

```
https://github.com/survivejs/webpack-book/compare/v1.9.0...v2.0.9
```

The page shows you the individual commits that went to the project between the given version range. You can also see the lines that have changed in the book.

The current version of the book is **2.0.9**.

本书当前的本书是2.0.9

## Getting Support
## 得到支持

If you run into trouble or have questions related to the content, there are several options:
对于书中内容，你可能会有一些问题。可以通过下面的方式进行反馈：

* Contact me through [GitHub Issue Tracker](https://github.com/survivejs/webpack-book/issues).
* 通过  *[GitHub Issue Tracker](https://github.com/survivejs/webpack/issues)*联系原作者

* Join me at [Gitter Chat](https://gitter.im/survivejs/webpack).
* 加入到[Gitter Chat](https://gitter.im/survivejs/webpack)

* Send me email at [info@survivejs.com](mailto:info@survivejs.com).
* 给原作者发邮件*[info@survivejs.com](http://info@survivejs.com)*


* Ask me anything about webpack or React at [SurviveJS AmA](https://github.com/survivejs/ama/issues).

* 直接向作者提关于Webpack和React的问题：*[SurviveJS AmA](https://github.com/survivejs/ama/issues)*

If you post questions to Stack Overflow, tag them using **survivejs**. You can use the hashtag **#survivejs** on Twitter for the same result.

在Stack Overflow上提问请用survivejs标记，我将会注意到。推特上，可用#survivejs。

## Additional Material

## 其他资料

You can find more related material from the following sources:
你可以从以下资源中找到更多相关的资料：

* Join the [mailing list](https://eepurl.com/bth1v5) for occasional updates.
* Follow [@survivejs](https://twitter.com/survivejs) on Twitter.

* Twitter上关注[@survivejs](https://twitter.com/survivejs)

* Subscribe to the [blog RSS](https://survivejs.com/atom.xml) to get access interviews and more.
* 订阅[blog RSS](https://survivejs.com/atom.xml)来获取访问权限。

* Subscribe to the [Youtube channel](https://www.youtube.com/channel/UCvUR-BJcbrhmRQZEEr4_bnw).

* 订阅[Youtube channel](https://www.youtube.com/channel/UCvUR-BJcbrhmRQZEEr4_bnw)。

* Check out [SurviveJS related presentation slides](https://presentations.survivejs.com/).
* 查看[SurviveJS相关演示幻灯片](https://presentations.survivejs.com/)。

## Acknowledgments
## 致谢

Big thanks to [Christian Alfoni](http://www.christianalfoni.com/) for helping me craft the first version of this book. That inspired the whole SurviveJS effort. The version you see now is a complete rewrite.

非常感谢[Christian Alfoni](http://www.christianalfoni.com/)帮助我制作这本书的第一个版本。这激励着整个SurviveJS项目。您现在看到的版本是全部重写后的版本。

The book wouldn’t be half as good as it's without patient editing and feedback by my editors [Jesús Rodríguez](https://github.com/Foxandxss), [Artem Sapegin](https://github.com/sapegin), and [Pedr Browne](https://github.com/Undistraction). Thank you.

如果没有细心的编辑和我的编辑[Jesús Rodríguez](https://github.com/Foxandxss), [Artem Sapegin](https://github.com/sapegin), and [Pedr Browne](https://github.com/Undistraction)的反馈，本书将远远达不到现在的质量。

This book wouldn’t have been possible without the original "SurviveJS - Webpack and React" effort. Anyone who contributed to it deserves my thanks. You can check that book for more accurate attributions.

没有最开始的版本"SurviveJS - Webpack and React"的努力。任何支持了那本书的人都应该得到我的致谢。你可以用那本书中得到更详细的原因。

Thanks to Mike "Pomax" Kamermans, Cesar Andreu, Dan Palmer, Viktor Jančík, Tom Byrer, Christian Hettlage, David A. Lee, Alexandar Castaneda, Marcel Olszewski, Steve Schwartz, Chris Sanders, Charles Ju, Aditya Bhardwaj, Rasheed Bustamam, José Menor, Ben Gale, Jake Goulding, Andrew Ferk, gabo, Giang Nguyen, @Coaxial, @khronic, Henrik Raitasola, Gavin Orland, David Riccitelli, Stephen Wright, Majky Bašista, Gunnari Auvinen, Jón Levy, Alexander Zaytsev, Richard Muller, Ava Mallory (Fiverr), Sun Zheng’an, Nancy (Fiverr), Aluan Haddad, Steve Mao, Craig McKenna, Tobias Koppers, Stefan Frede, Vladimir Grenaderov, Scott Thompson, Rafael De Leon, Gil Forcada Codinachs, Jason Aller, @pikeshawn, and many others who have contributed direct feedback for this book!

感谢  Mike "Pomax" Kamermans, Cesar Andreu, Dan Palmer, Viktor Jančík, Tom Byrer, Christian Hettlage, David A. Lee, Alexandar Castaneda, Marcel Olszewski, Steve Schwartz, Chris Sanders, Charles Ju, Aditya Bhardwaj, Rasheed Bustamam, José Menor, Ben Gale, Jake Goulding, Andrew Ferk, gabo, Giang Nguyen, @Coaxial, @khronic, Henrik Raitasola, Gavin Orland, David Riccitelli, Stephen Wright, Majky Bašista, Gunnari Auvinen, Jón Levy, Alexander Zaytsev, Richard Muller, Ava Mallory (Fiverr), Sun Zheng’an, Nancy (Fiverr), Aluan Haddad, Steve Mao, Craig McKenna, Tobias Koppers, Stefan Frede, Vladimir Grenaderov, Scott Thompson, Rafael De Leon, Gil Forcada Codinachs, Jason Aller, @pikeshawn, and many others who have contributed direct feedback for this book!
