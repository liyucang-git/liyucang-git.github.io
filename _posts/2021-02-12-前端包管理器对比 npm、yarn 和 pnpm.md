---
layout: post
title: 前端包管理器对比 npm、yarn 和 pnpm
subtitle: 为什么现在我更推荐 pnpm 而不是 npm/yarn
date: 2021-02-12
author: Li Yucang
catalog: true
tags:
  - 包管理
  - 前端工程化
---

# 前端包管理器对比 npm、yarn 和 pnpm

## 前言

本文先从前端包管理器的发展开始说起，对比 npm、yarn 和 pnpm 的差异，最后再通过详细介绍 pnpm 的特性来说明为什么现在前端包管理更推荐使用 pnpm。

## 前端包管理器的发展

### 没有包管理器

依赖(dependency)是别人为了解决一些问题而写好的代码，即我们常说的第三方包或三方库。

一个项目或多或少的会有一些依赖，而你安装的依赖又可能有它自己的依赖。

比如，你需要写一个base64编解码的功能，你可以自己写，但为什么要自己造轮子呢?大多数情况下，一个可靠的第三方依赖经过多方测试，兼容性和健壮性会比你自己写的更好。

项目中的依赖，可以是一个完整的库或者框架，比如 react 或 vue;可以是一个很小的功能，比如日期格式化;也可以是一个命令行工具，比如 eslint。

如果没有现代化的构建工具，即包管理器，你需要用`<script>`标签来引入依赖。

此外，如果你发现了一个比当前使用的依赖更好的库，或者你使用的依赖发布了更新，而你想用最新版本，在一个大的项目中，这些版本管理、依赖升级将是让人头疼的问题。

于是包管理器诞生了，用来管理项目的依赖。

它提供方法给你安装依赖(即安装一个包)，管理包的存储位置，而且你可以发布自己写的包。

### npm1/2

初代npm(Node.js Package Manager)随着Node.js的发布出现了。

**npm install 原理**

主要分为两个部分, 首先，执行 npm install 之后，包如何到达项目 node_modules 当中。其次，node_modules 内部如何管理依赖。

执行命令后，首先会构建依赖树，然后针对每个节点下的包，会经历下面四个步骤:

1、将依赖包的版本区间解析为某个具体的版本号；

2、下载对应版本依赖的 tar 包到本地离线镜像；

3、将依赖从离线镜像解压到本地缓存；

4、将依赖从缓存拷贝到当前目录的 node_modules 目录；

然后，对应的包就会到达项目的node_modules当中。

在 npm1、npm2 中呈现出的是嵌套结构，比如下面这样:

![](/img/localBlog/4386dbd291331a140b8353dcec8b961fe99300.jpeg)


这会导致3个问题：

1、依赖层级太深，会导致文件路径过长的问题，尤其在 window 系统下；

2、大量重复的包被安装，文件体积超级大。比如跟 foo 同级目录下有一个baz，两者都依赖于同一个版本的lodash，那么 lodash 会分别在两者的 node_modules 中被安装，也就是重复安装；

3、模块实例不能共享。比如 React 有一些内部变量，在两个不同包引入的 React 不是同一个模块实例，因此无法共享内部变量，导致一些不可预知的 bug；


### npm3/yarn

从 npm3 开始，包括 yarn，都着手来通过扁平化依赖的方式来解决上面的这个问题：

![](/img/localBlog/213ed4b459fed61d4d2236af5c9e10423628da.png)

所有的依赖都被拍平到node_modules目录下，不再有很深层次的嵌套关系。这样在安装新的包时，根据 node require 机制，会不停往上级的node_modules当中去找，如果找到相同版本的包就不会重新安装，解决了大量包重复安装的问题，而且依赖层级也不会太深。

但是扁平化带来了新的问题：

1、package.json里并没有写入的包竟然也可以在项目中使用了(Phantom - 幻影依赖)。

2、node_modules安装的不稳定性（Doppelgangers - 分身依赖）。

3、平铺式的node_modules算法复杂，耗费时间。

