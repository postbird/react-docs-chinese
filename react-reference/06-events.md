# SyntheticEvent

- 英文原文：[https://reactjs.org/docs/events.html](https://reactjs.org/docs/events.html)

本参考指南记录了构成 React 事件系统一部分的 `SyntheticEvent` 包装器。请参阅 [Handling Events](https://reactjs.org/docs/handling-events.html) 文档导航了解更多。

----

## 概览

你的事件处理程序将传递 `SyntheticEvent` 的实例，这是一个跨浏览器原生事件的跨浏览器包装。它具有与浏览器的原生事件相同的接口，包括 `stopPropagation()` 和 `preventDefault()`，除了这些事件在所有浏览器上的工作方式相同。

如果你发现由于某种原因需要基础的浏览器事件，则只需要使用 `nativeEvent` 属性即可获取它。每个 `SyntheticEvent` 对象都有以下属性：

```javascript
boolean bubbles
boolean cancelable
DOMEventTarget currentTarget
boolean defaultPrevented
number eventPhase
boolean isTrusted
DOMEvent nativeEvent
void preventDefault()
boolean isDefaultPrevented()
void stopPropagation()
boolean isPropagationStopped()
DOMEventTarget target
number timeStamp
string type
```

> **注意：**
> 从 v0.14 开始，从事件处理程序返回 `false` 将不再停止事件传播。相反，应该手动触发 `e.stopPropagation()` 或 `e.preventDefault()` 

### 事件池

`SyntheticEvent` 是事件池的。这意味着在事件回调被调用之后，`SyntheticEvent` 对象将被重用并且所有的属性将无效。这是出于性能原因。因此，你不能以异步方式访问该事件。

```javascript
function onClick(event) {
  console.log(event); // => nullified object.
  console.log(event.type); // => "click"
  const eventType = event.type; // => "click"

  setTimeout(function() {
    console.log(event.type); // => null
    console.log(eventType); // => "click"
  }, 0);

  // Won't work. this.state.clickEvent will only contain null values.
  this.setState({clickEvent: event});

  // You can still export event properties.
  this.setState({eventType: event.type});
}
```

> **注意：**
> 如果要以异步方式访问事件属性，则应该在事件上调用 `event.persist()` ，该事件将从事件池中移除合成事件，并允许用户代码保留对事件的引用。

----

### 支持的事件

React 使得事件标准化，以便它们在不同的浏览器中具有一致的属性。

下面的事件处理程序由冒泡阶段的事件触发。要在捕获阶段注册事件处理程序，请将 `Capture` 附加到事件名称上；例如，你可以使用 `onClickCature` 来处理捕获阶段的点击事件，而不是使用 `onClick`。

- [Clipboard 事件](#clipboard-events)

- [Composition 事件](#composition-events)

- [键盘事件](#keyboard-events)

- [焦点事件](#focus-events)

- [表单事件](#form-events)

- [鼠标事件](#mouse-events)

- [选择事件](#selection-events)

- [触摸事件](#touch-events)

- [UI 事件](#ui-events)

- [滚轮事件](#wheel-events)

- [媒体事件](#media-events)

- [图片事件](#image-events)

- [动画事件](#animation-events)

- [过渡事件](#transition-events)

- [其他事件](#other-events)

----

## API 文档参考

### Clipboard Events

事件名称：

```javascript
onCopy onCut onPaste
```

属性：

```javascript
DOMDataTransfer clipboardData
```

----

### Composition Events

事件名称：

```javascript
onCompositionEnd onCompositionStart onCompositionUpdate
```

属性：

```javascript
string data
```

----

### Keyboard Events

事件名称：

```javascript
onKeyDown onKeyPress onKeyUp
```

属性：

```javascript
boolean altKey
number charCode
boolean ctrlKey
boolean getModifierState(key)
string key
number keyCode
string locale
number location
boolean metaKey
boolean repeat
boolean shiftKey
number which
```

`key` 属性可以采用 [DOM3 级事件规范](https://www.w3.org/TR/uievents-key/#named-key-attribute-values) 中记录的任何值。

----

### Focus Events

事件名：

```javascript
onFocus onBlur
```

这些关注事件适用于 React DOM 中的所有元素，而不仅仅是表单元素。

属性：

```javascript
DOMEventTarget relatedTarget
```

----

### Form Events

活动名称：

```javascript
onChange onInput onInvalid onSubmit
```

有关 onChange 事件更多信息，参阅 [Forms](https://reactjs.org/docs/forms.html)

----

### Mouse Events

事件名称：

```javascript
onClick onContextMenu onDoubleClick onDrag onDragEnd onDragEnter onDragExit
onDragLeave onDragOver onDragStart onDrop onMouseDown onMouseEnter onMouseLeave
onMouseMove onMouseOut onMouseOver onMouseUp
```

`onMouseEnter` 和 `onMouseLeave` 事件从左侧的元素传播到正在进入的元素，而不是普通的冒泡，并且没有捕获阶段。

属性：

```javascript
boolean altKey
number button
number buttons
number clientX
number clientY
boolean ctrlKey
boolean getModifierState(key)
boolean metaKey
number pageX
number pageY
DOMEventTarget relatedTarget
number screenX
number screenY
boolean shiftKey
```

----

### Selection Events

事件名称：

```javascript
onSelect
```

---

### Touch Events

事件名称：

```javascript
onTouchCancel onTouchEnd onTouchMove onTouchStart
```
属性：

```javascript
boolean altKey
DOMTouchList changedTouches
boolean ctrlKey
boolean getModifierState(key)
boolean metaKey
boolean shiftKey
DOMTouchList targetTouches
DOMTouchList touches
```

-----

### UI Events

事件名称：

```javascript
onScroll
```

属性：

```javascript
number detail
DOMAbstractView view
```

----

### Wheel Events

事件名称：

```javascript
onWheel
```

属性：

```javascript
number deltaMode
number deltaX
number deltaY
number deltaZ
```
----

### Media Events

事件名称：

```javascript
onAbort onCanPlay onCanPlayThrough onDurationChange onEmptied onEncrypted
onEnded onError onLoadedData onLoadedMetadata onLoadStart onPause onPlay
onPlaying onProgress onRateChange onSeeked onSeeking onStalled onSuspend
onTimeUpdate onVolumeChange onWaiting
```

----

### Image Events

事件名称：

```javascript
onLoad onError
```

----

### Animation Events

事件名称：

```javascript
onAnimationStart onAnimationEnd onAnimationIteration
```

属性：

```javascript
string animationName
string pseudoElement
float elapsedTime
```

----

### Transition Events

事件名称：

```javascript
onTransitionEnd
```

属性:

```javascript
string propertyName
string pseudoElement
float elapsedTime
```

----

### Other Events

```javascript
onToggle
```
