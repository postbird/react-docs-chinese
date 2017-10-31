# Reconciliation

- 英文原文：[https://reactjs.org/docs/reconciliation.html][1]

React提供了一个声明式的API，因此每次更新你无须关心究竟发什么什么改变。这使得开发应用更加的便捷，但是可能并不是很明显的理解React内部是如何实现的。本文介绍React在组件更新时如何进行"diffing"(差异)算法选择，以便实现足够快的高性能应用。

## Motivation (动机)

当你使用React的时候，在某个时间点时，你可以认为`render()`函数用来创建了一个React元素的树。当一个state或者是props更新的时候，`render()`函数将返回一个不同的React元素的树。此时，React需要计算如何能够高效的匹配最新的树并进行UI的更新。

将一颗树转换成另一颗树的最小操作的算法有一些通用的解决方案，然而，[目前的算法][2]的复杂度是O(n3),其中n是树中元素的数量。

如果在React中使用这种算法的话，渲染1000个元素需要进行十亿次比较，这过于耗费资源。相反，React基于两个设想实现了一个O(n)的启发式算法：

1. 不同类型的两个元素将产生不同的树。

2. 开发者能够指定通过一个`key`的prop来表示那些子元素在不同的渲染中是不需要变动（stable）的

实际使用过程中，上面的设想对于几乎所有的实际情况都有效。

----

## Diffing 算法

在比较两个树的时候，React首先会比较两个根节点。根元素的类型不同，他们的表现也就不同。

### 不同类型的元素

每当根元素是不同类型时，React会销毁旧的树，并从头开始构建新的树。从`<a>`到`<img>`，或者从`<Article>`到`<Comment>`，或者从`<Button>`到`<div>`，这些都会导致完全的重构。

当销毁旧的树的时候，原来的DOM节点也都被销毁。组件示例会调用`componentWillUnmount`方法。当建立一个新树的时候，新的DOM节点会被插入到DOM中。组件实例会执行`componentWillMount`和`componentDidMount`方法。与旧树相关联的任何 state 都会丢失。

根下面的任何组件也将被卸载并且它们的 state 也会被破坏：

```html
<div>
  <Counter />
</div>

<span>
  <Counter />
</span>
```

上面的代码会导致将原来的`Counter`组件销毁，并且重新挂在一个新的。

### 相同类型的DOM元素

当比较两个相同类型的 React DOM 元素时，React 会查看两个元素的属性，将底层的DOM节点保留，仅仅更新那些改变的属性。比如：

```html
<div className="before" title="stuff" />

<div className="after" title="stuff" />
```
通过对上面两个元素的比较，React 知道只需要将底层DOM节点的`className`更新即可。

当DOM的`style`的更新时，React 也是只更新需要更新的属性。如：

```html
<div style={{color: 'red', fontWeight: 'bold'}} />

<div style={{color: 'green', fontWeight: 'bold'}} />
```
上面两个元素在进行转换的时候，React 只修改 `color` 样式，而不会去修改`fontWeight`的样式。

处理完DOM节点后，React将递归其子节点。

### 相同类型的组件元素

组件进行更新时，组件实例是相同的，因此 state 在重新渲染过程中保持不变。React更新底层组件实例的props来匹配新的元素，并且底层组件实例会调用`componentWillRecieiveProps()`和`componentWillUpdate()`。

然后调用`render()`方法，并且 diff 算法对之前的结果和新的结果进行递归。

### 子节点的递归

默认情况下，当对 DOM 节点的子节点进行递归的时候，React 只会在同一时间遍历子节点的两个列表，并在存在差异的时候进行更改。

例如，挡在元素的最后增加一个子节点的话，两个树之间的转换的效率会比较好：

```html
<ul>
  <li>first</li>
  <li>second</li>
</ul>

<ul>
  <li>first</li>
  <li>second</li>
  <li>third</li>
</ul>
```

React 会匹配到前两个子节点：`<li>first</li>` 和 `<li>second</li>` ，并且之后将 `<li>third</li>` 插入到新的树中。

然而如果都是按照上面的算法进行简单的实现，则在元素的最开始位置插入子节点会导致性能低下。比如，下面的两个树的转换的效率是比较差的：

```html
<ul>
  <li>Duke</li>
  <li>Villanova</li>
</ul>

<ul>
  <li>Connecticut</li>
  <li>Duke</li>
  <li>Villanova</li>
</ul>
```

上面的示例中，React会更新每一个子节点，而不会意识到 `<li>Duke</li>` 和 `<li>Villanova</li>` 是可以保持不变的。这是一个低效率的问题。

### Keys

为了解决上面的问题，React 支持使用一个 `key` 属性。当子节点拥有 keys 属性的时候，React 使用 key 将原始树中的子节点和新树中的子节点进行匹配。比如，给上面的低效率转换示例增加一个 key 能够提高转换效率：


```html
<ul>
  <li key="2015">Duke</li>
  <li key="2016">Villanova</li>
</ul>

<ul>
  <li key="2014">Connecticut</li>
  <li key="2015">Duke</li>
  <li key="2016">Villanova</li>
</ul>
```

现在，React 知道 key 值是 `2014` 的是新的元素，而 `2015` 和 `2016` 则是仅仅移动了一下位置而已。

实际上，找到一个 key 通常并不困难。你要显示的元素可能具有唯一的 ID ，所以 key 刚好可以来自你的数据：

```html
<li key={item.id}>{item.name}</li>
```

如果不是这种情况，你可以向模型中增加一个 ID 属性，或者将内容的某些部分 hash 成一个密钥。关键点是 key 在其相邻的节点中是唯一的即可，而不是在全局环境中唯一。

最后的手段，也可以将数组中的索引（index）作为 key，不过如果数组不会重新排序的话，会很好的工作，如果会重新排序，则会变得很慢。

----

## Tradeoffs　（权衡）

重点要记住，reconciliation 算法是一个实现细节。React 能够根据某个 action 渲染整个 app；而最终的结果是一样的。需要明白的是，在这种情况下，冲渲染意味着会调用所有组件的 `readner` 方法，但这不说明 React 会卸载并重新挂载这些组件。它仅仅是在前面章节中提到的规则进行差异比较(diffing 算法)。

为了使得通常情况下用例更快，我们会优化启发式算法。现在的实现中，你可以表述某个子树已经在起相邻子树中进行了移动这一事实，但是你不能指出该子树已经移动到别的什么地方。所以该算法会重新渲染完整的子树。

由于 React 依赖于启发式算法，所以如果这背后的假设（译者注：参照前面的两个假设）不满足条件，性能会受到影响：

1. 该算法不会去尝试匹配不同组件类型的子树。如果你发现自己的两个组件类型的输出非常相似，则可能需要使得他们的类型一致。在实际使用过程中，我们并没有发现这是一个问题。
2. Keys 应当是稳定的，可预测且唯一的。不稳定的 key （比如使用 `Math.random` 生成的键）将导致许多的组件实例和 DOM 节点会被不必要的更新，这可能会导致子组件的性能下降以及 state 的丢失。







[1]: https://reactjs.org/docs/reconciliation.html
[2]: http://grfia.dlsi.ua.es/ml/algorithms/references/editsurvey_bille.pdf