**Phantom**

package.json 中我们只声明了A，B ~ F都是因为扁平化处理才放到和A同级的 node_modules 下，理论上在项目中写代码时只可以使用A，但实际上B ~ F也可以使用，由于扁平化将没有直接依赖的包提升到 node_modules 一级目录，Node.js没有校验是否有直接依赖，所以项目中可以非法访问没有声明过依赖的包。

**Doppelgangers**

比如B和C都依赖了F，但是依赖的F版本不一样：

![](/img/localBlog/61c9a51874710cbdd5655819b0d50164711307.png)


依赖结构的不确定性表现是扁平化的结果不确定，以下2种情况都有可能，取决于package.json中B和C的位置。

![](/img/localBlog/55c9f777076ab7b0f3a4370f8ecfced187089e.png)

### npm5.x/yarn - 带有lock文件的平铺式的node_modules

该版本引入了一个lock文件，以解决node_modules安装中的不确定因素。 这使得无论你安装多少次，都能有一个一样结构的node_modules。 这也是为什么lock文件应该始终包含在版本控制中并且不应该手动编辑的原因。

然而，平铺式的算法的复杂性，以及Phantom、性能和安全问题仍未得到解决。

### pnpm - 基于符号链接的node_modules结构

pnpm(Performance npm)的作者Zoltan Kochan发现 yarn 并没有打算去解决上述的这些问题，于是另起炉灶，写了全新的包管理器。

pnpm复刻了npm所有的命令，所以使用方法和npm一样，并且在安装目录结构上做了优化，特点是善用链接，且由于链接的优势，大多数情况下pnpm的安装速度比yarn和npm更快。

pnpm生成node_modules主要分为两个步骤：

**1、基于硬连接的node_modules**

```
.
└── node_modules
    └── .pnpm
        ├── foo@1.0.0
        │   └── node_modules
        │       └── foo -> <store>/foo
        └── bar@1.0.0
            └── node_modules
                └── bar -> <store>/bar
```

乍一看，结构与npm/yarn的结构完全不同，第一手node_modules下面的唯一文件夹叫做.pnpm。在.pnpm下面是一个`<PACKAGE_NAME＠VERSION>`文件夹，而在其下面`<PACKAGE_NAME>`的文件夹是一个content-addressable store的硬链接。 当然仅仅是这样还无法使用，所以下一步软链接也很关键。

**2、用于依赖解析的软链接**

* 用于在foo内引用bar的软链接

* 在项目里引用foo的软链接

```
.
└── node_modules
    ├── foo -> ./.pnpm/foo@1.0.0/node_modules/foo
    └── .pnpm
        ├── foo@1.0.0
        │   └── node_modules
        │       ├── foo -> <store>/foo
        │       └── bar -> ../../bar@1.0.0/node_modules/bar
        └── bar@1.0.0
            └── node_modules
                └── bar -> <store>/bar
```

当然这只是使用pnpm的node_modules结构最简单的例子！但可以发现项目中能使用的代码只能是package.json中定义过的，并且完全可以做到没用无用的安装。peers dependencies的话会比这个稍微复杂一些，但一旦不考虑peer的话任何复杂的依赖都可以完全符合这种结构。

例如，当foo和bar同时依赖于lodash的时候，就会像下图这样的结构。

```
.
└── node_modules
    ├── foo -> ./.pnpm/foo@1.0.0/node_modules/foo
    └── .pnpm
        ├── foo@1.0.0
        │   └── node_modules
        │       ├── foo -> <store>/foo
        │       ├── bar -> ../../bar@1.0.0/node_modules/bar
        │       └── lodash -> ../../lodash@1.0.0/node_modules/lodash
        ├── bar@1.0.0
        │   └── node_modules
        │       ├── bar -> <store>/bar
        │       └── lodash -> ../../lodash@1.0.0/node_modules/lodash
        └── lodash@1.0.0
            └── node_modules
                └── lodash -> <store>/lodash
```

