# Web Components

- 英文原文：[https://reactjs.org/docs/web-components.html][1]

React 和 [Web Components(Web组件)][2]  用来解决不同的问题， Web 组件为可重用的组件提供了强大的封装，而 React 则提供了一个声明库，可以使 DOM 和数据保持同步。

这两者是互补的，作为开发人员，你可以在 Web 组件中自由的使用 React，或者在 React 中使用 Web 组件。


大多数使用 React 的人不适用 Web 组件，但是你可能会希望去使用，尤其是如果你使用 Web 组件开发的第三方 UI 组件。

----

## 在 React 中使用 Web 组件

```javascript
class HelloMessage extends React.Component {
  render() {
    return <div>Hello <x-search>{this.props.name}</x-search>!</div>;
  }
}
```

> **注意：** Web 组件通常暴露一个必须的 API。例如，视频的 Web 组件可能会提供 `play()` 和 `pause()` 方法，要访问 Web 组件的 API，你需要通过 ref 去直接与 DOM 节点进行交互。 如果你使用第三方 Web 组件，最佳的解决方案就是 编写一个 React 组件，将 Web 组件进行二次封装。

> Web 组件发出的事件可能无法通过 React 渲染树进行正确的传递，你需要手动的将事件处理绑定到你的 React 组件中


一个常见的混合方式是 Web 组件使用 "class" 而不是使用 "className"。

```javascript
function BrickFlipbox() {
  return (
    <brick-flipbox class="demo">
      <div>front</div>
      <div>back</div>
    </brick-flipbox>
  );
}
```

----

## 在 Web 组件中使用 React


```javascript
class XSearch extends HTMLElement {
  connectedCallback() {
    const mountPoint = document.createElement('span');
    this.attachShadow({ mode: 'open' }).appendChild(mountPoint);

    const name = this.getAttribute('name');
    const url = 'https://www.google.com/search?q=' + encodeURIComponent(name);
    ReactDOM.render(<a href={url}>{name}</a>, mountPoint);
  }
}
customElements.define('x-search', XSearch);
```

> **注意：** 如果你使用 `Babel` 转换class，上面的代码**不会起作用**， [查看这个问题的讨论][3]







[1]: https://reactjs.org/docs/web-components.html
[2]: https://developer.mozilla.org/en-US/docs/Web/Web_Components
[3]: https://github.com/w3c/webcomponents/issues/587



