---
title: 理解AngularJS Providers
date: 2017-05-27 14:17:30
categories: 前端开发
tags: [AngularJS,Provider]
---

AngularJS 1.x提供了如下5个provider方法，使用这些方法能创建可被注入的服务。但是如何使用这些方法，以及这些方法之间有什么区别呢？所以这篇博客重点记录下这些方法的区别，来帮助自己理解它们。

* [value](#value)
* [factory](#factory)
* [service](#service)
* [provider](#constant)
* [constant](#constant)

# value

这个方法用来定义常量，并且能在运行阶段注入Controller中。比如我们可以创建一个简单的服务叫"clientId"，并提供一个用于身份验证的id。

```js
var myApp = angular.module('myApp', []);
myApp.value('clientId', '12345678');

myApp.controller('DemoController', ['clientId', function DemoController(clientId) {
  this.clientId = clientId;
}]);
```

```html
<html ng-app="myApp">
  <body ng-controller="DemoController as demo">
    Client ID: {{demo.clientId}}
  </body>
</html>
```

注：常量值可以为JS的任意基本类型

# factory

`value`方法非常简单，但缺少创建服务时常常需要的一些重要功能。所以`factory`方法增加了以下功能：

* 使用其他服务的能力
* 服务初始化
* 延时初始化

`factory`方法使用0个或多个参数（其他服务）的函数构建新的服务，并且此函数的返回值为工厂方法创建的服务实例。

注：AngularJS中所有的服务都是单例，注射器injector最多使用这些provider方法一次来创建服务对象，并将它们缓存下来。

同样的例子，我们通过`factory`实现一下：

```js
myApp.factory('clientId', function clientIdFactory() {
  return '12345678';
});

// 这里只是返回一个字符串，可能用value更好点。如果需要运算，可以这样写
myApp.factory('apiToken', ['clientId', function apiTokenFactory(clientId) {
  var encrypt = function(data1, data2) {
    return (data1 + ':' + data2).toUpperCase();
  };

  var secret = window.localStorage.getItem('myApp.secret');
  var apiToken = encrpty(clientId, secret);

  return apiToken;
}]);
```

`factory`方法可以创建任何类型的服务，比如JavaScript基本类型，对象字面量，函数或者自定义类型的实例。

# service

可能我们经常需要使用自定义类型编写面向对象的代码。

```js
function UnicornLauncher(apiToken) {
  this.launchedCount = 0;
  this.launch = function() {
    this.lanuchedCount++;
  };
}

myApp.factory('unicornLauncher', ['apiToken', function(apiToken) {
  return new UnicornLauncher(apiToken);
}]);
```

上面是通过`factory`实现对象实例化，然而这里需要我们自己初始化实例，所以Angular提供了`service`这个方法，它接受一个构造函数作为参数。

```js
myApp.service('unicornLauncher', ['apiToken', UnicornLauncher]);
```

# provider

`provider`是核心方法，其他方法都是在基于它的语法糖。它的作用是在应用启动前提供可配置的API。通常用在一些可重用的服务，并且这些服务在不同的应用程序之间需要不同配置。

```js
myApp.provider('unicornLauncher', function UnicornLauncherProvider() {
  var useTinfoilShielding = false;

  this.useTinfoilShielding = function(value) {
    useTinfoilShielding = !!value;
  };

  this.$get = ["apiToken", function unicornLauncherFactory(apiToken) {

    // let's assume that the UnicornLauncher constructor was also changed to
    // accept and use the useTinfoilShielding argument
    return new UnicornLauncher(apiToken, useTinfoilShielding);
  }];
});

myApp.config(["unicornLauncherProvider", function(unicornLauncherProvider) {
  unicornLauncherProvider.useTinfoilShielding(true);
}]);
```

在应用启动之前，AngularJS调用`config`方法对服务完成配置，运行阶段会初始化所有的服务，所以，在配置阶段是访问不到服务，即不可注入。一旦配置阶段结束，就不能与provider交互。

# constant

由于`config`函数在没有服务可用的配置阶段运行，所以`value`方法创建的值或对象也无法访问。所以为了在配置阶段能访问到一些常量，引入了`constant`函数，用法和`value`类似。

# 特殊用途对象

由`Controller`，`Directive`，`Filter`，`Animation`创建的对象，和`factory`写法类似，在函数中返回对象实例，Controller对象除外。

# 总结

* 注射器使用这些方法创建两种类型的对象：服务和特殊用途对象
* 五种方法定义如何创建对象：value，factory，service，provider，constant
* factory和service是最常用的方法，它们之间唯一的区别是，service对于创建自定义类型的对象更好，而factory方法可用生成JavaScript基本类型和函数
* provider是核心方法，其他所有方法只是其上的语法糖
* provider是最复杂的方法，除非你需要构建可重用的代码块，否则最好别用它
* 除Controller之外，所有特殊用途的对象均通过factory方法定义
