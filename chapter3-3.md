# 浏览器自动刷新

一些像[LiveReload](http://livereload.com/)和[Browsersync](http://www.browsersync.io/)允许当我们修改我们的应用的时候，浏览器自行刷新，他们甚至可以米面CSS变动时候的自动刷新。

首先，我们需要开启webpack的*watch*模式，你可以通过`webpack --watch`来开启。一旦开启了这个功能，它将会检测文件发生的变化，然后自行编译打包。一个我们知道的工具*webpack-dev-server*在更高的层次上完成了这个功能。

*webpack-dev-server*是一个运行在内存中的开发服务器。当你开发应用的时候，浏览器自行刷新。它仍然支持webpack的一个更高级的功能**热替换(HMR)**，他使得浏览器不会使得整个页面的状态都进行刷新，这提高了开发像React之类应用的效率。

## 开始使用*webpack-dev-server*

首先执行

```bin
npm i webpack-dev-server --save-dev
```

通过上面的载入，你可以创建命令在`npm bin`指定的目录中，在哪里，你可以运行*webpack-dev-server*，最容易的方法是运行*webpack-dev-server --inline*。`--inline`其中了一个服务器的*inline*模式，可以重写结果的url地址。

### 在项目中使用 webpack-dev-server

在项目中使用*webpack-dev-server*，我们可以遵循之前的思路，在*package.json*中加入新的`scripts`

**package.json**

```js
...
"scripts": {
  "start": "webpack-dev-server",
  "build": "webpack"
},
...
```

我们可以将`--inline`的部分写在webpack的配置里面，我建议你的npm *scripts* 要保持足够的简洁，将复杂的地方都给配置文件来完成，你可以把注意力直接集中到配置文件本身上来。

如果你现在执行*npm run start*或者*npm start*，你可以在你的终端下看到这些内容，

```bin
> webpack-dev-server

[webpack-validator] Config is valid.
http://localhost:8080/webpack-dev-server/
webpack result is served from /
content is served from .../webpack-demo
Hash: 2dca5a3850ce5d2de54c
Version: webpack 1.13.0
```

这意味着开发服务器已经其中，你可以在你的浏览器访问[http://localhost:8080](http://localhost:8080)看到结果。

如果你修改了代码，你应该会在终端上看到输出，问题是这时候浏览器并不会自动刷新，它需要我们接下来的配置。

![image of hello](http://survivejs.com/webpack/images/hello_01.png)

> 如果你没有在浏览器上看到任何东西，尝试着改变端口，可能之前的端口被占用了，你可以使用`netstat -na | grep 8080`来查看是否8080被占用。这个命令可能会根据你的平台而有相应的改变。

## 配置热替换(HMR)

热替换的功能处于*webpack-dev-server*的上层，它通过一个借口来替换当前的模块。例如，*style-loader*可以不刷新地更新CSS，通过HMR来替换CSS显得很容易，因为它不会包含任何的状态。

HMR也可以应用于JavaScript，但是考虑到JavaScript的状态，它比CSS要复杂。在*配置React*的章节中，我们将会讨论如何将它与React结合起来，并且你可以把这个思想利用在其他的地方。

你可以通过`webpack-dev-server --inline --hot`来使用这个功能，`--hot`激活了HMR的模块通过一个为特定目的设计的插件，并且编写一个JavaScript文件接入的入口。

## 为HMR定义配置

为了让我们的配置可以更好的维护，我将会分离HMR的配置，我们将保持*webpack.config.js*简单并且可重用，我们可以将它发布至npm的平台上，这个时候就可以重用了。

我们将配置的部分放至*libs/parts.js*，

**libs/parts.js**

```js
const webpack = require('webpack');

exports.devServer = function(options) {
  return {
    devServer: {
      // Enable history API fallback so HTML5 History API based
      // routing works. This is a good default that will come
      // in handy in more complicated setups.
      historyApiFallback: true,

      // Unlike the cli flag, this doesn't set
      // HotModuleReplacementPlugin!
      hot: true,
      inline: true,

      // Display only errors to reduce the amount of output.
      stats: 'errors-only',

      // Parse host and port from env to allow customization.
      //
      // If you use Vagrant or Cloud9, set
      // host: options.host || '0.0.0.0';
      //
      // 0.0.0.0 is available to all network devices
      // unlike default `localhost`.
      host: options.host, // Defaults to `localhost`
      port: options.port // Defaults to 8080
    },
    plugins: [
      // Enable multi-pass compilation for enhanced performance
      // in larger projects. Good default.
      new webpack.HotModuleReplacementPlugin({
        multiStep: true
      })
    ]
  };
}
```

我们可以将这段代码以后重用起来，然后在主配置里连接他，

**webpack.config.js**

```js
const path = require('path');
const HtmlWebpackPlugin = require('html-webpack-plugin');
const merge = require('webpack-merge');
const validate = require('webpack-validator');

const parts = require('./libs/parts');
...
// Detect how npm is run and branch based on that
switch(process.env.npm_lifecycle_event) {
  case 'build':
    config = merge(common, {});
    break;
  default:
//  config = merge(common, {});
    config = merge(
      common,
      parts.devServer({
        // Customize host/port here if needed
        host: process.env.HOST,
        port: process.env.PORT
      })
    );

}

module.exports = validate(config);
```

执行`npm start`之后浏览**localhost:8080**，试着去修改*app/component.js*，它应该会自动刷新浏览器，你要知道刷新JavaScript是复杂的，而CSS是比较简单的，你可以了解刷新css在下一章节。

## 在Windows, Ubuntu和Vagrant上使用HMR

你可能会在Windows, Ubuntu和Vagrant遇到一些问题，尝试着这样去解决，

**libs/parts.js**

```js
const webpack = require('webpack');

exports.devServer = function(options) {
  return {

    watchOptions: {
      // Delay the rebuild after the first change
      aggregateTimeout: 300,
      // Poll using interval (in ms, accepts boolean too)
      poll: 1000
    },

    devServer: {
      ...
    },
    ...
  };
}
```

如果默认的配置不能工作的话，就使用上面这个，他根据文件系统，变得更加资源敏感。

## 从网络上连接开发服务器

我们很有必要定制主机和端口号根据我们的环境（如windows上是`SET PORT=3000`, Unix上是`export PORT=3000`）,这个当你使用其他的设备来访问很重要，默认的配置在大多数情况下已经足够了。

为了连接你的服务器，你需要知道你的机器上的ip，在Unix上使用`ifconfig | grep inet`，在Windows上，使用`ipconfig`。也可以通过一些如[node-ip](https://www.npmjs.com/package/node-ip)的npm包。特殊的在windows机器上你需要设置`HOST`来配置。

## 其他配置*webpack-dev-server*的方法

我们可以通过*webpack-dev-server*的配置在终端中使用。但是我发现在webpack的配置中来配置可以增加可读行性，而且便于发现问题。

另外一种选择是，我们可以使用一个Express的服务器，并且将*webpack-dev-server*充当它的一个[中间件](https://webpack.github.io/docs/webpack-dev-middleware.html)。还可以通过[Node.js API](https://webpack.github.io/docs/webpack-dev-server.html#api)来配置。这是一个好的方案如果你想更灵活的来控制它。

> [dotenv](https://www.npmjs.com/package/dotenv)允许你通过*.env*文件来定义你的环境变量，这个可以给你的开发带来一定的方便。

> CLI与Node.js API有一定的[区别](https://github.com/webpack/webpack-dev-server/issues/106)，这就是为什么有一些人喜欢完全的使用Node.js的API。

## 结论

在本章节中，我们讨论了如何设置webpack来自动刷新你的浏览器，你可以把这个特性运用在CSS上，我们将会在下一节来讨论它。
