---
title: ' windows下node版本管理:nvm-windows'
date: 2017-01-13 09:18:18
categories: nvm
tags: [nvm,node.js,nvm-windows]
---
最近工作上接手了两个项目，可它们依赖的node版本不同，于是想到了之前用的**nvm**（Node Version Manager）。

> [https://github.com/creationix/nvm](https://github.com/creationix/nvm)

之前安装nvm的方式是通过`npm install nvm`，而新版本可以通过脚本或者手动安装。目前，nvm没有提供windows的支持，但是在其文档中提到了**nvm-windows**这个工具。

> [https://github.com/coreybutler/nvm-windows](https://github.com/coreybutler/nvm-windows)

### 安装

在安装nvm-windows前，需要做以下步骤：

1. 卸载系统中已有的node.js
2. 删除node.js安装目录（例如`C:\Program Files\nodejs`）
3. 删除npm包的目录（例如`C:\Users<user>\AppData\Roaming\npm`）

![打开release页面](release.png)

打开release页面，下载最新版本的安装包。

![安装](install.png)

### 更新

更新也很简单，直接下载最新版的nvm-windows安装即可。它将安全的覆盖文件。

### 使用方法

* `nvm install <version> [arch]`：version可以是指定的node.js版本或者`latest`（最新版），arch可以是`32`、`64`或者`all`
* `nvm list [available]`：列出当前已经安装的node.js版本，`available`参数列出可安装版本
* `nvm use <version> [arch]`：切换node.js版本

这里就介绍几个常用命令，更多命令请自行看文档。
