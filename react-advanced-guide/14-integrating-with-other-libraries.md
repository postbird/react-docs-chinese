# Integrating with Other Libraries

- 英文原文：[https://reactjs.org/docs/integrating-with-other-libraries.html][1]

React 可用于任何 Web 应用程序。它可以嵌入到其他应用程序中，并且一些情况下，其他的应用也能够嵌入到 React 中。这篇文档会举一下比较常见的用例，将关注点放在和 [`jQuery`][2] 和 [`Backbone`][3] 上，但是这种思想也适用于组件与其他现有的任何代码集成。

----

## 和 DOM 操作插件进行集成

React 无法知道在 React 之外进行的 DOM 操作。它是根据自己内部的表现形式来确定是否更新，如果同一个 DOM 节点被另一个库操作，React 会无法识别并且无法恢复。

这并不意味着将 React 与 其他操作 DOM 的方式进行集成是不现实的甚至难以理解的，你只需要关心每个部分完成什么操作即可。

避免冲突的最简单的方法就是阻止 React 组件更新。你可以通过渲染没有触发 React 更新条件的元素，如空的 `<div />`。

### 如何解决问题：

为了演示上述问题，我们来封装一个通用的 jQuery 插件的包装器。

我们将给根节点的 DOM 元素增加一个 [ref][4]。在 `componentDidMount` 中，我们将得到一个引用，所以我们可以将其传递给 jQuery 插件。

为了防止在挂载后 React 操作 DOM，我们从 `render()` 方法中返回一个空的 `<div/>`。这个 `<div/>` 元素没有属性或者是子节点，所以 React 没有更新它的条件，能够让 jQuery 插件自由的管理 DOM 的一部分。

```javascript
class SomePlugin extends React.Component {
  componentDidMount() {
    this.$el = $(this.el);
    this.$el.somePlugin();
  }

  componentWillUnmount() {
    this.$el.somePlugin('destroy');
  }

  render() {
    return <div ref={el => this.el = el} />;
  }
}

```

注意，我们定义了 `componentDidMount` 和 `componentWillUnMount` 两个[生命周期的钩子函数][5]。许多 jQuery 插件将事件侦听器附加到 DOM 上，因此在 `componentWillUnmount` 中将其取消是非常重要的。如果插件没有提供清理方法，你可能不得不提供自己的方法，记住要删除所有事件的侦听器，以防止内存泄露。


### 与 jQuery 选择插件

关于这些概念的一个例子：为 [Chosen][6] 插件编写一个最小的包装容器，增加了 `<select>` 输入框。

> **注意：**
> 虽然这种方式是可行的，但并不意味着这是 React 应用程序中最佳的方法。我们建议你在能够使用 React 组件的地方使用组件。React组件在 React 的应用程序中更容易重用，并且通常能够更好的控制器行为和表现形式。

首先，我们来看看 Chosen 是如何操作 DOM。如果你在一个 `<select>` DOM 节点上调用它，它会读取原始 DOM 节点的属性，用一个内敛的样式隐藏它，然后在 `<select>` 之后插入一个单独的具有自己的表现形式的 DOM 节点。然后它会触发 jQuery 事件来通知我们发生的变动。

假设下面是我们正在使用我们的 `<Chosen>` 包装器的 React 组件的 API：

```javascript
function Example() {
  return (
    <Chosen onChange={value => console.log(value)}>
      <option>vanilla</option>
      <option>chocolate</option>
      <option>strawberry</option>
    </Chosen>
  );
}
```

为了简单起见，我们将其作为[不可控组件][7]来实现。

首先，我们在 `render()` 方法中创建一个空组件，返回的 `<select>` 包装在一个 `<div>` 中:

```javascript
class Chosen extends React.Component {
  render() {
    return (
      <div>
        <select className="Chosen-select" ref={el => this.el = el}>
          {this.props.children}
        </select>
      </div>
    );
  }
}

```

请注意我们是在 `<div>` 中包装 `<select>`的。这是必要的，因为 Chosen 会在传递给它的 `<select>` 节点后面附加另一个 DOM 元素。然后，对于 React 来说，`<div>` 总是只有一个子节点。这也是我们确保 React 的更新不会与 Chosen 插入的额外 DOM 节点相冲突。重要的是，如果你在 React 流之外修改 DOM，则必须确保 React 不会去操作这些 DOM 节点。

接下来，我们将实现生命周期钩子函数。我们需要通过参考 `componentDidMount` 中的 `<select>` 节点初始化 Chosen，并将其在 `componentWillUnmount` 中销毁：

```javascript
componentDidMount() {
  this.$el = $(this.el);
  this.$el.chosen();
}

componentWillUnmount() {
  this.$el.chosen('destroy');
}
```

[Try it on CodePen][8]

注意，React 对于 `this.el` 没有特别的意义，它只是因为我们之前已经在 `render()` 方法中指定了这个字段而能够正常工作。


```javascript
<select className="Chosen-select" ref={el => this.el = el}>
```

这足够使得我们的组件被渲染，但是我们也希望得到有关值变化的通知。为了做到这一点，我们将在 `<select>` 上订阅由 Chosen 管理的 jQuery 的 `change` 事件。

我们不会将 `this.props.onChange` 直接传递给 Chosen,因为组件的 prop 可能会在之后发生变化，并且其包括事件处理程序。 相反，我们声明一个调用 `this.props.onChange` 的 `handleChange()` 方法，并将其订阅到 jQuery 的`change` 事件上。

```javascript
componentDidMount() {
  this.$el = $(this.el);
  this.$el.chosen();

  this.handleChange = this.handleChange.bind(this);
  this.$el.on('change', this.handleChange);
}

componentWillUnmount() {
  this.$el.off('change', this.handleChange);
  this.$el.chosen('destroy');
}

handleChange(e) {
  this.props.onChange(e.target.value);
}
```

[Try it on CodePen][9]

最后，还有一件事情需要做。在 React 中，props 可以随着事件的推移而改变。比如，如果父组件的 state 发生变化，`<Chosen>` 组件能够获取到不同的子节点。这意味着在集成点上，我们手动的更新 DOM 来响应更新是非常重要的，因为 React 不再管理我们的 DOM。

Chosen 的文档表明我们可以使用 jQuery 的 `trigger()` API 去通知它关于原始 DOM 元素的更改。我们将让 React 在 `<select>` 内部更新 `this.props.children`，但是我们还将增加一个 `componentDidUpdate` 生命周期钩子，通知 Chosen 关于子列表中的更改：

```javascript
componentDidUpdate(prevProps) {
  if (prevProps.children !== this.props.children) {
    this.$el.trigger("chosen:updated");
  }
}
```
这样，当 React 管理的 `<select>` 子项更改时，Chosen 将指导更新它的 DOM 元素。

完整的 Chosen 组件的实现如下所示：

```javascript
class Chosen extends React.Component {
  componentDidMount() {
    this.$el = $(this.el);
    this.$el.chosen();

    this.handleChange = this.handleChange.bind(this);
    this.$el.on('change', this.handleChange);
  }
  
  componentDidUpdate(prevProps) {
    if (prevProps.children !== this.props.children) {
      this.$el.trigger("chosen:updated");
    }
  }

  componentWillUnmount() {
    this.$el.off('change', this.handleChange);
    this.$el.chosen('destroy');
  }
  
  handleChange(e) {
    this.props.onChange(e.target.value);
  }

  render() {
    return (
      <div>
        <select className="Chosen-select" ref={el => this.el = el}>
          {this.props.children}
        </select>
      </div>
    );
  }
}
```

[Try it on CodePen][10]


----

## 与其他的 view 库集成

由于 `ReactDOM.render()` 的灵活性，React 能够嵌入到其他的应用程序中。

虽然 React 在启动时将单个根 React 组件加载到 DOM 中，但也可以为 UI 的独立部分多次调用 `ReactDOM.render()`,这可以与按钮一样小或者与应用程序一样大。

事实上，这也是 Facebook 在使用 React 的方式。这使得 我们可以逐个编写应用程序，并将其与现有的服务器生成的模板和其他的客户端代码相结合。


### 使用 React 替换基于字符串的渲染

以前的 Web 应用程序中，常见的模式是将 DOM 块作为字符串进行描述，并将其插入到 DOM 中，比如：`$el.html(htmlString)`. 这些方面很适合引入 React，只需要将字符串的渲染重写为 React 的组件即可。


所以下面的 jQuery 实现的内容...

```javascript
$('#container').html('<button id="btn">Say Hello</button>');
$('#btn').click(function() {
  alert('Hello!');
});
```

... 能够通过 React 组件重写：

```javascript
function Button() {
  return <button id="btn">Say Hello</button>;
}

ReactDOM.render(
  <Button />,
  document.getElementById('container'),
  function() {
    $('#btn').click(function() {
      alert('Hello!');
    });
  }
);
```

自此，你可以开始将更多的逻辑转移到组件中，并开始更多的采用 React 去实现。例如，在组建中，最好不要依赖 ID 属性，因为可以多次渲染相同的组件。相反，我们将使用 [React 事件系统][11] 直接在 React 的 `<button>` 元素上注册 click 处理函数：

```javascript
function Button(props) {
  return <button onClick={props.onClick}>Say Hello</button>;
}

function HelloButton() {
  function handleClick() {
    alert('Hello!');
  }
  return <Button onClick={handleClick} />;
}

ReactDOM.render(
  <HelloButton />,
  document.getElementById('container')
);
```

[Try it on CodePen.][12]

你可以创建许多这样的隔离的组件，并且使用 `ReactDOM.render()` 将她们渲染到不同的 DOM 容器中。逐渐的，当你将更多的应用转换为 React 时，你将能够把它们组合成较大的组件，并且将 `ReactDOM.render()` 调用高更高的层级中。

### 在 Backbone View 中嵌入 React

[Backbone][13] 视图通常使用 HTML 字符串或者通过字符串生成模板函数来为其 DOM 元素创建内容。这个过程也可以使用 React 渲染。

下面，我们将创建一个名为 `ParagraphView` 的 `Backbone` 视图，它将覆盖 Backbone 的 `render()` 函数，从而将 React 的 `<Paragraph>` 组件渲染到 Backbone (`this.el`) 提供的 DOM 元素中。这里，我们同样使用 [ReactDOM.render()][14]:
 
```javascript
function Paragraph(props) {
  return <p>{props.text}</p>;
}

const ParagraphView = Backbone.View.extend({
  render() {
    const text = this.model.get('text');
    ReactDOM.render(<Paragraph text={text} />, this.el);
    return this;
  },
  remove() {
    ReactDOM.unmountComponentAtNode(this.el);
    Backbone.View.prototype.remove.call(this);
  }
});
```
[Try it on CodePen.][15]


在 `remove` 方法中调用 `ReactDOM.unmountComponentAtNode()` 方法是非常重要的，以便于在卸载的时候，React 注销与组件树相关联的事件处理程序以及其他的资源。

当从 React 树中删除组件时，会自动执行清理工作，但是由于我们正在删除整个树，所以我们必须调用这个方法。


----


## 与模型层集成

虽然通常会使用单项数据流，如 [React state][16]，[Flux][17] 或者是 [Redux][18] ，但是 React 组件可以使用其他框架和库的模型层。

### 在 React 组件中使用 Backbone 的 Models

将 [Backbone][13] 的模型层与 React 组件相结合的最简单的方式就是监听各个 change 事件，并且手动强制更新。

负责渲染模型的组件将监听 `change` 事件，而负责渲染整个集合的组件将监听 `add` 和 `remove` 事件。在这两种情况下，调用 [this.forceUpdate()][19] 来使用新的数据重新渲染该组件。

在下面这个例子中，`List` 组件使用 `Item` 组件呈现单个项目来显示 Backbone 集合。

```javascript
class Item extends React.Component {
  constructor(props) {
    super(props);
    this.handleChange = this.handleChange.bind(this);
  }

  handleChange() {
    this.forceUpdate();
  }

  componentDidMount() {
    this.props.model.on('change', this.handleChange);
  }

  componentWillUnmount() {
    this.props.model.off('change', this.handleChange);
  }

  render() {
    return <li>{this.props.model.get('text')}</li>;
  }
}

class List extends React.Component {
  constructor(props) {
    super(props);
    this.handleChange = this.handleChange.bind(this);
  }

  handleChange() {
    this.forceUpdate();
  }

  componentDidMount() {
    this.props.collection.on('add', 'remove', this.handleChange);
  }

  componentWillUnmount() {
    this.props.collection.off('add', 'remove', this.handleChange);
  }

  render() {
    return (
      <ul>
        {this.props.collection.map(model => (
          <Item key={model.cid} model={model} />
        ))}
      </ul>
    );
  }
}
```
[Try it on CodePen.][20]

### 从 Backbone Models 中提取数据

上述方法需要你的 React 组件了解 Backbone 模型和集合内容。如果你以后计划迁移到另一个数据管理解决方案，你可能希望尽可能将关于 Backbone 的部分的代码集中起来方便迁移。

一种解决方案是将模型的属性作为普通数据随时更改，并将其保留在一个单一的位置。下面这个 [higher-order component(高阶组件)][21] 是将 Backbone 模型的所有属性提取到 state 中，将数据传递到包装组件中。

这样，只有高阶组件需要了解 Backbone 模型的内部部件，并且应用程序中的大多数组件可以保持与 Backbone 无关。

在下面的例子中，我们将拷贝模型的属性去初始化 state。我们订阅了 `change` 事件（并且在卸载时取消订阅），当触发 `change` 事件时，我们使用模型的当前属性更新状态。最后，我们确保 如果 `model` prop 自身发生变化，我们需要取消订阅旧的模型，并且订阅新的模型、

请注意，下面的例子并不关心与 Backbone 使用的相关细节，但是它会给你一个解决这些问题的通用方法的思考：

```javascript
function connectToBackboneModel(WrappedComponent) {
  return class BackboneComponent extends React.Component {
    constructor(props) {
      super(props);
      this.state = Object.assign({}, props.model.attributes);
      this.handleChange = this.handleChange.bind(this);
    }

    componentDidMount() {
      this.props.model.on('change', this.handleChange);
    }

    componentWillReceiveProps(nextProps) {
      this.setState(Object.assign({}, nextProps.model.attributes));
      if (nextProps.model !== this.props.model) {
        this.props.model.off('change', this.handleChange);
        nextProps.model.on('change', this.handleChange);
      }
    }

    componentWillUnmount() {
      this.props.model.off('change', this.handleChange);
    }

    handleChange(model) {
      this.setState(model.changedAttributes());
    }

    render() {
      const propsExceptModel = Object.assign({}, this.props);
      delete propsExceptModel.model;
      return <WrappedComponent {...propsExceptModel} {...this.state} />;
    }
  }
}
```

为了演示如何使用它，我们讲 `NameInput` React 组件连接到 Backbone 模型，并在每次输入更改时，更新它的 `firstName` 属性：

```javascript
function NameInput(props) {
  return (
    <p>
      <input value={props.firstName} onChange={props.handleChange} />
      <br />
      My name is {props.firstName}.
    </p>
  );
}

const BackboneNameInput = connectToBackboneModel(NameInput);

function Example(props) {
  function handleChange(e) {
    model.set('firstName', e.target.value);
  }

  return (
    <BackboneNameInput
      model={props.model}
      handleChange={handleChange}
    />

  );
}

const model = new Backbone.Model({ firstName: 'Frodo' });
ReactDOM.render(
  <Example model={model} />,
  document.getElementById('root')
);
```
[Try it on CodePen][22]

这种方式不限于 Backbone。你可以通过订阅其生命周期钩子函数中的 changes 以及可选择性的将数据复制到本地的 React state，将 React 与任何模型库配合使用。




[1]: https://reactjs.org/docs/integrating-with-other-libraries.html
[2]: https://jquery.com/
[3]: http://backbonejs.org/
[4]: https://reactjs.org/docs/refs-and-the-dom.html
[5]: https://reactjs.org/docs/react-component.html#the-component-lifecycle
[6]: https://harvesthq.github.io/chosen/
[7]: https://reactjs.org/docs/uncontrolled-components.html
[8]: http://codepen.io/gaearon/pen/qmqeQx?editors=0010
[9]: http://codepen.io/gaearon/pen/bWgbeE?editors=0010
[10]: http://codepen.io/gaearon/pen/xdgKOz?editors=0010
[11]: https://reactjs.org/docs/handling-events.html
[12]: http://codepen.io/gaearon/pen/RVKbvW?editors=1010
[13]: http://backbonejs.org/
[14]: https://reactjs.org/docs/react-dom.html#render
[15]: http://codepen.io/gaearon/pen/gWgOYL?editors=0010
[16]: https://reactjs.org/docs/lifting-state-up.html
[17]: http://facebook.github.io/flux/
[18]: http://redux.js.org/
[19]: https://reactjs.org/docs/react-component.html#forceupdate
[20]: http://codepen.io/gaearon/pen/GmrREm?editors=0010
[21]: https://reactjs.org/docs/higher-order-components.html
[22]: http://codepen.io/gaearon/pen/PmWwwa?editors=0010