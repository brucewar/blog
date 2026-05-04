---
title: RESTful web API文档生成器
date: 2017-05-10 13:45:49
categories: Node.js
tags: [apidoc,RESTful API,文档]
---

>问：开发业务模块代码最重要的是什么？<br>答：API接口文档

如果你是后台开发，是否会有以下困扰：

* 开发API接口，还要通过wiki写接口文档，严重影响效率
* 接口不是一次就能定下来的，后续可能还要维护，所以还需要去修改文档

如果你是前端工程师，是否有过下面的困扰：

* 返回的数据怎么少字段，后台什么时候改了接口

我们总是吐槽文档不全，文档写的不好，然而却没有想到开发的同时顺便把文档也写了。这篇博文的重点就是为大家介绍一款生成RESTful web API文档的神器——[apidoc](http://apidocjs.com)，GitHub 4000多star。这款神器通过源代码中的注释生成最终的文档，所以需要按照规范编写注释。

# 安装和使用

## 前言

文中的例子全部使用Javadoc-Style编写，也可以在所有支持Javadoc的语言（如C#, Go, Dart, Java, JavaScript, PHP, TypeScript等）中使用。其他语言可以使用它们特定的多行注释。

```java
/**
 * This is a comment.
 */
```

## 安装

```bash
npm install apidoc -g
```

## 使用

```bash
apidoc -i myapp/ -o apidoc/ -t mytemplate/
```

使用`mytemplate/`下的模板文件，为`myapp/`目录下的所有文件创建api文档，并保存在`apidoc/`目录下。如果不带任何参数，apiDoc为当前目录以及子目录下的所有`.cs`, `.dart`, `.erl`, `.go`, `.java`, `.js`, `.php`, `.py`, `.rb`, `.ts`文件创建文档，并保存在`./doc/`目录。

## 命令行参数

```bash
# 查看命令行参数
apidoc -h
```

列出部分重要参数：

|参数|描述|示例|
|------|------|------|
|-f, --file-filters|通过正则过滤出需要解析的文件（可以使用多个-f）。默认为`.cs`, `.dart`, `.erl`, `.go`, `.java`, `.js`, `.php`, `.py`, `.rb`, `.ts`。|仅解析.js和.ts文件：<br>`apidoc -f ".*\\.js$" -f ".*\\.ts$"`|
|-i, --input|源文件或项目目录|`apidoc -i myapp/`|
|-o, --output|存放生成的文档的目录|`apidoc -o apidoc/`|
|-t, --template|为生成的文档使用模板，也可以创建和使用自己的模板|`apidoc -t mytemplate/`|

## Grunt模块

作者也为大家开发了一款grunt的打包工具[http://github.com/apidoc/grunt-apidoc](http://github.com/apidoc/grunt-apidoc)。

```bash
npm install grunt-apidoc --save-dev
```

## 模板

apiDoc默认模板：使用了[handlebars](http://handlebarsjs.com/)，[Bootstrap](http://twitter.github.io/bootstrap/)，[RequireJS](http://requirejs.org/)和[jQuery](http://jquery.com/)为输出的`api_data.js`和`api_project.js`文件生成html页面。

apiDoc默认使用一个复杂的模板，支持如下功能：

* 版本管理：查看不同版本的API
* 比较：查看一个API的两个版本之间的区别

你也可以使用自己创建的模板，为apiDoc生成的文件`api_data.js`, `api_project.js`或者json格式的文件`api_data.json`, `api_project.json`。

模板源代码：[https://github.com/apidoc/apidoc/tree/master/template](https://github.com/apidoc/apidoc/tree/master/template)

## 扩展

apiDoc也可以扩展自己的参数，具体细节请查看[apidoc/apidoc-core](https://github.com/apidoc/apidoc-core)项目的`lib/parsers/`, `lib/workers/`目录。

## 配置

apiDoc提供了两种配置方式，要么在工程根目录添加`apidoc.json`文件，要么在`package.json`中添加`apidoc`字段。

`apidoc.json`

```js
{
  "name": "example",
  "version": "0.1.0",
  "description": "apiDoc basic example",
  "title": "Custom apiDoc browser title",
  "url" : "https://api.github.com/v1"
}
```

`package.json`

```js
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

apidoc.json配置字段

|字段|描述|
|------|------|
|name|工程名称。如果`apidoc.json`中不包含此字段，则由`package.json确定`|
|version|工程版本号。如果`apidoc.json`中不包含此字段，则由`package.json确定`|
|description|工程描述。如果`apidoc.json`中不包含此字段，则由`package.json确定`|
|title|文档页面标题|
|url|api前缀，如：`https://api/github.com/v1`|
|sampleUrl|测试api方法的表单请求链接更多细节请查看[`@apiSampleRequest`](#apiSampleRequest)|
|header||
|&nbsp;&nbsp;&nbsp;&nbsp;title|被包含的header.md文件的导航文本（查看[Header/Footer](#header-footer)）|
|&nbsp;&nbsp;&nbsp;&nbsp;filename|被包含的header.md文件（必须为markdown文件）的文件名|
|footer||
|&nbsp;&nbsp;&nbsp;&nbsp;title|被包含的footer.md文件的导航文本|
|&nbsp;&nbsp;&nbsp;&nbsp;filename|被包含的footer.md文件（必须为markdown文件）的文件名|
|order|输出的api名和api分组名的顺序列表，未定义名称的api自动在后面展示。<br>`"order": ["Error", "Define", "PostTitleAndError", "PostError"]`|

apiDoc默认模板特定的配置字段

|字段|类型|描述|
|------|------|------|
|template|||
|forceLanguage|String|禁用浏览器自动语言检测，并设置一个特定的语言。例如：`de`, ` en`。[可选语言](https://github.com/apidoc/apidoc/tree/master/template/locales)
|withCompare|Boolean|开启与旧版本API比较，默认`true`|
|withGenerator|Boolean|在页尾输出生成信息，默认`true`|
|jQueryAjaxSetup|Object|为Ajax请求设置[默认参数](http://api.jquery.com/jquery.ajaxsetup/)|

## Header/Footer

在`apidoc.json`添加header和footer。

```js
{
  "header": {
    "title": "My own header title",
    "filename": "header.md"
  },
  "footer": {
    "title": "My own footer title",
    "filename": "footer.md"
  }
}
```

# 示例

## 继承

你可以将文档中多次使用的部分放到一个定义中。如下：

```js
/**
 * @apiDefine UserNotFoundError
 *
 * @apiError UserNotFound The id of the User was not found.
 *
 * @apiErrorExample Error-Response:
 *     HTTP/1.1 404 Not Found
 *     {
 *       "error": "UserNotFound"
 *     }
 */

/**
 * @api {get} /user/:id Request User information
 * @apiName GetUser
 * @apiGroup User
 *
 * @apiParam {Number} id Users unique ID.
 *
 * @apiSuccess {String} firstname Firstname of the User.
 * @apiSuccess {String} lastname  Lastname of the User.
 *
 * @apiSuccessExample Success-Response:
 *     HTTP/1.1 200 OK
 *     {
 *       "firstname": "John",
 *       "lastname": "Doe"
 *     }
 *
 * @apiUse UserNotFoundError
 */

/**
 * @api {put} /user/ Modify User information
 * @apiName PutUser
 * @apiGroup User
 *
 * @apiParam {Number} id          Users unique ID.
 * @apiParam {String} [firstname] Firstname of the User.
 * @apiParam {String} [lastname]  Lastname of the User.
 *
 * @apiSuccessExample Success-Response:
 *     HTTP/1.1 200 OK
 *
 * @apiUse UserNotFoundError
 */
```

**继承只能包含一层**，多层级将会降低行内代码的可读性，增加复杂度。

## 版本控制

保存之前定义的文档块，用于不同版本的接口比较，因此前端开发很容易看到哪些地方有变化。

在修改接口文档之前，将历史文档块复制到文件`_apidoc.js`。

所以在每个文档块设置[`@apiVersion`](#apiVersion)非常重要。

# apiDoc参数

结构体参数[`@apiDefine`](#apiDefine)用于将文档块定义为可复用的整体，并且可以包含在api文档块中。

除了其它定义块，一个定义的块中可以包含所有的参数（如[`@apiParam`](#apiParam)）。

## @api

必须包含这个标记，否则apiDoc将会忽略文档块。定义块[`@apiDefine`](#apiDefine)不需要`@api`。

```
@api {method} path [title]
```
|字段|描述|
|------|------|
|method|请求方法名：`DELETE`, `GET`, `POST`, `PUT`, ...。更多信息请查看[Wikipedia HTTP请求方法](https://zh.wikipedia.org/wiki/%E8%B6%85%E6%96%87%E6%9C%AC%E4%BC%A0%E8%BE%93%E5%8D%8F%E8%AE%AE#.E8.AF.B7.E6.B1.82.E6.96.B9.E6.B3.95)|
|path|请求路径|
|title `可选`|用于导航的简称|

```js
/**
 * @api {get} /user/:id
 */
```

## @apiDefine

定义一个通用块或者权限块，每个块中只能有一个`@apiDefine`，使用[`@apiUse`](#apiUser)导入通用块。

```
@apiDefine name [title]
    [description]
```

|字段|描述|
|------|------|
|name|块或值的唯一名称|
|title `可选`|简称，只用在命名函数中，如[`@apiPermission`](#apiPermission)和[`@apiParam(name)`](#apiParam)|
|description `可选`|下一行开始的详细说明，可以多行，仅用在命名函数，如[`@apiPermission`](#apiPermission)|

```js
/**
 * @apiDefine MyError
 * @apiError UserNotFound The <code>id</code> of the User was not found.
 */

/**
 * @api {get} /user/:id
 * @apiUse MyError
 */
```

```js
/**
 * @apiDefine admin User access only
 * This optional description belong to to the group admin.
 */

/**
 * @api {get} /user/:id
 * @apiPermission admin
 */
```

## @apiDeprecated

标记废弃的API

```
@apiDeprecated [text]
```

|字段|描述|
|------|------|
|text|多行文本|

```js
/**
 * @apiDeprecated */

/**
 * @apiDeprecated use now (#Group:Name).
 *
 * Example: to set a link to the GetDetails method of your group User
 * write (#User:GetDetails)
 */
```

## @apiDescription

API的详细描述

```
@apiDescription text
```

|字段|描述|
|------|------|
|text|多行描述文本|

```js
/**
 * @apiDescription This is the Description.
 * It is multiline capable.
 *
 * Last line of Description.
 */
```
## @apiError

返回的错误参数

```
@apiError [(group)] [{type}] field [description]
```

|字段|描述|
|------|------|
|(group) `可选`|所有参数按这个名称分组，如果未设置此字段，默认为`Error 4xx`，可以使用[`@apiDefine`](#apiDefine)设置标题和描述|
|{type} `可选`|返回类型，例如：`{Boolean}`，`{Number}`，`{String}`，`{Object}`，`{String[]}`（字符串数组）等|
|field|返回的标识符|
|description|字段描述|

```js
/**
 * @api {get} /user/:id
 * @apiError UserNotFound The <code>id</code> of the User was not found.
 */
```

## @apiErrorExample

返回的错误示例，输出为格式化的代码

```
@apiErrorExample [{type}] [title]
    example
```

|字段|描述|
|------|------|
|type `可选`|响应格式|
|title `可选`|示例简称|
|example|详细示例，支持多行文本|

```js
/**
 * @api {get} /user/:id
 * @apiErrorExample {json} Error-Response:
 * HTTP/1.1 404 Not Found
 * {
 *   "error": "UserNotFound"
 * }
 */
 ```

## @apiExample

API的使用示例，输出为格式化的代码

```
@apiExample [{type}] title
    example
```

|字段|描述|
|------|------|
|type `可选`|代码语言|
|title|示例简称|
|example|详细示例，支持多行文本|

```js
/**
 * @api {get} /user/:id
 * @apiExample {curl} Example usage:
 * curl -i http://localhost/user/4711
 */
```

## @apiGroup

应该一直使用，定义API文档块所属的分组。分组用于生成输出中的主导航，结构定义不需要`@apiGroup`

```
@apiGroup name
```

|字段|描述|
|------|------|
|name|分组名称，也用作导航名称|

```js
/**
 * @api {get} /user/:id
 * @apiGroup User
 */
```

## @apiHeader

描述传递给API的请求头，类似[`@apiParam`](#apiParam)，只不过输出文档在参数之上。

```
@apiHeader [(group)] [{type}] [field=defaultValue] [description]
```

|字段|描述|
|------|------|
|(group) `可选`|所有参数按此名称分组，如果未设置，默认为`Parameter`，你也可以在[`@apiDefine`](#apiDefine)中定义名称和描述|
|{type} `可选`|参数类型，例如：`{Boolean}`，`{Number}`，`{String}`，`{Object}`，`{String[]}`（字符串数组）等
|field|变量名|
|[field]|带括号标识此参数可选|
|=defaultValue `可选`|参数默认值|
|描述 `可选`|字段描述|

```js
/**
 * @api {get} /user/:id
 * @apiHeader {String} access-key Users unique access-key.
 */
```

## @apiHeaderExample

请求头参数示例

```
@apiHeaderExample [{type}] [title]
    example
```

|字段|描述|
|------|------|
|type `可选`|请求格式
|title `可选`|示例简称|
|example|详细示例，支持多行文本|

```js
/**
 * @api {get} /user/:id
 * @apiHeaderExample {json} Header-Example:
 * {
 *   "Accept-Encoding": "Accept-Encoding: gzip, deflate"
 * }
 */
```

## @apiIgnore


`@apiIgnore`块不会被解析，如果你在源代码中留下过期或者未完成的api，并且不想将其发布到文档中，这个标识符非常有用。

**将其放在块的顶部**

```
@apiIgnore [hint]
```

|字段|描述|
|------|------|
|hint `可选`|忽略原因的简短信息|

```js
/**
 * @apiIgnore Not finished Method
 * @api {get} /user/:id
 */
```

## @apiName

应该永远使用，定义API文档块的名称，名称将用于生成输出文档中的子导航。结构定义不需要`@apiName`

```
@apiName name
```

|字段|描述|
|------|------|
|name|API的唯一名称，可以为不同的[`@apiVersion`](#apiVersion)定义相同的名称|

```js
/**
 * @api {get} /user/:id
 * @apiName GetUser
 */
```

## @apiParam

定义传递给API的参数

```
@apiParam [(group)] [{type}] [field=defultValue] [description]
```

|字段|描述|
|------|------|
|(group) `可选`|所有参数按此名称分组，默认为`Parameter`，也可以在[`@apiDefine`](#apiDefine)中定义名称和描述|
|{type} `可选`|参数类型，例如：`{Boolean}`，`{Number}`，`{String}`，`{Object}`，`{String[]}`（字符串数组）等|
|{type{size}} `可选`|变量大小信息<br>`{string{..5}}` 最多5个字符的字符串<br>`{string{2..5}}` 最少2个字符，最多5个字符的字符串<br>`{Number{100-999}}` 介于100和999之间的数字|
|{type=allowedValues} `可选`|变量允许的值<br>`{string="small"}` 只能包含"small"的字符串<br>`{string="small","huge"}` 包含"small"或"huge"的字符串<br>`{number=1,2,3,99}` 允许为1,2,3,99中的一个值<br>`{string {..5}="small","huge"}` 最多5个字符的字符串，并且只能包含"small"和"huge"|
|field|参数名|
|[field]|带括号标识此参数可选|
|=defaultValue `可选`|参数默认值|
|description `可选`|参数描述|

```js
/**
 * @api {get} /user/:id
 * @apiParam {Number} id Users unique ID.
 */

/**
 * @api {post} /user/
 * @apiParam {String} [firstname] Optional Firstname of the User.
 * @apiParam {String} lastname Mandatory Lastname.
 * @apiParam {String} country="DE" Mandatory with default value "DE".
 * @apiParam {Number} [age=18] Optional Age with default 18.
 *
 * @apiParam (Login) {String} pass Only logged in users can post this.
 *        In generated documentation a separate
 *        "Login" Block will be generated.
 */
```

## @apiParamExample

请求参数示例

```
@apiParamExample [{type}] [title]
    example
```

|字段|描述|
|------|------|
|type `可选`|请求格式|
|title `可选`|示例简称|
|example|详细示例，支持多行文本|

```js
/**
 * @api {get} /user/:id
 * @apiParamExample {json} Request-Example:
 * {
 *   "id": 4711
 * }
 */
```

## @apiPermission

权限名称，如果用[`@apiDefine`](#apiDefine)定义名称，生成的文档将会包含额外的名称和描述

```
@apiPermission name
```

|字段|描述|
|------|------|
|name|权限唯一的名称|

```js
/**
 * @api {get} /user/:id
 * @apiPermission none
 */
```

## @apiSampleRequest

此参数配合`apidoc.json`配置中的`sampleUrl`参数使用，如果配置了`sampleUrl`，所有API方法都将有api测试表单，并追加在[`@api`](#api)结束位置。如果未配置`sampleUrl`，仅包含`@apiSampleRequest`的方法有测试表单。
如果在方法块中配置了`@apiSampleRequest url`，这个url作为请求地址（当它以http开头，会覆盖`sampleUrl`）
如果配置了`sampleUrl`，并且想在指定方法不包含测试表单，可以在文档块中配置`@apiSampleRequest off`。

```
@apiSampleRequest url
```

|字段|描述|
|------|------|
|url|测试api服务地址<br>`@apiSampleRequest http://www.example.com`<br>`@apiSampleRequest /my_test_path`<br>`@apiSampleRequest off`

```js
// Configuration parameter sampleUrl: "http://api.github.com"
/**
 * @api {get} /user/:id
 */

// Configuration parameter sampleUrl: "http://api.github.com"
/**
 * @api {get} /user/:id
 * @apiSampleRequest http://test.github.com/some_path/
 */

// Configuration parameter sampleUrl: "http://api.github.com"
/**
 * @api {get} /user/:id
 * @apiSampleRequest /test
 */

// Configuration parameter sampleUrl: "http://api.github.com"
/**
 * @api {get} /user/:id
 * @apiSampleRequest off
 */

// Configuration parameter sampleUrl is not set
/**
 * @api {get} /user/:id
 * @apiSampleRequest http://api.github.com/some_path/
 */
```

## @apiSuccess

成功返回的参数

```
@apiSuccess [(group)] [{type}] field [description]
```

|字段|描述|
|------|------|
|(group) `可选`|所有参数按此名称分组，默认为`Success 200`，可以在[`@apiDefine`](#apiDefine)中定义名称和描述
|{type} `可选`|返回类型，例如：`{Boolean}`，`{Number}`，`{String}`，`{Object}`，`{String[]}`（字符串数组）等
|field|返回标识字段|
|description `可选`|字段描述|

```js
/**
 * @api {get} /user/:id
 * @apiSuccess {String} firstname Firstname of the User.
 * @apiSuccess {String} lastname Lastname of the User.
 */

// 带(group)示例
/**
 * @api {get} /user/:id
 * @apiSuccess (200) {String} firstname Firstname of the User.
 * @apiSuccess (200) {String} lastname Lastname of the User.
 */

// 带对象示例
/**
 * @api {get} /user/:id
 * @apiSuccess {Boolean} active Specify if the account is active.
 * @apiSuccess {Object} profile User profile information.
 * @apiSuccess {Number} profile.age Users age.
 * @apiSuccess {String} profile.image Avatar-Image.
 */

// 带数组示例
/**
 * @api {get} /users
 * @apiSuccess {Object[]} profiles List of user profiles.
 * @apiSuccess {Number} profiles.age Users age.
 * @apiSuccess {String} profiles.image Avatar-Image.
 */
```

## @apiSuccessExample

成功响应的信息，按格式化代码输出

```
@apiSuccessExample [{type}] [title]
    example
```

|字段|描述|
|------|------|
|type `可选`|响应格式|
|title `可选`|示例简称|
|example|详细示例，支持多行文本|

```js
/**
 * @api {get} /user/:id
 * @apiSuccessExample {json} Success-Response:
 * HTTP/1.1 200 OK
 * {
 *   "firstname": "John",
 *   "lastname": "Doe"
 * }
 */
```

## @apiUse

包含一个[`@apiDefine`](#apiDefine)定义的块。如果与[`@apiVersion`](#apiVersion)一起使用，将包含相同的或最近的一个

```
@apiUse name
```

|字段|描述|
|------|------|
|name|定义块的名称|

```js
/**
 * @apiDefine MySuccess
 * @apiSuccess {string} firstname The users firstname.
 * @apiSuccess {number} age The users age.
 */

/**
 * @api {get} /user/:id
 * @apiUse MySuccess
 */
```

## @apiVersion

配置文档块的版本，也可以在[`@apiDefine`](#apiDefine)中使用，具有相同组和名称的API，可以在生成的文档中比较不同的版本，因此你或前端开发人员可以回溯自上一个版本以来API中的更改

```
@apiVersion version
```

|字段|描述|
|------|------|
|version|支持简单的版本控制（主版本号.次版本号.补丁号）。更多信息请参考[http://semver.org/](http://semver.org/)|

```js
/**
 * @api {get} /user/:id
 * @apiVersion 1.6.2
 */
```

到此为止，你应该对apidoc熟悉了一大半了吧！其实作为开发者，一定要养成写注释、写文档的习惯，也是程序员进阶的必要条件。

> 谨以此文鞭策自己。
