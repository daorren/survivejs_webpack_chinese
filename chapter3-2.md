# 分解配置文件

最开始的时候，webpack的配置文件可以包含在一个单独的文件中，但是随着程序的增长，你可能需要分解这个配置才更好地对它进行管理。你很有必要将他们根据环境来进行分类，然后你就很好的控制了打包后的内容。如何的分类并没有绝对正确的方法，但是一般有下面几条，

- 在多个文件中维护配置，通过Webpack的`--config`选项来连接，共享配置通过模块间的imports。你可以在[webpack/react-starter](https://github.com/webpack/react-starter)这个项目中看到具体的使用。
- 将配置放入一个库中，示例[HenrikJoreteg/hjs-webpack](https://github.com/HenrikJoreteg/hjs-webpack)
- 维护一个文件的配置，然后设立分支。通过*npm*的脚本（如`npm run test`）来决定使用哪个配置。npm在他的环境变量中设置了这些信息，你可以匹配他们来做你想做的事情。

我推荐使用最后的方案，我们可以很清楚的知道这里发生了什么，我开发了一个小的工具名为[webpack-merge](https://www.npmjs.org/package/webpack-merge)，我将会展示如何使用它。

## 设置webpack-merge

首先执行

```sh
npm i webpack-merge --save-dev
```

在项目中添加了*webpack-merge*

**webpack.config.js**

```js
const path = require('path');
const HtmlWebpackPlugin = require('html-webpack-plugin');

const merge = require('webpack-merge');


const PATHS = {
  app: path.join(__dirname, 'app'),
  build: path.join(__dirname, 'build')
};


// module.exports = { 


const common = {

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


var config;

// Detect how npm is run and branch based on that
switch(process.env.npm_lifecycle_event) {
  case 'build':
    config = merge(common, {});
    break;
  default:
    config = merge(common, {});
}

module.exports = config;

```

当我们做了这个改变之后，虽然说实际的功能仍然与之前一样，但是我们有了更多的扩展空间。我们可以将**热替换**接入进来使得浏览器自动刷新使得开发模式更好地提高了我们的效率。

## 连接 *webpack-validator*

为了使得我们更容易地开发配置，我们需要引入一个[webpack-validator](https://www.npmjs.com/package/webpack-validator)在我们的项目中，它可以提醒我们的配置与标准配置规则不合适的地方，这对我们学习和使用webpack起到了重要的作用。

首先安装：

```bin
npm i webpack-validator --save-dev
```

我们之后就可以将其引入到项目中了。

**webpack.config.js**

```js
const path = require('path');
const HtmlWebpackPlugin = require('html-webpack-plugin');
const merge = require('webpack-merge');
const validate = require('webpack-validator');
// ...
// module.exports = config;
// module.exports = validate(config);
```

如果你编写了错误的webpack配置，它将会给你提示和一下优雅的解决方法。

## Conclusion

尽管分解配置是一个非常微不足道的技术方案，但是它极大地扩展了我们的使用webpack的适用场景。每一种方案都有优点和缺点，我发现这个方案对于中小型的项目很有帮助，对于大型项目可能会有其他的解决方案。

现在我们已经拥有了足够的扩展空间，在下一章节中，我将会告诉你如何来利用webpack完成浏览器的自动刷新。
