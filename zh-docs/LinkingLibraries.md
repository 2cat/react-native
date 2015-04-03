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

点击主项目文件（代表 `.xcodeproj` 的文件），选择 `Build Phases`
Click on your main project file (the one that represents the `.xcodeproj`)
select `Build Phases` and drag the static library from the `Products` folder
insed the Library you are importing to `Link Binary With Libraries`

![](/react-native/img/AddToBuildPhases.png)

### Step 3

Not every library will need this step, what you need to consider is:

_Do I need to know the contents of the library at compile time?_

What that means is, are you using this library on the native site or just in
JavaScript? If you are just using it in JavaScript, you are good to go!


This step is not necessary for all libraries that we ship we React Native but
`PushNotificationIOS` and `LinkingIOS`.

In the case of the `PushNotificationIOS` for example, you have to call a method
on the library from your `AppDelegate` every time a new push notifiation is
received.

For that we need to know the library's headers. To achieve that you have to go
to your project's file, select `Build Settings` and search for `Header Search
Paths`. There you should include the path to you library (if it has relevant
files on subdirectories remember to make it `recursive`, like `React` on the
example).

![](/react-native/img/AddToSearchPaths.png)
