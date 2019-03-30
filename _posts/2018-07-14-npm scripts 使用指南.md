---
layout: post
title: npm scripts 使用指南
subtitle: 彻底搞懂npm script脚本
date: 2018-07-14
author: Li Yucang
catalog: true
tags:
    - npm
---

# npm scripts 使用指南

##  npm 脚本

npm 允许在package.json文件里面，使用scripts字段定义脚本命令。

````
{
  // ...
  "scripts": {
    "build": "node build.js"
  }
}
````

上面代码是package.json文件的一个片段，里面的scripts字段是一个对象。它的每一个属性，对应一段脚本。比如，build命令对应的脚本是node build.js。

命令行下使用npm run命令，就可以执行这段脚本。

````
$ npm run build
# 等同于执行
$ node build.js
````

这些定义在package.json里面的脚本，就称为 npm 脚本。它的优点很多。

1. 项目的相关脚本，可以集中在一个地方。
2. 不同项目的脚本命令，只要功能相同，就可以有同样的对外接口。用户不需要知道怎么测试你的项目，只要运行npm run test即可。
3. 可以利用 npm 提供的很多辅助功能。

## 原理

npm 脚本的原理非常简单。每当执行npm run，就会自动新建一个 Shell，在这个 Shell 里面执行指定的脚本命令。因此，只要是 Shell（一般是 Bash）可以运行的命令，就可以写在 npm 脚本里面。

比较特别的是，npm run新建的这个 Shell，会将当前目录的node_modules/.bin子目录加入PATH变量，执行结束后，再将PATH变量恢复原样。

这意味着，当前目录的node_modules/.bin子目录里面的所有脚本，都可以直接用脚本名调用，而不必加上路径。比如，当前项目的依赖里面有 Mocha，只要直接写mocha test就可以了。

````
"test": "mocha test"
而不用写成下面这样。
"test": "./node_modules/.bin/mocha test"
````

由于 npm 脚本的唯一要求就是可以在 Shell 执行，因此它不一定是 Node 脚本，任何可执行文件都可以写在里面。

npm 脚本的退出码，也遵守 Shell 脚本规则。如果退出码不是0，npm 就认为这个脚本执行失败。

由于 npm 脚本就是 Shell 脚本，因为可以使用 Shell 通配符。

````
"lint": "jshint *.js"
````

上面代码中，*表示任意文件名，果要将通配符传入原始命令，防止被 Shell 转义，要将星号转义。

## 传参

向 npm 脚本传入参数，要使用--标明。

````
  "scripts": {
    "dev": "node build/dev-server.js",
  }

  npm run dev:
  node build/dev-server.js

  npm run dev  -- demo:
  node build/dev-server.js "demo"

  npm run dev  --  demo webtt:
   node build/dev-server.js "demo" "webtt"
````

## 执行顺序

如果 npm 脚本里面需要执行多个任务，那么需要明确它们的执行顺序。

如果是并行执行（即同时的平行执行），可以使用&符号。

````
$ npm run script1.js & npm run script2.js
````

如果是继发执行（即只有前一个任务成功，才执行下一个任务），可以使用&&符号。

````
$ npm run script1.js && npm run script2.js
````

这两个符号是 Bash 的功能。此外，还可以使用 node 的任务管理模块：script-runner、npm-run-all、redrun，这里后面有提到。

## 钩子

npm 脚本有pre和post两个钩子。举例来说，build脚本命令的钩子就是prebuild和postbuild。

````
"prebuild": "echo I run before the build script",
"build": "cross-env NODE_ENV=production webpack",
"postbuild": "echo I run after the build script"
````

用户执行npm run build的时候，会自动按照下面的顺序执行：

````
npm run prebuild && npm run build && npm run postbuild
````

因此，可以在这两个钩子里面，完成一些准备工作和清理工作。下面是一个例子：

````
"clean": "rimraf ./dist && mkdir dist",
"prebuild": "npm run clean",
"build": "cross-env NODE_ENV=production webpack"
````

npm 默认提供下面这些钩子：

````
prepublish，postpublish
preinstall，postinstall
preuninstall，postuninstall
preversion，postversion
pretest，posttest
prestop，poststop
prestart，poststart
prerestart，postrestart
````

自定义的脚本命令也可以加上pre和post钩子。比如，myscript这个脚本命令，也有premyscript和postmyscript钩子。

## 变量

