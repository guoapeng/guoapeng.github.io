---
title: 如何创建nodejs模块进阶版
date: 2021/10/10
tags: 
- Nodejs
- Webpack
---

## 项目背景介绍

## 项目目标

### 发布后的包要支持以下特性

- 基于Webpack 5
- 使用TypeScript编写源码
- 模块支持:
  - 在浏览器环境下使用，通过\<script\>标签来引入这个类库
  - 通过NPM安装使用
  - 兼容 ES6(ES2015) 的模块系统、CommonJS 和 AMD 模块规范

## 首先创建一个nodejs工程

- 创建用来包含工程文件的文件夹例如
  
  ```bash
    # 创建用来存放工程的目录
    mkdir h5-audio-player
    # 进入目录
    cd h5-audio-player
  ```

- 初始化nodejs工程
  - 使用npm init 命令来初始化项目，此时它会要求输入一下项目信息，根据自己的实际情况输入即可，
  　后续我们会介绍工程文件的各参数的含义，后续还有机会手动调整参数．

    ```bash
    $npm init
    package name: (audioplayer)
    version: (1.0.0) 0.1.0
    description: a javascript library or utility for creating audio player on web  or server or mobile side
    entry point: (index.js)
    test command:
    git repository:
    keywords: audioplayer
    author: eagle.guo
    license: (MIT)
    About to write to your_proj_path/h5-audio-player/package.json:
    {
    "name": "audioplayer",
    "version": "0.1.0",
    "description": "a javascript library or utility for creating audio player on web  or server or mobile side",
    "main": "index.js",
    "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
    },
    "keywords": [
    "audioplayer"
    ],
    "author": "eagle",
    "license": "MIT"
    }
    Is this OK? (yes) yes

    ```

- 初始化完成后, 项目的目录结构如下:

    ```bash
        $tree -L 2
        audioplayer
        └── package.json
    ```

  - 可以看到初始化命令帮我们做的工作就是创建了一个package.json文件，里面包含了包名，版本等等信息，如果对package.json熟悉，手写或从其他项目拷贝一份来修改也是可以的，而且了解package.json相关的规范是我们开发nodejs项目必须要掌握的内容．可以使用npm help package.json查看详细信息．
  
## 编写源文件

## 用webpack编译源文件

- 在配置webpack之前，首先说明一下，我们为什么要使用webpack?
webpack听起来像一个打包器，实际上它更像是一个代码编译器，它有丰富的插件和loader，可以将各种类型代码（typescript, coffeescript, ES6 script, css, sass, less, 图片，字体等等）编译成兼容性较强的低端javascript,　而且低到什么程度可以通过配置文件来设定．这样编译出来的代码可以被大多数浏览器直接执行．这里说的是大多数浏览器，不是所有浏览器，因为我们的定位是制作一个npm模块，正如我们在文章开头就设定的目标兼容 ES6(ES2015) 的模块系统、CommonJS 和 AMD 模块规范．我们不可能知道使用者到底要兼容到什么程度，我们也不希望把代码编译成非常低端的js代码，然而这部分幸好可以被另一个比较牛X的模块Babel覆盖到, 用户可以引入我们的模块使用Babel根据他们自身的需求再次编译成更低端的javascript代码．至于一些遗留系统，没有使用到前端工程化方案的系统，也可以在项目外使用Babel Cli命令行工具，将我们的包编译成低端代码后导出使用．这里不得不感慨一下，互联网行业真是一个全球通力合作的行业，就我个人的眼界，全球还没有任何行业合作的规模有这么大，每个从业者自发地贡献一点力量一个模块，整个生态变得越来越庞大．这也是我为什么要坚持写博客，开源自己的模块的原因，分享自己的经验，踩过的坑，希望能帮助更多的人，也是互联网人应该拥有的精神，和引以为豪的事情，共勉! 如果你像以前的我一样，没时间没能力共享代码，当你见到用心写作的博文，代码库，在力所能及的情况下，记得回到原文打赏他们，当然这里不是指我，指那些像我一样996以后还有精力干到现在凌晨2点半的互联网创作者们，让他们有更充足的动力创作更多好的博客，或模块．

### 配置webpack

