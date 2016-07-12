# 开始

确保你正在使用最新版的[Node.js](http://nodejs.org/)，我推荐使用最新的LTS（长时间支持）版本，并且你还要确保`node`和 `npm`可以正常在你的终端中使用。

上面完整的配置可以参照[GitHub](https://github.com/survivejs-demos/webpack-demo)，如果你不了解，可以作为参照。

## 创建你的程序

在开始之前，你应该为你的程序建立一个文件夹，并且创建一个*package.json*文件，npm通过这个文件来管理依赖，下面是一个基本的命令。

```sh
mkdir webpack-demo
cd webpack-demo
npm init -y # -y 生成了 *package.json*
```

你可以手动编写*package.json*来获得更高的灵活性，虽然我们可以通过*npm*来进行大部分的关于程序进本信息的配置，但是我们还是需要了解文件的具体用法，你可以通过查看[package.json选项](https://docs.npmjs.com/files/package.json)来获得更多信息。

## 安装Webpack

虽然webpack可以全局安装(`npm i webpack -g`)，我仍然推荐将其作为你项目的一个依赖，这个可以避免一些版本上的错误。

这个项目应该与**持续集成（CI）**更好地结合起来，一个持续集成系统可以本地安装依赖，编译并且使用，最终之间推送结果到服务器上。

安装一个Webpack到你的项目，执行

```sh
npm i webpack --save-dev # 或者使用 -D 作为缩写。
```

你应该看到webpack在你的*package.json*中的`devDependencies`中，这时候，webpack被安装到了本地的*node_modules*目录下，并且npm为其生成了一个可以执行的entry。

## 执行webpack

你可以通过使用`npm bin`来查看可执行文件的路径，大多数情况下，他应该存在于*./node_modules/.bin*， 。尝试在这个路径执行webpack通过`node_modules/.bin/webpack`。

当你执行过上面片段之后，你应该看到一个版本，一个接口导航的地址和一个很长的参数说明。我们不需要使用其中的大多数，但是我们可以知道这个工具已经被正确执行了。

```sh
webpack-demo $ node_modules/.bin/webpack
webpack 1.13.0
Usage: https://webpack.github.io/docs/cli.html

Options:
  --help, -h, -?
  --config
  --context
  --entry
...
  --display-cached-assets
  --display-reasons, --verbose, -v

Output filename not configured.
```

## 文件夹结构

一个程序不能单单的只有*package.json*，我们需要丰富它的内容。让我们一开始先实现一个小的web站点，他可以加载我们通过webpack打包的JavaScript代码，当我们完成这个功能的时候，文件目录看起来是这个样子：

```sh
- app/
  - index.js
  - component.js
- build/
- package.json
- webpack.config.js
```

我们可以将*app/*下面的文件打包放在*build/*下面，为了能达到这个目的，我们需要配置对应的*webpack.config.js*文件。

## 配置资源

我想你并不会厌倦`Hello world`的示例，我们可以像这样设置一个模块

**app/component.js**

```js
module.exports = function () {
  var element = document.createElement('h1');

  element.innerHTML = 'Hello world';

  return element;
};
```

下一步，我们需要对我们的应用配置一个入口，他通过`require`，包含了我们的模块然后在DOM上渲染。

**app/index.js**

```js
var component = require('./component');

document.body.appendChild(component());
```

## 配置webpack

我们需要告诉webpack如何操作我们刚才的资源，我们创建了一个*webpack.config.js*文件，webpack和它的开发服务器可以检测到这些文件。

为了让代码更好的可以维护，我们使用[html-webpack-plugin](https://www.npmjs.com/package/html-webpack-plugin)来为我们的程序生成*index.html*，在项目中安装它：

```sh
npm i html-webpack-plugin --save-dev
```

下面是完整的配置来在build目录下生成bundle

**webpack.config.js**

```js
const path = require('path');
const HtmlWebpackPlugin = require('html-webpack-plugin');

const PATHS = {
  app: path.join(__dirname, 'app'),
  build: path.join(__dirname, 'build')
};

module.exports = {
  // Entry accepts a path or an object of entries.
  // We'll be using the latter form given it's
  // convenient with more complex configurations.
  entry: {
    app: PATHS.app
  },
  output: {
    path: PATHS.build,
    filename: '[name].js'
  },
  plugins: [
    new HtmlWebpackPlugin({
      title: 'Webpack demo'
    })
  ]
};
```

这个`entry`路径可以是一个相对的地址，这个时候可以使用[context](https://webpack.github.io/docs/configuration.html#context)来配置查找的上下文。大部分的配置字段最好都用一个绝对的路径，我建议在任何时候都使用绝对路径来避免冲突。

如果你执行`node_modules/.bin/webpack`，你将会看到下面的输出：

```sh
Hash: 2a7a7bccea1741de9447
Version: webpack 1.13.0
Time: 813ms
     Asset       Size  Chunks             Chunk Names
    app.js    1.69 kB       0  [emitted]  app
index.html  157 bytes          [emitted]
   [0] ./app/index.js 80 bytes {0} [built]
   [1] ./app/component.js 136 bytes {0} [built]
Child html-webpack-plugin for "index.html":
        + 3 hidden modules
```

这里告诉了我们很多，下面将进行注释：

- `Hash: 2a7a7bccea1741de9447` 本次build的hash值，你可以使用这个通过`[hash]`字段来验证资源。我们将详细的在*Adding Hashes to Filenames*章节中详细讨论
- `Version: webpack 1.13.0` webpack的版本
- `Time: 813ms` 打包所需要的时间
- `app.js 1.69 kB 0 [emitted] app` 打包后资源的名称，大小，打包后关联的**块**的id，状态信息和块名称。
- `[0] ./app/index.js 80 bytes {0} [built]` 被打包资源的id，名称，大小，被打到块的id以及状态
- `Child html-webpack-plugin for "index.html":` 这个是插件的输出，在这个例子中是*html-webpack-plugin*的输出。
- `+ 3 hidden modules`告诉你webpack生成的东西，在`node_modules`目录下，你可以使用`webpack --display-modules`展现详细的信息，也可以参照[Stack Overflow](https://stackoverflow.com/questions/28858176/what-does-webpack-mean-by-xx-hidden-modules)来查看更多的解释。

观察`build/`下面生成的内容，如果你观察仔细，你会发现这些内容就是webpack打包出来的内容，你可以通过打开`build/index.html`文件来运行，在OS X的环境下，通过`open ./build/index.html`

## 添加一个打包的快捷方式

通过执行`node_modules/.bin/webpack`的命令来执行webpack看起来有点多余，我们可以解决这个问题。我们可以通过npm和*package.json*作为一个任务执行器来做一些如下的配置。

**package.json**

```js
...
"scripts": {
  "build": "webpack"
},
...
```
你可以执行这些脚本通过*npm run*。例如，在这个例子中，使用*npm run build*，你将会得到与刚才同样的结果。这个生效是因为npm把*node_modules/.bin*这个文件路径加载了起来，所以我们才可以通过`"build": "webpack"`来达到执行`"node_modules/.bin/webpack"`同样的效果。

## 结论

我们已经完成了一个非常基本的webpack配置，尽管他看起来并不是很好，每次我们修改，我们都需要手动执行`npm build`,然后刷新浏览器，体验十分不好。

这个时候我们就需要webpack的进阶用法了，为了更好的学习进阶的用法，我们将会在下一节中先讨论如何分解webpack的配置。
