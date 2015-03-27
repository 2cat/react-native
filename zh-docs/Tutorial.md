---
id: tutorial
title: 教程
layout: docs
category: Quick Start
permalink: docs/tutorial.html
next: videos
---

## 前言

该教程目的是让你快速地开始使用 React Native 开发 iOS 应用。如果你想知道 React Native 是什么，为什么 Facebook 构造这个工具，这篇[博文](https://code.facebook.com/posts/1014532261909640/react-native-bringing-modern-web-techniques-to-mobile/)解释了这些问题。

我们假设你已经有用 React 开发网站的经验了。如果没有经验，你可以在[ React 网站](http://facebook.github.io/react/)学习它。


## 环境设置

React Native 需要 OSX ， Xcode ， Homebrew ， node ， npm ，和 [watchman](https://facebook.github.io/watchman/docs/install.html) 。 [Flow](https://github.com/facebook/flow) 是可选的。

在安装了这些依赖之后，有两个简单的命令用于设置一个 React Native 项目的所有开发需要的内容。

1、`npm install -g react-native-cli`

    react-native-cli 是一个完成余下设置工作的命令接口。可以通过 npm 安装。这将会安装一个 `react-native` 命令到你的终端中。该命令你只需要成功运行一次。

2、`react-native init AwesomeProject`

    该命令获取 React Native 源码和相关依赖，然后创建一个新的 Xcode 项目到 `AwesomeProject/AwesomeProject.xcodeproj` 。


## 开发

你现在可以在 Xcode 中打开这个新项目（ `AwesomeProject/AwesomeProject.xcodeproj` ），简单地通过 cmd+R 构建和运行。这么做的同时也会启动一个 node 服务器，用于开启实时代码加载功能。通过这个功能，你可以按下 cmd+R 在模拟器中看到你做的变化，而不用在 Xcode 中重新编译。

在这篇教程中，我们将会构建一个电影应用的简单版本，该应用从电影院获取 25 条电影数据，然后展示在 ListView 中。


### Hello World

`react-native init` 将会复制 `Examples/SampleProject` 到你命名项目的地方，在这里是 AwesomeProject 。这是一个简单的 hello world 应用。你可以编辑 `index.ios.js` ，然后在模拟器中按下 cmd+R 来观察变化。


### 原型数据（ Mocking data ）

在我们书写获取真正的 Rotten Tomatoes 数据之前，让我们构造一些数据模型。在 Facebook ，我们通常在 JS 文件的顶部 require 的下面声明常量，但是对于以下常量，随便放在什么地方都可以。 `index.ios.js` 中：

```javascript
var MOCKED_MOVIES_DATA = [
  {title: 'Title', year: '2015', posters: {thumbnail: 'http://i.imgur.com/UePbdph.jpg'}},
];
```


### 渲染一条电影数据

我们将会渲染电影的标题，年份，以及缩略图。因为缩略图是 React Native 中的一个图片组件，在下面的 React 依赖列表中添加 Image 。

```javascript
var {
  AppRegistry,
  Image,
  StyleSheet,
  Text,
  View,
} = React;
```

现在改变 render 函数，以便于渲染前面提到的电影数据，而不是显示 hello world。

```javascript
  render: function() {
    var movie = MOCKED_MOVIES_DATA[0];
    return (
      <View style={styles.container}>
        <Text>{movie.title}</Text>
        <Text>{movie.year}</Text>
        <Image source={{uri: movie.posters.thumbnail}} />
      </View>
    );
  }
```

按下 cmd+R ，你应该看到 “Title” 位于 “2015” 之上。注意 Image 并没有渲染出任何内容，因为我们没有指定想渲染的图片元素的宽度和高度，这可以通过样式（ styles ）来指定。在我们改变样式的同时，清除不再使用的样式。

```javascript
var styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
    backgroundColor: '#F5FCFF',
  },
  thumbnail: {
    width: 53,
    height: 81,
  },
});
```

最后，我们需要应用样式到 Image 组件上：

```javascript
        <Image
          source={{uri: movie.posters.thumbnail}}
          style={styles.thumbnail}
        />
```

按下 cmd+R ，然后图片应该会渲染出来了。

<div class="tutorial-mock">
  <img src="/react-native/img/TutorialMock.png" />
</div>


### 添加一些样式

太棒了，已经渲染出来了数据。现在让它看起来更漂亮。我想把文本放在图片的右侧，把标题字体放大并保持居中：

```
+---------------------------------+
|+-------++----------------------+|
||       ||        Title         ||
|| Image ||                      ||
||       ||        Year          ||
|+-------++----------------------+|
+---------------------------------+
```

我们将需要添加另一个容器，用于在水平布局的组件中放置垂直布局的组件。

```javascript
      return (
        <View style={styles.container}>
          <Image
            source={{uri: movie.posters.thumbnail}}
            style={styles.thumbnail}
          />
          <View style={styles.rightContainer}>
            <Text style={styles.title}>{movie.title}</Text>
            <Text style={styles.year}>{movie.year}</Text>
          </View>
        </View>
      );
```

没有改变太多代码，在文本附近添加了一个容器，然后移到 Image 之后（因为要放在 Image 的右侧）。让我们看看样式变成什么样子了：

```javascript
  container: {
    flex: 1,
    flexDirection: 'row',
    justifyContent: 'center',
    alignItems: 'center',
    backgroundColor: '#F5FCFF',
  },
```

我们使用弹性盒子（ FlexBox ）来布局 - 参考[这个伟大的指南](https://css-tricks.com/snippets/css/a-guide-to-flexbox/)来学习弹性盒子（ FlexBox ）。

在上面的代码片段中，我们简单地添加了 `flexDirection: 'row'` ，这使得我们的主容器的子级布局在水平方向上而不是垂直方向。

现在，给 JS `style` 对象添加另一个样式：

```javascript
  rightContainer: {
    flex: 1,
  },
```

这意味着 `rightContainer` 填充父容器中 Image 占用后的空间。如果还不理解，可以给 `rightContainer` 添加一个 `backgroundColor` 样式，然后尝试移除 `flex: 1` 。你将会看到这引起容器的尺寸变化到刚好包住子级的大小。

给文本添加样式相当直接：

```javascript
  title: {
    fontSize: 20,
    marginBottom: 8,
    textAlign: 'center',
  },
  year: {
    textAlign: 'center',
  },
```

继续前进，按下 cmd+R ，然后你将看到更新后的界面。

<div class="tutorial-mock">
  <img src="/react-native/img/TutorialStyledMock.png" />
</div>

### 获取真正的数据

从 Rotten Tomatoes 的 API 中获取数据对于学习 React Native 来说并不重要，因此不用担心，快速地阅读这一节。

将下列常量添加到文件的顶部（通常位于 require 下面），创建用于获取数据的 REQUEST_URL 。

```javascript
var API_KEY = '7waqfqbprs7pajbz28mqf6vz';
var API_URL = 'http://api.rottentomatoes.com/api/public/v1.0/lists/movies/in_theaters.json';
var PAGE_SIZE = 25;
var PARAMS = '?apikey=' + API_KEY + '&page_limit=' + PAGE_SIZE;
var REQUEST_URL = API_URL + PARAMS;
```

给应用添加一些初始化 state ，以便于可以通过检查 `this.state === null` 来辨别当前是否加载了电影数据。当请求返回的时候，我们可以使用 `this.setState({moves: moviesData})` 来设置该数据。添加下列代码到 render 函数的上面， React 类的里面。

```javascript
  getInitialState: function() {
    return {
      movies: null,
    };
  },
```

我们想要在组件完成加载之后发送请求。 `componentDidMount` 是一个 React 将会调用一次的函数，就在组件加载之后被调用。

```javascript
  componentDidMount: function() {
    this.fetchData();
  },
```

现在，添加上面使用到的 `fetchData` 函数到主组件中。该方法将负责获取数据。你只需要在解析了 promise 链之后调用 `this.setState({movies: data})` ，因为 React 的工作方式就是 `setState` 实际上出发一个重新渲染的事件，然后 render 函数将会注意到 `this.state.movies` 不再是 `null` 。注意，在 promise 链的最后调用了 `done()` - 总是确保调用 `done()` ，否则抛出的错误将无法捕获。

```javascript
  fetchData: function() {
    fetch(REQUEST_URL)
      .then((response) => response.json())
      .then((responseData) => {
        this.setState({
          movies: responseData.movies,
        });
      })
      .done();
  },
```

现在修改 render 函数，如果没有电影数据的话就渲染一个加载视图，否则渲染第一条电影数据。

```javascript
  render: function() {
    if (!this.state.movies) {
      return this.renderLoadingView();
    }

    var movie = this.state.movies[0];
    return this.renderMovie(movie);
  },

  renderLoadingView: function() {
    return (
      <View style={styles.container}>
        <Text>
          Loading movies...
        </Text>
      </View>
    );
  },

  renderMovie: function(movie) {
    return (
      <View style={styles.container}>
        <Image
          source={{uri: movie.posters.thumbnail}}
          style={styles.thumbnail}
        />
        <View style={styles.rightContainer}>
          <Text style={styles.title}>{movie.title}</Text>
          <Text style={styles.year}>{movie.year}</Text>
        </View>
      </View>
    );
  },
```

现在按下 cmd+R ，然后你应该会看见“ Loading movies ”。请求返回之后，将会渲染出从 Rotten Tomatoes 取到的第一条电影数据。

<div class="tutorial-mock">
  <img src="/react-native/img/TutorialSingleFetched.png" />
</div>

## ListView

现在让我们修改该应用，在一个 `ListView` 中渲染所有的数据，而不是仅仅渲染第一条电影数据。

为什么单独的 `ListView` 比渲染出所有这些元素或者把这些元素放在一个 `ScrollView` 里面要好？尽管 React 很快，但是渲染一个可能无止境的元素列表还是会变得很慢。 `ListView` 规划好要渲染的视图，以便于仅需要展示在可视区的元素，对于那些已经渲染了但是位于可视区域之外的元素会从本地的视图树中移除。

首先的首先：在文件顶部添加引入 `ListView` 的代码。

```javascript
var {
  AppRegistry,
  Image,
  ListView,
  StyleSheet,
  Text,
  View,
} = React;
```

现在修改 render 函数，以便于一旦获取到数据就在 ListView 中渲染出来而不是显示一条电影数据。

```javascript
  render: function() {
    if (!this.state.loaded) {
      return this.renderLoadingView();
    }

    return (
      <ListView
        dataSource={this.state.dataSource}
        renderRow={this.renderMovie}
        style={styles.listView}
      />
    );
  },
```

`DataSource` 是一个接口， `ListView` 用来判断哪一行视图在更新过程中发生了改变。

你将注意到我们使用了 `this.state` 中的 `dataSource` 。下一步就是添加一个从 `getInitialState` 返回的空的 `dataSource` 到这个对象中。同时，既然我们把数据存储在了 `dataSource` 之中，我们不应该再使用 `this.state.movies` ，避免存放了两份同样的数据。我们可以使用 state 上的布尔属性（ `this.state.loaded` ）来判断数据获取是否完成。

```javascript
  getInitialState: function() {
    return {
      dataSource: new ListView.DataSource({
        rowHasChanged: (row1, row2) => row1 !== row2,
      }),
      loaded: false,
    };
  },
```

下面是修改了的 `fetchData` 方法，相应地更新 state ：

```javascript
  fetchData: function() {
    fetch(REQUEST_URL)
      .then((response) => response.json())
      .then((responseData) => {
        this.setState({
          dataSource: this.state.dataSource.cloneWithRows(responseData.movies),
          loaded: true,
        });
      })
      .done();
  },
```

最后，我们把 `ListView` 组件需要的样式添加到 `styles` JS 对象中：
```javascript
  listView: {
    paddingTop: 20,
    backgroundColor: '#F5FCFF',
  },
```

下面是最终的结果：

<div class="tutorial-mock">
  <img src="/react-native/img/TutorialFinal.png" />
</div>

要让示例成为一个功能完备的应用，还有很多工作要做，比如这些功能：添加导航，搜索，无穷滚动加载，等等。访问[电影实例](https://github.com/facebook/react-native/tree/master/Examples/Movies)，实现了上述所有功能。


### 最终的源码

```javascript
/**
 * Sample React Native App
 * https://github.com/facebook/react-native
 */
'use strict';

var React = require('react-native');
var {
  AppRegistry,
  Image,
  ListView,
  StyleSheet,
  Text,
  View,
} = React;

var API_KEY = '7waqfqbprs7pajbz28mqf6vz';
var API_URL = 'http://api.rottentomatoes.com/api/public/v1.0/lists/movies/in_theaters.json';
var PAGE_SIZE = 25;
var PARAMS = '?apikey=' + API_KEY + '&page_limit=' + PAGE_SIZE;
var REQUEST_URL = API_URL + PARAMS;

var AwesomeProject = React.createClass({
  getInitialState: function() {
    return {
      dataSource: new ListView.DataSource({
        rowHasChanged: (row1, row2) => row1 !== row2,
      }),
      loaded: false,
    };
  },

  componentDidMount: function() {
    this.fetchData();
  },

  fetchData: function() {
    fetch(REQUEST_URL)
      .then((response) => response.json())
      .then((responseData) => {
        this.setState({
          dataSource: this.state.dataSource.cloneWithRows(responseData.movies),
          loaded: true,
        });
      })
      .done();
  },

  render: function() {
    if (!this.state.loaded) {
      return this.renderLoadingView();
    }

    return (
      <ListView
        dataSource={this.state.dataSource}
        renderRow={this.renderMovie}
        style={styles.listView}
      />
    );
  },

  renderLoadingView: function() {
    return (
      <View style={styles.container}>
        <Text>
          Loading movies...
        </Text>
      </View>
    );
  },

  renderMovie: function(movie) {
    return (
      <View style={styles.container}>
        <Image
          source={{uri: movie.posters.thumbnail}}
          style={styles.thumbnail}
        />
        <View style={styles.rightContainer}>
          <Text style={styles.title}>{movie.title}</Text>
          <Text style={styles.year}>{movie.year}</Text>
        </View>
      </View>
    );
  },
});

var styles = StyleSheet.create({
  container: {
    flex: 1,
    flexDirection: 'row',
    justifyContent: 'center',
    alignItems: 'center',
    backgroundColor: '#F5FCFF',
  },
  rightContainer: {
    flex: 1,
  },
  title: {
    fontSize: 20,
    marginBottom: 8,
    textAlign: 'center',
  },
  year: {
    textAlign: 'center',
  },
  thumbnail: {
    width: 53,
    height: 81,
  },
  listView: {
    paddingTop: 20,
    backgroundColor: '#F5FCFF',
  },
});

AppRegistry.registerComponent('AwesomeProject', () => AwesomeProject);
```
