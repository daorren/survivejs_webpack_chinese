# 简介

[Webpack](https://webpack.github.io/) 通过解决web开发的打包问题，极大地提高了web开发的效率。它可以将各种资源如JavaScript, CSS, 和HTML打包成一个浏览器可以接受的形式。如果你熟练地运用webpack, 它可以将你在web开发中遇到的大量的痛点消除。
由于webpack是一个配置驱动的应用，所以它并不是一个非常容易上手的工具，这本书的目的就是帮助你熟练地掌握webpack。

## 什么是webpack？

浏览器是用来解析HTML,JavaScript和CSS文件的。一个最简单的开发方式是直接编写许多文件，然后将他们用浏览器来直接解析，这会让它们显得非常笨重，这个情况在我们开发web应用的时候非常常见。

一个解决这个问题的原始方法是将这些文件打包成一个文件，很显然这个不是一个很好的解决方案，你有时需要将它们分解开来，你甚至需要动态的加载你所需要的依赖，这个问题地复杂性随着应用地开发而逐渐提高。

Webpack就是来解决这个问题的. 它可以允许你使用不同的工作流和不同的工具来得到同样的结果。事实上，这些已经足够了，一些构建工具，如Grunt和Gulp，可以帮助你做这些事情，但是需要你编写许多配置。

## webpack改变了什么？

Webpack采用了另外的一条路径，它允许你将你的项目看作是一个依赖图，你需要一个`index.js`在你的项目中，它使用标准的`import`语句描述了项目所需要的依赖，而且你可以描述你的样式文件和其他的资源文件通过同样的方式。

Webpack为你做了所有预处理的事情，然后把符合你之前配置的bundles交给你。这些声明的配置是非常强大的，但是很难学习。然而，如果你理解了webpack是如何工作的，它会成为你不可或缺的工具。本书就是教你如何运用它的。

## 你将如何学习？

这本书作为[webpack 官方文档](https://webpack.github.io/docs/)的补充，由于官方文档包含了非常多的内容，它可能不是一个开始学习时候可以使用到的工具，本书的目的就是简化webpack的学习曲线同时给有经验的开发者一些灵感。

你将会学习到如何根据不同的环境（如开发和生产）来配置webpack，你还会学习到webpack的一些进阶的用法，它将会让你十分收益。

## 这本书是如何组织的？

最开始的两章为你介绍webpack和它的一些基本的内容，你将会使用基本的配置来适应项目的不同环境，前面的章节都是任务导向性的，你可以收藏它，防止你以后忘记了webpack的一些具体的用法。

书的后面一部分讨论了一些进阶的话题，你将会学到如何加载具体类型的文件，你也将会加深对webpack打包机制的理解，利用它创建一些包，编写一些loader。

## 谁适合阅读本书？

我希望你有一些基本的JavaScript和Node.js的知识，你应该使用过npm。如果你还了解一些webpack，那就更好了，通过学习，你会加深对它们的理解。

如果你已经了解了webpack，这本书还是有一些知识可以提供给你，浏览此书看看有没有适合你的部分，我最大努力的在本书中提供了webpack的棘手内容。

如果你学习地很困难，尝试着向社区围绕着本书寻求帮助，我们在那里等着你，任何关于本书的回应将使得书的内容变得更好。

## 我将如何开始？

如果你不太了解webpack，前两章你要仔细地学习，你可以略过其他的内容只选择你所感兴趣的。如果你已经了解了webpack，挑选你认为有价值的内容。

## 书的版本

本书收到了大量的更新，很难在这里罗列出版本，我维护了更新内容在本书的[博客](http://survivejs.com/blog/)中，他可以让你了解到每个版本所更新的内容，同样的，使用Github来了解更新，也是一个不错地方法，我推荐使用Github，你可以这样使用，

```
https://github.com/survivejs/webpack/compare/v1.2.0...v1.3.1
```
这个页面可以让你方便地查看到指定版本之间的提交，它虽然排除了私有的章节，但也足够了。

目前的版本是 1.3.1

本书还在创作中，欢迎提交反馈和讨论，通过反馈我可以将书创作地更好，你甚至可以将本书修改成适合你的版本，这都是允许的。

书的一部分收入将会用来支持本书和webpack本身。

## 获得帮助

没有任何地书是完美的，你可以提交issue来对书的内容提问，下面是提issue的一些途径，

- 通过[github的issue](https://github.com/survivejs/webpack/issues)联系我。
- 通过[Gitter](https://gitter.im/survivejs/webpack)来聊天。
- 在Twitter关注官方账号[@survivejs](https://twitter.com/survivejs)来获取官方的新闻，也可以通过[@bebraw](https://twitter.com/bebraw)直接联系我。
- 发送邮件到[info@survivejs.com](info@survivejs.com)
- 通过[SurviveJS AmA](https://github.com/survivejs/ama/issues)来问我任何关于webpack和react的问题

如果你在stack Overflow上提问，将这些问题加上 **survivejs** 的标签，我可以很方便地注意到它们，你可以在Twitter使用**#survivejs**话题来达到同样的效果。 

## 声明

我声明 SurviveJS 的信息通过下面的渠道发布：

- [邮件列表](http://eepurl.com/bth1v5)
- [Twitter](https://twitter.com/survivejs)
- [Blog RSS](http://survivejs.com/atom.xml)

## 感谢

感谢 Christian Alfoni 帮助我编写了本书的第一版，他鼓舞了我整个SurviveJS的项目，你现在看到的版本已经修改了很多了。

这本书也离不开耐心编写和整理反馈的我的编辑Jesús Rodríguez Rodríguez，感谢他。

这本书同样也离不开SurviveJS - Webpack and React这个项目的支持，我对曾经贡献过这个项目的所有人表示感谢，你可以通过本书获得更多的贡献。

感谢 Mike "Pomax" Kamermans, Cesar Andreu, Dan Palmer, Viktor Jančík, Tom Byrer, Christian Hettlage, David A. Lee, Alexandar Castaneda, Marcel Olszewski, Steve Schwartz, Chris Sanders和其他直接贡献本书的人。
