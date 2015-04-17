# React Native [![Build Status](https://travis-ci.org/facebook/react-native.svg?branch=master)](https://travis-ci.org/facebook/react-native)

React Native 使你能够运用 JavaScript 和 [React](http://facebook.github.io/react) 的相关知识，基于本地平台构建世界级的应用。 React Native 关注的焦点是开发者能够高效地开发所有相关平台的应用 - 学习一样东西就可以做任何事情。 Facebook 把 React Native 运用于大量的生产应用，并且将会继续向 React Native 投入人力财力。

## 本地 iOS 组件

在基于 React Native 的开发中，你可以使用标准的平台组件，比如 iOS 上的 `UITabBar` 和 `UINavigationController` 。这使你应用的界面看起来风格没有改变，不用考虑其它平台系统是怎么样的，保证了应用的高质量。这些组件可以通过使用它们对应的 React 组件简单地集成到你的应用中，比如 _TabBarIOS_ 和 _NavigatorIOS_ 。

```javascript
var React = require('react-native');
var { TabBarIOS, NavigatorIOS } = React;

var App = React.createClass({
  render: function() {
    return (
      <TabBarIOS>
        <TabBarIOS.Item title="React Native" selected={true}>
          <NavigatorIOS initialRoute={{ title: 'React Native' }} />
        </TabBarIOS.Item>
      </TabBarIOS>
    );
  },
});
```

## 异步执行

所有 JavaScript 应用代码和本地平台代码之间的操作都是异步的，本地模块也可以使用另外的线程。这意味着可以在非主线程中解码图片，在后台保存到磁盘，计量文本和计算布局而不阻塞 UI ，等等。最后， React Native 应用是自然流动的和响应式的。本地代码和 JavaScript 代码的交互是完全格式化的，这就使得我们能够利用谷歌开发者工具在整个应用运行的时候来调试 JavaScript 代码，不论是在模拟器中还是物理设备中。

![](http://facebook.github.io/react-native/img/chrome_breakpoint.png)


## 触摸处理

iOS 有一个非常强大的系统叫做响应链（ Responder Chain ），用于处理复杂的视图层级结构中的触摸操作，在 web 中没有对应的东西。 React Native 实现了一个类似的响应系统，提供了高层次的组件，比如 TouchableHighlight 正确集成滚动控件和其它一些元素而不需要任何额外的配置。

```javascript
var React = require('react-native');
var { ScrollView, TouchableHighlight, Text } = React;

var TouchDemo = React.createClass({
  render: function() {
    return (
      <ScrollView>
        <TouchableHighlight onPress={() => console.log('pressed')}>
          <Text>Proper Touch Handling</Text>
        </TouchableHighlight>
      </ScrollView>
    );
  },
});
```


## 弹性盒子和样式
视图布局应该是简单的，这就是为什么我们从 web 引入了弹性盒子布局到 React Native。弹性盒子使构建大多数常规的 UI 布局变得简单，比如带有外边距和内边距的框的并排放置和嵌套。 React Native 也支持常见的 web 样式，比如 `fontWeight` ；还支持 `StyleSheet` 抽象，这种抽象机制使得你可以在组件附近声明所有的样式和布局，然后在行内使用声明的样式（通过 style 属性），这样代码就比较优化了。

```javascript
var React = require('react-native');
var { Image, StyleSheet, Text, View } = React;

var ReactNative = React.createClass({
  render: function() {
    return (
      <View style={styles.row}>
        <Image
          source={{uri: 'http://facebook.github.io/react/img/logo_og.png'}}
          style={styles.image}
        />
        <View style={styles.text}>
          <Text style={styles.title}>
            React Native
          </Text>
          <Text style={styles.subtitle}>
            Build high quality mobile apps using React
          </Text>
        </View>
      </View>
    );
  },
});
var styles = StyleSheet.create({
  row: { flexDirection: 'row', margin: 40 },
  image: { width: 40, height: 40, marginRight: 10 },
  text: { flex: 1, justifyContent: 'center'},
  title: { fontSize: 11, fontWeight: 'bold' },
  subtitle: { fontSize: 10 },
});
```

## Polyfills

React Native 专注于改变视图层代码书写的方式。其它的代码，我们参照 web 的通用标准，然后在合适的地方实现那些 API 。你可以使用 npm 来安装 JavaScript 库，这些库在 React Native 的基础上实现一些功能，比如 `XMLHttpRequest` ， `window.requestAnimationFrame` ， `navigator.geolocation` 。我们正在逐步扩展这些有用的 API ，并且也很高兴开源社区能贡献代码。

```javascript
var React = require('react-native');
var { Text } = React;

var GeoInfo = React.createClass({
  getInitialState: function() {
    return { position: 'unknown' };
  },
  componentDidMount: function() {
    navigator.geolocation.getCurrentPosition(
      (position) => this.setState({position}),
      (error) => console.error(error)
    );
  },
  render: function() {
    return (
      <Text>
        Position: {JSON.stringify(this.state.position)}
      </Text>
    );
  },
});
```

## 扩展性（ Extensibility ）

使用 React Native ，在不写一行本地代码的前提下创建一个伟大的 app 是很有可能的，但是 React Native 也可以通过自定义的本地视图和模块来轻松扩展 - 这意味着你可以重用已经创建好的任何东西，也能够引入和使用你最喜欢的本地库。为了在 iOS 中创建一个简单的模块，先创建一个实现 `RCTBridgeModule` 协议的新类，在 `RCT_EXPORT_METHOD` 中包装你想导出到 JavaScript 中的函数。另外，类本身必须用 `RCT_EXPORT_MODULE()` 显示导出。

```objc
// Objective-C

#import "RCTBridgeModule.h"

@interface MyCustomModule : NSObject <RCTBridgeModule>
@end

@implementation MyCustomModule

RCT_EXPORT_MODULE();

// Available as NativeModules.MyCustomModule.processString
RCT_EXPORT_METHOD(processString:(NSString *)input callback:(RCTResponseSenderBlock)callback)
{
  callback(@[[input stringByReplacingOccurrencesOfString:@"Goodbye" withString:@"Hello"]]);
}
@end
```

```javascript
// JavaScript

var React = require('react-native');
var { NativeModules, Text } = React;

var Message = React.createClass({
  render: function() {
    getInitialState() {
      return { text: 'Goodbye World.' };
    },
    componentDidMount() {
      NativeModules.MyCustomModule.processString(this.state.text, (text) => {
        this.setState({text});
      });
    },
    return (
      <Text>{this.state.text}</Text>
    );
  },
});
```

自定义 iOS 视图可以通过继承 `RCTViewManager` 来导出，实现一个 `-view` 方法，通过 `RCT_EXPORT_VIEW_PROPERTY` 宏导出属性。然后在一个简单的 JavaScript 文件中对接上。

```objc
// Objective-C

#import "RCTViewManager.h"

@interface MyCustomViewManager : RCTViewManager
@end

@implementation MyCustomViewManager
- (UIView *)view
{
  return [[MyCustomView alloc] init];
}

RCT_EXPORT_VIEW_PROPERTY(myCustomProperty);

@end`}
```

```javascript
// JavaScript

var MyCustomView = createReactIOSNativeComponentClass({
  validAttributes: { myCustomProperty: true },
  uiViewClassName: 'MyCustomView',
});
```
## 运行例子

- `git clone https://github.com/facebook/react-native.git`
- `cd react-native && npm install`
- `cd Examples`

现在，可以在 Xcode 中打开任何示例并运行。

更多更深入的文档内容，教程，请参看 [React Native website](http://facebook.github.io/react-native/docs/getting-started.html) 。


## 文档目录

* 快速开始
  - [入门](https://github.com/reactjs-cn/react-native/blob/master/zh-docs/GettingStarted.md)
  - [教程](https://github.com/reactjs-cn/react-native/blob/master/zh-docs/Tutorial.md)
  - [视频](https://github.com/reactjs-cn/react-native/blob/master/zh-docs/Videos.md)
* 指南
  - [样式](https://github.com/reactjs-cn/react-native/blob/master/zh-docs/Style.md)
  - [手势响应系统（ Gesture Responder System ）](https://github.com/reactjs-cn/react-native/blob/master/zh-docs/GestureResponderSystem.md)
  - [本地模块（ iOS ）](https://github.com/reactjs-cn/react-native/blob/master/zh-docs/NativeModulesIOS.md)
  - [链接已有库](https://github.com/reactjs-cn/react-native/blob/master/zh-docs/LinkingLibraries.md)
  - [调试](https://github.com/reactjs-cn/react-native/blob/master/zh-docs/Debugging.md)
  - [测试](https://github.com/reactjs-cn/react-native/blob/master/zh-docs/Testing.md)
  - 在真实设备上运行
  - 与已有 App 集成
  - JavaScript 环境

