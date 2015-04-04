---
id: debugging
title: 调试
layout: docs
category: Guides
permalink: docs/debugging.html
next: testing
---

## 调试 React Native 应用
为了调试你的 react 应用的代码，按下列步骤做：

 1、在 iOS 模拟器里运行应用。
 2、按下 ```Command + D``` ，会打开一张网页，网页地址是 [http://localhost:8081/debugger-ui](http://localhost:8081/debugger-ui)。（现在仅支持 Chrome 浏览器）
 3、启用 [在捕获到异常的时候暂停（ Pause On Caught Exceptions ）](http://stackoverflow.com/questions/2233339/javascript-is-there-a-way-to-get-chrome-to-break-on-all-errors/17324511#17324511)，可以获得更好的调试体验。
 4、按下 ```Command + Option + I``` 来打开 Chrome 的开发者工具，或者通过 ```View``` -> ```Developer``` -> ```Developer Tools``` 打开。
 5、你现在应该可以像平常一样调试应用了。

### 可选的
给谷歌 Chrome 浏览器安装 [React Developer Tools](https://chrome.google.com/webstore/detail/react-developer-tools/fmkadmapgofadopljbjfkapdkoienihi?hl=en) 扩展。如果在开发者工具打开的时候选中 ```React``` 标签页，会让你能够查看到视图层级结构。

## 热加载
为了达到热加载的目的，按下列步骤做：

1、在 iOS 模拟器里运行应用。
2、按下 ```Control + Command + Z``` 。
3、你现在会看到 `Enable/Disable Live Reload` ， `Reload` 和 `Enable/Disable Debugging` 选项。