npm 脚本有一个非常强大的功能，就是可以使用 npm 的内部变量。

首先，通过npm_package_前缀，npm 脚本可以拿到package.json里面的字段。比如，下面是一个package.json。

````
{
  "name": "foo", 
  "version": "1.2.5",
  "scripts": {
    "view": "node view.js"
  }
}
````

那么，变量npm_package_name返回foo，变量npm_package_version返回1.2.5。

我们也可以通过环境变量process.env对象，拿到package.json的字段值：

````
// test.js
console.log(process.env.npm_package_name); // foo
console.log(process.env.npm_package_version); // 1.2.5
````

npm_package_前缀也支持嵌套的package.json字段：

````
"repository": {
  "type": "git",
  "url": "xxx"
},
scripts: {
  "view": "echo $npm_package_repository_type"
}
````

## 实战

### 编译scss为css

我是SCSS的重度用户，平时工作都依赖SCSS。为了将SCSS编译成CSS，我使用了node-sass。首先，我们需要安装node-sass，在命令行下运行以下代码：

    npm install --save-dev node-sass

这个命令会在当前目录下安装node-sass，并添加到`package.json`的devDependencies对象中。当其他人使用你的项目时会非常方便，因为他们已经有了运行项目所需的所有内容。只要安装过一次，使用时在命令行运行以下代码即可：

    node-sass --output-style compressed -o dist/css src/scss

让我们看一下这个命令做了什么：从后向前看，查找`src/scss`目录的SCSS文件，输出（-o 标识）编译的CSS到`dist/css`目录，压缩输出文件（使用 --output-style 标识，设置选项值为"compressed"）。
现在我们知道了在命令行中如何工作，那么让我们把它放到npm scirpt中。在`package.json`的scripts对象中添加如下内容:

````
"scripts": {
  "scss": "node-sass --output-style compressed -o dist/css src/scss"
}
````

现在回到命令行并运行：

    npm run scss

可以看到这样运行的输出结果和直接在命令行使用node-sass命令得到的结果一致。

### 使用PostCSS自动给CSS加前缀

我们已经能够把SCSS编译成CSS，现在我们希望通过Autoprefixer和PostCSS自动给CSS代码添加厂商前缀，我们可以通过空格分隔的方式从而同时安装多个模块：

    npm install --save-dev postcss-cli autoprefixer

因为PostCSS默认不做任何事情，所以我们安装了两个模块。PostCSS依赖其他的插件来处理你提供的CSS，比如Autoprefixer。
安装并保存必要工具到devDependencies后，在你的scripts对象中添加一个新任务:

````
"scripts": {
  ...
  "autoprefixer": "postcss -u autoprefixer -r dist/css/*"
}
````

这个任务的意思是：嗨postcss，使用(-u标识符)Autoprefixer替换（-r标识符）`dist/css`目录下的所有`.css`文件，给他们添加厂商前缀代码。就是这样简单！想要修改默认浏览器前缀？只要给脚本添加如下代码即可:

````
"autoprefixer": "postcss -u autoprefixer --autoprefixer.browsers '&gt; 5%, ie 9' -r dist/css/*"
````

再次申明，配置你自己的构建代码有很多选项可以使用：postcss-cli 和 autoprefixer。

### JavaScript 代码检查

对于写代码来说，保持标准格式和样式是非常重要的，它能够确保错误最小化并提高开发效率。"代码检查"帮助我们自动化的完成了这个工作，所以我们通过使用 eslint 来进行代码检查。

再次如上文所述，安装eslint的包，这次让我们使用快捷方式:

    npm i -D eslint

这和如下代码是一样的效果：

    npm install --save-dev eslint

安装完成后，我们给eslint配置一些运行代码的基本规则。使用如下代码开始一个向导：

````
eslint --init // 译者注：这里直接使用会抛错eslint找不到，因为这种使用方法必须安装在全局，即通过 npm install i -g eslint方式安装
````

我建议选择"回答代码风格问题"并回答提问的相关问题。这个过程中eslint会在你的项目根目录下生成一个新文件，并检测你的相关代码。
现在，让我们把代码风格检测任务添加到`package.json`的scripts对象中：

````
"scripts": {
  ...
  "lint": "eslint src/js"
}
````

我们的任务仅有13字符！它会查找`src/js`目录下的所有JavaScript文件，并根据刚才生成的规则进行代码检测。当然，如果感兴趣的话你可以详细配置各种规则：get crazy with the options

