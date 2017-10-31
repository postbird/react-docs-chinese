# Portals

- 英文原文：[https://reactjs.org/docs/portals.html][1]

Portals 提供了一种一流的方式去将子节点渲染到存在于父组件的 DOM 层次结构之外的 DOM 节点。

```javascript
ReactDOM.createPortal(child, container)
```

第一个参数（`child`）是任何能够[被渲染的 React 子组件][2]，比如一个节点元素（element），string，或者是 fragment。第二个参数（`container`）是一个 DOM 元素。

----

## Usage （用法）

通常，如果你的组件的 render 方法返回一个元素时，它作为最接近的父节点的子节点挂载到 DOM 中：

```javascript
render() {
  // React mounts a new div and renders the children into it
  return (
    <div>
      {this.props.children}
    </div>
  );
}
```

但是，有时候要把子节点插入 DOM 中的不同位置时，是有用的：

```javascript
render() {
  // React does *not* create a new div. It renders the children into `domNode`.
  // `domNode` is any valid DOM node, regardless of its location in the DOM.
  return ReactDOM.createPortal(
    this.props.children,
    domNode,
  );
}
```

使用 portals 的典型场景是如果一个父组件有一个 `overflow:hidden` 或者是 `z-index` 的样式，但是你需要子节点在视觉上 “break out” （打破）这个父容器，比如 对话框，选项卡或者提示工具等。

> **注意：** 需要记住的是，当使用 portals 的时候，你需要确保遵循正确的辅助功能指南。




[1]: https://reactjs.org/docs/portals.html
[2]: https://reactjs.org/docs/react-component.html#render