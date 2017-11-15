# Glossary of React Terms (React 术语表)

- 英文原文：[https://reactjs.org/docs/glossary.html](https://reactjs.org/docs/glossary.html)

## Single-Page Application 单页应用

单页应用程序是一个加载单个 HTML 页面以及应用程序运行所需要的所有必须资源（如 JavaScript 和 CSS）的应用程序。任何与页面或者后续页面的交互都不需要往返服务器，这意味着页面不会被重新加载。

虽然你可以在 React 中构建一个单页应用程序，但这不是必须的。React 也可以用于增强现有网站的小部分，增强其交互性。用 React 编写的代码，可以与服务器上的 markup （如 PHP） 或其他客户端库一起和平共存。实际上，这正是 Facebook 使用 React 的方式。

----

## ES6，ES2015，ES2016 等

这些缩略图都是指 ECMAScript 语言规范标准的最新版本，这是 JavaScript 语言的一个实现。ES6 版本（也称 ES2015）包括许多以前版本的新增功能，例如：箭头函数，类，模板字符串，`let` 和 `const` 语句等。你可以在[这里](https://en.wikipedia.org/wiki/ECMAScript#Versions)查看更多的特性。

---- 

## Compilers 编译器

JavaScript 编译器接受 JavaScript 代码，将其转传并以不同的格式返回 JavaScript 代码。最常见的用例是采用 ES6 语法，并将其转换为旧版浏览器能够解释的语法。[Babel](https://babeljs.io/) 是在 React 中使用最多的编译器。

----

## Bundlers 打包工具

打包工具将 JavaScript 和 CSS 代码作为单独的模块（通常有数百个）编写，并将它们组合成为浏览器能够更好地优化的几个文件。一些在 React 应用程序中常用的打包工具包括 [Webpack](https://webpack.js.org/) 和 [Browserify](http://browserify.org/)。

----

## 包管理

包管理工具是允许你管理项目中依赖项的工具。[npm](https://www.npmjs.com/) 和 [Yarn](http://yarnpkg.com/) 是 React 应用程序中常用的两个包管理器。他们都是同一个 npm 包注册表的客户端。

---

## CDN

CDN 代表内容传送网络。 CDN 从全球各地的服务器网络提供缓存的静态内容。

----

## JSX

JSX 是 JavaScript 的语法扩展。它类似于模板语言，但是它具有 JavaScript 的全部功能。JSX 被编译为 `React.createElement()` 进行调用，并最终返回名为 “React Element” 的普通 JavaScript 对象。要了解 JSX 的基本介绍，请参阅[此处的文档](https://reactjs.org/docs/introducing-jsx.html)，要深入了解 JSX 请参阅[这里](https://reactjs.org/docs/jsx-in-depth.html)

React DOM 使用 小驼峰的属性命名约定而不是 HTML 的属性名称。例如 `tabindex` 在 JSX 需要写 `tabIndex`。而 `class` 属性需要写 `className` 因为 `class` 在 JavaScript 中是保留字：

```javascript
const name = 'Clementine';
ReactDOM.render(
  <h1 className="hello">My name is {name}!</h1>,
  document.getElementById('root')
);
```

----

## [Elements](https://reactjs.org/docs/rendering-elements.html)

React 元素是 React 应用程序的构建块。可能会将 elements 和更广为人知的 “Component” 混为一谈，一个 element 描述了你想在屏幕上看到的东西。React 元素是不可变的。

```javascript
const element = <h1>Hello, world</h1>;
```

通常元素不是直接使用，而是从组件中返回。

----

## [Components](https://reactjs.org/docs/components-and-props.html)

React 组件是一些小的，可重用的代码片段，它返回一个 React 元素供页面渲染。React 组件的最简单版本是一个普通的 JavaScript 函数，返回一个 React 元素：

```javascript
function Welcome(props) {
  return <h1>Hello, {props.name}</h1>;
}
```

组件同样能够使用 ES6 类：

```javascript
class Welcome extends React.Component {
  render() {
    return <h1>Hello, {this.props.name}</h1>;
  }
}
```

组件可以分解成不同的功能块，并在其他组件中使用。组件可以返回其他组件、数组、字符串和数字。一个好的经验法则是，如果你的用户界面的一部分被多次使用（按钮，面板，头像），或者自己（应用程序，FeedStory，评论）足够复杂，这是很好的可重用组件的实践。组件名称也应使用大写字母（`<Wrapper/>` 而不是 `<wrapper/>`）。有关渲染组件的更多内容，请[参阅此文档](https://reactjs.org/docs/components-and-props.html#rendering-a-component)。

### [props](https://reactjs.org/docs/components-and-props.html)

`props` 是 React 组件的输入。它们是从父组件传递给子组件的数据。

请记住，`props` 是只读的，不应该以任何方式修改它们：

```javascript
// 错误
props.number = 42;
```

如果你需要修改某些值来响应用户输入或者网络响应，请改用 `state`。


### props.children

`props.children` 可用于每个组件。它包含组件的开始和结束标记之间的内容。如：

```html
<Welcome>Hello world!</Welcome>
```

字符串 `Hello World` 在 `Welcome` 组件中的 `props.children` 是可用的。

```javascript
function Welcome(props){
  return <p>{props.children}</p>;
}
```
对于定义为类的组件，请使用 `this.props.children`：

```javascript
class Welcome extends React.Component {
  render() {
    return <p>{this.props.children}</p>;
  }
}
```

### [state](https://reactjs.org/docs/state-and-lifecycle.html#adding-local-state-to-a-class)

当与组件关联的某些数据随着时间变化时，组件需要 `state` 。例如，`Checkbox` 组件可能需要一个 `isChecked` state，而一个 `NewsFeed` 组件可能希望跟踪其 state 中的  `fetchedPosts`。

`props` 和 `state` 之间最重要的区别是 `props` 是从父组件传递的，而 `state` 是组件本身管理的。一个组件不能更改它的 `props`，但是它可以更改它的 `state`。要做到这一点，需要调用 `this.setState()` 方法。只有定义为类的组件才拥有 state。

对于每一个特定的变化数据，应该只有一个“拥有”它的 state 的组件。不要尝试同步两个不同组件的 state。相反，应当进行[状态提升](https://reactjs.org/docs/lifting-state-up.html)，将其提升到最接近的共同组建，并把它作为道具传递给它们两个。

----

## [Lifecycle Methods](https://reactjs.org/docs/state-and-lifecycle.html#adding-lifecycle-methods-to-a-class)


生命周期方法是在组建的不同阶段之行的自定义功能。当组件被创建并被插入 DOM （[挂载](https://reactjs.org/docs/react-component.html#mounting)），组件更新的是偶，以及组件被卸载或从 DOM 中移除的时候，都有相对应的方法。

----


## [可控组件](https://reactjs.org/docs/forms.html#controlled-components) vs [不可控组件](https://reactjs.org/docs/uncontrolled-components.html)

React 有两种不同的方法来处理表单的输入。

一个由 React 控制的输入表单元素被称为 *可控组件*。当用户将数据输入到可空组件时，会触发 change 事件处理方法，并且你的代码将决定输入是否有效（通过使用更新的值重新渲染）。如果你不重新渲染，那么表单元素将保持不变。

一个*不可控组件*像 React 元素之外的表单元素一样的工作。当用户输入数据到表单域（输入框、下拉菜单）时，更新后的信息会被反映出来，而不需要 React 做任何使用。然而，这也意味着你不能强迫这个输入域有一个固定的 value。

在大多数情况下，你应该使用可空组件。

----

## [Keys](https://reactjs.org/docs/lists-and-keys.html)


“key” 是创建元素数组时需要包含的特殊字符串属性。键可以帮助 React 识别哪些项目已经更改，添加或删除。key 应当是数组中的元素的一个稳定的标识。

key 只需要在同一个数组中的兄弟元素中是唯一的。它不需要在整个应用程序或者是整个单组件中是唯一的。

不要使用如 `Math.random()` 之类的东西传递给键。密钥在重新渲染时有一个 “稳定的身份” 是非常重要的，这样 React 可以确定何时添加、删除或者重新排序项目。理想情况下，key 应当对应来自于你的数据的唯一且稳定的标识符，如 `post.id`。

----

## [Refs](https://reactjs.org/docs/refs-and-the-dom.html)

React 支持的可以附加到任何组件的特殊属性。`ref` 属性可以是一个字符串或一个回调函数。当 `ref` 属性是回调函数时，该函数接收底层 DOM 元素或实例（取决于元素的类型）作为其参数。这能够是你直接访问 DOM 元素或者组件实例。

谨慎使用 refs。如果你发现自己经常在应用中 “使用 refs” ，请考虑更好的[自顶向下的数据流](https://reactjs.org/docs/lifting-state-up.html)模式。

----

## [Events](https://reactjs.org/docs/handling-events.html)

使用 React 元素处理事件有一些语法上的差异：

- React 事件处理程序使用 小驼峰法明明，而不是小写。

- 使用 JSX，你需要传递一个函数作为事件处理程序，而不是一个字符串。

---

## [Reconciliation](https://reactjs.org/docs/reconciliation.html)

当组件的 props 或者 state 更改的时候，React 通过比较新返回的元素和先前渲染的元素来决定是否需要实际的 DOM 更新。当它们不相等的时候，React 将更新DOM ，这个过程称为 “reconciliation”。