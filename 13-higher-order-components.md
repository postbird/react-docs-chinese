# Higher-Order Components

- 英文原文：[https://reactjs.org/docs/higher-order-components.html][1]

一个高阶组件（HOC）是 React 中用于组件重用逻辑的一种高级技术。HOCs 本身不是 React API 的一部分，他们从 React 的构成性质出现的一种模式。

具体来说，**高阶组件是一个获取组件并且返回新组件的方法**。

```javascript
const EnhancedComponent = higherOrderComponent(WrappedComponent);
```

组件是将 props 转换成 UI，而高阶组件则是将一个组件转换成另一个组件。

HOCs 在第三方的 React 库中很常见，比如 Redux 的 [connect][2] 和 Relay 的 [createContainer][3]。

本文中，我们将讨论为什么高阶组件是有用的，以及如何编写自己的高阶组件。

----

## 使用 HOCs 来解决 交叉问题(Cross-Cutting Concerns)

> **请注意：**
>我们以前建议使用 mixins 作为处理交叉问题的一种方式。然而，我们已经意识到，使用 mixins 创造的麻烦比它们的价值更多。[了解更多][4]关于我们为什么要放弃 mixins 以及如何转换现有的组件。


组件是 React 中代码重用的主要单元。然而，你会发现某些模式并不适合传统的组件。

比如，假设你有一个 `CommonList` 组件来订阅一个外部数据源来呈现注释列表：

```javascript
class CommentList extends React.Component {
  constructor(props) {
    super(props);
    this.handleChange = this.handleChange.bind(this);
    this.state = {
      // "DataSource" is some global data source
      comments: DataSource.getComments()
    };
  }

  componentDidMount() {
    // Subscribe to changes
    DataSource.addChangeListener(this.handleChange);
  }

  componentWillUnmount() {
    // Clean up listener
    DataSource.removeChangeListener(this.handleChange);
  }

  handleChange() {
    // Update component state whenever the data source changes
    this.setState({
      comments: DataSource.getComments()
    });
  }

  render() {
    return (
      <div>
        {this.state.comments.map((comment) => (
          <Comment comment={comment} key={comment.id} />
        ))}
      </div>
    );
  }
}
```

之后，你写了一个订阅博客文章的组件，它遵循类似的模式：

```javascript
class BlogPost extends React.Component {
  constructor(props) {
    super(props);
    this.handleChange = this.handleChange.bind(this);
    this.state = {
      blogPost: DataSource.getBlogPost(props.id)
    };
  }

  componentDidMount() {
    DataSource.addChangeListener(this.handleChange);
  }

  componentWillUnmount() {
    DataSource.removeChangeListener(this.handleChange);
  }

  handleChange() {
    this.setState({
      blogPost: DataSource.getBlogPost(this.props.id)
    });
  }

  render() {
    return <TextBlock text={this.state.blogPost} />;
  }
}
```

`CommonList` 和 `BlogPost` 是不一样的 —— 他们在 `DataSource` 上调用的方法不同，并且呈现的输出内容也不同。但是它们的大部分是相同的：

- 在 mount 上，在 `DataSource` 添加一个更改监听器 

- 在监听器内部，当 数据源发生变化时，调用 `setState` 方法

- 在 unmount 时，移除更改监听器

你可以想象，在一个大型的应用中，这种订阅 `DataSource` 并且调用 `setState` 的模式会一再发生。我们希望通过一个抽象，允许我们在一个地方定义这个逻辑，并且将其分享到许多组件。这是高阶组件非常卓越的地方。


我们可以编写一个创建组件的函数，像是 `CommonList` 和 `BlogPost`, 它们都订阅 `DataSource`, 这个函数会接受一个子组件作为其参数，且该子组件接收订阅的数据作为 prop。让我们试试这个 `withSubscription` 函数:

```javascript
const CommentListWithSubscription = withSubscription(
  CommentList,
  (DataSource) => DataSource.getComments()
);

const BlogPostWithSubscription = withSubscription(
  BlogPost,
  (DataSource, props) => DataSource.getBlogPost(props.id)
);
```
第一个参数是包装的组件，第二个参数是检索我们需要的数据，给出一个 `DataSource` 和 当前的 prop。


当  `CommonListWithSubscription` 和 `BlogPostWithSubscription` 被渲染的时候，`CommonenList` 和 `BlogPost` 会将从 `DataSource` 检索到的最新的数据作为 `data` prop 进行传递：

```javascript
/ This function takes a component...
function withSubscription(WrappedComponent, selectData) {
  // ...and returns another component...
  return class extends React.Component {
    constructor(props) {
      super(props);
      this.handleChange = this.handleChange.bind(this);
      this.state = {
        data: selectData(DataSource, props)
      };
    }

    componentDidMount() {
      // ... that takes care of the subscription...
      DataSource.addChangeListener(this.handleChange);
    }

    componentWillUnmount() {
      DataSource.removeChangeListener(this.handleChange);
    }

    handleChange() {
      this.setState({
        data: selectData(DataSource, this.props)
      });
    }

    render() {
      // ... and renders the wrapped component with the fresh data!
      // Notice that we pass through any additional props
      return <WrappedComponent data={this.state.data} {...this.props} />;
    }
  };
}
```

注意，HOC 不会修改传入的组件，也不会使用继承来复制其行为。相反，HOC 通过将原始组件包装在容器组件中来进行组合。HOC 是具有零副作用的纯函数。

就是这样！包装的组件接收容器的所有 props，以及用于渲染输出的新的 prop 和 `data`。HOC 不关心数据如何使用或者为什么使用，包装的组件也不关心数据来自哪里。

因为 `withSubscription` 是一个正常的函数，你可以添加任意数量的参数。例如，你可能希望对 `data` 这个 prop 的名称进行可配置，已进一步隔离 HOC 和 包装组件。或者你可以接收一个参数用来配置 `shouldComponentUpdate`,或者配置数据源（data source）。这些都是可能的，因为 HOC 完全控制组件的定义。


和组件一样，`withSubscription` 和 包装的组件之间的联系都是基于 props 的。这使得能够轻松的将一个 HOC 变成另外一个 HOC，只要它们为包装的组件提供相同的 props。如果你更新数据获取的库的话，这可能很有用。


----


## 不改更改原始组件，而是应当使用组合

要抵制在 HOC 内部修改组件原型（以及其他方式突变）的冲动。

```javascript
function logProps(InputComponent) {
  InputComponent.prototype.componentWillReceiveProps = function(nextProps) {
    console.log('Current props: ', this.props);
    console.log('Next props: ', nextProps);
  };
  // The fact that we're returning the original input is a hint that it has
  // been mutated.
  return InputComponent;
}

// EnhancedComponent will log whenever props are received
const EnhancedComponent = logProps(InputComponent);
```

上面代码中有一些问题。一个是输入的组件不能与增强组件分开重用。更重要的是，如果将另一个 HOC 应用于 `EnhancedComponent` *也将* 更改 `componentWillReceiveProps`，第一个 HOC 的将会被覆盖。这个 HOC 也不再使用没有生命周期方法的 function 定义的组件。

突变 HOC  是一种泄露抽象 —— 消费者必须知道它们是如何执行的，以便于在 HOC 中避免这种问题。

相比于突变，HOC 应当使用组合，将输入的组件包装在容器组件中：

```javascript
function logProps(WrappedComponent) {
  return class extends React.Component {
    componentWillReceiveProps(nextProps) {
      console.log('Current props: ', this.props);
      console.log('Next props: ', nextProps);
    }
    render() {
      // Wraps the input component in a container, without mutating it. Good!
      return <WrappedComponent {...this.props} />;
    }
  }
}
```

这个 HOC 和上面突变版本的 HOC 具有相同的功能，同时避免了发生冲突。它对 class 创建的组件和 function 创建的组件同样友好。并且，它是一个纯粹的函数，它可以与其他 HOC 甚至它自身进行组合。

你可能已经注意到 HOC 和称为**容器组件（Container Components）**的模式之间的相似之处。容器组件是将高阶和低级部分进行功能分离的一种方法。容器管理像是订阅和state（状态）这些东西，并且将 props 传递给处理如渲染 UI 这样的组件。HOC 使用容器作为实施的一部分，你可以认为 HOC 是参数化容器组件的一种定义方式。

----

## 约定：将不相关的 props 传递给包装组件

HOC 向组件添加功能，它不应该大幅改变这些功能。而是希望从 HOC 返回的组件具有与包装组件类似的接口。

HOC 应当传递与其具体的关心的内容无关的 props，大部分的 HOC 有一个如下的渲染方法：

```javascript
render() {

  // 过滤出不需要向下传递的props（只是HOC需要的props）

  const { extraProp, ...passThroughProps } = this.props;

  // 将 props 注入到包裹的组件，这些通常是状态值或者是实例方法

  const injectedProp = someStateOrInstanceMethod;

  // 通过包装端组件 传递 props
  return (
    <WrappedComponent
      injectedProp={injectedProp}
      {...passThroughProps}
    />
  );
}
```

这种约定有助于确保 HOC 尽可能的灵活和可重用。


----

## 约定：最大化组合

不是所有的 HOC 看起来都一样，有时候它们只接收一个参数，即包装的组件：


```javascript
const NavbarWithRouter = withRouter(Navbar);
```

通常， HOC 接收其他参数。在下面 Relay 的这个例子中，配置对象用于指定组件的数据依赖关系：

```javascript
const CommentWithRelay = Relay.createContainer(Comment, config);
```

HOC 常像下面这样子：

```javascript
// React Redux's `connect`
const ConnectedComment = connect(commentSelector, commentActions)(CommentList);
```

什么鬼？！如果你将其分开，会更加容易的理解。

```javascript
// connect 是一个返回一个函数的函数
const enhance = connect(commentListSelector, commentListActions);
// 返回的函数就是一个 HOC，且 HOC 会返回一个与 Redux 的 store 相关联的组件
const ConnectedComment = enhance(CommentList);
```

换句话说，`connect` 是一个高阶函数，用于返回一个高阶的组件（HOC）。

这样子可能看起来混乱且没必要，但是它有一个有用的属性。像是 `connect` 方法这样的单参数 HOC，有一个特点就是 `Component => Component`.输出类型和输入类型相同的函数真的非常容易的组合在一起。

```javascript
// Instead of doing this...
const EnhancedComponent = withRouter(connect(commentSelector)(WrappedComponent))

// ... you can use a function composition utility
// compose(f, g, h) is the same as (...args) => f(g(h(...args)))
const enhance = compose(
  // These are both single-argument HOCs
  withRouter,
  connect(commentSelector)
)
const EnhancedComponent = enhance(WrappedComponent)
```

(同样的属性也允许 `connect` 以及其他增强型的 HOC 用作装饰器，一个实验性的 JavaScript 提案)。

这种 `构建` 有效的函数由被许多第三方库支持包括 lodash (像是 [lodash.flowRight][5])，[Redux][6] 和 [Ramda][7]。

----

## 约定：包含显示的 Name 方便调试

由 HOC 创建的容器组件和其它组件一样会在 [React Developer Tools][8] 中显示，为了方便调试，需要选择一个显示的名称来表明这是 HOC 的返回结果。

最常见的技术是给包装的组件一个显示的名称。因此，如果您的高阶组件被定义为 `withSubscription`，并且被包装的组件的显示名称是 `CommonList`，则使用 `WithSubscription(CommonList)` 作为显示名称：

```javascript
function withSubscription(WrappedComponent) {
  class WithSubscription extends React.Component {/* ... */}
  WithSubscription.displayName = `WithSubscription(${getDisplayName(WrappedComponent)})`;
  return WithSubscription;
}

function getDisplayName(WrappedComponent) {
  return WrappedComponent.displayName || WrappedComponent.name || 'Component';
}
```
----

## 注意事项：

高阶组件有几个注意事项，如果你刚接触 React，可能并不是特别明显。

### 不要在 render 方法中使用高阶组件

React 的 diffing 算法（称为 reconciliation）使用组件标识来确定它是否应该更新现有的子树还是将其丢弃并插入新的子树。如果从 `redner`返回的组件和之前渲染的组件是完全相等的(`===`)，则 React 会对子树进行递归比较。如果他们不相等，则先前的子树将被完全卸载。

通常，你不需要考虑从这个问题。但是，对于 HOC 来说很重要，因为这意味着你不能将 HOC 应用到一个组件的 render 方法中的组件。

```javascript
render() {
  // 一个新的 EnhancedComponent 在每次渲染都被创建
  // EnhancedComponent1 !== EnhancedComponent2
  const EnhancedComponent = enhance(MyComponent);
  // 会导致子树每次都被挂载和卸载
  return <EnhancedComponent />;
}
```

这里的问题不仅仅在于性能 —— 重新挂载一个组件会导致该组件及其所有子组件的 state 都会丢失。

相反，在组建定义之外应用 HOC，以便于生成的组件只能创建一次。那么，它的身份在渲染中是一致的。无论如何，这通常是你想要的结果。

在极少数的情况下，你需要动态的应用 HOC，你还可以在组建的生命周期方法或者其构造函数中执行这个操作。


### 静态方法必须被拷贝

有时候，在 React 组件上定义一个静态方法很有用。例如，Relay 容器暴露了一个静态方法 `getFragment` 来便于构建 GraphQL fragments。

当你将 HOC 应用在组件上的时候，原始组件将被容器组件所包装。这意味着新组建没有原始组件的任何方法。

```javascript
// 定义一个静态方法
WrappedComponent.staticMethod = function() {/*...*/}
// 应用 hoc
const EnhancedComponent = enhance(WrappedComponent);

// enhanced 组件没有任何的静态方法
typeof EnhancedComponent.staticMethod === 'undefined' // 
```

要解决这个问题，你可以在返回结果之前将静态方法拷贝到容器上：

```javascript
function enhance(WrappedComponent) {
  class Enhance extends React.Component {/*...*/}
  // 必须知道哪些静态方法需要宝贝
  Enhance.staticMethod = WrappedComponent.staticMethod;
  return Enhance;
}
```

然而，这需要你明确知道哪些方法需要拷贝。你可以通过[hoist-non-react-statics][9] 来自动拷贝所有非 React 本身的静态方法。

```javascript
import hoistNonReactStatic from 'hoist-non-react-statics';
function enhance(WrappedComponent) {
  class Enhance extends React.Component {/*...*/}
  hoistNonReactStatic(Enhance, WrappedComponent);
  return Enhance;
}
```

另外一种可能的解决方案是将静态方法和组件本身分开导出：

```javascript
// Instead of...
MyComponent.someFunction = someFunction;
export default MyComponent;

// ...export the method separately...
export { someFunction };

// ...and in the consuming module, import both
import MyComponent, { someFunction } from './MyComponent.js';
```


### Refs 无法传递

虽然高阶组件的使用惯例是通过所有的 props 传递给组件，但是无法传递 refs。这是因为 `ref` 并不是真正的 prop 属性，而是 React 的特殊处理的。如果你在一个HOC返回的组件中的元素上加了一个 ref ，ref 指的是最外城的容器组件的实例，而不是被包装的组件。

如果你发现自己面临这个问题，理想的解决方案是找出避免使用 ref 的方法。有时候， React 使用的新手会依赖于 ref，这种情况下，使用 prop 是更好的解决方案。

也就是说，有些时候，refs 是需要规避的 —— React 也不支持使用 refs。聚焦一个 input 输入框是你可能希望通过命令控制组件的一个例子，在这种情况下，一种解决方案是传递一个 ref 回调函数作为一个普通的 prop，并且给它一个不同的名称：

```javascript
function Field({ inputRef, ...rest }) {
  return <input ref={inputRef} {...rest} />;
}

// 在 HOC 中包装 Field
const EnhancedField = enhance(Field);

// 在类组件的渲染方法中
<EnhancedField
  inputRef={(inputEl) => {
    // This callback gets passed through as a regular prop
    this.inputEl = inputEl
  }}
/>

// Now you can call imperative methods
this.inputEl.focus();
```

这并不是一个完美的解决防范。我们倾向于将 ref 作为库的一部分内容，而不是要用户手动的去处理它们。我们正在探索这个问题的解决方法以便于使用 HOC 是可观察的。









[1]: https://reactjs.org/docs/higher-order-components.html
[2]: https://github.com/reactjs/react-redux/blob/master/docs/api.md#connectmapstatetoprops-mapdispatchtoprops-mergeprops-options
[3]: https://facebook.github.io/relay/docs/api-reference-relay.html#createcontainer-static-method
[4]: https://reactjs.org/blog/2016/07/13/mixins-considered-harmful.html
[5]: https://lodash.com/docs/#flowRight
[6]: http://redux.js.org/docs/api/compose.html
[7]: http://ramdajs.com/docs/#compose
[8]: https://github.com/facebook/react-devtools
[9]: https://github.com/mridgway/hoist-non-react-statics
