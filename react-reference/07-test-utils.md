# Test Utilities

- 英文原文：[https://reactjs.org/docs/test-utils.html](https://reactjs.org/docs/test-utils.html)

引入：

```javascript
import ReactTestUtils from 'react-dom/test-utils'; // ES6
var ReactTestUtils = require('react-dom/test-utils'); // ES5 with npm
```

---

## 概览

`ReactTestUtils` 使你可以轻松地在你选择的测试框架中测试 React 组件。在 Facebook 我们使用 [Jest](https://facebook.github.io/jest/) 进行无痛的 JavaScript 测试。通过 Jest 网站的 [React Tutorial](http://facebook.github.io/jest/docs/en/tutorial-react.html#content) 了解如何开始使用 Jest。

> **注意：**
> Airbnb 已经发布了一个名为 Enzyme 的测试实用程序，它可以很容易地断言，操作和遍历 React Compoennts 的输出。如果你决定使用单元测试实用程序与 Jest 或 任何其他测试运行器一起使用，则值得检查：[http://airbnb.io/enzyme/](http://airbnb.io/enzyme/)

- [Simulate](#simulate)

- [renderIntoDocument()](#renderintodocument)

- [mockComponent()](#mockcomponent)

- [isElement()](#iselement)

- [isElementOfType()](#iselementoftype)

- [isDOMComponent()](#isdomcomponent)

- [isCompositeComponent()](#iscompositecomponent)

- [isCompositeComponentWithType()](#iscompositecomponentwithtype)

- [findAllInRenderedTree()](#findallinrenderedtree)

- [scryRenderedDOMComponentsWithClass](#scryrendereddomcomponentswithclass)

- [findRenderedDOMComponentWithClass](#findrendereddomcomponentwithclass)

- [scryRenderedDOMComponentsWithTag](#scryrendereddomcomponentswithtag)

- [findRenderedDOMComponentWithTag](#findrendereddomcomponentwithtag)

- [scryRenderedComponentsWithType](#scryrenderedcomponentswithtype)

- [findRenderedComponentWithType](#findrenderedcomponentwithtype)

----

## API 文档参考

## Shallow rendering

在为 React 编写单元测试的时候，浅渲染可能会比较有帮助。浅渲染使你可以渲染“一级深度”的组件，并能够确定 render 方法返回的内容，而不用担心未实例化或渲染的子组件的行为。这不需要请求 DOM。

> **注意：**
> 浅渲染已经移动到了 `react-test-renderer/shallow`.
> [在参考页面上了解更多关于浅层渲染的内容](https://reactjs.org/docs/shallow-renderer.html)。


----

## 其他实用程序程序

### Simulate

```javascript
Simulate.{eventName}(
  element,
  [eventData]
)
```

使用可选的 `eventData` 事件数据在 DOM 节点上模拟事件分发。

`Simulate` 对 [React 的每个事件](https://reactjs.org/docs/events.html#supported-events) 都有一个方法。

**点击一个元素**

```javascript
// <button ref={(node) => this.button = node}>...</button>
const node = this.button;
ReactTestUtils.Simulate.click(node);
```

**更改输入字段的值，然后按下 ENTER 键**

```javascript
// <input ref={(node) => this.textInput = node} />
const node = this.textInput;
node.value = 'giraffe';
ReactTestUtils.Simulate.change(node);
ReactTestUtils.Simulate.keyDown(node, {key: "Enter", keyCode: 13, which: 13});
```

> **注意：**
> 你必须提供你在组件中使用的任何事件属性（例如，keyCode，等等），因为 React 不会为你创建任何这些属性。

----

### renderIntoDocument()

```javascript
renderIntoDocument()
```

将 React 元素渲染到文档中分离的 DOM 节点中。**这个功能需要一个 DOM**。

> **注意：**
> 在导入 React **之前**，你需要全局可用的 `window`,`window.document` 和 `window.document.createElement`。否则 React 会认为它不能访问 DOM，像 `setState` 这样的方法将无法工作。

----

### mockComponent()

```javascript
mockComponent(
  componentClass,
  [mockTagName]
)
```

将模拟的组件模块传递给此方法，以便使用有用的方法来扩充它，以便将其用作虚拟 React 组件。不像往常那样渲染，组件将变成一个简单的 `<div>`（或如果提供了 mockTagName 的话就是其他标签），它包含任何提供的子项。

> **注意：**
> `mockComponent()` 是一个传统的 API。相反我们建议使用[浅渲染](#shallow-rendering)或 [jest.mock()](https://facebook.github.io/jest/docs/en/tutorial-react-native.html#mock-native-modules-using-jestmock)


----

### isElement()

```javascript
isElement(element)
```

如果 `element` 是任何的 React 元素，则返回 `true`。

-----

### isElementOfType()

```javascript
isElementOfType(
  element,
  componentClass
)
```

如果 `element` 是类型是 React 的 `componentClass`(组件类)，则返回 `true`。

----

### isDOMComponent()

```javascript
isDOMComponent(instance)
```

如果 `instance` 是一个 DOM 组件（如 `<div>` 或 `<span>`），则返回 `true`。


---

### isCompositeComponent()

```javascript
isCompositeComponent(instance)
```

如果 `instance` 是用户自定义的组件（如 class 或 function），则返回 `true`。

----


### isCompositeComponentWithType()

```javascript
isCompositeComponentWithType(
  instance,
  componentClass
)
```

如果 `instance` 是 React 的 `componentClass` 类型，则返回 `true`。

----


### findAllInRenderedTree()

```javascript
findAllInRenderedTree(
  tree,
  test
)
```

遍历 `tree` 中的所有组件并且满足 `test(component)` 返回 `true`。这本身并没有什么用处，但它被用作其他测试应用的基础。

----

### scryRenderedDOMComponentsWithClass()

```javascript
scryRenderedDOMComponentsWithClass(
  tree,
  className
)
```

查找渲染树中所有的 DOM 元素，这些 DOM 元素是类名匹配 `className` 的 DOM 组件。

----

### findRenderedDOMComponentWithClass()

```javascript
findRenderedDOMComponentWithClass(
  tree,
  className
)
```
像 [scryRenderedDOMComponentsWithClass()](#scryrendereddomcomponentswithclass) 相似，但期望得到一个结果并返回那个结果，或者如果除了一个之外还有其他数目的匹配，则抛出异常。

----

### scryRenderedDOMComponentsWithTag()

```javascript
scryRenderedDOMComponentsWithTag(
  tree,
  tagName
)
```

在渲染的树中查找所有的 DOM 元素，这些 DOM 元素是 tag 名称匹配 `tagName` 的 DOM 组件。

----

### findRenderedDOMComponentWithTag()

```javascript
findRenderedDOMComponentWithTag(
  tree,
  tagName
)
```

像 [scryRenderedDOMComponentsWithTag()](#scryrendereddomcomponentswithtag) 一样，但期望得到一个结果并返回这个结果，或者如果除了一个之外还有其他数目的匹配，则抛出异常。

----

### scryRenderedComponentsWithType()

```javascript
scryRenderedComponentsWithType(
  tree,
  componentClass
)
```

查找 `instances` 的组件类型与 `componentClass` 相同。

----

### findRenderedComponentWithType()

```javascript
findRenderedComponentWithType(
  tree,
  componentClass
)
```

和 [scryRenderedComponentsWithType()](#scryrenderedcomponentswithtype) 相同，但期望得到一个结果并返回该结果，或者如果除了一个之外还有其他数目的匹配，则抛出异常。



