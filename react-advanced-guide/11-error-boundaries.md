# Error  Boundaries

- 英文原文：[https://reactjs.org/docs/error-boundaries.html][1]

以往，组件内部的 JavaScript 错误会破坏 React 内部的状态，并且导致下一次渲染的时候，发生隐藏的错误（[emit][2] [cryptic][3] [errors][4]） 。这些错误经常由应用程序较早的代码，但是 React
没有提供在组建中优雅的处理他们的方法，也不能恢复。

----


## 错误边界的介绍

UI 中的 Javascript 的错误不应该破坏整个应用程序。为了解决 React 用户的这个问题，React 16 引入了一个 “错误边界” 的概念。

错误边界是在**其子组件树的任意位置都能捕获 JavaScript 错误的 React 组件，能够记录这些错误，并且显示一个备用的组件，而不是将整个组件树崩溃掉**。错误边界能够在其下面的组件树的渲染过程中、在生命周期函数中，以及在它们的构造函数中捕获错误。

> 注意：
>错误边界**不会**捕获以下错误：
> - 事件处理（[了解更多][5]）
> - 异步操作：（如 `setTimeout` 和 `requestAnimationFrame`）
> - 服务端渲染
> - 在错误边界本身抛出错误（而不是在子组件中）


如果在一个类构建的组件中定义了新的生命周期方法 `componentDidCatch(error, info)` ,这个组件将成为错误边界：

```javascript
class ErrorBoundary extends React.Component {
  constructor(props) {
    super(props);
    this.state = { hasError: false };
  }

  componentDidCatch(error, info) {
    // 显示备用 UI
    this.setState({ hasError: true });
    // 你也可以向服务端报告错误日志
    logErrorToMyService(error, info);
  }

  render() {
    if (this.state.hasError) {
      // 你可以渲染任何的备用 UI
      return <h1>Something went wrong.</h1>;
    }
    return this.props.children;
  }
}
```

之后你可以将其作为常规的组件：

```html
<ErrorBoundary>
  <MyWidget />
</ErrorBoundary>
```

`componentDidCatch()` 方法的工作方式和 JavaScript 的 `catch{}` 比较相似，不过是作用在组件上的。只有类组件可以是错误边界。在实践过程中，大多数时候，你可能会希望只声明一次错误边界，并且在整个应用程序中去使用它。

请注意，**错误边界只能在其下面的组件树中捕获错误**，错误边界本身不能捕获错误。如果一个错误边界尝试渲染错误消息失败，这个错误会传递给其上面最接近的错误边界。这也是很像 `catch{}` 在 JavaScript 中的作用。

## componentDidCatch 参数

`error` 是一个被抛出的错误。

`info` 是具有 `componentStack` 属性的对象。这个属性拥有抛出错误期间的组件堆栈的信息。

```javascript
//...
componentDidCatch(error, info) {
  
  /* Example stack information:
     in ComponentThatThrows (created by App)
     in ErrorBoundary (created by App)
     in div (created by App)
     in App
  */
  logComponentStackToMyService(info.componentStack);
}

//...
```

## 在线示例

查看这个通过 [React 16 Beta 版本][6]使用[错误边界的例子][7]。

----

## 在什么地方使用错误边界

错误边界的粒度取决于你自己。你可能会封装顶级的路由组件去向用户显示“发生的错误消息”，就像服务端框架处理崩溃消息一样。你也可以将每个小部件包装在各个错误编辑内，来防止其他的应用程序也崩溃。


----

## 未捕获错误的新的处理方式

这是一个非常重要的改变。**对于 React 16 来说,没有被任何的错误边界所捕获的错误将导致整个 React 组件树的卸载**。

我们讨论了这个决定，但是根据我们的经验，将崩溃的 UI 留在远处比完全移除该组 UI 更加的糟糕。例如，像 Messenger 这样的产品中，可以发现，崩溃的 UI 可能会导致某个用户像错误的对象发送消息。同样，对于一个支付的应用来说，显示错误的金额比不渲染该内容更糟糕。


这个变化意味着当你迁移到 React 16 时，你可能会发现应用中以前你没注意的一些地方出现了崩溃。添加错误边界，能够可以在出现问题的时候提供更好的用户体验。

例如，Facebook Messager 把边栏、信息面板和会话日志以及消息输入的内容都封装到单独的错误边界中。如果其中一个 UI 中的某些组件崩溃，其他的组件将继续工作。

我们也鼓励你使用 JS 错误报告服务（或者构建自己的错误报告服务）以便于你能够在生产环境下得到更多的未处理的异常信息，从而去修复这些问题。

----

## 组件堆栈跟踪：

React 16 会将渲染期间发生的所有错误打印到控制台，即使应用程序可能会意外的吞掉这些错误。除了错误消息和 JavaScript 堆栈之外，它还提供组件堆栈跟踪。现在，你可以在组件树中看到发生错误的位置：

![https://d33wubrfki0l68.cloudfront.net/ff92e5adee15dada70cc0f74f8ae1e7ca10b0a46/5624e/static/f1276837b03821b43358d44c14072945-5795d.png][8]

你也可以在组件堆跟踪中查看文件名和行号。默认情况下，[Create React App][9]也支持这个操作：

![https://d33wubrfki0l68.cloudfront.net/3e72936ddca683e889b17436d31d7ee27d2e2724/73ec9/static/45611d4fdbd152829b28ae2348d6dcba-db8f4.png][10]

如果你不使用 Create React App，你可以将 [这个插件][11] 添加到你的Babel配置中。 请注意，它仅在开发中适用，**生产环境下必须禁用**。

> 注意：堆栈跟踪中组件的名称取决于 [Function.name][12] 属性。如果你需要兼容尚未支持该属性的就浏览器（如：IE11），请在打包的应用中使用 `Function.name`的 polyfill ，如 [function.name-polyfill][13].另外你可以在所有的组件上显示的设置 [displayName][14] 属性。

----

## try/catch 如何？

`try/catch`是伟大的，但是它只适用于命令式的代码：

```javascript
try {
  showButton();
} catch (error) {
  // ...
}
```
然而，React 组件是声明式的，并且制定需要*呈现的内容*:

```html
<Button />
```
错误边界保留了组件的声明式这一性质，并且会按照你的期望进行工作。例如。即使由于 `componentDidUpdate` 中 `setState` 引起的一个错误是发生在组件树的深处，但是这个错误依旧会传递到最接近的错误边界。

----

## 事件处理程序如何？

错误边界**不会**捕获事件处理程序中的错误。

React 不需要错误边界从事件处理程序中进行恢复。与渲染方法和生命周期的钩子不同，事件处理程序在渲染过程中不会执行。所以，如果它们抛出错误，React 依然能够在屏幕上进行渲染。

如果你需要在事件处理程序内容捕获异常，使用 JavaScript 的 `try/catch` 来做：

```javascript
class MyComponent extends React.Component {
  constructor(props) {
    super(props);
    this.state = { error: null };
  }
  
  handleClick = () => {
    try {
      // Do something that could throw
    } catch (error) {
      this.setState({ error });
    }
  }

  render() {
    if (this.state.error) {
      return <h1>Caught an error.</h1>
    }
    return <div onClick={this.handleClick}>Click Me</div>
  }
}
```
请注意，上述示例演示了常规的 JavaScript 错误捕获，而不是使用错误边界。

----


## React 15 的变化

React 15 包含了对不同方法名称下错误边界的不同名称 `unstable_handleErrot`。这个方法不再起作用，并且在 React 16 版本中，你需要将其转为 `componentDidCatch`。

对于这种变化，我们提供了一个 [codemod][15] 来自动转换。





[1]: https://reactjs.org/docs/error-boundaries.html
[2]: https://github.com/facebook/react/issues/4026
[3]: https://github.com/facebook/react/issues/6895
[4]: https://github.com/facebook/react/issues/8579
[5]: https://reactjs.org/docs/error-boundaries.html#how-about-event-handlers
[6]: https://github.com/facebook/react/issues/10294
[7]: https://codepen.io/gaearon/pen/wqvxGa?editors=0010
[8]: https://d33wubrfki0l68.cloudfront.net/ff92e5adee15dada70cc0f74f8ae1e7ca10b0a46/5624e/static/f1276837b03821b43358d44c14072945-5795d.png
[9]: https://github.com/facebookincubator/create-react-app
[10]: https://d33wubrfki0l68.cloudfront.net/3e72936ddca683e889b17436d31d7ee27d2e2724/73ec9/static/45611d4fdbd152829b28ae2348d6dcba-db8f4.png
[11]: https://www.npmjs.com/package/babel-plugin-transform-react-jsx-source
[12]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function/name
[13]: https://github.com/JamesMGreene/Function.name
[14]: https://reactjs.org/docs/react-component.html#displayname
[15]: https://github.com/reactjs/react-codemod#error-boundaries