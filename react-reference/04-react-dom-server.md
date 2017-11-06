# ReactDOMServer

- 英文原文：[https://reactjs.org/docs/react-dom-server.html](https://reactjs.org/docs/react-dom-server.html)

`ReactDOMServer` 对象使你能够将组件渲染为静态标记。通常，它在节点服务器上使用：

```javascript
// ES modules
import ReactDOMServer from 'react-dom/server';
// CommonJS
var ReactDOMServer = require('react-dom/server');
```

----

## 概览

下面的方法在服务器和浏览器环境中都能使用：

- [renderToString()](#rendertostring)

- [renderToStatickMarkup](#rendertostaticmarkup)

下面附加的方法依赖于**只在服务器上可用**的包（`stream`），并且不能在浏览器上工作：

- [renderToNodeString()](#rendertonodestring)

- [renderToStaticNodeStream()](#rendertostaticnodestream)

----

## API 参考

### renderToString()

```javascript
ReactDOMServer.renderToString(element)
```

将 React 元素渲染为基本的 HTML。React 将返回一个 HTML 字符串。你可以使用此方法在服务器上生成 HTML，并在初始请求时发送标记以加快页面加载速度，并允许搜索引擎抓取你的页面以实现 SEO 的目的。

如果你在已经具有此服务器渲染标记的节点上调用了 [ReactDOM.hydrate()](https://reactjs.org/docs/react-dom.html#hydrate)，则 React 将保留它并仅附加事件处理程序，从而使您拥有非常高效的首次加载体验。

----

### renderToStaticMarkup()

```javascript
ReactDOMServer.renderToStaticMarkup(element)
```

和 [renderToString](#rendertostring) 类似，除了这不会创建 React 在内部使用的额外的 DOM 属性，如 `data-reactroot`。如果你想使用 React 作为一个简单的静态页面生成器，这很有用，因为去除额外的属性，可以节省一些字节。

如果你打算在客户端上使用 React 来交互标记，请不要使用此方法。而是在服务器上使用 [renderToString](#rendertostring)，而在客户端上使用 [ReactDOM.hydrate()](https://reactjs.org/docs/react-dom.html#hydrate)。

----

### renderToNodeStream()

```
ReactDOMSercer.renderToNodeStream(element)
```

将 React 元素渲染到其最初的 HTML。返回一个输出 HTML 字符串的可读流。这个流输出的 HTML 完全等于 [ReactDOMServer.renderToString()](#rendertostring) 返回的内容。你可以使用此方法在服务器上生成 HTML，并在初始请求时发送标记以加快页面加载速度，并允许搜索引擎抓取你的页面以实现 SEO 的目的。

如果你在已经具有此服务器渲染标记的节点上调用 [ReactDOM.hydrate()](https://reactjs.org/docs/react-dom.html#hydrate)，则 React 将保留它并仅附加事件处理程序，从而使你拥有非常高效的首次加载体验。

> **注意：**
> 仅在服务器上可用。 API 在浏览器上无法起作用。
> 从这个方法返回的流，将返回一个以 utf-8 编码的字节流，如果你需要另一种编码的流，请查看像 [iconv-lite](https://www.npmjs.com/package/iconv-lite) 这样的项目，该项目为转码文本提供转换流。


----

### renderToStaticNodeStream()

```javascript
ReactDOMServer.renderToStaticNodeStream(element)
```

和 [renderToNodeStream](#rendertonodestream) 方法类似，这不会创建 React 在内部使用的其他 DOM 属性，如 `data-reactroot`。如果你想使用 React 作为一个简单的静态页面生成器，这很有用。因为去除额外的属性可以节省一些字节。

这个流输出的 HTML 完全等于 [ReactDOMServer.renderToStaticMarkup](#rendertostaticmarkup) 返回的内容。

如果你计划在客户端上使用 React 来交互标记，请不要使用这个方法。而是在服务器上使用 [renderToNodeStream](#rendertonodestream)，在客户端上使用 [ReactDOM.hydrate](https://reactjs.org/docs/react-dom.html#hydrate)。

> **注意：**
> 仅在服务器起作用。这个 API 在浏览器无法工作。
> 从这个方法返回的流，将返回一个以 utf-8 编码的字节流，如果你需要另一种编码的流，请查看像 [iconv-lite](https://www.npmjs.com/package/iconv-lite) 这样的项目，该项目为转码文本提供转换流。