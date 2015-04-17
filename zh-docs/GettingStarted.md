---
id: getting-started
title: 快速开始
layout: docs
category: Quick Start
permalink: docs/getting-started.html
next: tutorial
---

## 需要（ Requirements ）

* 1、OS X - 该项目目前仅支持 iOS ， Xcode 仅在 Mac 系统上运行。
* 2、Xcode 新手？ 从 Mac 应用商店[下载它](https://developer.apple.com/xcode/downloads/)。
* 3、建议使用 [Homebrew](http://brew.sh/) 安装 node ， watchman ， 和 flow 。
* 4、`brew install node` 。 [node](https://nodejs.org/) 或者 [npm](https://docs.npmjs.com/) 新手？
* 5、`brew install watchman` 。 我们建议安装 [watchman](https://facebook.github.io/watchman/docs/install.html) ， 否则你可能遇到 node 文件监听 bug 。
* 6、`brew install flow` 。如果你想使用 [flow](http://www.flowtype.org) 。

## 快速开始

- `npm install -g react-native-cli`
- `react-native init AwesomeProject`

在新创建的文件夹 `AwesomeProject/` 中

- 打开 `AwesomeProject.xcodeproj` ，然后在 Xcode 中点击运行（ run ）。
- 在你选择的文本编辑器中打开 `index.ios.js` ，然后编辑一些行
- 在 iOS 模拟器中按下 cmd+R 键（[两次](http://openradar.appspot.com/19613391)）来重新加载应用，观察你做的变化！

恭喜你！你刚刚成功地运行和修改了你的第一个 React Native 应用。

