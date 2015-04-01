---
id: nativemodulesios
title: 本地模块 (iOS)
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

在我们的 `CalendarManager` 示例中，如果我们想传递事件日期给本地模块，我们必须要把它转换成字符串或者数字
In our `CalendarManager` example, if we want to pass event date to native, we have to convert it to a string or a number:

```objective-c
- (void)addEventWithName:(NSString *)name location:(NSString *)location date:(NSInteger)secondsSinceUnixEpoch
{
  RCT_EXPORT(addEvent);
  NSDate *date = [NSDate dateWithTimeIntervalSince1970:secondsSinceUnixEpoch];
}
```

As `CalendarManager.addEvent` method gets more and more complex, the number of arguments will grow. Some of them might be optional. In this case it's worth considering changing the API a little bit to accept a dictionary of event attributes, like this:

```objective-c
- (void)addEventWithName:(NSString *)name details:(NSDictionary *)details
{
  RCT_EXPORT(addEvent);
  NSString *location = [RCTConvert NSString:details[@"location"]]; // ensure location is a string
  ...
}
```

and call it from JavaScript:

```javascript
CalendarManager.addEvent('Birthday Party', {
  location: '4 Privet Drive, Surrey',
  time: date.toTime(),
  description: '...'
})
```

> **NOTE**: About array and map
>
> React Native doesn't provide any guarantees about the types of values in these structures. Your native module might expect array of strings, but if JavaScript calls your method with an array that contains number and string you'll get `NSArray` with `NSNumber` and `NSString`. It's developer's responsibility to check array/map values types (see [`RCTConvert`](https://github.com/facebook/react-native/blob/master/React/Base/RCTConvert.h) for helper methods).

# Callbacks

> **WARNING**
>
> This section is even more experimental than others, we don't have a set of best practices around callbacks yet.

Native module also supports a special kind of argument - callback. In most cases it is used to provide function call result to JavaScript.

```objective-c
- (void)findEvents:(RCTResponseSenderBlock)callback
{
  RCT_EXPORT();
  NSArray *events = ...
  callback(@[[NSNull null], events]);
}
```

`RCTResponseSenderBlock` accepts only one argument - array of arguments to pass to JavaScript callback. In this case we use node's convention to set first argument to error and the rest - to the result of the function.

```javascript
CalendarManager.findEvents((error, events) => {
  if (error) {
    console.error(error);
  } else {
    this.setState({events: events});
  }
})
```

Native module is supposed to invoke callback only once. It can, however, store the callback as an ivar and invoke it later. This pattern is often used to wrap iOS APIs that require delegate. See [`RCTAlertManager`](https://github.com/facebook/react-native/blob/master/React/Modules/RCTAlertManager.m).

If you want to pass error-like object to JavaScript, use `RCTMakeError` from [`RCTUtils.h`](https://github.com/facebook/react-native/blob/master/React/Base/RCTUtils.h).

## Implementing native module

The native module should not have any assumptions about what thread it is being called on. React Native invokes native modules methods on a separate serial GCD queue, but this is an implementation detail and might change. If the native module needs to call main-thread-only iOS API, it should schedule the operation on the main queue:


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

The same way if the operation can take a long time to complete, the native module should not block. It is a good idea to use `dispatch_async` to schedule expensive work on background queue.

## Exporting constants

Native module can export constants that are instantly available to JavaScript at runtime. This is useful to export some initial data that would otherwise require a bridge round-trip.

```objective-c
- (NSDictionary *)constantsToExport
{
  return @{ @"firstDayOfTheWeek": @"Monday" };
}
```

JavaScript can use this value right away:

```javascript
console.log(CalendarManager.firstDayOfTheWeek);
```

Note that the constants are exported only at initialization time, so if you change `constantsToExport` value at runtime it won't affect JavaScript environment.


## Sending events to JavaScript

The native module can signal events to JavaScript without being invoked directly. The easiest way to do this is to use `eventDispatcher`:

```objective-c
- (void)calendarEventReminderReceived:(NSNotification *)notification
{
  NSString *eventName = notification.userInfo[@"name"];
  [self.bridge.eventDispatcher sendAppEventWithName:@"EventReminder"
                                               body:@{@"name": eventName}];
}
```

JavaScript code can subscribe to these events:

```javascript
var subscription = DeviceEventEmitter.addListener(
  'EventReminder',
  (reminder) => console.log(reminder.name)
);
...
// Don't forget to unsubscribe
subscription.remove();
```
For more examples of sending events to JavaScript, see [`RCTLocationObserver`](https://github.com/facebook/react-native/blob/master/Libraries/Geolocation/RCTLocationObserver.m).