### 混淆压缩JavaScript文件

让我们继续，下面我们需要使用uglify-js来混淆压缩JavaScript文件，首先需要安装uglify-js：

    npm i -D uglify-js

然后我们可以在`package.json`里创建压缩混合任务：

````
"scripts": {
  ...
  "uglify": "mkdir -p dist/js && uglifyjs src/js/*.js -m -o dist/js/app.js"
}
````

npm scripts的任务的本质是：可以重复执行的、命令行任务的快捷方式（别名），这也是npm scripts的优点之一。这就意味着你可以直接在脚本里使用标准命令行代码！这个任务使用了两个标准命令行特性：mkdir 和 &&。

这个任务的第一部分“ mkdir -p dist/js ”：如果不存在目录（-p标识）就创建一个目录结构（mkdir），创建成功后执行uglifyjs命令。&&帮助你连接多个命令，如果前一个命令成功执行的话，就分别顺序执行剩余的命令。

这个任务的第二部分告诉uglifyjs针对`src/js/`目录下的所有JS文件（`*.js`），使用"mangle"命令（-m 标识），输出结果到`dist/js/app.js`文件中。这里是uglifyjs工具的全部配置选项 list of options。

让我们来更新一下uglify任务，创建一个`dist/js/app.js`的压缩版本，链接另外一个uglifyjs的命令并传参给"compress"（-c标识）。

````
"scripts": {
  ...
  "uglify": "mkdir -p dist/js && uglifyjs src/js/*.js -m -o dist/js/app.js && uglifyjs src/js/*.js -m -c -o dist/js/app.min.js"
}
````


### 压缩图片

下面我们将进行图片压缩的工作。根据httparchive.org的数据统计，网络上前1000名的网站平均页面大小为1.9M,其中图片占了1.1M（with images accounting for 1.1mb of that total）。所以提高网页加载速度的其中一个好办法就是减小图片大小。

安装 imagemin-cli:

    npm i -D imagemin-cli

Imagemin非常棒，它可以压缩大多数图片类型，包括GIF、JPG、PNG和SVG。 使用如下代码可以将一整个文件夹的图片全部压缩：

````
"scripts": {
  ...
  "imagemin": "imagemin src/images dist/images -p",
}
````

这个任务告诉imagemin找到并压缩`src/images`中的所有图片并输出到`dist/images`中。-p标志在允许的情况下将图片处理成渐进图片。

### SVG精灵（Sprites）

关于SVG的讨论近年来逐渐火热，SVG有众多优点：在所有的设备上保持松散结构、可通过CSS编辑、对读屏软件友好。然而，SVG编辑软件经常会产生大量的冗余代码。幸运的是，svgo可以帮助你自动删除这些冗余信息（我们马上就会安装svgo）。

接下来我们来安装svg-sprite-generator，用于自动处理并整合多个SVG文件为一个SVG文件（更多处理方案：more on that technique here）。

    npm i -D svgo svg-sprite-generator

你现在应该已经熟悉了这个过程——添加一个任务在你的`package.json`scripts对象中：

````
"scripts": {
  ...
  "icons": "svgo -f src/images/icons && mkdir -p dist/images && svg-sprite-generate -d src/images/icons -o dist/images/icons.svg"
}
````

注意icons任务通过两个&&引导符做了三件事情： 1.使用svgo传参一个SVG目录（-f标识），这个操作压缩了目录内的全部SVG文件；2.如果不存在'dist/images'目录则创建该目录（使用mkdir -p命令）；3.使用svg-sprite-generator，传参一个SVG目录（-d标识）以及输出处理后的SVG文件的目录路径名（-o标识）。

### 通过BrowserSync提供服务、自动监测并注入变更

最后一个插件是BrowserSync，它可以做如下事情：启动一个本地服务，向连接的浏览器自动注入更新的文件，并同步浏览器的点击和滚动。安装并添加任务的代码如下：

````
npm i -D browser-sync
"scripts": {
  ...
  "serve": "browser-sync start --server --files 'dist/css/*.css, dist/js/*.js'"
}
````

BrowserSync任务默认使用当前根目录下的路径启动一个服务器（--server标识），--files标识告诉BrowserSync去监测`dist`目录的CSS或JS文件，一旦有任何变化，则自动向页面注入变化的文件。

