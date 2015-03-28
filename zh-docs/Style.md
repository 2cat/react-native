---
id: style
title: 样式
layout: docs
category: Guides
permalink: docs/style.html
next: gesture-responder-system
---

React Native 没有实现 CSS ，但是依赖 JavaScript 来设置应用的样式。这是一个有争议的决定，你可以查看这个幻灯片来了解背后的基本原理。

<script async class="speakerdeck-embed" data-id="2e15908049bb013230960224c1b4b8bd" data-ratio="2" src="//speakerdeck.com/assets/embed.js"></script>

## 声明样式

在 React Native 中声明样式的方式如下：

```javascript
var styles = StyleSheet.create({
  base: {
    width: 38,
    height: 38,
  },
  background: {
    backgroundColor: '#222222',
  },
  active: {
    borderWidth: 2,
    borderColor: '#00ff00',
  },
});
```

`StyleSheet.create` 构造函数是可选的，但是它具备一些重要的优点。该函数将传入的值转换为整数，整数值指向一个内部的映射表，通过这个机制可以确保传入的值是不可改变的。通过把样式表创建放在源码文件结束位置，也可以确保样式对象在应用中只创建了一次而不是每次渲染的时候都创建一次。

所有的属性名和属性值是 web 样式的一个子集。对于布局， React Native 实现了 [Flexbox](/react-native/docs/flexbox.html)。

## 使用样式

所有的核心组件都接受一个样式

```javascript
<Text style={styles.base} />
<View style={styles.background} />
```

也接受一组样式

```javascript
<View style={[styles.base, styles.background]} />
```

这和 `Object.assign` 的行为是一样的：为了避免值的冲突，右侧的数组元素将会有更高的优先级，但是像 `false` 、 `undefined` 和 `null` 这种假值将会被忽略。一种通常的做法是基于某些条件运算符添加样式。

```javascript
<View style={[styles.base, this.state.active && styles.active]} />
```

最后，如果实在需要，你也可以在渲染的时候创建样式对象，但是非常不推荐这样做。把这些对象放在样式数组的最后。

```javascript
<View
  style={[styles.base, {
    width: this.state.width,
    height: this.state.width * this.state.aspectRatio
  }]}
/>
```

## 传递样式参数

为了让组件能指定子组件的样式，可以传递样式参数。使用 `View.propTypes.style` 和 `Text.propTypes.style` 确保传入的是样式数据。

```javascript
var List = React.createClass({
  propTypes: {
    style: View.propTypes.style,
    elementStyle: View.propTypes.style,
  },
  render: function() {
    return (
      <View style={this.props.style}>
        {elements.map((element) =>
          <View style={[styles.element, this.props.elementStyle]} />
        )}
      </View>
    );
  }
});

// ... in another file ...
<List style={styles.list} elementStyle={styles.listElement} />
```
