# React Top-Level API

- 英文原文：[https://reactjs.org/docs/react-api.html](https://reactjs.org/docs/react-api.html)

`React` 是 React 库的入口点。如果你从 `<script>` 标签加载 React，则这些顶级 API 可以在 `React` 全局环境中使用。如果你使用 ES6 和 npm，你可以通过 `import React from 'react'` .如果你使用 npm 中使用 ES5，你可以通过 `var React = require('react')`。

----


## 概览

### Components（组件）

React 组件使你能够将 UI 拆分成独立的可重复使用的部分，并分别考虑每个部分。可以通过将 `React.Component` 或 `React.PureComponent` 进行子类化定义 React 组件。

- [React.Component](#reactcomponent)

- [React.PureComponent](#reactpurecomponent)

如果你不使用 ES6 的 class，你可以使用 `create-react-class` 模块。有关更多系想你，请参阅使用 [不使用 ES6 的 React](https://reactjs.org/docs/react-without-es6.html) 了解更多信息。


### 创建 React 元素

我们建议 [使用 JSX](https://reactjs.org/docs/introducing-jsx.html) 来描述你的 UI 应该是什么样子的。每个 JSX 元素只是用于调用 [React.createElement(#createelement)]() 的语法糖。如果使用 JSX ，通常不会直接调用以下方法。

- [createElement()](#createelement)

- [createFactory()](#createfactory)

有关更多的信息，请参阅 [不使用 JSX 的 React](https://reactjs.org/docs/react-without-jsx.html)

### 变换元素

`React` 同时提供一些其他的 API：

- [cloneElement()](#cloneelement)

- [isValidElement()](#isvalidelement)

- [React.Children](#reactchildren)

----

## API 参考

### React.Component

`React.Component` 是 [使用 ES6 class](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Classes) 定义的 React 组件的基类：

```javascript
class Greeting extends React.Component {
  render() {
    return <h1>Hello, {this.props.name}</h1>;
  }
}
```

请参阅 [React.Component API 参考](https://reactjs.org/docs/react-component.html)获取与基础 `React.Component`类相关的方法和属性列表。

----

### React.PureComponent

`React.PureComponent`和[React.Component](#reactcomponent)完全相同，但实现了与浅层 `prop` 和 `state` 比较的 [shouldComponentUpdate()](#shouldcomponentupdate)。


如果你的 React 组件的 `render()` 方法在给定相同的 prop 和 state 的情况下呈现相同的结果，则可以使用 `React.PureComponent` 在某些情况下提升性能。

> **注意：**
> `React.PureComponent` 的 `shouldComponentUpdate()` 只是比较浅的比较对象。如果这些包含复杂的数据结构，则可能会产生更深层次差异的假阴性结果。只有当你期望拥有简单的 prop 和 state 时才扩展 `PureComponent`，或者当你知道深层数据结构已经更改时使用 [forceUpdate](#forceupdate)。或者，考虑使用[不可变对象](https://facebook.github.io/immutable-js/)来进行嵌套数据的快速比较。

----

### cloneElement()

```javascript
React.cloneElement(
  element,
  [props],
  [...children]
)
```

使用一个 `element` 克隆并返回一个新的 React 元素作为起点。由此产生的元素具有组合起来的原始元素的 prop 和 新的 prop 。新的 子元素会取代现有的子元素。原始元素的 `key` 和 `ref` 会被保留。

`React.cloneElement()` 几乎等价于：

```html
<element.type {...element.props} {...props}>{children}</element.type>
```

然而，它也保留了 `ref`。这意味着，如果你得到了一个带 `ref` 的子元素，你不会意外的从你的父组件那里得到。你会在你的新的 element 上拿到相同的 `ref`。

这个 API 的引入主要是作为被废弃的 `React.addons.cloneWithProps()` 的替代方案。

----

### createFactory()

```javascript
React.createFactory(type)
```

返回一个产生给定类型的 React 元素的函数。像 [React.createElement()](#createElement)一样，type 参数可以是标签名称字符串（如 `div` 或 `span`），或者是一个 [React.Component](https://reactjs.org/docs/components-and-props.html) 类型（一个 class 或者是一个 function）。

这个帮助器被认为是遗留的问题，我们鼓励你直接使用 JSX 或者使用 `React.createElement()`。

----

### isValidElement()

```javascript
React.isValidElement(object)
```

验证一个对象是否是 React 元素。返回 `true` 或者是 `false`

----

### React.Children

`React.Children` 提供了用于处理 `this.props.children` 不透明数据结构的实用程序。


#### React.Children.map

```javascript
React.Children.map(children, function[(thisArg)])
```

对 `thisArg` 的每个直接 `children` 中的 `this` 调用一个函数。如果 `children` 是一个 keyed 片段（可遍历对象）或者是数据，它将被遍历：该函数永远不会被传递给容器对象。如果 `children` 是一个 `null` 或者是 `undefined`,返回 `null` 或者是 `undefined`，而不是一个数组。 

#### React.Children.forEach

```javascript
React.Children.forEach(children, function[(thisArg)])
```

像 [React.Children.map()](#reactchildrenmap) 一样，但不会返回数组。


#### React.Children.count

```javascript
React.Children.count(children)
```

返回 `children` 的组件的总数量，等于调用 `map` 或者是 `forEach` 的回调次数。

#### React.Children.only

```javascript
React.Children.only(children)
```

验证 `children` 只有一个孩子（一个 React 元素）并且返回它。否则，这个方法抛出错误。

> **注意：**
> `React.Children.only()` 不接受 [React.Children.map()](#reactchildrenmap) 的返回值，因为它是一个数组而不是一个 React 元素。

#### React.Children.toArray

```javascript
React.Children.toArray(children)
```

将 `children` 的每个子项的键的数组形式返回一个不透明的数据结构。如果你想在你的渲染方法中操作 children 的集合，这非常有用，特别是如果你在传递之前想重新排序或者分割 `this.props.children` 。

> **注意：**
> `React.Children.toArray()` 在展平 children 列表的时候更改 key。也就是说，`toArray` 给返回的数组中的每个键添加前缀，以便每个元素的键都被作用于包含它的输入数组。

