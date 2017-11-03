# React Without JSX

- 英文原文： [https://reactjs.org/docs/react-without-jsx.html][1]

使用React时，JSX并不是必须要使用的。如果你不想在构建项目的时候配置JSX的编译的时候，也可以不使用JSX。

其实每一个JSX的元素都是`React.createElement(component,props,...children)`的语法糖而已。因此，所有你能通过JSX实现的内容，均可以通过JavaScript完成。

比如，下面通过JSX实现的代码：

```javascript
class Hello extends React.Component {
  render() {
    return <div>Hello {this.props.toWhat}</div>;
  }
}

ReactDOM.render(
  <Hello toWhat="World" />,
  document.getElementById('root')
);
```

可以被编译成如下不使用JSX的代码：

```javascript
class Hello extends React.Component {
  render() {
    return React.createElement('div', null, `Hello ${this.props.toWhat}`);
  }
}

ReactDOM.render(
  React.createElement(Hello, {toWhat: 'World'}, null),
  document.getElementById('root')
);

```
如果你想了解JSX如何被编译成JavaScript代码的更多例子，可以试试[在线Babel编译器][2]

组件可以由普通的字符串提供，也可以由`React.Component`的子类提供，当然，也可以通过无状态的函数构建。

如果你不喜欢多次的键入`React.createElement`,也可以通过一个短的别名来使用：

```javascript
const e = React.createElement;

ReactDOM.render(
  e('div', null, 'Hello World'),
  document.getElementById('root')
);
```

如果你使用这种短别名的形式来使用`React.createElement`，它几乎可以同样方便的不借助JSX来使用React。

或者可以通过社区一些项目所提供的更简洁的语法，如[react-hyperscript][3] 和 [hyperscript-helpers][4].








[1]: https://reactjs.org/docs/react-without-jsx.html
[2]: https://babeljs.io/repl/#?babili=false&evaluate=true&lineWrap=false&presets=es2015%252Creact%252Cstage-0&code=function%20hello()%20%7B%0A%20%20return%20%3Cdiv%3EHello%20world!%3C%252Fdiv%3E%253B%0A%7D
[3]: https://github.com/mlmorg/react-hyperscript
[4]: https://github.com/ohanhi/hyperscript-helpers