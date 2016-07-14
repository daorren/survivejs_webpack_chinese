# 自动刷新CSS

在本节中我们将会讨论如何使得CSS在我们的项目中自动刷新，一个很简洁的事情是，webpack在这个场景中不需要强制性的全部刷新，并且，我们可以做一些更智能的事情。

## 加载CSS

在开始之前，我们需要安装两个loader

```bin
npm i css-loader style-loader --save-dev
```

我们现在拥有了需要的loaders，接下来，我们需要将他们与webpack连接起来。

**libs/part.js**

```js
...
exports.setupCSS = function(paths) {
  return {
    module: {
      loaders: [
        {
          test: /\.css$/,
          loaders: ['style', 'css'],
          include: paths
        }
      ]
    }
  };
}
```

我们需要将其与我们的主配置文件连接，

**webpack.config.js**

```js
...
switch(process.env.npm_lifecycle_event) {
  case 'build':
//  config = merge(common, {});
    config = merge(
      common,
      parts.setupCSS(PATHS.app)
    );
    break;
  default:
    config = merge(
      common,
      parts.setupCSS(PATHS.app),
      ...
    );
}
module.exports = validate(config);
```

这个配置文件告诉我们凡是以`.css`结尾的文件都将会传入到给定的loader中，`test`字段并没有匹配到js文件，这些loaders是从右向左执行的。

在这个例子中*css-loader*首先被执行，接着是*style-loader*。*css-loader*可以处理css文件中的`@import`和`url`语句，*style-loader*可以处理JavaScript中的`require`语句，就想类似于Sass和LESS一样的CSS预处理器。

> loaders作用于将一些资源转换为另一些资源，loaders直接可以被串行化，就想Unix中的piping一样，你可以参照文章[什么是loaders](http://webpack.github.io/docs/using-loaders.html)，和[loaders列表](http://webpack.github.io/docs/list-of-loaders.html)。

> 如果`include`字段没有设置，webpack将转化项目中所有符合的文件，这极大地降低了性能，所以，在配置loaders的时候，配置`include`字段是非常必要的。当然，还有一个`exclude`字段用于排除相关文件。

## 初始化CSS

事实上CSS文件是这个样子的，

**app/main.css**

```css
body {
  background: cornsilk;
}
```

我们接下来需要让webpack连接到它，但是webpack只能通过`require`语句来连接一个文件。

**app/index.js**

```js
require('./main.css');
...
```

现在执行`npm start`，如果你使用了默认端口，在浏览器上打开`http://localhost:8080`。

打开这个css文件，改变背景颜色为`lime`(`background: lime`)，样式变得更加美观了一点。

![image of hello2](http://survivejs.com/webpack/images/hello_02.png)

> 另外一个加载CSS的方法是针对CSS文件定义一个单独的入口。

## 理解CSS范围和CSS组件

当你使用这种方式来引用了一个CSS文件，webpack将会在你引用的地方将样式打包进这个文件的bundle中。假设你在使用*style-loader*，webpack将把他包含在html的`<style>`标签中，这意味着默认情况下这个样式的默认范围就是全局的。

特殊的，你可以使用[CSS Modules](https://github.com/css-modules/css-modules)来定义非全局的CSS。webpack的CSS loader支持这个特性，需要你通过`css?modules`来激活这个功能，然后将你的全局css通过`:global(body){ ... }`来进行声明。

> 关于query的语法`css?modules`将会在*Loader Definition*的章节中详细描述，并且在webpack中有多重方法可以达到同样的效果。

在这个例子中，可以通过`require`语句来带给你非全局的class，你可以将它针对元素进行绑定，就像下面这样。

**app/main.css**

```js
:local(.redButton) {
  background: red;
}
```

我们接下来可以这样绑定元素

**app/component.js**

```js
var styles = require('./main.css');
...
// Attach the generated class name
element.className = styles.redButton;
```

尽管这种用法感到有点奇怪，使用非全局的方式可以减少你许多处理CSS的错误，我们将会在我们的项目中使用一些传统的写法，但是这种方式我们也需要了解。我想一种组合模式一样，值得我们学习。

## 结论

在本节中，你学习了如何在开发过程中搭建webpack并且自动刷新浏览器，在下一章节中，我们将讨论一个能极大提高开发效率的功能——sourcemaps。