你可以同时打开多个浏览器（甚至在不同的设备上），他们都会实时更新文件变化的！

### 分组任务

使用以上任务我们可以做到如下功能：

* 编译SCSS到CSS并自动添加厂商前缀
* 对Javascript进行代码检查及混淆压缩
* 压缩图片
* 整合整个文件夹内的SVG文件为一个SVG文件
* 启动一个本地服务并向连接至该服务的浏览器自动注入更新。
这还不是全部内容！

#### 合并CSS任务

我们会添加一个新的任务，用于合并两个CSS相关的任务（处理SASS和执行加前缀的Autoprefixer），有了这个任务我们就不用分别执行两个相关任务了：

````
"scripts": {
  ...
  "build:css": "npm run scss && npm run autoprefixer"
}
````

当你运行npm run build:css时，这个任务会告诉命令行去执行npm run scss，当这个任务成功完成后，会接着（&&）执行 npm run autoprefixer。

#### 合并JavaScript任务

就像css任务一样，我们可以把JavaScript任务也链接到一起以方便执行：

````
"scripts": {
  ...
  "build:js": "npm run lint && npm run uglify"
}
````

现在，我们可以通过npm run build:js一步调用，来进行代码检测、混淆压缩JavaScript代码了！

#### 合并剩余任务

对于图片任务、其他剩余构建任务，我们可以用相同的方法把他们变成一个任务：

````
"scripts": {
  ...
  "build:images": "npm run imagemin && npm run icons",
  "build:all": "npm run build:css && npm run build:js && npm run build:images",
}
````

### 变更监控

我们的任务不断的需要对文件做一些变更，我们不断的需要切回命令行去运行相应的任务。针对这个问题，我们可以添加一个任务来监听文件变更，让文件在变更的时候自动执行这些任务。这里我推荐使用onchange插件，安装方法如下：

    npm i -D onchange

让我们来给CSS和JavaScript设置监控任务：

````
"scripts": {
  ...
  "watch:css": "onchange 'src/scss/*.scss' -- npm run build:css",
  "watch:js": "onchange 'src/js/*.js' -- npm run build:js",
}
````

这些任务可以分解如下：onchange需要你传参想要监控的文件路径（字符串），这里我们传的是SCSS和JS源文件，我们想要运行的命令跟在--之后，这个命令当路径内的文件发生增删改的时候就会被立即执行。

让我们再添加一个监控命令来完成我们的npm scripts构建过程:

再添加一个包，parallelshell：

    npm i -D parallelshell

再次给scriopts对象添加一个新任务：

````
"scripts": {
  ...
  "watch:all": "parallelshell 'npm run serve' 'npm run watch:css' 'npm run watch:js'"
}
````

parallelshell支持多个参数字符串，它会给npm run传多个任务用于执行。

为什么时候parallelshell去合并多个任务，而不是像之前的任务一样使用&&呢？最开始我也尝试这么做了，但是问题是：&&链接多个命令到一块，需要等待每一个命令成功完成后才会执行下一个任务。然而当我们运行watch命令时，这些命令一直都不会结束！这样我们就会卡在一个无限循环里。

因此，使用parallelshell使得我们可以同时执行多个watch命令。（译者注：后来在评论里有人推荐使用npm-run-all插件来代替parallelshell，它支持这种用法可以一次性检测全部watch任务更加方便："watch": "npm-run-all --parallel serve watch:*"）

这个任务使用了BrowserSync的npm run serve任务启动了一个服务，然后对CSS和JavaScript文件执行了监控命令，一旦CSS或JavaScript文件有变更，监控任务就会执行相应的构建（build）任务。由于BrowserSync被设置成监控`dist`目录下的变更，所以它会自动的向相关联的URL内注入新的文件，真是太棒了！

### 其他实用任务

npm有大量可以实用的插件（ots of baked in tasks ），让我们再添加一个新的任务来看看这些插件对构建脚本的影响：

````
"scripts": {
  ...
  "postinstall": "npm run watch:all"
}
````

上文有提到，postinstall钩子会在install完成后运行，当你在命令行中执行npm install完成后会执行npm run watch:all。当团队合作时这个功能非常有用，当别人复制了你的项目并运行了npm install的时候，我们的watch:all任务就会马上执行，别人马上就会准备好一切开发环境：启动一个服务、打开一个浏览器窗口、监控文件变更。