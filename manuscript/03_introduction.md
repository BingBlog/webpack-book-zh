对于Web开发来说，[Webpack](https://webpack.js.org/)解决了一个最基本的问题－集成打包，进而简化了开发过程。Webpack可以对各类资源进行处理，例如JavaScript,CSS和HTML，将它们转化成浏览器容易处理的格式。解决了这个问题，也就让Web开发不再那么痛苦。

但是由于它复杂的配置驱动方法，Webpack并不是容易掌握的工具。本教程的目的就是帮助你开始学习Webpack，然后逐步超越基础！

## Webpack到底是什么？

Web浏览器可以处理HTML，JavaScript和CSS。最简单的开发方式就是编写出浏览器可以直接处理的文件。但问题是，开发的应用最终变得笨重。尤其是在进行Web应用开发的时候，我们也总会遇到这个问题。

最简单的方式是将JavaScript代码都打包成一个文件，然后浏览器仅加载一次。但是这也存在问题，随着体积的增长，单个文件变得难以加载，你就需要使用其它的方法。最常用的做法就是，再次把文件拆分以便利于缓存。你甚至需要根据需求动态地来加载各个依赖。

随着应用程序的发展，处理它的复杂性也随之增长。Webpack就是用来解决这个问题的。它通过静态分析解决了前面所描述的问题。这个过程使当下前端开发中最基本的问题得到大部分的解决。一旦很好的解决了这个问题，你将受益匪浅。

Webpack并不是解决这个问题的唯一选择，各种不同的工具也早已出现。任务运行管理工具，如Grunt和Gulp，是高级工具中更好的例子。但是常常遇到的问题是，需要你手动编写工作流。将这个问题交给一个打包工具，例如webpack，是一个进步。


### Webpack如何改变现状？
Webpack采用了另外一种方式。它让你将你的项目视为一个依赖关系图。你可以在项目中的index.js文件中通过使用标准的```require```或者```import```语法来加载所有依赖。你也可以用同样的方式引入样式文件和其它资源。

Webpack基于你指定的配置和你的代码，完成了所有的预处理工作，然后生成了代码包。这些配置的声明方式很强大，但是也有一点儿难学。不过，只要你理解了Webpack的工作原理，它便会成为一个不可却少的工具。这本书就是让你越过最开始的那个小山坡，然后能够走的更远。

## 你将学到什么？
这本书是对*[Webpack的官方文档](https://webpack.github.io/docs/)*的补充。官方文档在很多方面有更深入讲解，本书可以作为官方文档的伴侣。俗话说，“万事开头难”，这本书就像一个概览，可以让你开始学习时不那么痛苦，如果你是有一定基础的使用者，本书也会让你温故而知新。

您将学会开发一个基本的Webpack配置工具，可以分别达到开发和生产的目的。你也会学习很多先进的技术，使你从Webpack强大的功能中获益匪浅。

>本书基于webpack2。如果你想使用webpack1，[官方迁移文档](https://webpack.js.org/guides/migrating/)中包含了所有主要版本的更改。[webpack-cli的GitHub官方库](https://github.com/webpack/webpack-cli)中也有方法可以让你从webpack1迁移到webpack2。

## 这本书如何组织的？
本书从讲解何为webpack开始。然后，你将会看到很多部分，每部分从不同的方向讨论了webpack。在这些章节中，你将学习如何开发webpack配置。（在之后的实际开发中）它们也可以作为参考资料。
本书包含一下几个章节：
 - **Developing－开发**部分讲解如何初始化并启动webpack。本章介绍了诸如热加载的特性，并讲解如何撰写易于维护的配置文件。
 - **Styling－样式**部分花了很大的篇幅来讲解样式相关的话题。你将学习如何通过加载样式文件，如何在设置中引入自动前缀等技术。
 - **Loading**部分详细讲解了定义webpack的加载器，并向你介绍了如何加载诸如图片、字体、JavaScript等资源。
 - **Building－构建**部分介绍了source maps，打包的想法和代码拆分。你也将学习如何组织你自己的构建。
 - **Optimizing－优化**部分将你的构建放到最终的生产质量的层面上，向你介绍了许多小技巧，来使其变体积变得更小。同时，你也将学习如何调整webpack的性能。
 - **Output－输出**部分讨论了webpack的输出选项。虽然它的名字叫"webpack"，但它不仅限于web。你将会学习如何管理多页面配置并了解服务端渲染的基本概念。
 - **Techniques－技术**部分讨论了几个特定的想法，包括动态加载，web workers，国际化和部署应用程序。
 - **Packages－包**章节重点关注了npm和一些与webpack相关的结束。你将学习如何高效使用并且开发自己的npm包。
 - **Extending－扩展**部分想你展示了如何使用你自己的loaders-加载器和plugins-插件来扩展webpack。

 > 关于source map 没有找到好的译文，具体解释可见阮一峰的博客[JavaScript Source Map 详解](http://www.ruanyifeng.com/blog/2013/01/javascript_source_map.html)

在你阅读完本书所有主要的内容后，最后一章，有一个简短的总结，重述了本书的要点。本章节还包含一个检查清单，让你可以根据自己的项目来回顾本书所将的技术。

本书最后的附录包含了一些次要的话题以及对一些对主要内容更深入的探索。你可以根据自己的兴趣来有选择性的阅读。最后一个附录是疑难解答，可以帮助你理解如何调试webpack错误。

## 这书适合那些读者？

希望你对JavaScript和Node.js有基本的掌握。对npm的使用水平也到了初级阶段。如果你对Webpack有一些了解，那就太棒了。读完此书，将会加强你对这些工具的理解。

如果你是Webpack老手，你也能学到一些东西。浏览本书，看看你能否学习一些技术。我已经尽我所能的使本书包含了webpack这个工具所有的部分，甚至包括一些比较伤脑经的部分。

如果你仍然有困惑，可以尝试从本书的社区中寻找帮助。如果你卡住了或者有些地方理解不了，我们可以提供帮助。你的任评价都将会使本书的质量得到提升。

## 如何获得本书？
如果你不是很了解Webpack，建议仔细阅读前两章。后面的可以浏览下，先看感兴趣的部分。如果你已经对Webpack有一定了解，涉猎并选择你觉得有价值的技术点进行学习。

## 本书的版本
由于本书不断创新的步伐，并经过很多修补和改进，所以有很多个版本。我通过本书[官方博客](http://survivejs.com/blog/)对新老版本信息进行发布，版本信息会告诉你各个版本的差异。通过GitHub仓库检测也是一个很好的方法。推荐使用Github对比工具。例如：
```
https://github.com/survivejs/webpack-book/compare/v1.7.0...v1.8.7
```
在浏览器中输入该地址，页面会向你展示两个版本的提交信息。你将看到本书每一行的更改

本书当前的本书是1.8.7

本书是一项持续性的工作，非常欢迎通过各种渠道进行反馈。我将会基于需求来扩展本教程，使其可以更好的惠及你我他。你也可以为本书贡献出你的更改和源代码。

本书的部分利润将会用于工具自身的开发。

## 得到支持
“人无完人”，对于书中内容，你可能会有一些问题。可以通过下面的方式进行反馈：
 - 通过  *[GitHub Issue Tracker](https://github.com/survivejs/webpack/issues)*联系原作者
 - 加入到[Gitter Chat](https://gitter.im/survivejs/webpack)
 - 关注推特官方账号 *[@survivejs](https://twitter.com/survivejs)*或者直接联系作者*[@bebraw](https://twitter.com/survivejs)*
 - 给原作者发邮件*[info@survivejs.com](http://info@survivejs.com)*
 - 直接向作者提关于Webpack和React的问题：*[SurviveJS AmA](https://github.com/survivejs/ama/issues)*

在Stack Overflow上提问请用survivejs标记，我将会注意到。推特上，也可用#survivejs。


## 信息发布
我将通过一下频道发布SurviveJs的信息
 - *[Mailing list](http://eepurl.com/bth1v5)*
 - *[Twitter](https://twitter.com/survivejs)*
 - *[Blog RSS](http://survivejs.com/atom.xml)*

欢迎订阅

## 致谢
非常感谢Christian Alfoni帮助我制作这本书的第一个版本。这激励着整个SurviveJS项目。 您现在看到的版本是全部重写后的版本。

如果没有细心的编辑和我的编辑[Jesús Rodríguez](https://github.com/Foxandxss)的反馈，本书将远远达不到现在的质量。

没有最开始的版本"SurviveJS - Webpack and React"的努力，本书也将无缘与大家见面。任何支持了那本书的人都应该得到我的致谢。你可以用那本书中得到更详细的原因。

感谢Mike“Pomax”Kamermans，Cesar Andreu，Dan Palmer，ViktorJančík，Tom Byrer，Christian Hettlage，David A. Lee，Alexandar Castaneda，Marcel Olszewski，Steve Schwartz，Chris Sanders，Charles Ju，Aditya Bhardwaj，Rasheed Bustamam，José Menor，Ben Gale，Jake Goulding，Andrew Ferk，gabo，Giang Nguyen，@Coaxial，@khronic，Henrik Raitasola，Gavin Orland，David Riccitelli，Stephen Wright，MajkyBašista，Artem Sapegin，Gunnari Auvinen，JónLevy，Alexander Zaytsev，Richard Muller，Ava Mallory（Fiverr），Sun Zheng'an，Nancy（Fiverr），Aluan Haddad，Pedr Browne，Steve Mao，Craig McKenna，Tobias Koppers和许多其他人为本书贡献了直接反馈！