---
id: nativemodulesios
title: 本地模块（ iOS ）
layout: docs
category: Guides
permalink: docs/nativemodulesios.html
next: testing
---

有时应用需要访问平台 API ，并且 React Native 还没有提供相应的封装。可能你想重用一些已有的 Objective-C 或者 C++ 代码，而不是在 JavaScript 中重新实现。或者要写一些高性能的、多线程的代码，例如图片处理、网络处理、数据库操作以及界面渲染。

我们设计 React Native 是为了能够写真正的本地代码，充分利用平台的优势。这是一个更加高级的特性，我们不希望这个成为常规的开发流程的一部分，然而，这是必不可少的。如果 React Native 不支持你需要的某个本地特性，你应该自己实现。

本文是一篇更加高级的教程，展示如何构建一个本地模块。假设读者了解 Objective-C （ Swift 现在还不支持 ）和核心库（ Foundation, UIKit ）。

## iOS 日历模块示例

本教程将会使用 [iOS 日历 API](https://developer.apple.com/library/mac/documentation/DataManagement/Conceptual/EventKitProgGuide/Introduction/Introduction.html) 举例。我们想从 JavaScript 中能够访问 iOS 日历。

本地模块就是一个 Objective-C 类，该类实现了 `RCTBridgeModule` 协议。RCT 是 ReaCT 的简称。

```objective-c
// CalendarManager.h
#import "RCTBridgeModule.h"

@interface CalendarManager : NSObject <RCTBridgeModule>
@end
```

React Native 将不会暴露 `CalendarManager` 中的任何方法给 JavaScript ，除非明确说明。幸运地是，通过 `RCT_EXPORT` 可以非常简单地实现：

```objective-c
// CalendarManager.m
@implementation CalendarManager

- (void)addEventWithName:(NSString *)name location:(NSString *)location
{
  RCT_EXPORT();
  RCTLogInfo(@"Pretending to create an event %@ at %@", name, location);
}

@end
```

现在，在你的 JavaScript 文件中，可以像这样调用该方法：

```javascript
var CalendarManager = require('NativeModules').CalendarManager;
CalendarManager.addEventWithName('Birthday Party', '4 Privet Drive, Surrey');
```

注意，导出的方法名从 Objective-C 选择器的第一部分生成。有时这回导致生成一个不常用的 JavaScript 名字（比如示例中的那个名字）。你可以传递一个可选的参数给 `RCT_EXPORT` 来改变这个名字，比如：`RCT_EXPORT(addEvent)` 。

该方法的返回值应该总是 `void` 。 React Native 桥接是异步的，所以传递结果给 JavaScript 的唯一方式是使用回调函数，或者触发事件（见下文）。

## 参数类型

React Native 支持几种参数类型，可以从 JavaScript 代码传入到本地模块：

- string (`NSString`)
- number (`NSInteger`, `float`, `double`, `CGFloat`, `NSNumber`)
- boolean (`BOOL`, `NSNumber`)
- 该列表中任何数据类型组成的数组（ `NSArray` ）
- 映射（ `NSDictionary` ），键是字符串类型，值是该列表中的任何类型
- function (`RCTResponseSenderBlock`)

在我们的 `CalendarManager` 示例中，如果我们想传递事件日期给本地模块，我们必须要把它转换成字符串或者数字：

```objective-c
- (void)addEventWithName:(NSString *)name location:(NSString *)location date:(NSInteger)secondsSinceUnixEpoch
{
  RCT_EXPORT(addEvent);
  NSDate *date = [NSDate dateWithTimeIntervalSince1970:secondsSinceUnixEpoch];
}
```

随着 `CalendarManager.addEvent` 方法越来越复杂，参数的数量将会增加。某些参数可能是可选的。这时值得考虑改变一点 API 的形式，接受一个事件属性字典，就像这样：

```objective-c
- (void)addEventWithName:(NSString *)name details:(NSDictionary *)details
{
  RCT_EXPORT(addEvent);
  NSString *location = [RCTConvert NSString:details[@"location"]]; // ensure location is a string
  ...
}
```

在 JavaScript 中调用：

```javascript
CalendarManager.addEvent('Birthday Party', {
  location: '4 Privet Drive, Surrey',
  time: date.toTime(),
  description: '...'
})
```

> **注意**： 关于数组和映射
>
> React Native 不确保这些数据结构中的值的类型是正确的。你本地的模块可能希望得到一组字符串，但是如果 JavaScript 调用你的方法时传入一个包含数字和字符串的数组，将会得到带有 `NSNumber` 和 `NSString` 类型数据的 `NSArray` 数组。检查数组/映射中的值的类型（参考 [`RCTConvert`](https://github.com/facebook/react-native/blob/master/React/Base/RCTConvert.h) ）是开发者的责任。

# 回调

> **警告**
>
> 相对于其它部分，该部分内容更具有尝试性，关于回调，我们至今没有一套最佳实践。

本地模块也支持一种特殊的参数类型 - 回调函数。在大多数情况下用于将执行结果返回给 JavaScript 。

```objective-c
- (void)findEvents:(RCTResponseSenderBlock)callback
{
  RCT_EXPORT();
  NSArray *events = ...
  callback(@[[NSNull null], events]);
}
```

`RCTResponseSenderBlock` 仅接受一个参数 - JavaScript 回调函数，该函数需要传入一组参数。此处，我们采用 node 的约定，设置第一个参数为错误对象，剩下的为 Objective-C 函数的运行结果。

```javascript
CalendarManager.findEvents((error, events) => {
  if (error) {
    console.error(error);
  } else {
    this.setState({events: events});
  }
})
```

本地模块应该仅调用一次回调函数。但是也可以暂存下回调函数在将来调用。这种方式常用于包装需要代理的 iOS 接口。参考 [`RCTAlertManager`](https://github.com/facebook/react-native/blob/master/React/Modules/RCTAlertManager.m) 。

如果你想传递包含错误信息的对象给 JavaScript ，使用 `RCTUtils.h`](https://github.com/facebook/react-native/blob/master/React/Base/RCTUtils.h) 中的 `RCTMakeError` 。

## 实现本地模块

不要假定本地模块运行在哪个线程。 React Native 在一个独立的序列的 GCD 队列中调用本地模块，但这是一个实现细节，以后可能会改变。如果本地模块需要调用仅在主线程中执行的 iOS API，应该把该操作放到主队列计划任务中：


```objective-c
- (void)addEventWithName:(NSString *)name callback:(RCTResponseSenderBlock)callback
{
  RCT_EXPORT(addEvent);
  dispatch_async(dispatch_get_main_queue(), ^{
    // Call iOS API on main thread
    ...
    // You can invoke callback from any thread/queue
    callback(@[...]);
  });
}
```

同样，如果某个操作需要耗费很长时间完成，不要因此阻塞本地模块。使用 `dispatch_async` 把耗时的操作放到后台任务队列中是一个好办法。

## 导出常量

本地模块可以导出常量，运行的时候可以在 JavaScript 中马上使用。这对于导出一些初始化的数据非常有用，不再需要 Objecttive-C 和 JavaScript 的一次双向调用。

```objective-c
- (NSDictionary *)constantsToExport
{
  return @{ @"firstDayOfTheWeek": @"Monday" };
}
```

JavaScript 可以马上使用这个值：

```javascript
console.log(CalendarManager.firstDayOfTheWeek);
```

注意，该常量仅在初始化的时候导出，所以如果在运行时改变 `constantsToExport` 的值，将不会影响 JavaScript 环境。


## 发送事件给 JavaScript

本地模块可以发送事件信号给 JavaScript ，而不是直接调用。最简单的方式就是使用 `eventDispatcher` ：

```objective-c
- (void)calendarEventReminderReceived:(NSNotification *)notification
{
  NSString *eventName = notification.userInfo[@"name"];
  [self.bridge.eventDispatcher sendAppEventWithName:@"EventReminder"
                                               body:@{@"name": eventName}];
}
```

JavaScript 代码可以监听这些事件：

```javascript
var subscription = DeviceEventEmitter.addListener(
  'EventReminder',
  (reminder) => console.log(reminder.name)
);
...
// 别忘了取消监听
subscription.remove();
```
更多关于发送事件给 JavaScript 的示例，请参考 [`RCTLocationObserver`](https://github.com/facebook/react-native/blob/master/Libraries/Geolocation/RCTLocationObserver.m) 。