首先要安装webpack的一些工具包，然后要安装出来javascript, typescript的一些插件和loader, 如有必要还要安装一些处理css或less或者sass的．

#### 引入webpack工具包

第一步我们来按装webpack的工具包，并完成webpack的一些基本配置．
先安装webpack,　webpack-cli这两个nodejs模块. 如何你还没有按照nodejs模块，可以参考我的文章[安装并配置nodejs](https://pengtech.net/nodejs/install_and_config_nodejs.html).

```bash
   cnpm install --save-dev webpack webpack-cli
```

说明：

- cnpm： 是阿里为我们提供的一个方便下载npm包的工具
- install: 使用来安装npm包的命令，包将会被安装到当前目录的node_modules目录下
- save-dev: 使用该参数，被安装的包将会被作为项目依赖保存到package.json的dependencies-dev区域，
  作用是当他人获得项目后，只需要在项目根目录使用cnpm install即可安装所依赖的npm包，不再需要指定包名
- 执行该命令，cnpm会会首先去获取webpack, 和wepack-cli的package.json文件，分析依赖关系然后将webpack或webpack-cli的依赖包以及这两个包自身下载下来．
- 此处没有指定版本号，cnpm将会去npm registry根据semver规则找到最新的最新的包进行安装．semver规则可以参考[npm的官方文档](https://www.npmjs.cn/misc/semver/)
- 如果不想按照最新版本的webpack 和webpack-cli, 可以在命令行中指定版本，指定版本使用包名加@符合后面接具体的版本号, 例如：

  ```bash
  　　cnpm install --save-dev webpack@5.58.1 webpack-cli@4.9.0
  ```

  - 当使用指定的版本进行安装是需要自行确定包与包之间版本的兼容性
- 安装完成后还会生成一个package-lock.json的包名，这个包里存储的是安装包是从何处下载的，以及一些hash code之类的，这个文件最好也要checkin到代码库里面，以保证每个开发人员拿到代码后都能从相同的位置去下载依赖包，不受自身开发环境配置的影响而使用与其他开发人员不同的包，尤其是依赖包．

#### webpack基本配置

在项目的根目录下手动创建webpack.config.js

```json
const {resolve} = require('path');

module.exports= {
   //webpack的js入口, 里面定义打包资源
    entry: './src/index.js',
    output: {
        //audioplayer为输出文件的文件名前缀
        filename: 'audioplayer.bundle.js',
        //dest是destination的缩写，表示目标位置
        path: resolve(__dirname, 'dest'),
    },
    module: {
        rules: [
            //javascript处理规则配置
            {
                test: /\.js$/,
                //项目依赖不需要打包到最终的输出文件，所以需要排除
                exclude: /node_modules/
            }
        ]
    },
    mode: 'development'
}
```

说明:

- webpack.config.js是webpack的默认配置文件，意思是当在项目根目录下执行webpack命令时，如果不指定任何配置文件，webpack默认将使用此配置文件，如果不想使用默认配置可以使用 --config 选项指定配置文件例如:

  ```bash
  webpack --config webpack.config.dev.js
  ```

- 目前我们webpack项目的入口文件依然是'./src/index.js' 是一个javascript文件，而我们的终极目标是全部代码使用typescript编写，由于typescript编译需要更多的插件和更复杂的配置，而我们的重点是讲解wepback的基本配置，所以此处我们依然以javascript讲解webpack，我们将逐步进化到typescript.
- entry 用于指定webpack的入口文件，这里采用的single entry, 至于multiple entry本文不涉及，这块是属于webpack的编译打包比较高级的部分，可以参考webpack官网或者其他博文进行了解，本文的目标重点不在此，避免过于繁杂，然而理解了single　entry后，如果项目有需要，再配置multiple entry也不是难事．
- path 这个包是nodejs的核心包，不需要作为项目依赖引入．
- 目前我们项目的结构如下

  ```bash
  .
  ├── node_modules
         ├──一系列依赖包
  ├── package.json
  ├── README.md
  ├── src
  │   ├── audioplayer.js
  │   └── index.js
  └── webpack.config.js
  ```

- 这里重点看一下项目入口文件./src/index.js

  ```javascript
    import * as audioplayer from './audioplayer.js';
    module.exports = audioplayer;
  ```

  - 当webpack读取到index.js文件后，
  　如果把工程中各个模块的依赖关系当作一棵树，那么入口就是这棵依赖树的根，webpack会解析模块所依赖的文件或包，生成ATS树，然后根据webpack配置中定义的规则对模块进行处理，最终生成一个或多个js输出文件．
- 另外我们需要修改package.json文件，将main属性指向webpack的输出文件

  ```json
  "main": "dest/audioplayer.bundle.js",
  ```

### 运行webpack基本配置

至此我们的webpack基本配置已经完成，我们可以执行一下webpack命令来看它帮我们做了些什么．

```bash
$npx webpack
asset audioplayer.bundle.js 18.2 KiB [emitted] (name: main)
runtime modules 1020 bytes 4 modules
cacheable modules 12.9 KiB
./src/index.js 79 bytes [built] [code generated]
./src/audioplayer.js 12.8 KiB [built] [code generated]
webpack 5.58.1 compiled successfully in 249 ms
```

可以看出webpack帮我们编译了index.js和audioplayer.js两文件并生成了audioplayer.bundle.js文件
现在我们的目录结构变成了下面这个样子

```bash
$tree  -I "node_modules"
.
├── dest
│   └── audioplayer.bundle.js
├── package.json
├── README.md
├── src
│   ├── audioplayer.js
│   └── index.js
└── webpack.config.js
```

### webpack配置改造一：生成可发布的npm模块

- 在如何发布一个nodejs模块 - [如何发布一个nodejs模块](https://pengtech.net/nodejs/how_to_publish_node_modules.html)中我们讲过，
一个完整的可发布的nodejs模块应该包括：
  - package.json 这里面一个包含版本信息，依赖．
  - README.md一些关于模块的说明，让使用或打算使用包的人对模块能有一个清晰的认识．
  - license: 关于使用此模块的一些license信息
  - index.js: 这个为可选项，它的作用是如果package.json中没有指定main属性，nodejs默认将会从index.js开始加载模块
  - 主模块：模块主体，对于我们的模块即为dest/audioplayer.bundle.js
- 所以在这一节我们需要做的是添加license文件，另外将不需要发布出去的源文件以及webpack配置从可发布制品中忽略掉．
  - 添加license文件.
  - 将源文件和webpack配置排除在模块可发布制品之外，这里有多种方式，在基础篇
    - 第一种是排除法即使用黑名单的方式，创建.npmignore，将需要排除的文件夹和文件写入黑名单.npmignore,语法和.gitignore类似，另外如果项目下有.gitignore文件，.gitignore也是黑名单的一部分，即在.gitignore中包含的文件也会被排除在外．
    - 第二种是白名单的方式，即只包含需要发布的文件．白名单使用package.json 中的files属性进行定义，例如: 在devDependencies同级位置添加

      ```json
          "files": [
          "README.md",
          "LICENSE",
          "package.json",
          "dest/audioplayer.bundle.js"
        ]
      ```

    - 这里推荐使用白名单的方式，当项目添加新文件时不用考虑是否要发布出去，只有在需要发布时才去修改白名单．也可以两者相结合．
  - 这里npm似乎缺少了一个发布前查看哪些文件即将被发布的命令，很遗憾，我打算给他们提个issue改善这一点. 假如可发布制品中保护了，一些CI/CD中配置的一些用户名密码文件，这样发表出去还是挺危险的．

### webpack配置改造一：去掉polyfill填充

```bash
cnpm install --save-dev @babel/core @babel/cli @babel/preset-env babel-loader
```

### webpack配置改造二：编译typescript源代码

### 编译

## 测试与代码覆盖率

## 持续集成

## 发布模块到NPM社区

## 使用和升级

## 参考文档

[基于Webpack和ES6构建NPM包](https://blog.csdn.net/weixin_34327223/article/details/88012867)

[使用webpack打包组件和基础库并发布至npm](https://blog.csdn.net/xjl271314/article/details/106220492/)

[libraryTarget的几种选择我们来好好分析](https://zhuanlan.zhihu.com/p/108216236)
