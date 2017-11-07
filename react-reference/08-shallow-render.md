# Shallow Render

- 英文原文：[https://reactjs.org/docs/shallow-renderer.html](https://reactjs.org/docs/shallow-renderer.html)

导入依赖：

```javascript
import ShallowRenderer from 'react-test-renderer/shallow'; // ES6
var ShallowRenderer = require('react-test-renderer/shallow'); // ES5 with npm
```

----

## 概览：

在为 React 编写单元测试时，浅渲染可能会有帮助。浅渲染使你可以渲染“一级深度”的组件，并确定 render 方法返回的内容，而不用担心未实例化或未渲染的子组件的行为。这不需要请求 DOM。

例如，如果你有下面的组件：

```javascript
function MyComponent() {
  return (
    <div>
      <span className="heading">Title</span>
      <Subcomponent foo="bar" />
    </div>
  );
}
```

那么你可以断言：

```javascript
import ShallowRenderer from 'react-test-renderer/shallow';

// in your test:
const renderer = new ShallowRenderer();
renderer.render(<MyComponent />);
const result = renderer.getRenderOutput();

expect(result.type).toBe('div');
expect(result.props.children).toEqual([
  <span className="heading">Title</span>,
  <Subcomponent foo="bar" />
]);
```

浅测试目前有一些限制，即不支持 refs。

> **注意：**
> 我们还建议检查 Enzyme's 的 [浅渲染 API](http://airbnb.io/enzyme/docs/api/shallow.html)。它在相同的功能上提供了更好的更高级别的 API。

----

## 文档参考

### shallowRenderer.render()

你可以将 shallowRenderer 视为渲染正在测试的组件的 “地点”，并从中提取组件的输出内容。

`shallowRenderer.render()` 类似于 [ReactDOM.render()](https://reactjs.org/docs/react-dom.html#render)，但是它不需要 DOM，只能渲染一个等级深度。这意味着你可以测试与他们的子组件实施方式相关的组件。

### shallowRenderer.getRenderOutput()

在调用了 `shallowRenderer.render()` 之后，可以使用 `shallowRenderer.getRenderOutput()` 来获取浅渲染的输出结果。

你可以开始断言输出结果的事实。


