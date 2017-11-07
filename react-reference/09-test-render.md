# Test Renderer

- 英文原文：[https://reactjs.org/docs/test-renderer.html](https://reactjs.org/docs/test-renderer.html)

引入依赖：

```javascript
import TestRenderer from 'react-test-renderer'; // ES6
const TestRenderer = require('react-test-renderer'); // ES5 with npm
```

----

## 概览

这个包提供了一个 React 渲染器，可以用来渲染 React 组件到纯 JavaScript 对象，而不依赖 DOM 或本地的 mobile 环境。

从本质上来说，这个包可以很容易地抓取 React DOM 或 React Native 组件呈现的平台视图层次结构（类似 DOM 树）的快照，而无需使用浏览器或 [jsdom](https://github.com/tmpvar/jsdom)

例如：

```javascript
import TestRenderer from 'react-test-renderer';

function Link(props) {
  return <a href={props.page}>{props.children}</a>;
}

const testRenderer = TestRenderer.create(
  <Link page="https://www.facebook.com/">Facebook</Link>
);

console.log(testRenderer.toJSON());
// { type: 'a',
//   props: { href: 'https://www.facebook.com/' },
//   children: [ 'Facebook' ] }
```

你可以使用 Jest 的快照测试同能自动将 JSON 树的副本保存到文件中，并检查测试是否没有更改：[了解更多信息](http://facebook.github.io/jest/blog/2016/07/27/jest-14.html)

你也可以遍历输出来查找特定的节点并对它们进行断言。

```javascript
import TestRenderer from 'react-test-renderer';

function MyComponent() {
  return (
    <div>
      <SubComponent foo="bar" />
      <p className="my">Hello</p>
    </div>
  )
}

function SubComponent() {
  return (
    <p className="sub">Sub</p>
  );
}

const testRenderer = TestRenderer.create(<MyComponent />);
const testInstance = testRenderer.root;

expect(testInstance.findByType(SubComponent).props.foo).toBe('bar');
expect(testInstance.findByProps({className: "sub"}).children).toEqual(['Sub']);
```

### TestRender

- [TestRenderer.create()](#testrenderercreate)

### TestRenderer instance

- [testRenderer.toJSON()](#testrenderertojson)

- [testRenderer.toTree()](#testrenderertotree)

- [testRenderer.update()](#testrendererupdate)

- [testRenderer.unmount()](#testrendererunmount)

- [testRenderer.getInstance()](#testrenderergetinstance)

- [testRenderer.root](#testrendererroot)

### TestInstance

- [testInstance.find()](#testinstancefind)

- [testInstance.findByType](#testinstancefindbytype)

- [testInstance.findByProps](#testinstancefindbyprops)

- [testInstance.findAll](#testinstancefindall)

- [testInstance.findAllByType](#testinstancefindallbytype)

- [testInstance.findAllByProps](#testinstancefindallbyprops)

- [testInstance.instance](#testinstanceinstance)

- [testInstance.type](#testinstancetype)

- [testInstance.props](#testinstanceprops)

- [testInstance.parent](#testinstanceparent)

- [testInstance.children](#testinstancechildren)

----

## API 文档参考

### TestRenderer.create()

```javascript
TestRednerer.create(element, options);
```

用传递的 React 元素创建一个 `TestRenderer` 实例。它不适用真正的 DOM，但它仍然完全呈现组件树到内存中，所以你可以做出断言。返回的实例具有以下方法和属性。

### testRenderer.toJSON()

```javascript
testRenderer.toJSON()
```

返回表示渲染树的对象。该树只包含特定于平台的节点，如 `<div>` 或 `<View>` 及其 props，但是不包含任何用户编写的组件。这对于[快照测试](http://facebook.github.io/jest/docs/en/snapshot-testing.html#snapshot-testing-with-jest)非常方便。

### testRenderer.toTree()

```javascript
testRenderer.toTree()
```

返回表示渲染树的对象。与 `toJSON()` 不同，表示比 `toJSON()` 提供的表示更加详细，并包括用户编写的组件。你可能不需要这个方法，除非你在测试渲染器之上写了自己的断言库。

### testRenderer.update()

```javascript
testRenderer.update(element)
```

用新的根元素重新渲染内存树。这在根上模拟 React 更新。如果新元素与前一个元素具有相同的类型和键，则树会被更新。否则，它将重新安装一颗新的树。

### testRenderer.unmount()

```javascript
testRenderer.unmount()
```

卸载内存树，触发适当的生命周期事件。

### testRenderer.getInstance()

```javascript
testRenderer.getInstance()
```

返回根元素对应的实例（如果实用）。如果根元素是一个功能组件，因为它们没有实例，这将不起作用。

### testRenderer.root

```javascript
testRenderer.root
```

返回对测试树中特定节点断言有用的根 “test instance” 对象。你可以使用它来查找下面更深的其他 “test instance”。

### testInstance.find()

```javascript
testInstance.find(test)
```

找到 `test(testInstance)` 返回 `true` 的单个后台测试实例。如果 `test(testInstance)` 对于一个测试实例返回 `true`，则会引发错误。

### testInstance.findByType()

```javascript
testInstance.findByType(type)
```

使用提供的 `type` 查找单个后代测试实例。如果没有提供的 `type` 只有一个测试实例，则会引发错误。

### testInstance.findByProps()

```javascript
testInstance.findByProps(props)
```

用提供的 `props` 找到一个后代测试实例。如果没有提供的道具只有一个测试实例，它会抛出一个错误。

### testInstance.findAll()

```javascript
testInstance.findAll()
```

查找 `test(testInstance)` 返回 `true` 的所有后代测试实例。

### testInstance.findAllByType()

```javascript
testInstance.findAllByType(type)
```

使用提供的 `type` 查找所有后代测试实例。

### testInstance.findAllByProps()

```javascript
testInstance.findAllByProps(props)
```

用提供的 `prosp` 查找所有后代测试实例。

### testInstance.instance

```javascript
testInstance.instance
```

与此测试实例相对应的组件实例。它仅适用于类组件，因为 function 创建的组件没有实例。它匹配给定组件内的 `this` 值。

### testInstance.type

```javascript
testInstance.type
```

与此测试实例对应的组件类型。比如，一个 `<Button/>` 组件有一个 `Button` 的 type。

### testInstance.props

```javascript
testInstance.props
```

这个测试实例对应的道具。例如，一个 `<Button size="small"/>` 组件具有 `{size: 'small'}` 的props

### testInstance.parent

```javascript
testInstance.parent
```

这个测试实例的父测试实例。

### testInstance.children

```javascript
testInstance.children
```

这个测试实例的子测试实例。

----

## 思路

你可以将 `createNodeMock` 函数传递给 `TestRenderer.create` 作为选项，它允许自定义模拟参考。`createNodeMock` 接收当前元素作为参数，并且返回一个模拟 ref 对象。测试依赖于 refs 的组件时，这非常有用。


```javascript
import TestRenderer from 'react-test-renderer';

class MyComponent extends React.Component {
  constructor(props) {
    super(props);
    this.input = null;
  }
  componentDidMount() {
    this.input.focus();
  }
  render() {
    return <input type="text" ref={el => this.input = el} />
  }
}

let focused = false;
TestRenderer.create(
  <MyComponent />,
  {
    createNodeMock: (element) => {
      if (element.type === 'input') {
        // mock a focus function
        return {
          focus: () => {
            focused = true;
          }
        };
      }
      return null;
    }
  }
);
expect(focused).toBe(true);
```












