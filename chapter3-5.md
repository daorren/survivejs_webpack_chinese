# 使用sourcemaps

![image of sourcemap](http://survivejs.com/webpack/images/sourcemaps.png)

为了让我们更好的调试程序，我们可以设置soucemap方便对代码和样式进行调试。sourcemap可以帮助你快速定位问题发生的地方，wepack可以生成两种sourcemap,包括在bundle中的行内sourcemaps和单独的sourcemap文件两种形式.行内的sourcemap在开发过程中有着非常好的调试体验，单独的sourmap用于生产环境中，可以使得bundle的大小降低。

我下面将会给你演示如何使用javaScript的sourcemaps。当你想查看一个细节的时候，最好的方法是看它的文档。比如使用TypeScript你可能需要一个特定的标志来让它生效。

## 在开发过程中使用Sourcemap

为了能在开发中使用sourcemap，我们需要配置一个`eval-source-map`的配置。而在生产过程中，我们使用规范的单独的soucemap文件。

**webpack.config.js**

```js
...
switch(process.env.npm_lifecycle_event) {
  case 'build':
    config = merge(
      common,
      {
        devtool: 'source-map'
      },
      parts.setupCSS(PATHS.app)
    );
  default:
    config = merge(
      common,
      {
        devtool: 'eval-source-map'
      },
      parts.setupCSS(PATHS.app),
      ...
    );
}
module.exports = validate(config);
```

`eval-source-map`初始化构建的时候非常的缓慢，但是再次构建和生成文件的时候速度就会加快，你也可以使用类似`cheap-module-eval-source-map`和`eval`来加快速度，只不过他们生成了一个质量不高的sourcemap。所有的`eval`选项将会生成sourcemap作为你javascript代码的一部分。

你可以在你的工作中使用这些生成好的sourcemap，参照[Chrome](https://developer.chrome.com/devtools/docs/javascript-debugging)，[Firefox](https://developer.mozilla.org/en-US/docs/Tools/Debugger/How_to/Use_a_source_map)，[IE Edge](https://developer.microsoft.com/en-us/microsoft-edge/platform/documentation/f12-devtools-guide/debugger/#source-maps)和[Safari](https://developer.apple.com/library/safari/documentation/AppleApplications/Conceptual/Safari_Developer_Guide/ResourcesandtheDOM/ResourcesandtheDOM.html#//apple_ref/doc/uid/TP40007874-CH3-SW2)来获得更多的细节。

## webpack 支持的sourcemap类型

尽管两种类型的sourcemap，包括`eval-source-map`和`eval`在我们的开发过程中已经足够了，webpack给我们提供了更多类型的sourcemap，这些类型可以在你的bundles里自动生成，并且在生产环境中不可用。

下面的表格是从[文档](https://webpack.github.io/docs/configuration.html#devtool)中根据打包速度排列的支持sourcemap的列表。soucemap质量月底，打包速度越快。
