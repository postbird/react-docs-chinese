# Uncontrolled Components

- 英文原文：[https://reactjs.org/docs/uncontrolled-components.html][1]

**在大多数情况下，我们建议使用[可控的组件][2]实现表单。在可控的组件中，表单的数据通过React组件进行处理。当然还有一种替代解决方案是不可控的组件，在不可控组件中，表单数据由DOM本身去处理**


如果是编写不可控的组件，就不需要为每个state的更新编写一个事件处理函数，可以[通过ref获取DOM中的表单的值][3]。

例如，下面代码在不可控的组件中只有一个name的值:

```javascript
class NameForm extends React.Component {
  constructor(props) {
    super(props);
    this.handleSubmit = this.handleSubmit.bind(this);
  }

  handleSubmit(event) {
    alert('A name was submitted: ' + this.input.value);
    event.preventDefault();
  }

  render() {
    return (
      <form onSubmit={this.handleSubmit}>
        <label>
          Name:
          <input type="text" ref={(input) => this.input = input} />
        </label>
        <input type="submit" value="Submit" />
      </form>
    );
  }
}
```
[Try it on CodePan][4]

由于不可控组件中保留了数据在DOM中的真实来源，所以看起来在某些时候，不可控的组件更容易集成React和非React的代码。如果你想要快速但是dirty的话，这种方式也能够减少一些代码。通常，还是应当使用受控的组件。


如果不清楚为某种情况使用哪种类型（可控或不可控）的组件，可以在这篇[关于可控核不可控组件的建议][5]中寻找帮助。


## 默认值

在React的渲染周期中，from上的`value`属性会覆盖DOM中的value。如果使用不可控组件，经常需要通过React指定一个初始值，但后续更新的时候不受控制。在这种情况下，你可以通过`defaultValue`而不是`value`来实现。

> 译者注：如果指定的是`value`属性，之后是无法编辑输入框更改内容的，且`value`的更新会导致输入框内容同样更新，而使用`defaultValue`没有这种副作用。

```javascript
render() {
  return (
    <form onSubmit={this.handleSubmit}>
      <label>
        Name:
        <input
          defaultValue="Bob"
          type="text"
          ref={(input) => this.input = input} />
      </label>
      <input type="submit" value="Submit" />
    </form>
  );
}
```

同样，`<input type="checkbox">` and `<input type="radio">`支持`defaultChecked`,而`<select>`和`<textarea>`支持`defaultValue`.









[1]: https://reactjs.org/docs/uncontrolled-components.html
[2]: https://reactjs.org/docs/forms.html
[3]: https://reactjs.org/docs/refs-and-the-dom.html
[4]: https://codepen.io/gaearon/pen/WooRWa?editors=0010
[5]: http://goshakkk.name/controlled-vs-uncontrolled-inputs-react/