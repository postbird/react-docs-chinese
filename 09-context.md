# Context

- 英文原文：[https://reactjs.org/docs/context.html][1]

> **注意：** `React.PropTypes` 从 ***React v15.5*** 版本开始已经移入了一个不同的包中。 请使用 [the prop-types library instead][2] 去定义 `contextTypes`

> 我们提供一个 [codemod script][3] 来帮助自动转换


使用 Raect，可以很方便的跟踪 React 组件的数据流。当你浏览你的组件的时候，你可以看到那些 props 被传递，这可以使你的应用看起来更容易理解。

在某些情况下，你会希望通过组件树来传递数据的时候不需要在每一个级别的组件中传递 prop。你可以使用强大的 `context` API 来完成这个操作。


----

## 为什么使用 Context

> 译者注： 为了更好的理解 context API，并没有将 context 翻译成*上下文*

绝大多数的应用程序来说是不需要使用 context 的。

如果你希望你的应用程序是稳定的，请不要使用 context。它是一个实验性的 API，在将来的 React 版本中可能会移除。

如果你不熟悉像 [Redux][4] 或者是 [MobX][5] 这样的状态管理器，也请不要使用 context。对于许多实践中的应用来说，这些库以及他们和 React 的绑定是管理许多组件状态的不错的选择。Redux 可能是你目前遇到的问题的正确的解决方案，而不是 context。

如果你不是经验丰富的 React 开发人员，请不要使用 context，通常来说仅仅使用 props 和 state 来实现功能更好。

如果即使有上面这些警告，你还坚持使用 context 的话，尽量避免直接使用 context API，而是将其进行二次封装，以便于之后 API 更改的时候更容易升级。

----

## 如果使用 Context

假设你有下面结构的组件：

```javascript
class Button extends React.Component {
  render() {
    return (
      <button style={{background: this.props.color}}>
        {this.props.children}
      </button>
    );
  }
}

class Message extends React.Component {
  render() {
    return (
      <div>
        {this.props.text} <Button color={this.props.color}>Delete</Button>
      </div>
    );
  }
}

class MessageList extends React.Component {
  render() {
    const color = "purple";
    const children = this.props.messages.map((message) =>
      <Message text={message.text} color={color} />
    );
    return <div>{children}</div>;
  }
}
```
在上面这个示例中，我们会传递一个 `color` 的 prop 给 style，以便于用来设置 `Button` 和 `Message` 组件的样式。使用 context，我们能够自动的在组件树中传递：

```javascript
import PropTypes from 'prop-types';

class Button extends React.Component {
  render() {
    return (
      <button style={{background: this.context.color}}>
        {this.props.children}
      </button>
    );
  }
}

Button.contextTypes = {
  color: PropTypes.string
};

class Message extends React.Component {
  render() {
    return (
      <div>
        {this.props.text} <Button>Delete</Button>
      </div>
    );
  }
}

class MessageList extends React.Component {
  getChildContext() {
    return {color: "purple"};
  }

  render() {
    const children = this.props.messages.map((message) =>
      <Message text={message.text} />
    );
    return <div>{children}</div>;
  }
}

MessageList.childContextTypes = {
  color: PropTypes.string
};

```

通过给 `MessageList` (context 的提供者)增加 `childContextTypes` 和 `getChildContext` 方法，React 可以自动的在组件树中传递信息，并且子组件的任何组件（本例中是 `Button`）能够通过定义的 `contextTypes` 来访问。


如果 `contextTypes` 没有定义，则 `context` 是一个空对象。

----

## 父子组件耦合

Context 也能够构建一个父子组件交互的 API。比如，使用这种方式的 [React Router V4][6] :

```javascript
import { BrowserRouter as Router, Route, Link } from 'react-router-dom';

const BasicExample = () => (
  <Router>
    <div>
      <ul>
        <li><Link to="/">Home</Link></li>
        <li><Link to="/about">About</Link></li>
        <li><Link to="/topics">Topics</Link></li>
      </ul>

      <hr />

      <Route exact path="/" component={Home} />
      <Route path="/about" component={About} />
      <Route path="/topics" component={Topics} />
    </div>
  </Router>
);
```

在 `Router` 组件中传递的一些信息，每一个 `Link` 和 `Route` 都能够回传给外层的 `Router`.

在使用榆次类似的 API 构建组件之前，请先考虑是否有其他更好的选择。比如，你可以把整个 React 组件作为 props 进行传递。


----

## 在组件声明中期的方法中使用 context

如果 `contextTypes` 是在组件中定义的，则下面的[生命周期方法][7]接收一个附加的参数，也就是 `context` 对象：

- [constructor(props, context)][8]

- [componentWillReceiveProps(nextProps, nextContext)][9]

- [shouldComponentUpdate(nextProps, nextState, nextContext)][10]

- [componentWillUpdate(nextProps, nextState, nextContext)][11]

> **注意：** React16 中，`componentDidUpdate`不在接收 `prevContext`

----


## 在没有 state 的 fcuntion 构建的组件中使用 Context

无状态的 function 构建的组件也能够使用 `context` ,只要 `contextTypes` 被定义为函数的属性即可。下面代码是使用 function 构建的无状态 `Button` 组件：

```javascript
import PropTypes from 'prop-types';

const Button = ({children}, context) =>
  <button style={{background: context.color}}>
    {children}
  </button>;

Button.contextTypes = {color: PropTypes.string};
```
----

## 更新 context

**不要这样做（Don’t do it.）**

React 有一个用来更新 context 的 API ,但是它已经从根本上被否定了，你也不应该使用它。

当 state 和 props 发生更改时，会调用 `getChildContext` 方法。为了更新 context 中的数据，使用 `this.setState` 进行本地的 state 更新，这回触发一个新的 context，并且子组件会收到这些更改。

```javascript
import PropTypes from 'prop-types';

class MediaQuery extends React.Component {
  constructor(props) {
    super(props);
    this.state = {type:'desktop'};
  }

  getChildContext() {
    return {type: this.state.type};
  }

  componentDidMount() {
    const checkMediaQuery = () => {
      const type = window.matchMedia("(min-width: 1025px)").matches ? 'desktop' : 'mobile';
      if (type !== this.state.type) {
        this.setState({type});
      }
    };

    window.addEventListener('resize', checkMediaQuery);
    checkMediaQuery();
  }

  render() {
    return this.props.children;
  }
}

MediaQuery.childContextTypes = {
  type: PropTypes.string
};

```

有一个问题，如果通过组件的更改来提供 context 值，在使用这个值的子组件的父组件中的 `shouldComponentUpdate` 返回 `false` ，则子组件不会发生更新。

这完全不受使用 context 的组件所控制，所以基本上也没有一个可靠的方法解决 context 的更新。[这篇文章][12]很好的解释了为什么这是一个问题，以及如何解决这个问题。



[1]: https://reactjs.org/docs/context.html
[2]: https://www.npmjs.com/package/prop-types
[3]: https://reactjs.org/blog/2017/04/07/react-v15.5.0.html#migrating-from-react.proptypes
[4]: https://github.com/reactjs/redux
[5]: https://github.com/mobxjs/mobx
[6]: https://reacttraining.com/react-router
[7]: https://reactjs.org/docs/react-component.html#the-component-lifecycle
[8]: https://reactjs.org/docs/react-component.html#constructor
[9]:https://reactjs.org/docs/react-component.html#componentwillreceiveprops
[10]:https://reactjs.org/docs/react-component.html#shouldcomponentupdate
[11]: https://reactjs.org/docs/react-component.html#componentwillupdate
[12]:https://medium.com/@mweststrate/how-to-safely-use-react-context-b7e343eff076