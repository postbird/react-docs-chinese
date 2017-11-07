# JavaScript Environment Requirements

- 英文原文：[https://reactjs.org/docs/javascript-environment-requirements.html](https://reactjs.org/docs/javascript-environment-requirements.html)

React 16 依赖于集合类型 [**Map**](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Map) 和 [**Set**](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Set).如果你支持旧版浏览器和尚未提供这些功能的浏览器（如 IE <= 11），需要在你的打包环境中包含一个全局的 polyfill，比如 [core-js](https://github.com/zloirock/core-js) 或者 [babel-polyfill](https://babeljs.io/docs/usage/polyfill/)。

React 16 使用 core-js 来支持旧浏览器的多 polyfill 环境可能如下所示：

```javascript
import 'core-js/es6/map';
import 'core-js/es6/set';

import React from 'react';
import ReactDOM from 'react-dom';

ReactDOM.render(
  <h1>Hello, world!</h1>,
  document.getElementById('root')
);
```

React 依赖于 `requestAnimationFrame`（即使在测试环境中）。你可以使用 [raf](https://www.npmjs.com/package/raf) 这个包来 shim `requestAnimationFrame`：

```javascript
import 'raf/polyfill';
```