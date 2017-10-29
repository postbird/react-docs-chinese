# Typechecking With PropTypes

>**注意** ： `React.PropTypes`从`React v15.5`开始迁移到不同的包中，请使用`prop-types`库。我们提供一个[`codemod`脚本][1]来自动化转换

随着应用的增长，你可以通过**类型检查**来捕获大量的bug。

对于一些应用来说，你可以使用如[`Flow`][2]和[`TypeScript`][3]等JavaScript扩展来进行全局的应用类型检查。但是即使你不使用他们，React也有一些内置的类型检查功能。

如果需要在组件的props上进行类型检查，可以通过赋值特殊的`propTypes`进行：

```javascript
import PropTypes from 'prop-types';

class Greeting extends React.Component {
  render() {
    return (
      <h1>Hello, {this.props.name}</h1>
    );
  }
}

Greeting.propTypes = {
  name: PropTypes.string
};
```

`PropTypes`导出的是一系列验证器用于验证接收到的数据是否是可用的。上述例子中，我们使用了`PropTypes.string`.

如果接收了一个无效的prop，JavaScript控制台(console)会出现警告。出于性能原因，仅在开发模式下检查`propTypes`。

## PropTypes

下面是一些不同的验证器的示例：

```javascript
import PropTypes from 'prop-types';
import React,{Component} from 'react';

class MyComponent extends Component{};
MyComponent.propTypes = {
  // 声明的prop可以是一个特殊的JS基础变量，默认情况下，下面都是可选的
  optionalArray: PropTypes.array,
  optionalBool: PropTypes.bool,
  optionalFunc: PropTypes.func,
  optionalNumber: PropTypes.number,
  optionalObject: PropTypes.object,
  optionalString: PropTypes.string,
  optionalSymbol: PropTypes.symbol,

  // 下面示例能够渲染任何元素: numbers, strings, elements ，array, fragment
  optionalNode: PropTypes.node,

  // 需要是 React 元素
  optionalElement: PropTypes.element,

  // 可以声明 prop 是某个类的示例
  optionalMessage: PropTypes.instanceOf(Message),

  // 可以声明 prop 在某个 enum 中的一个
  optionalEnum: PropTypes.oneOf(['News', 'Photos']),

  // 用来验证prop对象中的每一个属性
  optionalUnion: PropTypes.oneOfType([
    PropTypes.string,
    PropTypes.number,
    PropTypes.instanceOf(Message)
  ]),

  // 验证 prop 数组的每个子元素的类型
  optionalArrayOf: PropTypes.arrayOf(PropTypes.number),

  // 检查 prop 对象的属性的类型
  optionalObjectOf: PropTypes.objectOf(PropTypes.number),

  // 用来检查 prop 对象的每个属性的类型
  optionalObjectWithShape: PropTypes.shape({
    color: PropTypes.string,
    fontSize: PropTypes.number
  }),

  // 检查 prop 是必须存在的（required）
  requiredFunc: PropTypes.func.isRequired,

  // 用来检查任意的数值都必须存在
  requiredAny: PropTypes.any.isRequired,

  // 你可以通过自定义验证器的方法来进行验证。
  // 自定义验证器应当返回一个抛出错误的Error对象。
  // 不要使用`console.warn`或者throw抛出错误，因为无法再 oneOfType 中使用
  customProp: function(props, propName, componentName) {
    if (!/matchme/.test(props[propName])) {
      return new Error(
        'Invalid prop `' + propName + '` supplied to' +
        ' `' + componentName + '`. Validation failed.'
      );
    }
  },

  // 你也可以为'arrayOf'和'objectOf'提供自定义验证器
  // 如果验证失败，应该返回一个Error对象
  // 数组或者对象的每一个key都会被调用这个验证器。
  // 此验证器的前面两个参数是数组或者是对象本身以及当前遍历的index(如数组下标或对象属性key)
  customArrayProp: PropTypes.arrayOf(function(propValue, key, componentName, location, propFullName) {
    if (!/matchme/.test(propValue[key])) {
      return new Error(
        'Invalid prop `' + propFullName + '` supplied to' +
        ' `' + componentName + '`. Validation failed.'
      );
    }
  })
};
```

## 检查单Children的类型

通过`PropTypes.element`，可以用来验证一个单children的类型：

```javascript
import PropTypes from 'prop-types';

class MyComponent extends React.Component {
  render() {
    // This must be exactly one element or it will warn.
    const children = this.props.children;
    return (
      <div>
        {children}
      </div>
    );
  }
}

MyComponent.propTypes = {
  children: PropTypes.element.isRequired
};
```

## Prop 默认值

你可以通过`defaultProps`来为`props`定义默认值：

```javascript
class Greeting extends React.Component {
  render() {
    return (
      <h1>Hello, {this.props.name}</h1>
    );
  }
}

// Specifies the default values for props:
Greeting.defaultProps = {
  name: 'Stranger'
};

// Renders "Hello, Stranger":
ReactDOM.render(
  <Greeting />,
  document.getElementById('example')
);
```
如果父组件没有指定`props`，则`defaultProps`能够指定一个默认的`this.props.name`.

当然，即使指定了`defaultProps`，`propTypes`的类型检查也会作用于`defaultProps`.





[1]: https://reactjs.org/blog/2017/04/07/react-v15.5.0.html#migrating-from-react.proptypes
[2]: https://flowtype.org/
[3]: https://www.typescriptlang.org/