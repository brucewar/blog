---
title: npm3.0新的依赖解决方案
date: 2016-12-20 23:02:20
categories: npm
tags: [npm,v3]
---

最近给我的装备（*Thinkpad S3-s431*）升了一下级，将原本用来加速缓存的`24G`固态硬盘换成了`128G`。所以得重装系统，然后一堆软件也得重装。包括Node.js。

安装了最新的Node.js（v6.9.2）,npm（v3.10.9）。由于`node_modules`里的文件夹结构太深，无法移动，只能去项目中使用`npm install`重新安装依赖，然后发现`node_modules`文件夹结构是这样的：

![npm v3](npmv3.png)

一个模块被分在了不同文件夹下，满足下好奇心，去看了[npm的官方文档](https://docs.npmjs.com/how-npm-works/npm3#npm-v3-dependency-resolution)。

果然NPM开发团队还是解决了这个包冗余和包结构太深的问题，下面我们来看看他们是如何做的。

npm2以一种嵌套的方式安装所有的依赖，而npm3将所有依赖都安装在主目录的`node_modules`下。如下图所示：

![npm2和npm3比较](npm3deps2.png)

APP依赖模块A，而模块A又依赖模块B，npm2的方案是将模块B安装在模块A的`node_modules`。npm3将模块A和模块B都安装在APP的`node_modules`中。假如APP依赖另一个模块C，而模块C又依赖另一个版本的模块B，我们看它们又有哪些区别：

![npm2和npm3不同版本模块管理比较](npm3deps4.png)

npm2中，会在模块A和C的`node_modules`下安装不同版本的模块B，而npm3先安装模块A的同时，将其依赖的模块B的1.0版本安装在APP模块下，为了防止模块冲突，同npm2的做法类似，将模块C依赖的模块B2.0安装在模块C下。

通过`npm ls`命令，我们可以看到模块依赖关系的树形结构图。如果想看项目主目录的依赖，可以使用如下命令：

```bash
npm ls --depth=0
```
