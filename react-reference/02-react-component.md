# React.Component

- 英文原文：[https://reactjs.org/docs/react-component.html](https://reactjs.org/docs/react-component.html)

[组件](https://reactjs.org/docs/components-and-props.html)让你能够把 UI 分成独立的，可重用的部分，并且将每个部分分开考虑。`React.Component` 由 [React](https://reactjs.org/docs/react-api.html) 提供。

-----

## 概览

`React.Component` 是一个抽象的基类，因此很少直接引用 `React.Component` 。相反，通常会对其进行子类化，并且至少定义一个 [render()](https://reactjs.org/docs/react-component.html#render) 方法。

一般来说，可以将一个 React 组件定义为一个 [JavaScript 类](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Classes)：

```javascript
class Greeting extends React.Component {
  render() {
    return <h1>Hello, {this.props.name}</h1>;
  }
}
```

如果你现在还没有使用 ES6，你可能会使用 [create-react-class](https://reactjs.org/docs/react-api.html#createclass) 来代替。可以看一下 [不使用 ES6 的 React](https://reactjs.org/docs/react-without-es6.html) 了解更多。

### 组件生命周期

每个组件都有几个“生命周期函数”，你可以在某些特定的时间进行重写。事件发生之前调用的是带 **will** 前缀的方法，而事件发生之后调用的是带 **did** 的方法。

#### Mounting （挂载）

当创建一个组件的示例并将其插入到 DOM 中的时候，将调用下列方法：

- [constructor()](#constructor)

- [conponentWithMount()](#componentwillmount)

- [render()](#render)

- [componentDidMount()](#componentdidmount)

#### Updating （更新）

更新可以由 props 或者是 state 更改引起。当一个组件被重新渲染时，下面方法将被调用：

- [componentWillReceiveProps()](#componentshouldreceiveprops)

- [shouldComponentUpdate()](#shouldcomponentupdate)

- [componentWillUpdate()](#componentwillupdate)

- [render()](#render)

- [componentDidUpdate()](#componentdidupdate)

#### Unmounting（卸载）

当一个组件从 DOM 中移除的时候，会调用下面方法：

- [componentWillUnmount()](#componentwillunmount)

## 错误处理

在渲染期间，生命周期或者任何子组件的构造函数中发生错误时调用此方法。

- [componentDidCatch()](#componentdidcatch)

### 其他 API

每个组件都提供了一些其他的 API：

- [setState()](#setstate)

- [forceUpdate()](#forceupdate)

### 类属性

- [defaultProps](#defaultprops)

- [displayName](#displayname)

### 实例属性

- [props](#props)

- [state](#state)

----

## API 参考

### render()

```javascript
render()
```

`render()` 方法是必须的。

调用时，应当检查 `this.props` 和 `this.state` 并返回以下类型之一：

- **React 元素**。 通常通过 JSX 创建，返回的元素可以是原生的 DOM 元素，如 `<div/>`，也可以是用户自定义的组件（`<MyComponent/>`）。

- **字符串和数字**。在 DOM 中渲染成文本节点。

- **Portals**。通过 [ReactDOM.createProtal](https://reactjs.org/docs/portals.html) 创建。

- `null`。不渲染任何内容。

- **Boolean**。不渲染任何内容。（主要为了支持 `return test && <Child/>`模式，其中 `test` 是 Boolean 类型）。

当返回 `null` 或者是 `false` 的时候，`ReactDOM.findDOMNode(this)` 会返回 `null`。

`render()` 方法应当是纯粹的，这意味着它不会修改组件的 state，每次调用时都会返回相同的结果，并且不会直接与浏览器进行交互。如果你需要和浏览器进行交互，请在 `componentDidMount()` 或者其他生命周期方法中执行。保持 `render()` 的纯粹性能够使得组件更直观。

> **注意：**
> 如果[shouldComponentUpdate()](#shouldcomponentupdate) 方法返回 `false`, `render()` 将不会被调用。


#### 其他

你可以通过数组从 `render()` 方法中返回多个项目。

```javascript
render() {
  return [
    <li key="A">First item</li>,
    <li key="B">Second item</li>,
    <li key="C">Third item</li>,
  ];
}
```

> **注意：**
> 不要忘记给每一个项目 [增加 key 属性](https://reactjs.org/docs/lists-and-keys.html#keys) 以避免 key 警告。

----


### constructor()

```javascript
constructor(props)
```

React 组件的构造函数在挂载之前被调用。在实现 `React.Component` 子类的构造函数时，应当在任何其他语句之前调用 `super(props)`。否则， `this.props` 在构造函数中是 undefined，这可能会导致错误。

避免在构造函数中引入任何副作用或涉及订阅的方法。对于这些用例，请在 `componentDidMount()` 方法中操作。

构造函数是初始化 state 的正确位置。实现这个，只需要将一个对象分配给 `this.state`；不要尝试在构造函数中调用 `setState()` 方法。够在函数也经常用来把事件踔厉程序绑定到类实例。

如果你不初始化 state，并且不绑定方法，则不需要为 React 组件实现构造函数。

在极少数情况下，可以根据 props 初始化 state。这有效的 “forks” props,并且通过初始化的 props 设置 state。下面是一个 `React.Component` 子类的有效的构造函数示例：

```javascript
constructor(props) {
  super(props);
  this.state = {
    color: props.initialColor
  };
}
```

小心这种模式，因为 state 不会随着 props 的更新而更新。相对于将同步 props 到 state 中，经常想做的是 [状态提升](https://reactjs.org/docs/lifting-state-up.html)。

如果你通过 state 来 “fork” props，你也可能想要实现 [componentWillReceiveProps(nextProps)]() 来使得 state 保持最新。但是状态提升更容易，且会更少的出现错误。

----

### componentWillMount()

```javascript
componentWillMount()
```

`componentWillMount()` 会在挂载前立即调用。它在 `render()` 方法之前调用，因此在该方法中同步调用 `setState()` 不会触发额外的渲染。一般来说，我们建议使用构造函数。

避免在此方法中引入任何副作用或这是订阅的函数，对于这些情况，请改用 `componentDidMount()`。

这是服务端渲染上唯一调用的生命周期钩子。

---

### componentDidMount()

```javascript
componentDidMount()
```

`componentDidMount()` 在挂载组件后立即被调用。需要 DOM 节点的初始化应该在这个函数中实现。如果需要从远程端点加载数据，这是一个实例化网络请求的好地方。

这个方法是设置任何订阅的好地方。如果你这样做，不要忘记在 `componentWillUnmount()` 中取消订阅。

在这个方法中调用 `setState()` 会触发一个额外的渲染，但是保证在同一个 tick 中刷新。这保证即使 `render()` 方法在这种情况下调用两次，用户也不会看到中间状态。请谨慎使用此模式，因为这通常会导致性能为题。但是使用类似模态框和工具提示的情况下，如需要在渲染依赖于其大小或位置的东西之前测量 DOM 节点时，这是有用的。

----

### componentWillReceiveProps()

```javascript
componentWillReceiveProps(nextProps)
```

当一个组件接收新的 props 的时候，`componentWillReceiveProps()` 会被调用。如果你需要更新 state 来响应 props 的更改（比如，重置 state），你可以比较 `this.props` 和 `nextProps` ，并在此方法中使用 `this.setState()` 来执行 state 的转换。

请注意，即使 props 没有更改， React 也可能调用这个方法，因此，如果你只想处理更改，请务必比较当前值和下一个值。父组件导致父组件重新渲染的时候，可能会发生这种情况。

React 在[挂载]()过程中不会用初始化的 props 调用 `componentWillCReceiveProps()` 方法。只有组件的一些 props 可能更新的时候，才会调用这个方法。调用 `this.setState()` 通常不会触发 `componentReceiveProps()`。

----


### shouldComponentUpdate()

```javascript
shouldComponentUpdate(nextProps, nextState)
```

使用 `shouldComponentUpdate()` 让 React 知道组件的输出结果是否不受当前 state 或者 props 变化的影响。默认行为是，每次 state 更改时重新渲染，在绝大多数情况下，你应当依赖默认行为。

当渲染新的 props 或 state 时，会调用 `shouldComponentUpdate()` 方法。默认为 true。在初始渲染或使用 `forceUpdate()` 方法时，不会调用此方法。

返回 `false` 不会阻止子组件在其 state 更改时重新渲染。

目前，如果 `shouldComponentUpdate()` 返回 `false`，则不会调用 [componentWillUpdate()](#componentwillupdate)、[render()](#render) 和 [componentDidUpdate()](#componentdidupdate) 三个方法。请注意，将来 React 可能会将 `shouldComponentUpdate()` 作为提示而不是严格的指令，并且发挥 false 仍然可能会导致组件的重新渲染。

如果在分析之后确定一个特殊的组件很慢， 则可以将其更改为从 [React.PureComponent](https://reactjs.org/docs/react-api.html#reactpurecomponent)继承，`React.PureComponent` 实现了浅层 `props` 和 `state` 比较的方法 `shouldComponentUpdate()`。如果你确定要手写这个，你可以将 `this.props`和 `nextProps` 以及 `this.state` 和 `nextState` 进行比较，并且返回 `false` 告诉 React 可以跳过这个更新。

我们不建议进行深度相等检查活在 `shouldComponentUpdate()` 中使用 `JSON.stringfy()`。这是非常低效的，会损害性能。

----


### componentWillUpdate()

```javascript
componentWillUpdate(nextProps,nextState)
```

当接收到新的 props 或 state 的时候，`componentWillUpdate()` 会立即被调用。使用这个钩子在更新发生之前执行一些准备工作。这个方法不是为初始渲染调用的。

请注意，你不能在这里调用 `this.setState()`,你可不应该做其他任何事情（比如 dispatch 一个 Redux 的action），在 `componentWillUpdate()` 返回之前触发一个 React 组件的更新。

如果你需要更新 state 以响应 props 的更改，请改用 `componentWillReceiveProps()`

> **注意：**
> 如果 [shouldComponentUpdate()](#shouldcomponentupdate) 返回 false ，则 `componentWillUpdate()` 不会被调用。


----

### componentDidUpdate()

```javascript
componentDidUpdate(nextProps, nextState)
```

`componentDidUpdate` 会在更新发生后立即调用。这个方法不是为初始渲染调用的。

当组件更新时，可以使用这个机会来操作 DOM。只要你将当前的 props 和 之前的 props 进行比较（例如，如果 props 没有更改，则可能不需要网络请求），这也是做网络请求的好地方。

> **注意：**
> 如果 [shouldComponentUpdate()](#shouldcomponentupdate) 返回 false，`componentDidUpdate()` 不会被调用。


----

### componentWillUnmount()

```javascript
componentWillUnmount()
```

当一个组件呗卸载和销毁前会调用 `componentWillUnmount()` 方法。在此方法中执行一些必要的清理工作，例如使定时器无效，取消网络请求或清理在 `componentDidMount()` 中创建的任何订阅等。

----

### componentDidCatch()

```javascript
componentDidCatch()
```

错误边界是 React 组件，可以在子组件树的任何位置捕获 JavaScript 错误，记录这些错误，并显示一个备用的 UI 而不是崩溃整个组间树。错误边界能够在渲染过程中、生命周期方法以及整个树中的构造方法中捕获错误。

如果一个类组件定义了这个生命周期方法， 它将成为一个错误边界。调用它中的 `setState()` 可以让你捕获子树中未处理的 JavaScript 错误，并显示一个后备 UI。只能使用错误边界从意外的异常中恢复，不要试图将他们用于控制流程。

更多细节，请查看 [React 16 的错误边界](#https://reactjs.org/blog/2017/07/26/error-handling-in-react-16.html)

> **注意：**
> 错误边界只能捕获其子树中的异常，错误边界本身的异常无法捕获。

----

### setState()

```javascript
setState(updater[, callback])
```

`setState()` 将对组件 state 的更改进行排队，并告诉 React 该组件及其子组件需要使用更新后的 state 进行重新渲染。这是用来更新用户界面以响应事件处理程序和服务器响应的主要方法。

把 `setState()` 看错一个请求，而不是一个立即的命令来更新组件。为了获得更好的感知性能， React 可能会延迟它，然后一次更新几个组件。React 不保证立即应用状态更改。

`setState()` 并不总是立即更新组件。它可能会批处理或推迟更新，直到更晚。这使得在调用 `setState()` 之后，立即读取 `this.state` 是一个潜在的陷阱。相反，使用 `componentDidUpdate()` 或 `setState()` 回调（`setState(updater,callback)`），其中的任何一个都保证在应用更新之后触发。如果你需要根据之前的状态设置状态，请阅读下面的 `updater` 参数。

除非 `shouldComponentUpdate()` 返回 false，否则`setState()` 将总是导致重新渲染。如果正在使用可变对象，并且在 `shouldComponentUpdate()` 中不能实现条件渲染逻辑，则只有当前 state 与先前 state 不同时调用 `setState()` 才能避免不必要的重新渲染。

第一个参数是一个带签名的 `updater` 函数：

```javascript
(prevState, props) => stateChange
```

`prevState` 是对前一个 state 的引用。它不应当被直接修改。相反，应当通过建立一个基于 `prevState` 和 `props` 的输入的新对象来表示变化，例如，假设我们想通过 `props.step` 增加一个 state 值：

```javascript
this.setState((prevState, props) => {
  return {counter: prevState.counter + props.step};
});
```

保证更新功能接收到的 `prevState` 和 `props` 都是最新的，更新器的输出和 `prevState` 是浅显的合并。

`setState()` 的第二个参数是可选的回调函数，一旦 `setState()` 完成并且组件被重新渲染，该函数将被执行。通常我们推荐使用 `componentDidUpdate()` 来代替这种逻辑。


你可以选择将一个对象作为第一个参数传递给 `setState()` 而不是函数：

```javascript
setState(stateChange[, callback])
```

这使 `stateChange` 到新状态的浅层合并，例如调整购物车的数量：

```javascript
this.setState({quantity:2})
```

这种形式的 `setState()` 也是异步的，同一个周期内的多个调用可以一起进行批处理。例如，如果你尝试在同一个周期多次增加一件物品的树立，那么将导致相当于：

```javascript
Object.assign(
  previousState,
  {quantity: state.quantity + 1},
  {quantity: state.quantity + 1},
  ...
)
```

随后的调用将覆盖同一周期中以前调用的值，因此数量只会增加一次。如果次啊一个状态取决于之前的状态，我们建议使用更新函数的形式：

```javascript
this.setState((prevState) => {
  return {counter: prevState.quantity + 1};
});
```

更多细节可以查看 [state 和 生命周期](https://reactjs.org/docs/state-and-lifecycle.html)。

----

### forceUpdate()

```javascript
component.forceUpdate(callback)
```

默认情况下， 当你组件的 state 或者 props 改变时，你的组件讲重新渲染。如果 `render()` 依赖于其他数据，则可以通过调用 `forceUpdate()` 来告诉 React 组件需要重新渲染。

调用 `forceUpdate()` 将导致在组件上调用 `render()` 方法，跳过 `shouldComponentUpdate()`。这将触发子组件的正常生命周期方法，包括每个子组件的 `shouldComponentUpdate()` 方法。如果标记更改， React 将只会更新 DOM。

通常你应噶尽量避免使用 `forceUpdate()` ，只能从 `render()` 中的 `this.props` 和 `this.state` 读取。

----

## 类属性

### defaultProps

`defaultProps` 可以被定义为组件类本身的属性，以设置该类的默认 props。这用于未定义的 props，但不适用于 null props。例如：

```javascript
class CustomButton extends React.Component {
  // ...
}

CustomButton.defaultProps = {
  color: 'blue'
};
```

如果没有提供 `props.color`，则默认设置为 `blue`:

```javascript
render(){
  return <CustomButton/>; // props.color will be set to blue
}
```

如果 `props.color` 被设置为 null，它的值将是 null：

```javascript
render() {
    return <CustomButton color={null} /> ; // props.color will remain null
  }
```

----

### displayName

`displayName` 字符串常用于调试信息。通常，你不需要明确的设置它，因为它可以从定义组件的函数或者类的名称推断出来。如果要为调试目的显示不同的名称，或者要创建高阶组件，则可能需要显示的设置它，有关详细信息，可以参阅[封装 DisplayName 来更好的调试](https://reactjs.org/docs/higher-order-components.html#convention-wrap-the-display-name-for-easy-debugging)。

----

## 实例属性

### props

`this.props` 包含由该组件的调用者定义的道具。请参阅[组件和 props](https://reactjs.org/docs/components-and-props.html) 了解更多 props 的介绍。

特别的，`this.props.children` 是一个特殊的 prop，通常由 JSX 表达式中的子标签定义，而不是标签本身。

### state

state 包含了此组件的一些特定数据，可能随着时间而改变。state 是用户定义的，它应该是一个普通的 JavaScript 对象。

如果你不在 `render()` 中使用，则该数据不应当出现在 state 中。例如，你可以将计时器 ID 直接放在示例本身属性上，而不是放在 state 中。

请参阅 [state 和 生命周期](https://reactjs.org/docs/state-and-lifecycle.html) 了解更多关于 state 的信息。

永远不要直接改变 `this.state`，因为之后调用 `setState()` 可能回去带你所做的改变。应当将 `this.state` 看做是不可变的。



