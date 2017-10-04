---
title: electron项目的打包
date: 2017-09-30 11:00:00
categories: node.js
tags:
  - electron
  - node.js
---
> [electron](https://electron.atom.io/)是一个可以让你使用web技术（HTML,CSS,JavaScript）来编写pc端程序的框架，如果你对这个框架还不了解的话，可以[点击链接](https://electron.atom.io/)去了解更多，本文只记录一些我使用electron编写程序后期，将程序进行打包的一些关注点。

`electron`的打包在我的开发工作中一直都是比较容易忽略但又非常重要的。前期沉浸在业务层的代码里边，终于熬到头了，准备打包了，结果却问题百出...

本人使用过的electron的打包工具不多，就只有两个而已：electron-packager跟electron-builder。

## [electron-packager](https://github.com/electron-userland/electron-packager)

electron-packager最大的缺陷是它并没有提供一个类似NSIS这样子的安装包工具的集合。如果你的项目本身就不依赖安装包，或者有能力自己来编写NSIS脚本的话，这个打包工具无疑是便利的。

> 下边的讲解有部分内容是复制于electron-packager的[Readme.md](https://github.com/electron-userland/electron-packager/blob/master/readme.md)，需要了解更多，可以移步于此。

### 安装

```sh
# npm 脚本使用
npm install electron-packager --save-dev

# 命令行全局安装
npm install electron-packager -g
```

### 命令行使用

一个比较典型的electron-packager使用案例如下：

```sh
electron-packager <sourcedir> <appname> --platform=<platform> --arch=<arch> [optional flags...]
```

electron-packager会根据你给的参数，实现以下步骤：

- 找到/下载当前使用的electron版本

- 使用这个版本的electron在目录为`<out>/<appname>-<platform>-<arch>`下创建一个electron应用（这个目录也是可以通过`optional flags`设置的）

- sourcedir为打包文件目录

- 在下边这两种情况下，`--platform`跟`--arch`参数是可以忽略掉的：
  - 如果你使用了`--all`替代，electron-packager会创建符合所有平台（electron支持的windows7+, macOs以及linux系统）的桌面应用
  - 如果你什么都没有指定的话，electron-packager则只会打包当前系统当前CPU位数的桌面应用

因为这个工具缺乏集成安装包的工具，所以，在后边发现了`electron-builder`之后，就没怎么使用了。如果你想了解它更多的帮助信息，可以查看官方文档，或者使用`electron-packager --help`查看。

## [electron-builder](https://www.electron.build/)

这个是我当前在使用的工具，它解决了一些我在打包的时候遇到的痛点：

- 压缩成安装包的集成，因为`electron-builder`是集成了NSIS的，这一点极为方便

- 重新编译native modules，这个痛点不知道有没有人遇到过，如果使用electron来编写pc端的桌面应用的时候，需要使用到一些原生（需要经过编译）的node模块的时候，复制粘贴什么的是不可行的，就算是重新使用`electron-rebuild`或者`electron-gyp`重新安装编译。

或许是因为`npm`的模块依赖目录结构的诟病，所以`electron-builder`强力推荐使用`yarn`来安装node modules。

### 安装

```sh
yarn add electron-builder --dev
```

### 使用

使用`electron-builder`来打包最简便的方式就是通过`package.json`来设置相关参数，实现我们需要的打包过程。

`electron-builder`是会检测`package.json`这个文件的，如果`name`，`version`，`description`，`author`这些参数缺失，会影响到打包的顺利进行，所以，我们需要把这几个参数给补全了。

接下来，本文就主要讲一些本人关心/需要设置的参数，如果并没有解决您遇到的问题，不妨去细看一下官方文档，或者上google，stack overflow去搜索您遇到的问题，说不定能找到相应的答案。

##### 接下来，需要在`package.json`里添加名为`build`的顶层的对象属性：

```json
"build": {
  "appId": "your.id",
  "productName": "your_app_name",
  "asar": true,
  "asarUnpack": [
    "something_you_do_want_to_unpack"
  ],
  "compression": "normal",
  "icon": "path/to/icon",
  "electronVersion": "1.6.11",
  "directories": {
    "buildResources": "build"
  },
  "nsis": {
    "oneClick": false,
    "allowToChangeInstallationDirectory": true,
    "language": "zh-Hans"
  }
}
```

- appId: 应用id

- productName: 产品名称，这个字段是控制你的应用打开程序的名称

- asar: 是否需要将项目编写的内容压缩成一个`asar`格式的文件。理论上这个是很有必要的，如果条件允许我们可以将所有的内容压缩成一个`asar`文件的话，我们的应用就大大的减少了文件数量，这在复制，或者安装的时候，速度会快上很多。但并不是所有的内容都是可以这么做的，下边会提到。

- compression: 压缩的形式，`electron-builder`提供了`store`，`normal`以及`maximun`三种形式给我们选择：其中，`store`打包时间最快，但仅适用于测试；而`maximum`虽然打包速度慢，但是打包出来的体积明显要的要小。所以，我们可以根据需求去设置不同的打包形式。

- icon: 这个字段用来设置应用的桌面icon以及browerWindow实例的icon（如果你没有将你的browerWindow设置为`frame: false`的话）

- electronVersion: 用来打包的electron版本号，一般为了安全起见，减少因为兼容性引发的bug，我觉得最好还是设定一个项目内统一的electron版本比较好。

- directoried: 打包的源目录

- nsis: NSIS的设置

  - oneClick: 是否为一键安装

  - allowToChangeInstallationDirectory: 是否允许用户修改安装目录，该值只有在`oneClick`为`false`的时候生效

  - language: 安装语言

很明显，`electron-builder`的设置选项远不止这些，以上说到的这些也只是我使用的时候比较关注到的几点。下边，我简单分享一下在编写`electron`应用的时候关于最后打包，需要注意的一些地方：

- 如果能够将应用里边自定义的内容以及一些依赖包（html，css，js以及一些node_modules，bower_components等）压缩到`asar`是最好的选择，这样子一来，整个应用的文件数量会因此而大大减少。这么做的好处是，无论是应用在安装，还是复制的速度上会快上很多。

- 但也不是所有的文件都适合打包到`asar`里边去的，举个简单的例子就是应用的`icon`，`electron`应用在启动的时候会根据入口文件给的`icon`路径去寻找这个icon文件。但如若这个文件被压缩到`asar`里边去的话，因为找不到对应路径下的文件，就会出现报错，从而启动应用失败。

- 甚至有些时候，我们需要去通过`node.js`或者其他子程序去读写文件，用于存储信息（比如说日志），这个时候这些读写文件的路径就不能随意的指定了，更不能指定为相对路径。本人现在觉得比较好的做法是使用`node.js`的api获取到当前系统账户的对应文件夹目录，然后创建一个该`electron`应用对应的文件夹（一般都习惯性的命名为`.<app_name>`），因为是对应的系统账户的文件夹，当前操作用户对其有读写权限，就不用担心读写权限的问题了。

- 当然，上边的这些操作在开发环境下调试是有些不方便的，如果是windows用户，我们需要经常去翻C盘找文件...（虽然我的win10系统只有一个盘）我目前的做法是写一个配置文件，里边配置好一些开发/生产环境的参数，然后在编写程序的时候去区分，然后获取不同的文件路径。