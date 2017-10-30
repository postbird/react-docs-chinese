# Optimizing Performance

- 英文原文：[https://reactjs.org/docs/optimizing-performance.html][1]

React内部使用了一些技术来最大限度的减少更新UI导致的DOM的操作数量。对于大多数应用来说，使用React不需要刻意去做专门的性能优化就能实现快速的用户界面。当然，也有一些方案可以加快你的React应用。


## 使用生产环境构建

如果你的React在测试过程中发现性能等问题，确保你使用的测试环境是最小的生产环境。

默认情况下，React包含许多有用的警告信息，这些警告在开发的时候非常有用，然而，也使得React更大更慢。因此在部署React应用的时候，应当使用生产版本。

如果你不确定构建应用的过程配置的是开发模式还是生产模式，你可以通过安装Chrome的[React Developer Tools ][2]来调试。如果是在生产环境访问一个React的网站，该扩展图片的背景是深色的：

![bg1][3]

如果访问的网站是开发模式下，扩展的图片背景是红色的：

![bg2][4]

一般来讲，在开发过程中会使用开发模式，而在部署过程会使用生产模式。

你可以在下面找到构建你的应用程序的文档说明。


### Create React App

如果你的项目使用 [Create React App][5] 构建，通过下面命令启动：

```bash
npm run build
```

上面命令会在你的项目的`build/`文件夹下构建一个生产版本。

请记住，只有在部署到生产之前才需要这样做。对于正常开发来说，使用`npm start`就足够了。

### 单文件部署（Single-Files Builds）

我们提供生产模式下的 React 和 React DOM 文件：

```html
<script src="https://unpkg.com/react@16/umd/react.production.min.js"></script>
<script src="https://unpkg.com/react-dom@16/umd/react-dom.production.min.js"></script>
``` 

请记住，只有结尾是`.production.min.js`是适合生产环境的。


### Brunch

如果使用Brunch构建高效的生产版本，安装[`uglify-js-brunch`][6]插件

```bash
# If you use npm
npm install --save-dev uglify-js-brunch

# If you use Yarn
yarn add --dev uglify-js-brunch
```

然后，如果要构建生产版本，在`build`命令上使用 `-p` 即可：

```bash
brunch build -p
```

请记住，只需要在构建生产环境版本的时候使用该命令。在开发中不应当使用这个插件（`uglify-js-brunch`）也不应当使用`-p`参数，因为不仅仅会隐藏有用的React警告，构建速度也会更慢。


### Browserify

如果使用`Browserify`构建生产环境版本，需要安装一下插件：

```bash
# If you use npm
npm install --save-dev envify uglify-js uglifyify 

# If you use Yarn
yarn add --dev envify uglify-js uglifyify 
```

如果要构建生产环境版本，确保安装一下午的transform（下面几个很重要）:

- [envify transform][7] 能够确保构建正确的生产环境版本。需全局安装(`-g`)

- [uglifyify][8] 用来移除一些生产环境的依赖。同样全局安装(`-g`)

- 最终，得到的打包文件通过管道传递给[uglify-js][9]进行处理.([了解为什么][10])

例如：

```bash
browserify ./index.js \
  -g [ envify --NODE_ENV production ] \
  -g uglifyify \
  | uglifyjs --compress --mangle > ./bundle.js
```

> **注意：** 包的名字是`uglify-js`,但是提供的名称是`uglifyjs`.<br/>
> 这不是拼错了

同样请记住，只需要在生产环境中使用即可。在开发环境中，不应当使用这些插件，因为会隐藏有用的警告信息，而且构建速度也非常慢。

### Rollup

如果使用 Rollup 构建生产环境版本，需要安装下面这些插件：

```bash
# If you use npm
npm install --save-dev rollup-plugin-commonjs rollup-plugin-replace rollup-plugin-uglify 

# If you use Yarn
yarn add --dev rollup-plugin-commonjs rollup-plugin-replace rollup-plugin-uglify 
```

如果要构建生产环境版本，确保安装下这些插件（非常重要）：

- [replace][11] 确保配置了正确的构建环境

- [commonjs][12] 插件提供了对 Rollup 的 `CommonJS` 支持

- [uglify][13] 插件 最终压缩和管理构建的包。


```javascript
plugins: [
  // ...
  require('rollup-plugin-replace')({
    'process.env.NODE_ENV': JSON.stringify('production')
  }),
  require('rollup-plugin-commonjs')(),
  require('rollup-plugin-uglify')(),
  // ...
]
```

完整的配置示例可以[查看简介][14]：

同样请注意，只需要在构建生产环境版本的时候使用上述方式。在开发环境下，不要使用`uglify`插件或者是`replace`插件的时候使用`production`，因为会隐藏有用的React警告，而且构建速度更慢。


### webpack

> **注意:** 如果你使用 Create Raect App ,请参照上面的介绍 [the instructions above][15] <br/>
> 如果你直接使用webpack进行构建的话，下面章节会有帮助。

如果使用webpack构建生产环境版本，确保在配置中包含如下插件：

```javascript
new webpack.DefinePlugin({
  'process.env': {
    NODE_ENV: JSON.stringify('production')
  }
}),
new webpack.optimize.UglifyJsPlugin()
```
你可以在 [webpack 文档][16] 中了解更多。

请记住，仅在生产环境中使用上述插件。开发环境中不应该使用`UglifyJsPlugin`或者是`DefinePlugin`,因为会隐藏有用的React警告信息，构建速度也更慢。


## 使用Chrome性能分析标签来分析React组件

在**开发模式**下，你可以使用支持的浏览器中的性能工具来可视化组件的安装、更新和卸载。如：

![bg3][17]

如果在Chrome中使用：

1. 使用一个查询字符串`?react_perf`来加载应用程序。（如：http://localhost:3000/?react_perf）

2. 打开Chrome开发者工具的[`Performance`][18](性能)选项卡然后开始**Record**(记录)

3. 执行你想要分析的操作。不要超过20s因为Chrome可能会挂起

4. 停止记录（Stop Recording）

5. React 事件被分在 **User Timing** 标签下

请注意，**这些指示标准（数字）是相对的（开发环境下），组件在生产环境下将呈现更快的速度**。不过，当不相关的UI更新出现错误的时候或者是发现更新的深度和频率等方面，它能够帮助你尽快的发现问题。

目前 Chrome 、 Edge 和 IE 支持这些特性，不过我们使用标准的[`User Timing API`][19] ，因此我们希望能够有更多的浏览器支持这些特性。 

----

## Avoid Reconciliation

React在内部构建和维护一系列UI，包含了组件返回的React元素。这种方式可以使用React避免创建DOM节点并且没必要访问已经存在的DOM节点，所以可能会比直接操作JavaScript对象慢一些。有些时候，称其为**"虚拟DOM"**，不过和React Native的工作方式一样。

当一个组件的 props 或者是 state 改变的时候，React比较新的元素与之前渲染的元素，从而决定是否需要返回更新后的DOM。如果他们不相等，React将返回新DOM。

在某些情况下，你的组件可以通过重写生命周期函数`shouldComponentUpdate`来进行加速，因为在重新渲染过程开始之前便会被触发。

这个函数默认返回`true`，表示Raect需要进行更新：

```javascript
shouldComponentUpdate(nextProps, nextState) {
  return true;
}
```

如果在某些情况下吗，你能够明确你的组件不需要进行更新，你可以从`shouldComponentUpdate`函数返回`false`，跳过重新渲染的过程，无论在该组件上调用`render()`还是之后调用`render()`都可以实现。

----

## Action中的 shouldComponentUpdate

下面是一个组件树。对于每一个组件，`SCU`表示`shouldComponentUpdate`返回什么值，而`vDOMq`表示渲染的React元素是否是等价的。最后，圆圈的颜色表示，组件是否需要进行重绘。

![chonghui][20]

因为**C2**的`shouldComponentUpdate`返回`false`,所以React没有尝试去重新渲染**C2**，并且也没有在**C4**和**C5**上触发`shouldComponentUpdate`.


对于**C1**和**C3**来说，`shouldComponentUpdate`返回`true`,所以React需要继续往下遍历虚拟DOM树，并且去检查他们。对于**C6**来说，`shouldComponentUpdate`返回`true`，并且由于渲染的元素与已经存在的元素不相等，所以React要更新DOM。

**C8** 的情况比较有趣，React会渲染该组件，但是由于其返回的React元素和之前渲染的元素等价，所以不会更新DOM。

请注意，上述情况中，React只需要对**C6**改变DOM，这是一定会发生的。然而对于**C8**来说，通过和已经渲染的元素进行比较，决定不改变DOM，而对于**C2**的子树和**C7**来说，它不需要和已经渲染的元素进行比较，甚至不会去调用`shouldComponentUpdate`，也不会调用`render()`方法。

---

## 例子：

如果你改变下面组件的唯一方式是通过`props.color`或`state.count`的改变的话，你可以通过`shouldComponentUpdate`进行检查：

```javascript
class CounterButton extends React.Component {
  constructor(props) {
    super(props);
    this.state = {count: 1};
  }

  shouldComponentUpdate(nextProps, nextState) {
    if (this.props.color !== nextProps.color) {
      return true;
    }
    if (this.state.count !== nextState.count) {
      return true;
    }
    return false;
  }

  render() {
    return (
      <button
        color={this.props.color}
        onClick={() => this.setState(state => ({count: state.count + 1}))}>
        Count: {this.state.count}
      </button>
    );
  }
}
```

上面代码中，`shouldComponentUpdate`仅仅检查`props.color`和`state.count`是否改变。如果这些值不更改，则组件不会进行更新。当组件变的复杂时，可以使用类似的方式，在`props`和`state`的字段中进行一些 **“浅比较”**，来确定组件是否需要进行更新。如果组件继承自React提供的`React.PureComponent`的话，使用这种模式更加的方便。所以上面的代码中，有一个更简单的实现：

```javascript
class CounterButton extends React.PureComponent {
  constructor(props) {
    super(props);
    this.state = {count: 1};
  }

  render() {
    return (
      <button
        color={this.props.color}
        onClick={() => this.setState(state => ({count: state.count + 1}))}>
        Count: {this.state.count}
      </button>
    );
  }
}
```

大多数情况下，你可以使用`React.PureComponent`而不需要自己编写`shouldComponentUpdate`.当然，它只是做一个比较浅的比较，因此，如果 props 或者 state 通过这种 **“浅比较”** 可能会错过组件的更新的话，则不能这样去使用。

这可能是更复杂的数据结构方面的问题，例如，假设你想通过一个`ListOfWords`组件来呈现通过逗号分隔的单词列表，并使用父组件`WordAdder`,可以单击按钮将单词添加列表中。下面的代码将无法正常工作：

```javascript
class ListOfWords extends React.PureComponent {
  render() {
    return <div>{this.props.words.join(',')}</div>;
  }
}

class WordAdder extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      words: ['marklar']
    };
    this.handleClick = this.handleClick.bind(this);
  }

  handleClick() {
    // This section is bad style and causes a bug
    const words = this.state.words;
    words.push('marklar');
    this.setState({words: words});
  }

  render() {
    return (
      <div>
        <button onClick={this.handleClick} />
        <ListOfWords words={this.state.words} />
      </div>
    );
  }
}
```

问题在于`PureComponent`会对`is.props.words`的新值和旧值之间进行一个浅比较，虽然`WordAdder`的`handleClick`方法会发生变化，但是这种变化实际上在进行`this.props.words`的比较的时候，即使数组中的word实际上已经改变，但是旧值和新值也是相等的。所以`ListOfWords`不会发生更新，即使它应当呈现新单词。

----

## 不突变数据

避免上述问题最简单的方法是不要使用 props 或者 state 操作变异值。比如上面`handleClick`方法可以使用`concat`进行重写。

```javascript
handleClick() {
  this.setState(prevState => ({
    words: prevState.words.concat(['marklar'])
  }));
}
```

ES6提供了数组的[分解运算符][21]来方便操作，如果你使用Create React App,下面代码默认就是起作用的：

```javascript
handleClick() {
  this.setState(prevState => ({
    words: [...prevState.words, 'marklar'],
  }));
};
```

你也可以通过类似的方式重写代码来避免突变。例如，加入有一个名为`colormap`的对象，如果要将`colormap.right`赋值成`blue`,可以如下编写：

```javascript
function updateColorMap(colormap) {
  colormap.right = 'blue';
}
```
如果避免原始对象的突变，可以通过[Object.assign][22]方法：

```javascript
function updateColorMap(colormap) {
  return Object.assign({}, colormap, {right: 'blue'});
}
```

`updateColorMap`方法现在返回一个新的对象，而不是仅仅突变原来的对象。

`Object.assign`是ES6的API，需要一个polyfill。

JavaScript的提案（ES7）中有一个更好的[对象分解运算符][23]来更方便的编写：

```javascript
function updateColorMap(colormap) {
  return {...colormap, right: 'blue'};
}
```

如果你是使用 Create React App,并且 `Object.assign`和对象分解运算符都默认起作用。


## 使用不可变的数据结构

[Immutable.js][24]是另一种解决此问题的方案。它通过结构共享提供不变的、持久的数据集合：

- *Immutable*（不可变性） ： 一旦创建，数据集合不能再另一个时间点更改

- *Persistent* （一致性）: 可以从先前的集合和已经创建的集合中返回一个新的集合，创建新的集合后，原来的数据集合仍然有效。

- *Structural Sharing* （结构共享）：应当尽可能通过原始集合的结构创建新的集合，从而减少结构复制提高性能。

不可变数据使得追踪数据变化变的容易。新的改动总是会返回一个新的对象，因此我们只需要检查对象的引用是否已经更改，例如，下面普通的JavaScript代码中：

```javascript
const x = { foo: 'bar' };
const y = x;
y.foo = 'baz';
x === y; // true
```
虽然 `y` 已经更改过了，但是它还是和 `X` 完全相等，你可以通过 `Immutable.js` 来编写类似代码：

```javascript
const SomeRecord = Immutable.Record({ foo: null });
const x = new SomeRecord({ foo: 'bar' });
const y = x.set('foo', 'baz');
const z = x.set('foo', 'bar');
x === y; // false
x === z; // true
```

在这种情况下，由于在突变 `x` 的时候返回了新的引用，所以我们可以使用完全相等（`x===y`）来验证存储在 `y`　中的新值和存储在　`x` 中的原始值不同。

另外还有两个库能够帮助我们来使用不可变的数据集合是：[seamless-immutable][25] 和 [immutable-helper][26] .

Immutable数据结构给你提供了一种追踪对象变化的更方便的方式，这也是我们需要实现的应用程序的`shouldComponentUpdate`。这可以为你提供良好的性能提升。



[1]: https://reactjs.org/docs/optimizing-performance.html
[2]: https://chrome.google.com/webstore/detail/react-developer-tools/fmkadmapgofadopljbjfkapdkoienihi
[3]: https://reactjs.org/static/d0f767f80866431ccdec18f200ca58f1-4f509.png
[4]: https://reactjs.org/static/e434ce2f7e64f63e597edf03f4465694-4f509.png
[5]: https://github.com/facebookincubator/create-react-app
[6]: https://github.com/brunch/uglify-js-brunch
[7]: https://github.com/hughsk/envify
[8]: https://github.com/hughsk/uglifyify
[9]: https://github.com/mishoo/UglifyJS2
[10]: https://github.com/hughsk/uglifyify#motivationusage
[11]: https://github.com/rollup/rollup-plugin-replace
[12]: https://github.com/rollup/rollup-plugin-commonjs
[13]: https://github.com/TrySound/rollup-plugin-uglify
[14]: https://gist.github.com/Rich-Harris/cb14f4bc0670c47d00d191565be36bf0
[15]: https://reactjs.org/docs/optimizing-performance.html#create-react-app
[16]: https://webpack.js.org/guides/production-build/
[17]: https://reactjs.org/static/64d522b74fb585f1abada9801f85fa9d-dcc89.png
[18]:https://developers.google.com/web/tools/chrome-devtools/evaluate-performance/timeline-tool
[19]: https://developer.mozilla.org/en-US/docs/Web/API/User_Timing_API
[20]: https://reactjs.org/static/5ee1bdf4779af06072a17b7a0654f6db-9a3ff.png
[21]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Spread_operator
[22]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/assign
[23]: https://github.com/sebmarkbage/ecmascript-rest-spread
[24]: https://github.com/facebook/immutable-js
[25]: https://github.com/rtfeldman/seamless-immutable
[26]: https://github.com/kolodny/immutability-helper