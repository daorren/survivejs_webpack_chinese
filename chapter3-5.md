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

下面的表格是从[文档](https://webpack.github.io/docs/configuration.html#devtool)中根据打包速度排列的支持sourcemap的列表。soucemap质量越低，打包速度越快。

|Sourcemap type|Quality   | Notes  |
|:-:|:-:|:-:|
| `eval`  | 生成代码  | 每个模块都被`eval`执行，并且存在`@sourceURL`  |
| `cheap-eval-source-map`  | 转换代码（行内）   | 每个模块被`eval`执行，并且sourcemap作为`eval`的一个dataurl  |
| `cheap-module-eval-source-map`  | 原始代码（只有行内）  | 同样道理，但是更高的质量和更低的性能  |
| `eval-source-map` | 原始代码  | 同样道理，但是最高的质量和最低的性能  |

你可以从`eval-source-map`开始试用，逐渐地转变成其他的选项感受一下性能的差距。

webpack还可以生成生产环境下友好的sourcemaps。它可有在浏览器需要的时候引用单独的文件。在这个场景下你依然可以很轻松地进行调试。下面是他们的功能特点。

|Sourcemap type|Quality   | Notes  |
|:-:|:-:|:-:|
| `cheap-source-map`  | 转换代码（行内）| 生成的sourcemap没有列映射，从loaders生成的sourcemap没有被使用  |
| `cheap-module-source-map`  | 原始代码（只有行内）  | 与上面一样除了每行特点的从loader中进行映射  |
| `source-map` | 原始代码  | 最好的sourcemap质量有完整的结果，但是会很慢 |

`source-map`在这里是一个默认值，尽管使用他生成代码会花费很长时间，换来了更高的质量。如果你不考虑生产环境下的sourcemap，你可以忽略此项。

还有一些其他影响sourcemap生成的参数。

```js
const config = {
  output: {
    //  你可以使用[file]，[id]和[hash]来修改生成sourcemap的文件名
    //  默认的选项在大多数情况下足够了
    sourceMapFilename: '[file].map', // Default

    // 这个是 sourcemap 文件名模板，依赖于devtool的选项，你一般情况下不要修改它
    devtoolModuleFilenameTemplate: 'webpack:///[resource-path]?[loaders]'
  },
  ...
};
```

> 这里有devtool的[官方文档](https://webpack.github.io/docs/configuration.html#output-sourcemapfilename)

## SourceMapDevToolPlugin

如果你想更好的控制sourcemap的生成，你可能需要使用`SourceMapDevToolPlugin`作为替代。通过这个你可以控制生成SourceMap的范围。当你使用插件的时候，你可以跳过`devtool`的选项。

下面是一个来自[官方文档的例子](https://webpack.github.io/docs/list-of-plugins.html#sourcemapdevtoolplugin)

```js
const config = {
  plugins: [
    new webpack.SourceMapDevToolPlugin({
      // 像loaders一样匹配文件
      test: string | RegExp | Array,
      include: string | RegExp | Array,

      // `exclude` 匹配文件名，而不是包名
      exclude: string | RegExp | Array,

      // 如果文件名设置了，输出这个文件
      // 参考 `sourceMapFileName`
      filename: string,

      // This line is appended to the original asset processed. For
      // instance '[url]' would get replaced with an url to the
      // sourcemap.
      append: false | string,

      // See `devtoolModuleFilenameTemplate` for specifics.
      moduleFilenameTemplate: string,
      fallbackModuleFilenameTemplate: string,

      module: bool, // If false, separate sourcemaps aren't generated.
      columns: bool, // If false, column mappings are ignored.

      // Use simpler line to line mappings for the matched modules.
      lineToLine: bool | {test, include, exclude}
    }),
    ...
  ],
  ...
};
```

## 为样式文件的Sourcemaps

你可以通过一个query参数使用样式的Sourcemap文件。就想*css-loader*,*sass-loader*,*less-loader*，除了一个`?sourceMap`（例如`css?sourceMap`）
