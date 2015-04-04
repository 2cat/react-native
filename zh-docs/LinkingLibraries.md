---
id: linking-libraries
title: 链接库（ Linking Libraries ）
layout: docs
category: Guides
permalink: docs/linking-libraries.html
next: debugging
---

不是每个应用都使用所有的本地功能，把支持所有本地功能的代码打包到发布包中将会增加其大小... 但是我们仍然想在需要的时候很简单地添加这些本地功能。

按照这个思路，我们暴露了很多这些本地功能作为独立的静态库。

对于大多数库来说，只需要简单地拖动两个文件，有时需要第三步，但是不会有第四步了。

_所有的我们和 React Native 一起打包的库都放在项目仓库根目录下的 `Libraries` 文件夹下。其中有些纯 JavaScript 代码，只需要使用 `require` 引入。其它库仍然依赖一些本地代码，此时就必须添加这些文件到应用中，否则应用将会在你尝试使用这个库的时候抛出错误。_

## 下面是链接包含本地代码的库的几个步骤

### 步骤 1

如果库包含本地代码，必定有一个 `.xcodeproj` 文件在库的文件夹下。
拖动这个文件到你 Xcode 中的项目（通常在 Xcode 中的 `Libaries` 组下面）；

![](http://facebook.github.io/react-native/img/AddToLibraries.png)

### 步骤 2

点击主项目文件（代表 `.xcodeproj` 的文件），选择 `Build Phases` ，从 `Products` 文件夹拖动静态库到 `Link Binary With Libraries` 。

![](http://facebook.github.io/react-native/img/AddToBuildPhases.png)

### 步骤 3

并不是每个库都需要这一步，你需要考虑的是：

_在编译的时候需要知道库的内容吗？_

也就是说，是在本地调用这个库还是只在 JavaScript 中？如果是在 JavaScript 中使用，那就接着往下看吧！

这一步对于我们和 React Native 一起打包的所有的库来说是不需要的，除了 `PushNotificationIOS` 和 `LinkingIOS` 。

比如说，对于 `PushNotificationIOS` 的情形，每次接收到新的推送提醒的时候，都必须要在 `AppDelegate` 中调用这个库的一个方法。

因此需要知道这个库的头文件内容。为了达到这个目的，必须打开项目文件，选择 `Build Settings` ，找到 `Header Search Paths` 。此处你应该包含库的路径（如果库在子目录中有相关的文件，记得`递归`包含，就像示例中的 `React` ）。

![](http://facebook.github.io/react-native/img/AddToSearchPaths.png)
