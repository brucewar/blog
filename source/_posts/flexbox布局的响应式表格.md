---
title: flexbox布局的响应式表格
date: 2017-07-06 08:57:19
categories: 前端开发
tags: [CSS,flex]
---

HTML `table`元素做响应式表格是非常难用的，我们需要大量的样板和嵌套的HTML来解决这样一个简单的问题。我们来探讨一种使用`div`和[Flexbox](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_Flexible_Box_Layout/Using_CSS_flexible_boxes)的替代方法。这将给我们带来的好处是能够创建在所有尺寸屏幕上都看起来很棒的响应式表格。

首先，解决方案将使用[SUIT CSS](http://inlehmansterms.net/2014/08/25/modular-css-with-suitcss/)以模块化方式写入[Sass](http://sass-lang.com/)。我们将使用一些Sass库帮助我们完成任务。[autoprefixer](https://autoprefixer.github.io/)用于帮助我们生成必要的Flexbox CSS供应商前缀，以及[Breakpoint](http://breakpoint-sass.com/)来帮助我们做media query。如果你喜欢用CSS，请随时将生成的CSS从链接复制到[Sassmeister](http://sassmeister.com/gist/b38aca96fc6024a28514)示例。

我们需要的是构建`Table`组件的3个基本类。首先，我们需要`Table`类，它将使用Flexbox来使所有的子节点（行）按列布局。接下来，我们需要一个`Table-row`类，它将使用Flexbox来使其所有的子节点（行/列）按行排列，而不需要包装。最后我们需要`Table-row-item`，它基本上是表格组件的一个单元格。现在，我们需要的是一个`Table-header`类，我们可以添加到任何行元素，以给它一个标题的样式。根据这些标准，我们可以为我们的组件编写HTML和Sass，如下所示。

![列等宽表格](table1.png)

```html
<div class="Table">
  <div class="Table-row Table-header">
    <div class="Table-row-item">Header1</div>
    <div class="Table-row-item">Header2</div>
    <div class="Table-row-item">Header3</div>
    <div class="Table-row-item">Header4</div>
  </div>
  <div class="Table-row">
    <div class="Table-row-item" data-header="Header1">row1 col1</div>
    <div class="Table-row-item" data-header="Header2">row1 col2</div>
    <div class="Table-row-item" data-header="Header3">row1 col3</div>
    <div class="Table-row-item" data-header="Header4">row1 col4</div>
  </div>
  <div class="Table-row">
    <div class="Table-row-item" data-header="Header1">row2 col1</div>
    <div class="Table-row-item" data-header="Header2">row2 col2</div>
    <div class="Table-row-item" data-header="Header3">row2 col3</div>
    <div class="Table-row-item" data-header="Header4">row2 col4</div>
  </div>
</div>
```

```scss
@import "breakpoint";

.Table {
  $light-color: #ffffff;
  $dark-color: #f2f2f2;
  $md: 500px;

  display: flex;
  flex-flow: column nowrap;
  justify-content: space-between;
  border: 1px solid $dark-color;
  font-size: 1rem;
  margin: 0.5rem;
  line-height: 1.5;

  // .Table-header
  &-header {
    display: none;
    @include breakpoint($md) {
      font-weight: 700;
      background-color: $dark-color;
    }
  }
  // .Table-row
  &-row {
    width: 100%;
    &:nth-of-type(even) { background-color: $dark-color; }
     &:nth-of-type(odd) { background-color: $light-color; }
    @include breakpoint($md) {
      display: flex;
      flex-flow: row nowrap;
      &:nth-of-type(even) { background-color: $light-color; }
      &:nth-of-type(odd) { background-color: $dark-color; }
    }
    // .Table-row-item
    &-item {
      display: flex;
      flex-flow: row nowrap;
      flex-grow: 1;
      flex-basis: 0;
      padding: 0.5em;
      word-break: break-word;
      &:before {
        content: attr(data-header);
        width: 30%;
        font-weight: 700;
      }
      @include breakpoint($md) {
        border: 1px solid $light-color;
        padding: 0.5em;
        &:before { content: none; }
      }
    }
  }
}
```

[在Sassmeister尝试这个例子](http://sassmeister.com/gist/b38aca96fc6024a28514)

根据此实现，我们可以轻松创建响应式表格。每行简单地将项目的数量分解成等于列。这使我们能够灵活地创建具有不同列数的表格。然而，同样的好处也有缺点。

我们的表格组件的问题是每个列的宽度相同。如果我们有的列包含的数据比其他列更宽或更窄会发生什么呢？幸运的是，Flexbox也使得这个很容易实现。我们可以简单地添加一些实用的工具类来设置不同列的Flexbox增长率。

![列不等宽表格](table2.png)

```scss
// generate Flexbox grow-rate utility classes
@for $i from 1 through 10 {
  .u-Flex-grow#{$i} {
    flex-grow: i;
  }
}
```

```html
<div class="Table">
  <div class="Table-row Table-header">
    <div class="Table-row-item u-Flex-grow2">Long Header1</div>
    <div class="Table-row-item">Header2</div>
    <div class="Table-row-item">Header3</div>
    <div class="Table-row-item">Header4</div>
    <div class="Table-row-item u-Flex-grow3">Longer Header5</div>
    <div class="Table-row-item">Header6</div>
    <div class="Table-row-item">Header7</div>
  </div>
  <div class="Table-row">
    <div class="Table-row-item u-Flex-grow2" data-header="Header1">row1 col1</div>
    <div class="Table-row-item" data-header="Header2">row1 col2</div>
    <div class="Table-row-item" data-header="Header3">row1 col3</div>
    <div class="Table-row-item" data-header="Header4">row1 col4</div>
    <div class="Table-row-item u-Flex-grow3" data-header="Header5">row1 col5</div>
    <div class="Table-row-item" data-header="Header6">row1 col6</div>
    <div class="Table-row-item" data-header="Header7">row1 col7</div>
  </div>
  <div class="Table-row">
    <div class="Table-row-item u-Flex-grow2" data-header="Header1">row2 col1</div>
    <div class="Table-row-item" data-header="Header2">row2 col2</div>
    <div class="Table-row-item" data-header="Header3">row2 col3</div>
    <div class="Table-row-item" data-header="Header4">row2 col4</div>
    <div class="Table-row-item u-Flex-grow3" data-header="Header5">row2 col5</div>
    <div class="Table-row-item" data-header="Header6">row2 col6</div>
    <div class="Table-row-item" data-header="Header7">row2 col7</div>
  </div>
  <div class="Table-row">
    <div class="Table-row-item u-Flex-grow2" data-header="Header1">row3 col1</div>
    <div class="Table-row-item" data-header="Header2">row3 col2</div>
    <div class="Table-row-item" data-header="Header3">row3 col3</div>
    <div class="Table-row-item" data-header="Header4">row3 col4</div>
    <div class="Table-row-item u-Flex-grow3" data-header="Header5">row3 col5</div>
    <div class="Table-row-item" data-header="Header6">row3 col6</div>
    <div class="Table-row-item" data-header="Header7">row3 col7</div>
  </div>
</div>
```

[在Sassmeister尝试这个例子](http://sassmeister.com/gist/de4eab2113204729ea50)

> 本文翻译自[http://inlehmansterms.net/2014/10/11/responsive-tables-with-flexbox/](http://inlehmansterms.net/2014/10/11/responsive-tables-with-flexbox/)