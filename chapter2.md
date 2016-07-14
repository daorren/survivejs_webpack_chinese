# webpack与其他工具的比较

至此，你了解到了webpack功能的强大，在此之前的开发中，我们将一些脚本机械地连接起来就感觉已经足够了，然而，时代变化了，分发你的javaScript越来越复杂。

这个问题随着单页应用（SPAs）的发展变得更加突出，这些单页应用通常会依赖许多库，我们现在有着许多如何加载这些库的解决方案，你可以一次性地全部加载完成，你也可以按需加载，webpack支持了许多这些灵活的解决方案。

Node.js和npm的发展，给了我们更多的想象空间，在npm之前，我们很难去处理依赖，现在，npm成为前端开发流行地解决依赖的方案，它使得管理依赖更加容易了。

## 构建工具和打包工具

在实际工作中，构建工具有很多，[Make](https://en.wikipedia.org/wiki/Make_%28software%29)就是其中的代表，它至今仍然具有无限地活力。为了将构建变得更容易，一些任务执行器（*task runners*），如[Grunt](http://gruntjs.com/)和[Gulp](http://gulpjs.com/)出现了，并且它们通过在npm上得插件而变得更加强大。

构建工具属于一个更高层次地工具，他们允许你做一些跨平台的事情。你最初使用构建工具遇到的问题就是分拣出各种资源并且将他们打包（bundle），这就是我们需要打包工具（*bundlers*）的原因，如[Browserify](http://browserify.org/),[Brunch](http://brunch.io/),[webpack](https://webpack.github.io/)等都是比较流行的打包工具。

让我们继续，[JSPM](http://jspm.io/)将对浏览器进行直接地包管理，他依赖于一个动态地模块加载器[System.js](https://github.com/systemjs/systemjs)。webpack2 将会支持System.js的语法。

与Browserify,Brunch和webpack不一样的是，JSPM跳过了开发时候的打包环节，你可以生成一个生产的包来使用他，你可以通过Glen Maddern的[视频](https://www.youtube.com/watch?t=33&v=iukBMY4apvI)来详细了解JSPM。

## Make

你可以说Make已经成为历史了，不错，这个诞生于1977年的工具，经历了各种各样的维护。Make允许你针对不同的目的编写出不同的任务，例如，你可能需要针对不同的环境如构建生产包，压缩文件，执行测试编写不同地任务，这个思想在其它的工具中也会体验出来。

虽然Make经常应用于C的项目中，但是这并不妨碍它用于其它语言，James Coglan在[《如何使用Make在JavaScript》](https://blog.jcoglan.com/2014/02/05/building-javascript-projects-with-make/)中详细介绍了如何在JavaScript项目中使用Make，如下面地代码

```make
PATH  := node_modules/.bin:$(PATH)
SHELL := /bin/bash

source_files := $(wildcard lib/*.coffee)
build_files  := $(source_files:%.coffee=build/%.js)
app_bundle   := build/app.js
spec_coffee  := $(wildcard spec/*.coffee)
spec_js      := $(spec_coffee:%.coffee=build/%.js)

libraries    := vendor/jquery.js

.PHONY: all clean test

all: $(app_bundle)

build/%.js: %.coffee
    coffee -co $(dir $@) $<

$(app_bundle): $(libraries) $(build_files)
    uglifyjs -cmo $@ $^

test: $(app_bundle) $(spec_js)
    phantomjs phantom.js

clean:
    rm -rf build
```

通过使用Make，你可以将你的任务通过使用make特定的语法和终端命令来进行模块化，然后很轻松地用webpack进行连接。

## Grunt

![Image of Grunt](http://survivejs.com/webpack/images/grunt.png)

Grunt是Gulp之前的主流，这得益于他的插件的设计思想，尽管插件的本身有些复杂。当这些插件的配置越来越复杂的时候，会变得特别难于理解。

这里是一个来自于[Grunt官方文档](http://gruntjs.com/sample-gruntfile)的例子，在这个配置中，我们定义了一个linting和一个watcher的任务，通过这个任务，我们可以在编写代码的时候实时地得到错误的警告。

**Gruntfile.js**

```js
module.exports = function(grunt) {
  grunt.initConfig({
    lint: {
      files: ['Gruntfile.js', 'src/**/*.js', 'test/**/*.js'],
      options: {
        globals: {
          jQuery: true
        }
      }
    },
    watch: {
      files: ['<%= lint.files %>'],
      tasks: ['lint']
    }
  });

  grunt.loadNpmTasks('grunt-contrib-jshint');
  grunt.loadNpmTasks('grunt-contrib-watch');

  grunt.registerTask('default', ['lint']);
};
```

实际上，你可以编写需要许多小任务来完成各种需求，Grunt的一个强大之处就是它隐藏了许多任务之间的连接操作。然而，时间长了，将会出现问题，你会对于任务地执行细节存在困惑。

> [grunt-webpack](https://www.npmjs.com/package/grunt-webpack)允许你在Grunt环境中使用webpack,你可以将关注点转移到使用webpack上。

## Gulp

![Image of Gulp](http://survivejs.com/webpack/images/gulp.png)

Gulp采用了不同的方式，它并不是通过配置每一个插件，而是直接地操作代码，Gulp依赖于底层的piping，如果你对于Unix非常熟悉，这里与Unix中的非常相似，它包括了加载器，过滤器和接收器三个组成部分。

加载器的作用是匹配文件，过滤器来操作这些文件（如生成JavaScript），最终，将这些文件传递给接收器（如一个文件夹）。下面是一个简单的*Gulpfile*示例，

**Gulpfile.js**

```js
var paths = {
    scripts: ['client/js/**/*.coffee', '!client/external/**/*.coffee']
};

// Not all tasks need to use streams
// A gulpfile is just another node program
// and you can use all packages available on npm
gulp.task('clean', function(cb) {
  // You can use multiple globbing patterns as
  // you would with `gulp.src`
  del(['build'], cb);
});

gulp.task('scripts', ['clean'], function() {
  // Minify and copy all JavaScript (except vendor scripts)
  // with sourcemaps all the way down
  return gulp.src(paths.scripts)
    .pipe(sourcemaps.init())
      .pipe(coffee())
      .pipe(uglify())
      .pipe(concat('all.min.js'))
    .pipe(sourcemaps.write())
    .pipe(gulp.dest('build/js'));
});

// Rerun the task when a file changes
gulp.task('watch', function() {
  gulp.watch(paths.scripts, ['scripts']);
});

// The default task (called when you run `gulp` from CLI)
gulp.task('default', ['watch', 'scripts']);
```

通过上面的配置代码，你可以在遇到困难的时候很容易地修改，你还可以包装已有的Node.js包成为Gulp插件。对比Grunt，你非常清晰这里面发生了什么。你可以对一些不具有共性的任务编写模块，这些也是新思想经常出现的地方。

> [gulp-webpack](https://www.npmjs.com/package/gulp-webpack)允许你在Gulp环境下使用webpack

> [fly](https://github.com/bucaran/fly)是一个与Gulp相似的工具，它依赖于ES6的generator

# Browserify

![Image of Browserify](http://survivejs.com/webpack/images/browserify.png)

处理javaScript模块是一个非常麻烦的事情，这门语言直到ES6的时候才诞生了模块化。因此，在这之前出现了许多解决方案，如[AMD](http://requirejs.org/docs/whyamd.html)等。

实际上，我们通常使用CommonJS和Node.js的形式，这个优势很明显就是你可以发送到npm上，避免重复地制造轮子。

[Browserify](http://browserify.org/)是一个解决模块化问题的方案，他可以将CommonJS的模块打包起来，你可以将它与Gulp一起使用。这里有一些小的转换工具作为进阶的用法，如[watchify](https://www.npmjs.com/package/watchify)提高了一个文件watcher允许你在开发的过程中打包。虽然browserify还需要一些完善但无疑它目前是一个比较好的工具。

Browserify的生态系统诞生了许多小模块，在这个方面，Brwoserify得益于Unix的设计思想，Browerify比webpack更易于接受，也是Webpack一个好的替代品。

## Brunch

![Image of Brunch](http://survivejs.com/webpack/images/brunch.png)

对比Gulp, Brunch在更高的抽象层进行操作，他像webpack一样，声明一个应用。下面是[Brunch官网](http://brunch.io/)的一个示例

```js
module.exports = {
  files: {
    javascripts: {
      joinTo: {
        'vendor.js': /^(?!app)/,
        'app.js': /^app/
      }
    },
    stylesheets: {
      joinTo: 'app.css'
    }
  },
  plugins: {
    babel: {
      presets: ['es2015', 'react']
    },
    postcss: {
      processors: [require('autoprefixer')]
    }
  }
};
```

Brunch通过使用一些命令，如`brunch new`, `brunch watch --server`和`brunch build --production`来进行任务的操作，这里面包含了许多内容而且可以使用插件扩展。

> 这里有一个Brunch[热替换](https://github.com/brunch/hmr-brunch)的方案。

## JSPM

![Image of JSPM](http://survivejs.com/webpack/images/jspm.png)

使用JSPM将不同于之前的工具，他通过在项目上安装一个很小的命令行实现，并且，他支持[SystemJS 插件](https://github.com/systemjs/systemjs#plugins)，允许你在你的项目中加载各种格式的文件。

考虑到JSPM仍然是一个年轻的项目，还有一些缺陷，如果你想体验可以试用一下，正如你所知道的，工具的目的是改变前端开发的环境，而且JSPM是其中强有力的竞争者。

## Webpack

![Image of Webpack](http://survivejs.com/webpack/images/webpack.png)

你可以说webpack是一个比browserify更加强大的工具。Browerify包含了众多小的工具，而webpack包含了一个核心功能，而且加了许多功能性的接口，这个核心的功能可以通过特定的loaders和plugins进行扩展。

webpack可以通过你项目中的`require`语句来生成你所需要的bundles。这个loader的机制可以使得CSS文件正常运行，并且支持`@import`语法。还有一些具体任务特定的插件，如压缩代码，本地化，热替换等等。

以一段代码作为示例，`require('style!css!./main.css')` 加载了*main.css*中的文件，并且通过CSS和style loaders从右向左进行处理。通过这个声明，将源文件提供给了webpack，更推荐使用webpack的配置来完成这一功能，下面是一个从[webpack官方指南](http://webpack.github.io/docs/tutorials/getting-started/)中得到的一个例子。

```js
var webpack = require('webpack');

module.exports = {
  entry: './entry.js',
  output: {
    path: __dirname,
    filename: 'bundle.js'
  },
  module: {
    loaders: [
      {
        test: /\.css$/,
        loaders: ['style', 'css']
      }
    ]
  },
  plugins: [
    new webpack.optimize.UglifyJsPlugin()
  ]
};
```

这个配置是使用JavaScript编写的，它的可扩展性非常的好。

这个配置的模型可能会使你感到一点迷惑，你可能会很难理解这里发生了什么，这在一些复杂的场景下显得更加突出，这些原因也是本书存在的价值。

## 为什么是Webpack？

我们为什么在Gulp和Grunt之上使用Webpack？webpack的强大之处在与处理复杂的打包问题，然而这个其他的工具也可能涉及。我推荐webpack是因为它支持热替换(HMR)，它还可以通过[babel-plugin-react-transform](https://github.com/gaearon/babel-plugin-react-transform)这个插件来使用React，我接下来会教你如何设置。

### 热替换

你可以已经对这些工具如[LiveReload](http://livereload.com/)或者[Browsersync](http://www.browsersync.io/)非常熟悉了，这些工具在你更改代码的时候可以自动的刷新页面。HMR把这个特性变得更加强大。在react的环境下，webpack允许程序维护自己的state,这听起来很简单，但是使得开发过程产生了巨大的变化。

需要注意的是，HMR在Browserify中通过[livereactload](https://github.com/milankinen/livereactload)进行实现，所以它不属于webpack的一个独有的特性。

### 分解bundle

除了HMR的特性，webpack的分解bundle的功能是十分强大的，他允许你通过多种的方式来分解bundles.你甚至可以在程序的执行过程中动态地加载他们。这种懒加载尤其在大的项目中十分有用，你可以加载你所需要的依赖在任何需要的时候。

### 资源哈希

通过webpack，你非常容易的将hash注入到bundle名称中（如 app.d587bbd6e38337f5accd.js)，当源码发生改变后，你可以验证浏览器是否发生了对应的改变。在理想状况下，分解bundle的特性允许浏览器只刷新一小部分。

### Loaders和Plugins

这些小的功能叠加起来，不出意外的话，你可以完成所有你想要的功能。而且如果你遗忘了一些东西，你可以通过loaders和plugins完成这些功能。

webpack有一个很复杂的学习曲线，即使是这样，这仍然是一个值得学习的工具，长时间学习它，你可以在今后的开发中节省大量的时间。如果你想了解更多与其他工具的对比，看一下[官方的对比](https://webpack.github.io/docs/comparison.html)。

### 结论

我希望通过本章帮助你了解到为什么webpack是一个值得学习的工具，他解决了许多web开发的通用问题，如果你很熟悉他，将会节省很多时间。

在接下来的章节中，我们将会详细学习webpack，你将会学习如何完成一个基础的开发和一个基本的配置，再往后的章节你将会探索一些高级的用法。

你应该讲webpack配合其他工具一起使用，因为他只是解决了打包的问题。当然，这些事情你不需要关心，使用*package.json*，`scripts`和webpack吧，接下来我们将进行详细的学习。