node_modules 中的 bar 和 foo 两个目录会软连接到 .pnpm 这个目录下的真实依赖中，而这些真实依赖则是通过 hard link 存储到全局的 store 目录中。

综合而言，本质上 pnpm 的 node_modules 结构是个网状 + 平铺的目录结构。这种依赖结构主要基于软连接(即 symlink)的方式来完成。

pnpm 是通过 hardlink 在全局里面搞个 store 目录来存储 node_modules 依赖里面的 hard link 地址，然后在引用依赖的时候则是通过 symlink 去找到对应虚拟磁盘目录下(.pnpm 目录)的依赖地址。

比如安装A，A依赖了B：

![](/img/localBlog/f1a984784cc4f415464644f0e48003526b662f.png)


1、安装依赖

A和B一起放到.pnpm中(和上面相比，这里没有耗时的扁平化算法)。

另外A@1.0.0下面是node_modules，然后才是A，这样做有两点好处：

* 允许包引用自身
* 把包和它的依赖摊平，避免循环结构

2、处理间接依赖

A平级目录创建B，B指向B@1.0.0下面的B。

3、处理直接依赖

顶层node_modules目录下创建A，指向A@1.0.0下的A。

对于更深的依赖，比如A和B都依赖了C：

![](/img/localBlog/a7072be886859a6f7a4647c455a91e01c2deaa.png)

## pnpm 详细介绍

### 什么是 pnpm 

pnpm 本质上就是一个包管理器，这一点跟 npm/yarn 没有区别，但它作为杀手锏的优势在于:

* 包安装速度极快；
* 磁盘空间利用非常高效。
* 支持 monorepo
* 安全性高

它的安装也非常简单。可以有多简单?

```
npm i -g pnpm
```

### 特性概览

#### 速度快
pnpm 安装包的速度究竟有多快？先以 React 包为例来对比一下:

![](/img/localBlog/WechatIMG26.png)

可以看到，作为黄色部分的 pnpm，在绝多大数场景下，包安装的速度都是明显优于 npm/yarn，速度会比 npm/yarn 快 2-3 倍。

#### 高效利用磁盘空间

**npm/yarn - 消耗磁盘空间的node_modules**

npm/yarn有一个缺点，就是使用了太多的磁盘空间, 如果你安装同一个包100次，100分的就会被储存在不同的node_modules文件夹下。 举一个常有的的例子，如果完成了一个项目，而node_modules没有删掉保留了下来，往往会占用大量的磁盘空间。 为了解决这个问题，我经常使用npkill。

```
$ npx npkill
```

可以扫描当前文件夹下的所有node_modules，并动态地删除它们。

**pnpm - 高效的使用磁盘空间**

pnpm将包存储在同一文件夹中（content-addressable store），只要当你在同一OS的同一个用户在下再次安装时就只需要创建一个硬链接。 MacOs的默认位置是~/.pnpm-store，甚至当安装同一package的不同版本时，只有不同的部分会被重新保存。 也就是说然后当你安装一个package时，如果它在store里，建立硬连接新使用，如果没有，就下载保存在store再创建硬连接。

在使用 pnpm 对项目安装依赖的时候，如果某个依赖在 sotre 目录中存在了话，那么就会直接从 store 目录里面去 hard-link，避免了二次安装带来的时间消耗，如果依赖在 store 目录里面不存在的话，就会去下载一次。

当然这里你可能也会有问题：如果安装了很多很多不同的依赖，那么 store 目录会不会越来越大？

答案是当然会存在，针对这个问题，pnpm 提供了一个命令来解决这个问题: `pnpm store | pnpm`。

同时该命令提供了一个选项，使用方法为 pnpm store prune ，它提供了一种用于删除一些不被全局项目所引用到的 packages 的功能，例如有个包 axios@1.0.0 被一个项目所引用了，但是某次修改使得项目里这个包被更新到了 1.0.1 ，那么 store 里面的 1.0.0 的 axios 就就成了个不被引用的包，执行 pnpm store prune 就可以在 store 里面删掉它了。

