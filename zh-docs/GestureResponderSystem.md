---
id: gesture-responder-system
title: 手势响应系统
layout: docs
category: Guides
permalink: docs/gesture-responder-system.html
next: nativemodulesios
---

在移动设备上的手势识别比 web 复杂得多。一个触摸操作有多个阶段，应用要判断用户此操作的意图是什么。例如，应用需要判断触摸操作是滚动，还是在小部件上的滑动，还是轻触。在触摸过程中这些甚至会发生变化。也可能会有多点同时触摸的情况。

触摸响应系统需要允许组件判断出触摸交互事件，不用关心父组件和子组件。该系统在 `ResponderEventPlugin.js` 中实现，源码中包含了更多详细的内容和文档。

### 最佳实践

用户在使用 web 应用和本地应用的时候可以感受到巨大的差异，这就是重要因素之一。每一个操作都应该包含下列的属性：

- 反馈/强调- 告诉用户什么在处理他们的触摸操作，当他们停止手势动作的时候会发生什么
- 能够中途取消- 当做一个手势操作的时候，用户应该能够通过移开手指来中途取消操作

这些特性使用户在使用一个应用的时候更加舒服，因为允许人们尽情与应用交互而不用担心操作出错。

### TouchableHighlight 和 Touchable*

该响应系统使用起来可能会非常复杂，所以我们为需要 “轻触” 的东西提供了一个抽象的 `Touchable` 实现。这使用了该响应系统，允许简单地配置声明轻触交互。在做传统 web 网站的时候，使用按钮和链接的地方，此时都应该换成 `TouchableHighlight` 。


## 响应器声明周期

一个视图可以成为触摸响应器，只要实现了正确的预先约定好的方法。有两个方法可以用于辨别某个视图是不是一个响应器：

 - `View.props.onStartShouldSetResponder: (evt) => true,` - 该视图是否要在触摸开始的时候响应？
 - `View.props.onMoveShouldSetResponder: (evt) => true,` - 手指在视图上每一次滑动都会调用：该视图是否要“声明”响应触摸操作？

如果视图返回 true ，尝试成为一个响应器，下列之一将会发生：

 - `View.props.onResponderGrant: (evt) => {}` - 该视图现在可以响应触摸事件了。是时候高亮某些部分来向用户展示发生了什么
 - `View.props.onResponderReject: (evt) => {}` - 有别的什么视图是当前手势操作的响应器，并且将不会释放响应权

如果视图正在响应手势操作，下列的处理器将会被调用：

 - `View.props.onResponderMove: (evt) => {}` - 用户正在移动手指
 - `View.props.onResponderRelease: (evt) => {}` - 在触摸结束的时候触发，例如“ touchUp ”
 - `View.props.onResponderTerminationRequest: (evt) => true` - 其它视图想成为响应器。该视图应该释放响应权吗？返回 true 允许释放响应权
 - `View.props.onResponderTerminate: (evt) => {}` - 视图的响应权被剥夺。可能在调用 `onResponderTerminationRequest` 之后被其它视图夺走，或者未通知的情况下被操作系统夺走（ iOS 上点击控制中心或者通知中心的时候）

`evt` 是一个人造的触摸事件对象，有下列几种形式：

 - `nativeEvent`
     + `changedTouches` - 一组所有的从上次事件改变了的触摸事件（Array of all touch events that have changed since the last event）
     + `identifier` - 该次触摸的 ID
     + `locationX` - 触摸的 X 坐标，相对于元素
     + `locationY` - 触摸的 Y 坐标，相对于元素
     + `pageX` - 触摸的 X 坐标，相对于屏幕
     + `pageY` - 触摸的 Y 坐标，相对于屏幕
     + `target` - 接收触摸事件的元素的节点 id
     + `timestamp` - 触摸发生的时间，对计算速度很有帮助
     + `touches` - 屏幕上当前所有触摸点事件对象

### 捕获ShouldSet 处理器

`onStartShouldSetResponder` 和 `onMoveShouldSetResponder` 在一个冒泡模型中调用，最深的节点最先调用。这意味着当多个视图给 `*ShouldSetResponder` 返回 true 的时候，最深的组件将会获得响应权。这在很多场景当中是满足需求的，因为确保所有控件和按钮都是可用的。

然而，某些时候父级想要确保自己是响应器，这可以在捕获阶段处理。在响应系统从最深组件冒泡之前，会有一个捕获过程，触发 `on*ShouldSetResponderCapture` 。因此，如果父视图想要在触摸开始的时候阻止子视图成为响应器，就应该有一个返回 true 的 `onStartShouldSetResponderCapture` 处理器。

 - `View.props.onStartShouldSetResponderCapture: (evt) => true,`
 - `View.props.onMoveShouldSetResponderCapture: (evt) => true,`

### PanResponder

对于更高层的手势识别，参考 [PanResponder](/react-native/docs/panresponder.html)。