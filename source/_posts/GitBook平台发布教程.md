---
title: GitBook平台发布教程
date: 2016-12-05 17:11:25
categories: GitBook
tags: [gitbook,电子书,教程]
---

最近兴趣所致翻译了一本英文的SVG教程，并将其托管在**GitHub**，部署于**GitBook**。为了给大家最直观的效果，献上教程部署地址：[https://svg.brucewar.me](https://svg.brucewar.me)。这也是我第一次翻译英文文档，也是我第一次使用GitBook部署电子书。教程虽然简单，但基本涉及了SVG的所有知识点。

> 喂喂喂，跑题了啊！

回到正题，因为部署电子书涉及的工具比较多，不必担心，我将按以下流程来各个击破：

* 本地生成电子书
* 托管GitHub
* 发布到GitBook
* 绑定自定义域名

## 本地生成电子书

GitBook官方提供了一个命令行工具（gitbook），可以使用git和markdown制作本地电子书并支持预览等功能。在安装这个命令行工具之前，你需要安装[Node.js](https://nodejs.org/zh-cn/)，官网已经提供了安装包，这里就不详细说明Node.js的安装方法了。然后便可以通过Node.js提供的包管理工具`npm`安装gitbook了。

### 安装gitbook

```bash
npm install gitbook-cli -g
```

等待片刻，可以使用下面命令确认是否安装成功：

```bash
gitbook -V

# result:
# CLI version: 2.3.0
# GitBook version: 3.2.2

# 查看gitbook提供的一些命令
gitbook -h
```

### 初始化电子书

首先为电子书建一个文件夹，然后通过命令行工具初始化电子书：

```bash
mkdir ebook && cd ebook
gitbook init
```

最终会在文件夹下生成两个文件：`README.md`和`SUMMARY.md`。前者主要是关于你的书的介绍，后者包含书的目录，即章节结构，格式如下：

```
* [第一章](chapter1.md)
	* [第一节](chapter1_section1.md)
	* [第二节](chapter1_section2.md)
* [第二章](chapter2.md)
```

下面你就可以使用markdown语法编写电子书啦！如果你想预览电子书，可以使用下面命令：

```bash
gitbook serve
```

第一次使用这个命令会有点慢，它会安装gitbook的一些额外工具。等待一会儿，会出现提示（Serving book on [http://localhost:4000](http://localhost:4000)），打开链接便可预览你创建的电子书啦！运行该命令会在电子书的文件夹下生成一个`_book`文件夹，里面的内容就是生成的HTML文件。

### 高级配置

gitbook也提供了一些高级配置信息，可以通过在电子书文件夹下添加一个`book.json`文件为其添加一些高级配置，主要的配置信息如下：

* title：电子书的名称
* author：作者
* description：本书的简单描述
* language：gitbook使用的语言
* links：左侧导航栏添加的链接
* styles：自定义页面样式
* plugins：配置使用的插件
* pluginsConfig：各个插件对应的配置
* gitbook：指定使用gitbook版本

```js
{
	"title": "SVG教程（中文翻译版）",
	"author": "brucewar <wjl891014@gmail.com>",
	"description": "SVG教程中文翻译版，也是本人第一次翻译英文教程。",
	"links": {
		"sidebar": {
			"博客": "http://brucewar.me",
			"打赏": "http://brucewar.me/donate/",
			"View on GitHub": "https://github.com/brucewar/svg-tutorial"
		}
	},
	"plugins": [
		"duoshuo",	// 多说评论
		"github",	// 展示github图标
		"ga",	// google analytics
		"ba"	// 百度统计
	],
	"pluginsConfig": {
		"duoshuo": {
			"short_name": "brucewar",
			"theme": "default"
		},
		"github": {
			"url": "https://github.com/brucewar"
		},
		"ga": {
			"token": "UA-87259783-2"
		},
		"ba": {
			"token": "80f89a3fe34a9f8e22c53f85908e2d6"
		}
	}
}
```

上面是我使用的一些配置。如果在你的配置中添加了插件，执行`gitbook serve`前，需要通过以下命令安装这些插件。

```bash
gitbook install
```

gitbook提供了一个插件平台，你可以去搜索你想要的插件，当然，你也可以为其贡献插件。

## 托管GitHub

首先，你需要有一个[GitHub](https://github.com)账号。然后创建一个公有仓库。然后，回到电子书目录下，使用git工具初始化一个仓库。并将本地仓库和远程GitHub仓库关联。

```bash
git init
git remote add origin 你的远程仓库地址（比如https://github.com/brucewar/svg-tutorial）

// 提交电子书
git add .
git commit -m "publish"
git push origin master
```

## 发布到GitBook

如果没有[GitBook](https://www.gitbook.com)账号，可以先注册一个。然后在新建一本book。然后需要做的就是将GitBook和GitHub进行关联。

![GitBook关联GitHub步骤](gitbook2github.jpg)

但是，这还不够，因为GitBook不知道何时构建你的电子书，所以这里我们就要用到GitHub的*webhook*功能。通俗点讲，就是在GitHub提交电子书的同时，让GitHub告诉GitBook，有更新了，你重新构建下。具体操作如下：

关联完GitHub仓库后，会出现如下界面，复制webhook url：

![复制Webhook URL](webhook_url.jpg)

打开GitHub仓库的设置页面，添加一个webhook：

![在GitHub添加webhook](webhook_add.jpg)

## 绑定自定义域名

此时，你可以通过GitBook提供的域名访问你刚才创建的电子书啦！一般域名格式是http://{author}.gitbooks.io/{book}/content，但是你也可以使用自定义的域名（首先你得买个域名）。

![绑定域名](custom_domain.jpg)