该命令推荐偶尔进行使用，但不要频繁使用，因为可能某天这个不被引用的包又突然被哪个项目引用了，这样就可以不用再去重新下载这个包了。

#### 支持 monorepo

随着前端工程的日益复杂，越来越多的项目开始使用 monorepo。之前对于多个项目的管理，我们一般都是使用多个 git 仓库，但 monorepo 的宗旨就是用一个 git 仓库来管理多个子项目，所有的子项目都存放在根目录的packages目录下，那么一个子项目就代表一个package。

pnpm 与 npm/yarn 另外一个很大的不同就是支持了 monorepo，体现在各个子命令的功能上，比如在根目录下 pnpm add A -r, 那么所有的 package 中都会被添加 A 这个依赖，当然也支持 --filter字段来对 package 进行过滤。

#### 安全性高

不知道你发现没有，pnpm 这种依赖管理的方式也很巧妙地规避了非法访问依赖的问题，也就是只要一个包未在 package.json 中声明依赖，那么在项目中是无法访问的。

但在 npm/yarn 当中是做不到的，那你可能会问了，如果 A 依赖 B， B 依赖 C，那么 A 就算没有声明 C 的依赖，由于有依赖提升的存在，C 被装到了 A 的node_modules里面，那我在 A 里面用 C，跑起来没有问题呀，我上线了之后，也能正常运行啊。不是挺安全的吗？

还真不是。

1、你要知道 B 的版本是可能随时变化的，假如之前依赖的是C@1.0.1，现在发了新版，新版本的 B 依赖 C@2.0.1，那么在项目 A 当中 npm/yarn install 之后，装上的是 2.0.1 版本的 C，而 A 当中用的还是 C 当中旧版的 API，可能就直接报错了。

2、如果 B 更新之后，可能不需要 C 了，那么安装依赖的时候，C 都不会装到node_modules里面，A 当中引用 C 的代码直接报错。

3、在 monorepo 项目中，如果 A 依赖 X，B 依赖 X，还有一个 C，它不依赖 X，但它代码里面用到了 X。由于依赖提升的存在，npm/yarn 会把 X 放到根目录的 node_modules 中，这样 C 在本地是能够跑起来的，因为根据 node 的包加载机制，它能够加载到 monorepo 项目根目录下的 node_modules 中的 X。但试想一下，一旦 C 单独发包出去，用户单独安装 C，那么就找不到 X 了，执行到引用 X 的代码时就直接报错了。

这些，都是依赖提升潜在的 bug。如果是自己的业务代码还好，试想一下如果是给很多开发者用的工具包，那危害就非常严重了。

### pnpm以外的解決法

**Yarn 要不要支持 symlinks（符号链接） 的讨论**

2016 年 11 月 10 日，一个大佬开了一个 issue：Looking for brilliant yarn member who has first-hand knowledge of prior issues with symlinking modules。

问：「听@thejameskyle 说，在开源之前，Yarn 曾经做过 symlinks（符号链接）。然而，它破坏了太多的生态系统而不能被认为是一个有效的选择。」如果您是 yarn 成员，对实际损坏的内容有第一手消息，并且可以从技术上解释原因，我恳请您回答这个问题。

然后@sebmck 回答了他的问题：说 symlinks（符号链接），对现有的生态系统不能很好地兼容。

总结了几点：

符号链接的优点：

1、稍微快一点的包安装；

2、更少的磁盘使用；

存在的缺点：

1、操作系统差异；

2、通过添加多种安装模式来降低 Yarn 的确定性；

3、文件观察不兼容；

4、现有工具兼容性差；

5、由循环引起的递归错误，等等...

这个 issue，讨论了很长的篇幅，这里就不继续了，感兴趣的可以去观摩一下。最后显然 yarn 的团队暂时不打算支持使用符号链接。

**npm global-style**

npm也曾经为了解决扁平式node_modules的问题提供过，通过指定global-style来禁止平铺node_modules，但这无疑又退回了嵌套式的node_modules时代的问题，所以并没有推广开来。

**dependency-check**

