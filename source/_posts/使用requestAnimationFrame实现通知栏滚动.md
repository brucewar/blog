---
title: 使用requestAnimationFrame实现通知栏滚动
date: 2020-05-19 22:21:33
categories: Web前端
tags: [requestAnimationFrame,React]
---
# requestAnimationFrame是什么
`window.requestAnimationFrame()` 告诉浏览器——你希望执行一个动画，并且要求浏览器在下次重绘之前调用指定的回调函数更新动画。该方法需要传入一个回调函数作为参数，该回调函数会在浏览器下一次重绘之前执行。如果想让浏览器在下一次重绘前继续更新下一帧动画，需要在回调函数中再次调用`window.requestAnimationFrame()`。

回调函数执行次数通常是每秒**60**次，但在大多数遵循W3C建议的浏览器中，回调函数执行次数通常与浏览器屏幕刷新次数相匹配。为了提高性能和电池寿命，因此在大多数浏览器里，当`requestAnimationFrame()` 运行在后台标签页或者隐藏的`<iframe>` 里时，`requestAnimationFrame()` 会被暂停调用以提升性能和电池寿命。

回调函数执行时，会被传入一个时间戳参数，表示当前执行回调函数的时刻。

## 如何使用
```js
let frameId = window.requestAnimationFrame(callback)

// 停止动画
window.cancelAnimationFrame()
```

## 优缺点

和`setTimeout(callback, 16)`类似，但又不同，`requestAnimationFrame`是通过浏览器刷新频率决定执行的最佳时机，动画不会出现卡顿现象。

* 优点
    * 动画保持60fps（每帧16ms），浏览器内部决定渲染的最佳时机
    * API简洁标准，维护成本低
* 缺点
    * 动画的开始/取消需要开发者自己控制
    * 浏览器标签未激活时，一切都不会执行
    * 老版本浏览器不支持IE9
    * Node.js不支持，无法用在服务器的文件系统事件
    
# 实现滚动通知栏

以下基于React实现了一个文字可滚动的通知栏：

```js
import React, { Component } from 'react'

export default class NoticeBar extends Component {
  constructor (props) {
    super(props)
    this.state = {
      closed: false
    }
    this.marquee = null
    this.aFrameId = null
  }
  componentDidMount () {
    if (window.requestAnimationFrame && this.props.marqueeText) {
      let scrollWidth = this.marquee.parentElement.offsetWidth
      let textWidth = this.marquee.offsetWidth
      let right = 0
      let marquee = () => {
        right++
        if (right >= textWidth - scrollWidth + 30) {
          right = 0
        }
        this.marquee.style.right = right + 'px'
        this.aFrameId = window.requestAnimationFrame(marquee)
      }
      if(textWidth > scrollWidth) this.aFrameId = window.requestAnimationFrame(marquee)
    }
  }
  componentWillUnmount () {
    // 为了提高性能，组件卸载前，记得停止动画
    window.cancelAnimationFrame(this.aFrameId)
    this.aFrameId = null
  }
  onClose = () => {
    if (this.aFrameId) {
      window.cancelAnimationFrame(this.aFrameId)
      this.aFrameId = null
    }
    this.setState({ closed: true })
  }
  render () {
    const { mode, marqueeText, onClick, children } = this.props
    const { onClose } = this
    const { closed } = this.state
    if (closed) return null
    return (
      <div className="notice-bar">
        <div className="notice-bar-icon"/>
        <div className="notice-bar-content">
          {!!children && children}
          {
            !!marqueeText && !children && (
              <div className="notice-bar-marquee-wrap">
                <div className="notice-bar-marquee" ref={ele => this.marquee = ele}>
                  {marqueeText}
                </div>
              </div>
            )
          }
        </div>
        {
          mode === 'link' && (
            <div className="notice-bar-operation" onClick={onClick}>
              <i className="icon-link"/>
            </div>
          )
        }
        {
          mode === 'closable' && (
            <div className="notice-bar-operation" onClick={onClose}>
              <i className="icon-close"/>
            </div>
          )
        }
      </div>
    )
  }
}
```

```scss
.notice-bar{
  width: 100%;
  height: 30px;
  border-radius: 6px;
  background: #ffe37e;
  font-size: 12px;
  color: #222;
  display: flex;
  align-items: center;
  padding: 0 0 0 9px;
  & > .notice-bar-icon{
    background: url("../img/notice-bar-icon.png") center no-repeat;
    background-size: 100%;
    width: 9px;
    height: 9px;
    margin-right: 6px;
    flex-shrink: 0;
    flex-grow: 0;
  }
  & > .notice-bar-content{
    margin-right: 12px;
    white-space: nowrap;
    overflow: hidden;
    text-overflow: ellipsis;
    flex: 1;
    & .notice-bar-marquee-wrap{
      overflow: hidden;
      & .notice-bar-marquee{
        white-space: nowrap;
        display: inline-block;
        position: relative;
      }
    }
  }
  & > .notice-bar-operation{
    height: 100%;
    display: flex;
    & .icon-close{
      background: url("../img/notice-bar-close.png") center no-repeat;
      background-size: 100%;
      width: 12px;
      height: 12px;
      display: block;
      align-self: flex-start;
      flex-shrink: 0;
      flex-grow: 0;
    }
    & .icon-link{
      background: url("../img/notice-bar-link.png") center no-repeat;
      background-size: 100%;
      width: 12px;
      height: 12px;
      display: block;
      align-self: center;
      flex-shrink: 0;
      flex-grow: 0;
      margin-right: 12px;
    }
  }
}
```