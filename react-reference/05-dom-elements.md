# DOM Elements

- 英文原文： [https://reactjs.org/docs/dom-elements.html](https://reactjs.org/docs/dom-elements.html)

React 实现了一个独立于浏览器的 DOM 系统，以提高性能和跨浏览器兼容性。我们借此清理了浏览器 DOM 实现中的一些粗糙边缘。

在 React 中，所有的 DOM 属性和 arrtibutes（包括事件处理程序）都应该是 camelCased（小驼峰法）。例如，HTML 属性 `tabindex` 对应 React 中的属性 `tabIndex`。`aria-*` 和 `data-*` 属性却是个例外，这个应该是都小写的。比如，你可以保持 `aria-label` 为 `aria-label` 不变。

----

## Arrtibutes 中的区别

React 和 HTML 之间有许多不同的属性：

### checked

`checkbox` 或者 `radio` 类型的 `input` 组件支持 `checked` 属性。你可以使用它来设置组件是否被选中。这对于构建可控组件非常有用。`defaultChecked` 是不受控制的等价物，它设置组件是否在首次挂载时被选中。

### className

要指定一个 CSS 类，请使用 `className` 属性。这是用于常规如 `<div/>` 和 `<a/>` 等这样的 DOM 元素 和 SVG 元素

如果你在 Web 组件中使用 React（这不是常见的用法），请改用 `class` 属性。

### dangerouslySetInnerHTML

`dangerouslySetInnerHTML` 是 React 在浏览器使用 `innerHTML` 的替代方法。通常来说，从代码设置 HTML 是存在风险的，因为很容易让你的用户无意中发现[跨站脚本攻击（XSS）](https://en.wikipedia.org/wiki/Cross-site_scripting)。因此，你可以直接从 React 中设置 HTML，但是你必须使用 `dangerouslySetInnerHTML` 并且使用 `__html` 键传递一个对象，来提醒自己危险，如：

```javascript
function createMarkup(){
  return {__html: 'First &middot; Second'};
}
function MyComponent() {
  return <div dangerouslySetInnerHTML = {createMarkup} />;
}
```

### htmlFor

由于 `for` 在 JavaScript 中是一个保留字，因此 React 元素使用 `htmlFor` 来代替。

### onChange

`onChange` 时间的行为和你预期的一样：只要表单字段发生更改，就会触发此事件。我们故意不适用现有的浏览器行为，因为 `onChange` 对于它的行为是不恰当的，React 依靠这个事件来实时处理用户的输入。

### selected

`seleced` 使用由 `<option>` 组件支持，你可以用它来设置组件是否被选中。这对于构建可空组件非常有用。

### style

`style` 属性接收一个带有 camelCased （小驼峰发法）模式的 JavaScript 对象，而不是一个 CSS 字符串。这与 DOM `style` 的 JavaScript 属性是一致的，更搞笑，并且防止 XSS 安全漏洞。例如：

```javascript
const divStyle = {
  color: 'blue',
  backgroundImage:'url(' + imgUrl + ')'
};

function HelloWorldComponent() {
  return <div style = {divStyle} > Hello World! </div>
}
``` 

请注意样式不是自动处理前缀的，要支持比较旧的浏览器，你需要提供相应的样式：

```javascript
const divStyle = {
  WebkitTransition: 'all', // note the capital 'W' here
  msTransition: 'all' // 'ms' is the only lowercase vendor prefix
};

function ComponentWithTransition() {
  return <div style={divStyle}>This should work cross-browser</div>;
}
```

style 的 key 应当是驼峰式的，以便于从 JS 访问 DOM 节点上的属性（如 `node.style.backgroundImage`）是一致的。除了 [ms](http://www.andismith.com/blog/2012/02/modernizr-prefixed/) 之外的前缀都应当以大写字母开头，这就是为什么 `WebkitTransition` 是大写的 “W”。

React 会自动附加一个 `px` 后缀到某些内敛样式属性，如：

```javascript
// This:
<div style={{ height: 10 }}>
  Hello World!
</div>;

// Becomes:
<div style="height: 10px;">
  Hello World!
</div>
```

不是所有的样式属性都转换为像素字符串。某些样式属性会保持无单位（如 `zoom`，`order`，`flex`）。在[这里](https://github.com/facebook/react/blob/4131af3e4bf52f3a003537ec95a1655147c81270/src/renderers/dom/shared/CSSProperty.js#L15-L59)可以看到完整的无单位属性列表。

### suppressContentEditableWarning

通常情况下，带有子元素的元素也被标记为 `contentEditable` 的时候会发出警告，因为它不起作用。这个属性会抑制该警告。除非你正在建立一个像 [Draft.js](https://facebook.github.io/draft-js/) 这样的库来管理 `contentEditable` ，否则不要使用它。


### value

`value` 属性是用来支持 `<input>` 和 `<textarea>` 组件的。你可以使用它来设置组件的值。这对于构建可空组件非常有用。 `defaultValue` 是不可控的等价值，用于设置组件在第一次挂载时的 value。

----

## 所有支持的 HTML 属性

从 React 16 开始，任何的[自定义](https://reactjs.org/blog/2017/09/08/dom-attributes-in-react-16.html) DOM 属性属性和标准的 DOM 属性都可以支持。

React 一直为 DOM 提供一个以 JavaScript 为中心的 API。由于 React 组件经常采用自定义和 DOM 相关的 props，React 就像 DOM API 一样使用 `camelCase（驼峰法）` 约定：

```html
<div tabIndex="-1" />      // Just like node.tabIndex DOM API
<div className="Button" /> // Just like node.className DOM API
<input readOnly={true} />  // Just like node.readOnly DOM API
```

这些道具与相应的 HTML 属性的工作方式类似，除了上面记载的特殊情况外。

React 支持的一些 DOM 属性包括：

```javascript
accept acceptCharset accessKey action allowFullScreen allowTransparency alt
async autoComplete autoFocus autoPlay capture cellPadding cellSpacing challenge
charSet checked cite classID className colSpan cols content contentEditable
contextMenu controls controlsList coords crossOrigin data dateTime default defer
dir disabled download draggable encType form formAction formEncType formMethod
formNoValidate formTarget frameBorder headers height hidden high href hrefLang
htmlFor httpEquiv icon id inputMode integrity is keyParams keyType kind label
lang list loop low manifest marginHeight marginWidth max maxLength media
mediaGroup method min minLength multiple muted name noValidate nonce open
optimum pattern placeholder poster preload profile radioGroup readOnly rel
required reversed role rowSpan rows sandbox scope scoped scrolling seamless
selected shape size sizes span spellCheck src srcDoc srcLang srcSet start step
style summary tabIndex target title type useMap value width wmode wrap
Similarly, all SVG attributes are fully supported:
```