光靠npm/yarn的话看似无法解决，所以基础社区的解决方案dependency-check也经常被用到。

```
$ dependency-check ./package.json --verbose
Success! All dependencies used in the code are listed in package.json
Success! All dependencies in package.json are used in the code
```

有了本文的基础，光是看到README内一段命令行的输出应该也能想象到dependency-check是如何工作的了吧！

果然和其他的解决方案比，pnpm显得最为优雅吧。

**hard link 和 symlink 兼容问题**

读到这里，可能有用户会好奇: 像 hard link 和 symlink 这种方式在所有的系统上都是兼容的吗？

实际上 hard link 在主流系统上(Unix/Win)使用都是没有问题的，但是 symlink 即软连接的方式可能会在 windows 存在一些兼容的问题，但是针对这个问题，pnpm 也提供了对应的解决方案：

在 win 系统上使用一个叫做 junctions 的特性来替代软连接，这个方案在 win 上的兼容性要好于 symlink。

或许你也会好奇为啥 pnpm 要使用 hard links 而不是全都用 symlink 来去实现。

实际上存在 store 目录里面的依赖也是可以通过软连接去找到的，nodejs 本身有提供一个叫做 --preserve-symlinks 的参数来支持 symlink，但实际上这个参数实际上对于 symlink 的支持并不好导致作者放弃了该方案从而采用 hard links 的方式。

### npm/yarn 与 pnpm 对比小结

npm/yarn - 缺点

* 扁平的node_modules结构允许访问没有引用的package。

* 来自不同项目的package不能共享，这是对磁盘空间的消耗。

* 安装缓慢，大量重复安装node_modules。

pnpm - 解决方案

* pnpm使用独创的基于symlink的node_modules结构，只允许访问package.json中的引入packages（严格）。

* 安装的package存储在一个任何文件夹都可以访问的目录里并用硬连接到各个node_modules，以节省磁盘空间（高效）。

* 有了上述改变，安装也会更快（快速）。

从官方网站上看，严格、高效、快速和对于monorepo的支持是pnpm的四大特点。

## pnpm 日常使用

说了这么多，估计你会觉得 pnpm 挺复杂的，是不是用起来成本很高呢？

恰好相反，pnpm 使用起来十分简单，如果你之前有 npm/yarn 的使用经验，甚至可以无缝迁移到 pnpm 上来。不信我们来举几个日常使用的例子。

### pnpm install

跟 npm install 类似，安装项目下所有的依赖。但对于 monorepo 项目，会安装 workspace 下面所有 packages 的所有依赖。不过可以通过 --filter 参数来指定 package，只对满足条件的 package 进行依赖安装。

当然，也可以这样使用，来进行单个包的安装:

```
// 安装 axios
pnpm install axios
// 安装 axios 并将 axios 添加至 devDependencies
pnpm install axios -D
// 安装 axios 并将 axios 添加至 dependencies
pnpm install axios -S
```

当然，也可以通过 --filter 来指定 package。

### pnpm update

根据指定的范围将包更新到最新版本，monorepo 项目中可以通过 --filter 来指定 package。

### pnpm uninstall

在 node_modules 和 package.json 中移除指定的依赖。monorepo 项目同上。举例如下:

```
// 移除 axios
pnpm uninstall axios --filter package-a
```

### pnpm link

将本地项目连接到另一个项目。注意，使用的是硬链接，而不是软链接。如:

```
pnpm link ../../axios
```

另外，对于我们经常用到npm run/start/test/publish，这些直接换成 pnpm 也是一样的，不再赘述。更多的使用姿势可参考官方文档。

### 总结

可以看到，虽然 pnpm 内部做了非常多复杂的设计，但实际上对于用户来说是无感知的，使用起来非常友好。并且，现在作者现在还一直在维护，目前 npm 上周下载量已经有 10w +，经历了大规模用户的考验，稳定性也能有所保障。

因此，我觉得无论是从背后的安全和性能角度，还是从使用上的心智成本来考虑，pnpm 都是一个相比 npm/yarn 更优的方案。
