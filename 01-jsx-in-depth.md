# JSX In Depth

- 英文文档：[https://reactjs.org/docs/jsx-in-depth.html][7]

本质上，JSX只是`React.createElement(component, props, ...children)`的语法糖，比如下面代码：
```html
<MyButton color="blue" shadowSize={2}>
  Click Me
</MyButton>
```

将被编译成：

```javascript
React.createElement(
  MyButton,
  {color: 'blue', shadowSize: 2},
  'Click Me'
)
```

如果没有子标签的话，也可以使用自闭和标签：

```html
<div className="sidebar" />
```

会被编译成：

```javascript
React.createElement(
  'div',
  {className: 'sidebar'},
  null
)
```

如果想测试JSX如何被编译成JavaScript，可以尝试[Babel在线编译器][1]

----

# 指定React的元素类型

JSX标签的第一部分明确了React元素的类型。

使用大写的JSX标签来表示React的一个组件。这些标签编译成对命名变量的直接的引用，所以，如果使用JSX标签`<Foo />`表达式，则`Foo`必须在作用域内。（译者注：必须引入组件）


## 作用域内必须引入React

由于JSX会被编译成`React.createElement`,因此在JSX的代码中，必须引入`React`库。

例如,下面代码中即使`React`和`CustomButton`没有直接在JavaScript中使用，但是这两者都必须在一起引入:

```javascript
import React from 'react';
import CustomButton from './CustomButton';

function WarningButton() {
  // return React.createElement(CustomButton, {color: 'red'}, null);
  return <CustomButton color="red" />;
}
```

如果使用`<script>`标签加载React，而不是使用模块化加载，React也是在作用域内的，此时React是全局作用域中。

## JSX使用点(`.`)操作符

在JSX中可以使用点(`.`)操作符使用React组件，如果通过一个模块导出多个React组件的话，使用点操作符非常的使用和方便。

例如，如果`MyComponents.DatePicker`是一个组件，可以在JSC中直接向下面这样使用：

```javascript
import React from 'react';

const MyComponents = {
  DatePicker: function DatePicker(props) {
    return <div>Imagine a {props.color} datepicker here.</div>;
  }
}

function BlueDatePicker() {
  return <MyComponents.DatePicker color="blue" />;
}
```
## 自定义组件必须大写


如果定义的组件使用小写开头，他会把`<div>`,`<span>`这样的内置组件作为`div`和`span`字符串传递给`React.createElement`。

如果以大写开头， 比如`<Foo/>`会被编译成`React.createElement(Foo)`,并且`Foo`对应在jsx文件中引入的组件.

React推荐使用大写开头定义组件，如果有小写开头的组件，则在jsx中使用前，应当将其赋值给大写的变量。

例如，下面的代码将不会起作用：

```javascript
import React from 'react';

// Wrong! This is a component and should have been capitalized:
function hello(props) {
  // Correct! This use of <div> is legitimate because div is a valid HTML tag:
  return <div>Hello {props.toWhat}</div>;
}

function HelloWorld() {
  // Wrong! React thinks <hello /> is an HTML tag because it's not capitalized:
  return <hello toWhat="World" />;
}
```

为了解决上面的问题，需要将`hello`改成`Hello`,并且使用`<Hello/>`:

```javascript
import React from 'react';

// Correct! This is a component and should be capitalized:
function Hello(props) {
  // Correct! This use of <div> is legitimate because div is a valid HTML tag:
  return <div>Hello {props.toWhat}</div>;
}

function HelloWorld() {
  // Correct! React knows <Hello /> is a component because it's capitalized.
  return <Hello toWhat="World" />;
}
```

## 在运行的时候选择组件

不能使用通用表达式来作为React的元素类型，如果你想使用一个通用表达式来指明元素类型，首先需要将其赋值给大写的变量。

这种情况经常发生在需要根据prop来渲染不同的组件的情况：

(下面代码会出错)

```javascript
import React from 'react';
import { PhotoStory, VideoStory } from './stories';

const components = {
  photo: PhotoStory,
  video: VideoStory
};

function Story(props) {
  // Wrong! JSX type can't be an expression.
  return <components[props.storyType] story={props.story} />;
}
```
为了解决这个问题,需要将标签赋值给大写的变量：

```javascript
import React from 'react';
import { PhotoStory, VideoStory } from './stories';

const components = {
  photo: PhotoStory,
  video: VideoStory
};

function Story(props) {
  // Correct! JSX type can be a capitalized variable.
  const SpecificStory = components[props.storyType];
  return <SpecificStory story={props.story} />;
}
```

----

# JSX中的Props

在JSX中指定props有几种不同的方式。


## 使用JavaScript表达式作为Props

对于`MyComponent`,`props.foo`的值是`10`,因为`1 + 2 + 3 + 4`结果是10.

由于`if`语句和`for`循环不是JavaScript的表达式，所以不能直接在JSX使用。不过，可以在JSX外部的代码中使用，比如：

```javascript
function NumberDescriber(props) {
  let description;
  if (props.number % 2 == 0) {
    description = <strong>even</strong>;
  } else {
    description = <i>odd</i>;
  }
  return <div>{props.number} is an {description} number</div>;
}
```

你可以在[条件渲染][2]和[循环][3]部分了解更多。

## 字符串文本

你可以将字符串文本作为prop,下面两个JSX表达式是等价的：

```html
<MyComponent message="hello world" />

<MyComponent message={'hello world'} />
```

如果将字符串作为prop，则prop的值是对HTML的转义，因此下面两个JSX表达式是等价的：

```html
<MyComponent message="&lt;3" />

<MyComponent message={'<3'} />
```

不过两者没直接的什么关系，只是提一下。

## Prop的默认值是`True`
如果没有给prop指定值，则默认值是`true`,因此下面两个JSX表达式是等价的：

```html
<MyTextBox autocomplete />

<MyTextBox autocomplete={true} />
```

一般我们不建议这样去使用，因为这将会和[ES6对象简写][4]冲突。

比如 `{foo}` 会被看做`{foo:foo}`的简写，而不是`{foo:true}`的简写。

> This behavior is just there so that it matches the behavior of HTML.

## 扩展运算符传递属性

如果你的`props`作为一个对象，如果在JSX中使用这个对象传递`props`，可以通过扩展运算符`...`展开props对象,下面两个组件是等价的：

```javascript
function App1() {
  return <Greeting firstName="Ben" lastName="Hector" />;
}

function App2() {
  const props = {firstName: 'Ben', lastName: 'Hector'};
  return <Greeting {...props} />;
}
```

你也可以在利用展开运算符实现一些特殊的props：

```javascript
const Button = props => {
  const { kind, ...other } = props;
  const className = kind === "primary" ? "PrimaryButton" : "SecondaryButton";
  return <button className={className} {...other} />;
};

const App = () => {
  return (
    <div>
      <Button kind="primary" onClick={() => console.log("clicked!")}>
        Hello World!
      </Button>
    </div>
  );
};
```

在上面的例子中，`kind`不会传递给DOM中的`<button>`元素，只会被处理。

而其他的props则会通过`...other`对象传递给DOM元素，可以看到传递了一个`onClick`和`children`.

虽然扩展运算符能够非常方便的将props传递，但是也会将一些不需要的props或者是无效的HTML属性传递给DOM，所以建议谨慎使用这种语法。

----

# JSX中children props

JSX表达式中,除了自闭和标签外还有非自闭和标签，中间的内容通过一个特殊的prop,`props.children`。下面有几种不同的方式来传递这些props：

## 字符串文本

你可以在标签中间放一个字符串，`props.children`将只是该字符串。这对于许多内置的HTML元素很有用，比如：

```html
<MyComponent>Hello world!</MyComponent>
```

上述代码在JSX中是有用的，并且`MyComponent`中的`props.children`是字符串`Hello world!`.HTML会进行转移，因此可以像下面这样子写HTML元素：

```html
<div>This is valid HTML &amp; JSX at the same time.</div>
```

JSX会移除行开始和结尾的空格，同时也会删除空行。

与标签相邻的新行也会被移除，字符串文字中间的新行会被压缩成一个空格。

例如：

```html
<div>Hello World</div>

<div>
  Hello World
</div>

<div>
  Hello
  World
</div>

<div>

  Hello World
</div>
```

## JSX子组件

同样，JSX可以将JSX组件作为子元素，这对于显示嵌套的组件很有用：

```html
<MyContainer>
  <MyFirstComponent />
  <MySecondComponent />
</MyContainer>
```

可以混合不同类型的子元素，所以可以同时一起使用字符串文本作为JSX的子元素。就像使用HTML一样使用JSX，因此既是有用的JSX也是有用的HTML：

```html
<div>
  Here is a list:
  <ul>
    <li>Item 1</li>
    <li>Item 2</li>
  </ul>
</div>
```

React的组件同样能够返回一个html元素数组：

```javascript
render() {
  // No need to wrap list items in an extra element!
  return [
    // Don't forget the keys :)
    <li key="A">First item</li>,
    <li key="B">Second item</li>,
    <li key="C">Third item</li>,
  ];
}
```
## JavaScript表达式作为子元素

你可以将任意的JavaScript表达式作为Children，通过一个闭合的`{}`即可。例如，下面两个是等价的：

```html
<MyComponent>foo</MyComponent>

<MyComponent>{'foo'}</MyComponent>
```

经常用于显示任意长度的JSX表达式，比如呈现一个`todos`列表：

```javascript
function Item(props) {
  return <li>{props.message}</li>;
}

function TodoList() {
  const todos = ['finish doc', 'submit pr', 'nag dan to review'];
  return (
    <ul>
      {todos.map((message) => <Item key={message} message={message} />)}
    </ul>
  );
}
```

JavaScript 表达式能够和其他类型的表达式混合使用，比如替代字符串模板：

```javascript
function Hello(props) {
  return <div>Hello {props.addressee}!</div>;
}
```

## Children传递方法

通常来说，JSX中的JavaScript表达式会被转为字符串、React元素或者是其他。然而，`props.children`和其他的prop一样能够传递任何数据，而不仅仅是React能够渲染的内容。例如，如果你有一个自定义组件，你可以将其作为`props.children`的回调。

```javascript
// Calls the children callback numTimes to produce a repeated component
function Repeat(props) {
  let items = [];
  for (let i = 0; i < props.numTimes; i++) {
    items.push(props.children(i));
  }
  return <div>{items}</div>;
}

function ListOfTenThings() {
  return (
    <Repeat numTimes={10}>
      {(index) => <div key={index}>This is item {index} in the list</div>}
    </Repeat>
  );
}
```

传递给自定义组件的Children可以是任意的内容，只要组件能够将其转换成能够渲染的内容即可。这种用法并不常见，如果想要扩展JSX的功能，可以这样去使用。


## Booleans,Null和Undefined会被忽略

`false`、`null`、`undefined`和`true`都是正确的Children，他们只是不会被渲染。

这些JSX表达式最后渲染的结果是一样的：

```html
<div />

<div></div>

<div>{false}</div>

<div>{null}</div>

<div>{undefined}</div>

<div>{true}</div>
```
这对于条件渲染React元素很有用，比如下面的例子(如果`showHeader`是`true`,则会渲染`<Header/>`)：

(译者注：这里觉得文档中的表述并不是非常的好，下面的表述某种程度上容易被理解成只会渲染`<Header/>`而不渲染 `<Content/>`,虽然这是不可能的)

>This JSX only renders a `<Header />` if showHeader is true:

```html
<div>
  {showHeader && <Header/>}
  <Content/>
</div>
```

不过需要注意的是，一些[`falsy`值(实际上就是JavaScirpt中一些类型转换)][5],比如`0`,依旧会在React中进行渲染。

例如下面的例子，代码中不会按照预期的渲染，因为`props.message`是空数组的时候，会被渲染出`0`，而不会忽略。（译者注：因为`0`不是标准的Boolean）

```html
<div>
  {props.messages.length &&
    <MessageList messages={props.messages} />
  }
</div>
```

为了解决上述问题，需要确保在使用`&&`运算符的时候，前面必须是`Boolean`类型的：

```html
<div>
  {props.messages.length > 0 &&
    <MessageList messages={props.messages} />
  }
</div>
```
反之，如果你想将`false`、`true`、`null`或者`undefined`渲染出来，你需要将其[转成string][6]:

```html
<div>
  My JavaScript variable is {String(myVariable)}.
</div>
```


[1]: https://babeljs.io/repl/#?babili=false&evaluate=true&lineWrap=false&presets=es2015%252Creact%252Cstage-0&code=function%20hello()%20%7B%0A%20%20return%20%3Cdiv%3EHello%20world!%3C%252Fdiv%3E%253B%0A%7D
[2]: https://reactjs.org/docs/conditional-rendering.html
[3]: https://reactjs.org/docs/lists-and-keys.html
[4]: https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Operators/Object_initializer#New_notations_in_ECMAScript_2015
[5]: https://developer.mozilla.org/en-US/docs/Glossary/Falsy
[6]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String#String_conversion
[7]: https://reactjs.org/docs/jsx-in-depth.html