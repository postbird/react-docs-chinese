# Refs and the DOM

- 英文文档：[https://reactjs.org/docs/refs-and-the-dom.html][1]

在典型的 React 数据流中，`props` 是父组件与子组件交互的唯一方式。如果需要修改子组件，可以通过修改 `props` 重新渲染。

然而，在有些情况下，除了使用典型的交互模式外，需要强制的修改子组件。要修改的子组件可能是React的示例，也可能是DOM元素，而React提供了一种可以同时作用于上述两种情况的解决方案。

## 什么时候使用Refs

下面是几个使用 Refs 的示例：

- 管理焦点状态(focus)、文本选择(text selection)、或者是媒体播放(media)

- 强制触发动画

- 与第三方的DOM库集成

在任何能够通过直接声明完成的事情中应当避免使用Refs。

例如，对于一个`Dialog`组件，应当提供一个`isOpen`的`prop`来控制它，而不是暴露`open()`和`close()`两个方法去操作。

## 不要过度使用Refs

可能你会倾向于在app中使用Refs来 **"触发某些事件"** ，如果是这样的话，请花点儿时间思考一下，在不同的组件层次结构中的state应当如何去分发（state的层次结构设计）。

通常来说，拥有satte的组件的层次结构应当较高（通过层次高的组件，将state通过props分发到子组件中）。

可以到[`Lifting State Up`][2]文档来查看示例。

## 在DOM元素上使用Ref

React提供了一个可以加到任何组件上面的特殊属性，`ref`属性使用了一个回调函数，并且回调函数在组件挂载(mounted)或者是销毁(unmounted)后立即执行。

当 `ref` 属性作用于 HTML 元素上时，`ref`的回调函数接收底层的DOM元素作为其参数，例如，下面代码中，使用`ref`回调存储DOM节点的引用。

```javascript
class CustomTextInput extends React.Component {
  constructor(props) {
    super(props);
    this.focusTextInput = this.focusTextInput.bind(this);
  }

  focusTextInput() {
    // Explicitly focus the text input using the raw DOM API
    this.textInput.focus();
  }

  render() {
    // Use the `ref` callback to store a reference to the text input DOM
    // element in an instance field (for example, this.textInput).
    return (
      <div>
        <input
          type="text"
          ref={(input) => { this.textInput = input; }} />

        <input
          type="button"
          value="Focus the text input"
          onClick={this.focusTextInput}
        />
      </div>
    );
  }
}
```
组件挂载是，React将DOM元素作为参数并且调用ref回调函数，而在卸载时将`null`作为参数调用ref的回调。

通过ref回调函数指定组件类中的某个属性为DOM元素是常见的使用方式，首选的方式是上面示例中的回调函数，当然还有更简短的使用方式：`ref={input => this.textInput = input}`.

## 在组件类上使用 Ref

当在声明为类的自定义组件上使用 `ref` 属性的时候,`ref`回调函数接收组件的已挂载实例作为回调的参数。

例如，如果我们想要在`CustomTextInput`组件上面模拟该组件挂在后立即触发点击事件：

```javascript
class AutoFocusTextInput extends React.Component {
  componentDidMount() {
    this.textInput.focusTextInput();
  }
  render() {
    return (
      <CustomTextInput
        ref={(input) => { this.textInput = input; }} />
    );
  }
}
```
需要注意的是，仅当`CustomTextInput`被声明为`class`的时候的时候才起作用：

```javascript
class CustomTextInput extends React.Component {
  // ...
}
```

## Refs 和 Function构造的组件

**你不应该将 ref 属性作用域 Function 声明的组件**，因为没有实例：

```javascript
function MyFunctionalComponent() {
  return <input />;
}

class Parent extends React.Component {
  render() {
    // This will *not* work!
    return (
      <MyFunctionalComponent
        ref={(input) => { this.textInput = input; }} />
    );
  }
}
```

如果你要使用 ref ，需要将组件转 class 定义的组件，就像使用生命周期的方法的时候一样，都需要将其转成 class 定义的组件。

当然，**在 function 声明的组件中使用 ref 的话**，按照正常的使用方式即可。

```javascript
function CustomTextInput(props) {
  // textInput must be declared here so the ref callback can refer to it
  let textInput = null;

  function handleClick() {
    textInput.focus();
  }

  return (
    <div>
      <input
        type="text"
        ref={(input) => { textInput = input; }} />

      <input
        type="button"
        value="Focus the text input"
        onClick={handleClick}
      />
    </div>
  );  
}
```

## 将 DOM Refs 暴露给父组件

在极少数情况，你可能将子组件的DOM节点传递给父组件。通常来讲不建议这样做，因为会破坏组件的封装。但是偶尔情况下，对于触发焦点或者得到子组件的DOM节点的大小或者位置很有帮助。

虽然你可以[向子组件增加一个ref属性][3],但是并不是理想的解决方案，因为获取到的是组件实例而不是DOM节点。

此外，这种方式对于 funcion 定义的组件无效。

在这种情况下，我们建议通过子组件暴露 ref 属性。子组件通过`prop`（如父组件传递过来的 inputRef）作为ref的属性添加到子组件的DOM节点上。从而实父组件能够通过中间的 inputRef 来使用子组件的DOM节点。

这种方法对于 class 声明的组件和 function 声明的组件都适用：

```javascript
function CustomTextInput(props) {
  return (
    <div>
      <input ref={props.inputRef} />
    </div>
  );
}

class Parent extends React.Component {
  render() {
    return (
      <CustomTextInput
        inputRef={el => this.inputElement = el}
      />
    );
  }
}
```

在上面的例子中，`Parent`把它的ref回调作为一个`inputRef` prop 传递给了 `CustomTextInput`，并且`CustomTextInput`把回调函数传递给了`<input>`.因此，`Paren`中的`this.inputElement`便是`CustomTextInput`组件中`<input>`节点。

值得注意的是，上面示例中`inputRef`并没有什么特殊的意义，只是一个正常的prop。不过，`<input>`上的`red`属性很重要，因为它告诉 React 通过一个ref链接到`<input>`节点。

即使`CustomTextInput`是通过 fucntion 定义的组件也依旧能够正常工作（译者注：上面的示例就说明了这一点）。原因在于，和 `ref`属性不同（[ref属性只能作用于class声明的组件，而不能作用于function声明的组件][4]），`inputRef`只是一个普通的prop，两种类型声明的组件都能够使用。


这种方式的另一个优点在于能够很好的作用于多重嵌套的组件。假设`Parent`不需要`input`这个DOM节点，但是`Parent`的父组件（如：`Grandparent`）需要使用`<input>`DOM节点。这事，可以通过为`Grandparent`指定一个`inputRef`，然后再通过`Parent`的`inputRef`传递到`CustomTextInput`.

```javascript

function CustomTextInput(props) {
  return (
    <div>
      <input ref={props.inputRef} />
    </div>
  );
}

function Parent(props) {
  return (
    <div>
      My input: <CustomTextInput inputRef={props.inputRef} />
    </div>
  );
}


class Grandparent extends React.Component {
  render() {
    return (
      <Parent
        inputRef={el => this.inputElement = el}
      />
    );
  }
}
```

上述例子中，首先在`Grandparent`声明一个ref的回调，然后传递给了`Parent`组件，传递的方式是使用了一个`inputRef`的prop。然后`Parent`组件将这个prop以同样的名称`inputRef`传递给了`CustomTextInput`。最后`CustomTextInput`组件读取`inputRef`参数，并将传递的函数作为ref的属性加到`<input>`节点上。所以`Grandparent`中的`this.inputElement`指向`CustomTextInput`的`<input>`节点。

考虑到所有可能遇到的情况，还是建议尽量不要暴露DOM的节点，但是这确实是一种有效的处理方法。当然，需要注意的是，这种方法需要向子组件中增加一些代码。

如果你无法控制子组件的实例，还可以通过[`findDOMNode()`][5]来作为最后的可选方案，不过不鼓励使用这种方法。

## 之前版本的API：Refs字符串

如果是使用之前版本React，可能会熟悉一个比较旧的`ref`的API，将`ref`属性值设置为一个字符串，比如：`textInput`,可以通过`this.refs.textInput`来操作DOM节点。

我们建议尽量不要使用这种方式，因为字符串的refs会[导致一些遗留的问题][6],很可能**在之后的版本中，移除该API**。

如果你正在使用`this.refs.inputText`去访问refs,我们建议你使用新的回调模式来替代。

## 注意事项

如果`ref`回调函数被定义为一个内敛的函数，组件更新时会被调用两次，第一次参数是`null`，第二次参数是DOM节点。

这是因为每次渲染穿件一个新的函数实例，因此React需要清除之前的引用，然后设置新的引用。

可以通过将`ref`回调函数定义为`类上的绑定方法`来避免此情况。不过，多数情况下，关系不大。







[1]: https://reactjs.org/docs/refs-and-the-dom.html
[2]: https://reactjs.org/docs/lifting-state-up.html
[3]: https://reactjs.org/docs/refs-and-the-dom.html#adding-a-ref-to-a-class-component
[4]: https://reactjs.org/docs/refs-and-the-dom.html#refs-and-functional-components
[5]: https://reactjs.org/docs/react-dom.html#finddomnode
[6]: https://github.com/facebook/react/pull/8333#issuecomment-271648615