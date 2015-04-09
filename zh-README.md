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

iOS 有一个非常强大的系统叫做响应链，
iOS has a very powerful system called the Responder Chain to negotiate touches in complex view hierarchies which does not have a universal analog on the web. React Native implements a similar responder system and provides high level components such as TouchableHighlight that integrate properly with scroll views and other elements without any additional configuration.

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
视图布局应该是简单的，这就是为什么我们从 web 引入了弹性盒子布局到 React Native。弹性盒子使构建大多数常规的 UI 布局变得简单，
Laying out views should be easy, which is why we brought the flexbox layout model from the web to React Native.  Flexbox makes it simple to build the most common UI layouts, such as stacked and nested boxes with margin and padding.  React Native also supports common web styles, such as `fontWeight`, and the `StyleSheet` abstraction provides an optimized mechanism to declare all your styles and layout right along with the components that use them and apply them inline.

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

React Native is focused on changing the way view code is written.  For the rest, we look to the web for universal standards and polyfill those APIs where appropriate. You can use npm to install JavaScript libraries that work on top of the functionality baked into React Native, such as `XMLHttpRequest`, `window.requestAnimationFrame`, and `navigator.geolocation`.  We are working on expanding the available APIs, and are excited for the Open Source community to contribute as well.

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

## Extensibility

It is certainly possible to create a great app using React Native without writing a single line of native code, but React Native is also designed to be easily extended with custom native views and modules - that means you can reuse anything you've already built, and can import and use your favorite native libraries.  To create a simple module in iOS, create a new class that implements the `RCTBridgeModule` protocol, and add `RCT_EXPORT` to the function you want to make available in JavaScript.

```objc
// Objective-C

#import "RCTBridgeModule.h"

@interface MyCustomModule : NSObject <RCTBridgeModule>
@end

@implementation MyCustomModule
- (void)processString:(NSString *)input callback:(RCTResponseSenderBlock)callback
{
  RCT_EXPORT(); // available as NativeModules.MyCustomModule.processString
  callback(@[[input stringByReplacingOccurrencesOfString:@"Goodbye" withString:@"Hello"];]]);
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

Custom iOS views can be exposed by subclassing `RCTViewManager`, implementing a `-view` method, and exporting properties with the `RCT_EXPORT_VIEW_PROPERTY` macro.  Then a simple JavaScript file connects the dots.

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

Further documentation, tutorials, and more on the [React Native website](http://facebook.github.io/react-native/docs/getting-started.html).
