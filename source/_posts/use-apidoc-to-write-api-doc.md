---
title: 利用apidoc在注释里写API文档
date: 2017-01-15 14:59:00
categories: node.js
tags:
  - node.js
  - apidoc
---
> 本文基于`node.js`的代码注释编写。

程序员都不喜欢写文档，但是程序员习惯写注释。所以今天给大家安利一个注释文档的工具——[apidoc](http://apidocjs.com)，一个在注释里边编写API文档的小工具！

**由于经验以及英文水平问题，文中很有可能会有错漏，各位大神如果看出来了，麻烦指出一下。谢谢～**

# 安装apidoc

```sh
npm install -g apidoc
```

# 运行
```sh
apidoc -i myapp/ -o apidoc/ -t mytemplate/
```

- -i 需要编译的包含注释的文件
- -o 输入文件夹名称
- t 模板

具体的详细帮助可以通过`apidoc -h`查看。

# 设置（apidoc.json）

我们可以通过`apidoc.json`文件来设置项目API文档的一些内容。例如：
```json
{ 
  "name": "example", 
  "version": "0.1.0", 
  "description": "apiDoc basic example",
  "title": "Custom apiDoc browser title", 
  "url" : "https://api.github.com/v1"
}
```
又或者通过`package.json`来设置apidoc的文档设置也是支持的。
```json
{ 
  "name": "example", 
  "version": "0.1.0", 
  "description": "apiDoc basic example", 
  "apidoc": { 
      "title": "Custom apiDoc browser title", 
      "url" : "https://api.github.com/v1" 
  }
}
```

# 简单介绍一下注释的编写

因为[apidoc官网](http://apidocjs.com)已经有详细的介绍了，所以这里就只是简单的介绍写apidoc常用的语法。

## @api

`@api`一般是必须编写的（除非你是用了[`@apiDefine`](http://apidocjs.com/#param-api-define)），不然apidoc编译器会忽略这段注释。

```js
/** 
 * @api {method} path [title]
 */
```
| Name     | Description     |
| :------------- | :------------- |
| method       | 请求的方法名称：如`GET`、`POST`等等       |
| path        | 请求路径  |
| title(可选)        | 一个简短的标题（用于导航跟文档标题）|

## @apiGroup

定于api归属的组名，生成的文档会把该api注释归类到该值对应的api组上。

```js
/**
 * @apiGroup name
 */
```
| Name     | Description     |
| :------------- | :------------- |
| name       | 组名称      |


## @apiName 

`@apiName`用于定义API文档的一个实例，并用作实例名称 。

```js
/**
 * @apiName name
 */
```
| Name     | Description     |
| :------------- | :------------- |
| name       | 实例名称      |

## @apiParam

`@apiParam`用于编写API的参数以及参数的解释。

```js
/**
 * @apiParam [(group)] [{type}] [field=defaultValue] [description]
 */
```
| Name     | Description     |
| :------------- | :------------- |
| (group) `可选`       | 参数归属组名，不填写组名，则默认设为`Paramter`      |
| {type} `可选`       | 参数数据类型，如`{String}`、`{Number}`、`{Array}`等等      |
| {type{size}} `可选` | 变量的大小信息 `{String{..5}}`参数类型为一个字符不超过5的字符串；`{String{2..5}}`参数为一个字符在2到5之间的字符串； |
| {type=allowedValues} `可选` | 参数允许值 `{string="small","huge"}`参数只能接受`small`或者`huge`的字符串 |
| field `可选`       | 参数名称      |
|  =defaultValue | 参数默认值 |
| description(可选)       | 描述      |

## @apiParamExample

`@apiParamExample`参数例子

```js
/**
 * @apiParamExample [{type}] [title]
 *    example
 */
```
| Name     | Description     |
| :------------- | :------------- |
| {type} `可选`       | 请求数据结构      |
| title `可选` | 例子的一个简短的标题 |
| example  | 例子的详细信息，可多个例子并存 |

## @apiSuccess

请求成功后的返回字段参数

```js
/**
 * @apiSuccess [(group)] [{type}] field [description]
 */
```
| Name     | Description     |
| :------------- | :------------- |
| (group) `可选`       | 参数归属组名，不填写组名，则默认设为`Success 200`      |
| {type} `可选` | 返回的数据类型，如`{String}`、`{Number}`等等 |
| field  | 返回的标示符（返回成功的状态码） |
| description `可选` | 描述 |

## @apiSuccessExample

请求成功后返回的字段参数例子

```js
/**
 * @apiSuccessExample [{type}] [title] example
 */
```
| Name     | Description     |
| :------------- | :------------- |
| {type} `可选`       | 请求数据结构      |
| title `可选` | 例子的一个简短的标题 |
| example  | 例子的详细信息，可多个例子并存 |

## @apiError & @apiErrorExample

这个的用法跟`@apiSuccess`、`@apiSuccessExample`的用法相类似。

# 举个栗子

下边举一个根据id获取文章资源API的例子

```js
/**
 * @api {get} /articles/:id 根据单个id获取文章信息
 * @apiName 根据id获取文章信息
 * @apiGroup Articles
 *
 * @apiParam (params) {String} id       文章id
 *
 * @apiSuccess {Array} article 返回相应id的文章信息
 *
 * @apiSuccessExample Success-Response:
 *    HTTP/1.1 200 OK
 *      {
 *        "tile": "文章标题2",
 *        "date": 1483941498230,
 *        "author": "classlfz",
 *        "content": "文章的详细内容"
 *       }
 *
 * @apiError (Error 4xx) 404 对应id的文章信息不存在
 *
 * @apiErrorExample Error-Response:
 *     HTTP/1.1 404 对应id的文章信息不存在
 *     {
 *       "error": err
 *     }
 */
```

接着我们运行`apidoc -i src/ -o docs`，就会看到项目里边多了一个docs的文件夹，打开里边的`index.html`，就可以看到生成的API文档了。如下图：

![FireShot Capture 1 - Custom apiDoc browser title_ - file____home_classlfz_github_expres.png](http://upload-images.jianshu.io/upload_images/1626912-5555c6d326baadf2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

这样子，我们就可以轻松的通过注释得到一个API文档了，而不需要在写完代码之后还要去打开word或者md文件去另外编写API文档......对于我本人而言，我是比较喜欢这样子的工作方式的。

如果项目是放在[github](https://github.com/)上边的话，我们可以使用[github pages](https://pages.github.com/)来[公布API文档](http://www.jianshu.com/p/f8cec06f2d8b)，方便协同开发。

# 写成gulp任务

因为平时用gulp比较多，这里再补上通过gulp任务来生成apidoc文档。

## 安装依赖

```sh
npm install gulp gulp-apidoc --save-dev
```

## 编写gulp代码

```js
// 构建apidoc
gulp.task('apidoc', (done) => {
  apidoc({
    src: 'src/',
    dest: 'docs',
    debug: true,
    includeFilters: ['.*\\.js$']
  }, done);
});
```