# webpack与其他工具的比较

至此，你了解到了webpack的功能的强大，在此之前，将一些脚本连接起来就已经足够了，然而，时代变化了，分发你的javaScript越来越复杂。

这个问题随着单页应用（SPAs）的发展变得更加棘手，这些单页应用通常会依赖许多库，我们现在有着许多如何加载这些库的解决方案，你可以一次性地全部加载完成，你也可以按需加载，webpack支持了许多这些灵活的解决方案。

Node.js和npm的发展，给力我们更多地想象空间，在npm之前，我们很难去处理依赖，现在，npm成为前端开发流行地解决依赖的方案，它使得管理依赖更加容易了。

## 构建工具和打包工具

在实际工作中，构建工具有很多，[Make](https://en.wikipedia.org/wiki/Make_%28software%29)就是其中的代表，它至今仍然具有无限地活力。为了将构建变得更容易，一些任务执行器（*task runners*），如[Grunt](http://gruntjs.com/)和[Gulp](http://gulpjs.com/)出现了，他们通过在npm上得插件而变得更加强大。

构建工具属于一个更高层次地工具，他们允许你做一些跨平台的事情。你最初使用构建工具遇到的问题就是分拣出各种资源并且将他们打包（bundles），这就是我们需要打包工具（*bundlers*）的原因，如[Browserify](http://browserify.org/),[Brunch](http://brunch.io/),[webpack](https://webpack.github.io/)。

让我们继续，[JSPM](http://jspm.io/)将对浏览器进行直接地包管理，他依赖于一个动态地模块加载器[System.js](https://github.com/systemjs/systemjs)。webpack2 将会支持System.js的语法。

与Browserify,Brunch和webpack不一样的是，JSPM跳过了开发时候的打包环节，你可以生成一个生产的包来使用他，你可以通过Glen Maddern的[视频](https://www.youtube.com/watch?t=33&v=iukBMY4apvI)来详细了解JSPM。

## Make

你可以说Make已经成为历史了，不错，这个诞生于1977年的工具，经历了各种各样的维护。Make允许你对于不同的目的编写不同的任务，例如，你可能需要对不同的目的如构建生产包，压缩文件，执行测试编写不同地任务，这个思想你在其它的工具中也能发现。

虽然Make经常应用于C的项目中，但是这并不妨碍它用于其它语言，James Coglan在[《如何使用Make在JavaScript》](https://blog.jcoglan.com/2014/02/05/building-javascript-projects-with-make/)中详细介绍了这个，如下面地代码

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

Grunt是Gulp之前的主流，这得益于他的插件的设计思想，尽管插件的本身有些复杂。当他们配置越来越复杂的时候，会变得特别难于理解。

这里是一个来自于[Grunt官方文档](http://gruntjs.com/sample-gruntfile)的例子，在这个配置中，我们定义了一个linting和一个watcher的任务，通过他们，我们可以在编写代码的时候实时地得到错误的警告。

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

实际上，你可以需要许多小任务来完成各种需求，Grunt的一个强大之处就是它隐藏了许多任务之间的连接操作。然而，时间长了，将会出现问题，它会对于任务地执行顺序非常地难于理解。

> [grunt-webpack](https://www.npmjs.com/package/grunt-webpack)允许你在Grunt环境中使用webpack,你可以将关注点转移到使用webpack上。

## Gulp

![Image of Gulp](http://survivejs.com/webpack/images/gulp.png)

Gulp采用了不同的方式，它并不是通过配置每一个插件，而是直接地操作代码，Gulp依赖于底层的piping，如果你对于Unix非常熟悉，这里与Unix中的非常相似，你拥有加载器，过滤器和接收器。

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

通过上面的配置代码，你可以在遇到困难的时候很容易地修改，你还可以包装已有的Node.js包成为Gulp插件。对比Grunt，你非常清晰这里面发生了什么。你将会对一些不具有共性的任务编写模块，这些也是新方法经常出现的地方。

> [gulp-webpack](https://www.npmjs.com/package/gulp-webpack)允许你在Gulp环境下使用webpack

> [fly](https://github.com/bucaran/fly)是一个与Gulp相似的工具，它依赖于ES6的generator

# Browserify

![Image of Browserify](http://survivejs.com/webpack/images/browserify.png)

处理javaScript模块是一个非常麻烦的事情，这门语言知道ES6的时候才诞生了模块化。因此，在这之前出现了许多解决方案，如[AMD](http://requirejs.org/docs/whyamd.html)。

实际上，我们通常使用CommonJS和Node.js的形式，这个优势很明显就是你可以发送到npm上避免重复地制造轮子。

[Browserify](http://browserify.org/)是一个解决模块化问题的方案，他可以将CommonJS的模块打包起来，你可以将它与Gulp一起使用。这里有一些小的转换工具作为进阶的用法，如[watchify](https://www.npmjs.com/package/watchify)提高了一个文件watcher允许你在开发的过程中打包。虽然browserify还需要一些完善但无疑它目前是一个比较好的工具。

Browserify的生态系统诞生了许多小模块，在这个方面，Brwoserify得益于Unix的设计思想，Browerify比webpack更易于接受，也是Webpack一个好的替代品。

## Brunch

![Image of Brunch](http://survivejs.com/webpack/images/brunch.png)

对比Gulp,Brunch在更高的抽象层进行操作，他像webpack一样，声明一个应用。下面是[Brunch官网](http://brunch.io/)的一个示例

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

Brunch通过使用一些命令，如`brunch new`, `brunch watch --server`和`brunch build --production`。它包含了许多内容而且可以使用插件扩展。

> 这里有一个Brunch[热替换](https://github.com/brunch/hmr-brunch)的方案。

## JSPM

![Image of JSPM](http://survivejs.com/webpack/images/jspm.png)

使用JSPM将不同于之前的工具，他通过在项目上安装一个很小的命令行实现，并且，他支持[SystemJS 插件](https://github.com/systemjs/systemjs#plugins)，允许你在你的项目中加载各种格式的文件。

考虑到JSPM仍然是一个年轻的项目，还有一些缺陷，如果你想体验可以试用一下，正如你所知道的，工具的目的是改变前端开发的环境，而且JSPM是其中强有力的竞争者。

## Webpack

![Image of Webpack](http://survivejs.com/webpack/images/webpack.png)

