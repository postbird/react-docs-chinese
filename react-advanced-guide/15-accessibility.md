# Accessilibity

- 英文原文：[https://reactjs.org/docs/accessibility.html][1]

## 为什么要无障碍访问？

Web 可访问性（也称为[a11y][2]是设计和创建每个人都能够使用的网站。无障碍访问支持对于使用辅助技术来解释网页是必须的。

React 完全支持构建无障碍访问的网站，通常使用标准的 HTML 技术。

----

## 标准和准则

### WCAG

[网站内容无障碍访问性指南][3] 提供了创建无障碍访问网站的指导原则。

下面的 WCAG 清单有大概的描述：

- [WCAG checklist from Wuhcag][4]

- [WCAG checklist from WebAIM][5]

- [Checklist from The A11Y Project][6] 

### WAI-ARIA

[网站无障碍访问性计划 - 可访问的富 Internet 应用程序][7]文档包含用于构建完全可访问的 JavaScript 小部件的技术

注意，JSX 完全支持所有的 `aria-*` 的 HTML 属性。虽然在 React 中大多数的 DOM 属性都是 `canekCased`（驼峰法）表示的，但是这些属性应当使用小写：

```html
<input
  type="text" 
  aria-label={labelText}
  aria-required="true"
  onChange={onchangeHandler}
  value={inputValue}
  name="name"
/>
```

----

## 无障碍表单

### Labeling

 每个 HTML 表单控件（如 `<input>` 和 `<textarea>`） 都需要被标记为可访问。我们需要提供向屏幕阅读器公开的描述性标签。

 下面的资源介绍了如何做：

 - [The W3C shows us how to label elements][8]

 - [WebAIM shows us how to label elements][9]

 - [The Paciello Group explains accessible names][10]

尽管这些标准 HTML 实践可以直接在 React 中使用，但请注意，`for` 属性在 JSX 中需要使用 `htmlFor`:

```html
<label htmlFor="namedInput">Name:</label>
<input id="namedInput" type="text" name="name"/>
```

### 通知用户错误

所有用户都需要了解错误信息。下面的链接显示了如何将错误文本暴露给屏幕阅读器：

- [The W3C demonstrates user notifications][11]

- [WebAIM looks at form validation][12]

----

## 焦点控制

确保你的 web 应用只能通过键盘完全控制：

- [WebAIM talks about keyboard accessibility][13]


### 键盘焦点和焦点轮廓

键盘焦点是指 DOM 中的当前元素，它被触发来接受来自键盘的输入。我们将其视为与下图所示的类似的焦点轮廓:

![焦点轮廓][14]

如果你使用另一个焦点轮廓实现替换它的话，只有通过 CSS 来去除这个焦点轮廓，例如通过设置 `outline:0`。

### 跳过所需的内容的机制

提供一种机制，允许用户跳过应用程序中的导航部分，因为这有助于加速键盘导航。

Skiplinks 或 Skip Navigation Links 是隐藏的导航链接，只有当键盘用户与界面交互时才显示。他们很容易实现与内部页面锚点和一些样式：

- [WebAIM - Skip Navigation Links][15]

同样，使用 `<main>` 和 `<aside>` 这样的标记元素来划分页面区域，因为辅助技术允许用户快速导航的这部分内容。

了解更多关于使用这些元素来增强可访问性的信息：

- [Deque University - HTML 5 and ARIA Landmarks][16]

### 编程管理焦点

我们的 React 应用程序在运行的时候不断修改 HTML DOM，有时会导致键盘焦点丢失或者设置成其他的元素。

为了解决这个问题，我们需要以正确的方向以编程的方式推动键盘焦点。例如，将模式窗口关闭后，给打开模式窗口的按钮重置一个焦点。


Mozilia 开发者社区（MDN）介绍了这一点，并且介绍了我们如何构建[可键盘导航的 JavaScript 小部件][17]。

在 React 中设置焦点，我们可以使用 [Refs to Components][18]

要使用这种方式，首先在组件类的 JSX 中创建一个元素：


```javascript
render() {
  // Use the `ref` callback to store a reference to the text input DOM
  // element in an instance field (for example, this.textInput).
  return (
    <input
      type="text"
      ref={(input) => { this.textInput = input; }} />

  );
}
```

那么我们可以在需要的时候在我们的组件中给它一个焦点：

```javascript
focus() {
  // Explicitly focus the text input using the raw DOM API
  this.textInput.focus();
}
```

一个焦点管理的很好的例子就是 [react-aria-modal][19]。这是一个相当罕见的完全无障碍 modal 窗口的例子。它不仅将初始焦点设置在取消按钮上（防止用户意外激活成功操作）而在在 modal 中捕获键盘焦点，也将焦点重置回最初触发 modal 的元素上。

> **注意：**
> 虽然这是一个非常重要的可访问性特性，但它也是一个应当谨慎使用的技术。使用它修复键盘焦点流时，不要使用预测用户如何使用应用程序。

----

## 更复杂的小部件

更复杂的用户体验不应意味着不太可取的用户体验。尽管通过尽可能接近 HTML 编码来实现可访问性是最容易实现的，但是即使是最复杂的小部件，也应当能够进行可访问性编码。

这里，我们需要 [ARIA Roles][20] 以及 [ARIA States and Properties][21] 的知识。这些工具箱拥有 JSX 完全支持的 HTML 属性，能够是我们构建完全可访问的网站以及功能强大的 React 组件。

每种类型的小部件具有特定的设计模式，并且预期由用户和用户代理通过某种方式起作用：

- [WAI-ARIA Authoring Practices - Design Patterns and Widgets][22]

- [Heydon Pickering - ARIA Examples][23]

- [Inclusive Components][24]

----

## 其他要点考虑

### 设置语言

当屏幕阅读器软件使用它来选择正确的语言设置时，指示页面文本的语言：

-[WebAIM - Document Language][25]

### 设置文档标题

设置文档 `<title>` 以正确描述当前页面内容，因为这可以确保用户保持对当前页面上下文的了解：

- [WCAG - Understanding the Document Title Requirement][26]

我们可以在 React 中使用 [React Document Tile 组件][27]

### 颜色对比

确保网站上的所有可读文本都有足够的色彩对比度，以保持低视力用户的最大可读性：

- [WCAG - Understanding the Color Contrast Requirement][28]

- [Everything About Color Contrast And Why You Should Rethink It][29]

- [A11yProject - What is Color Contrast][30]

手动计算网站中所有的适当颜色可能非常繁琐，你可以使用[Colorable 来计算整个可访问的颜色][31]

下面提到的 aXe 和 WAVE 工具都包括颜色对比测试，并且会报告对比误差。

如果你想扩展你的对比测试能力，你可以使用这些工具：

- [WebAIM - Color Contrast Checker][32]

- [The Paciello Group - Color Contrast Analyzer][33]

----

## 开发和测试工具

我们可以使用许多工具来协助我们创建无障碍访问的 Web 应用程序。

### 键盘

到目前为止，最简单也是最重要的检查之一是测试整个网站是否可以与键盘一起使用。做到这一点需要：

1. 拔出你的鼠标

2. 使用 `Tab` 和 `Shift + Tab` 浏览

3. 使用 `Enter` 去触发元素

4. 根据需要，使用键盘上的箭头键与某些元素（如菜单和下拉菜单）进行交互

### 开发助手

我们可以直接在我们的 JSX 代码中检查一些辅助功能，通常情况下，智能感知的 IDE 中已经提供了针对 ARIA 角色、状态和属性的智能感知检查。我们也可以通过访问以下工具：

#### eslint-plugin-jsx-a11y

ESLint 的 [eslint-plugin-jsx-a11y][34] 插件提供了 AST 风格检查用于检查 JSX 中的可访问性。许多 IDE 允许你将这些直接集成到代码分析和源代码窗口中。

[Create React App][35] 有这个插件以及激活该规则配置。如果要启用更多的辅助功能规则，则可以使用以下内容在项目的根目录下创建 `.eslintrc` 文件：

```json
{
  "extends": ["react-app", "plugin:jsx-a11y/recommended"],
  "plugins": ["jsx-a11y"]
}
```

### 在浏览器中测试可访问性

许多工具可以在浏览器的网页上运行可访问性检查。请将它们和本文提到的其他可访问性检查结合使用，因为它们只能测试 HTML 的技术可访问性。

#### aXe, aXe-core and react-axe

Deque Systems 为你的应用程序的自动化和端对端可访问性测试提供了 [ax-core][36],该模块包含 Selenuim 集成。

aXe 或者是 [The Accessibility Engine] 是构建在 `ax-core` 上的可访问性检查器浏览器扩展。

你也可以使用 [react-axe][38] 模块在开发和调试的时候直接将这些可访问性结果报告给控制台。

#### WebAIM WAVE

[Web 可访问性评估工具][39] 是另一种可访问性浏览器扩展。

#### Accessibility inspectors and the Accessibility Tree

[Accessibility Tree][40] 是 DOM 树的一个子集，它包含每个 DOM 元素的可访问对象，这些元素应该暴露给辅助技术，比如 屏幕阅读器。

在一些浏览器中，我们可以轻松查看辅助功能树中的每个元素的辅助功能信息：

- [Activate the Accessibility Inspector in Chrome][41]

- [Using the Accessibility Inspector in OS X Safari][42]

### 屏幕阅读器

使用屏幕阅读器进行测试也是可访问性测试的一部分。

请注意 浏览器和屏幕阅读器的组合使用。建议你在最适合你的屏幕阅读器的浏览器中测试你的应用程序。

#### NVDA in Firefox

[NonVisual Desktop Access][43] 或者是 NVDA 是一个广泛使用的开源 Windows 屏幕阅读器。

有关如何最佳的使用 NVDA ，请参阅以下指南：

- [WebAIM - Using NVDA to Evaluate Web Accessibility][44]

- [Deque - NVDA Keyboard Shortcuts][45]

#### VoiceOver in Safari

VoiceOver 是苹果设备上的集成屏幕阅读器。

有关如何激活和使用 VoiceOver ，请参阅以下指南：

- [WebAIM - Using VoiceOver to Evaluate Web Accessibility][46]

- [Deque - VoiceOver for OS X Keyboard Shortcuts][47]

- [Deque - VoiceOver for iOS Shortcuts][48]


#### JAWS in Internet Explorer

[Job Access With Speech][49] 或者 JAWS 是 Windows 上使用频繁的屏幕阅读器。

有关如何更好的使用 JAWS，请参阅下面的指南：

- [WebAIM - Using JAWS to Evaluate Web Accessibility][50]

- [Deque - JAWS Keyboard Shortcuts][51]




[1]: https://reactjs.org/docs/accessibility.html
[2]: https://en.wiktionary.org/wiki/a11y
[3]: https://www.w3.org/WAI/intro/wcag
[4]: https://www.wuhcag.com/wcag-checklist/
[5]: http://webaim.org/standards/wcag/checklist
[6]: http://a11yproject.com/checklist.html
[7]: https://www.w3.org/WAI/intro/aria
[8]: https://www.w3.org/WAI/tutorials/forms/labels/
[9]: http://webaim.org/techniques/forms/controls
[10]: https://www.paciellogroup.com/blog/2017/04/what-is-an-accessible-name/
[11]: https://www.w3.org/WAI/tutorials/forms/notifications/
[12]: http://webaim.org/techniques/formvalidation/
[13]: http://webaim.org/techniques/keyboard/
[14]: https://reactjs.org/static/dec0e6bcc1f882baf76ebc860d4f04e5-a1d23.png
[15]: http://webaim.org/techniques/skipnav/
[16]: https://dequeuniversity.com/assets/html/jquery-summit/html5/slides/landmarks.html
[17]: https://developer.mozilla.org/en-US/docs/Web/Accessibility/Keyboard-navigable_JavaScript_widgets
[18]: https://reactjs.org/docs/refs-and-the-dom.html
[19]: https://github.com/davidtheclark/react-aria-modal
[20]: https://www.w3.org/TR/wai-aria/roles
[21]: https://www.w3.org/TR/wai-aria/states_and_properties
[22]: https://www.w3.org/TR/wai-aria-practices/#aria_ex
[23]: http://heydonworks.com/practical_aria_examples/
[24]: https://inclusive-components.design/
[25]: http://webaim.org/techniques/screenreader/#language
[26]: https://www.w3.org/TR/UNDERSTANDING-WCAG20/navigation-mechanisms-title.html
[27]: https://github.com/gaearon/react-document-title
[28]: https://www.w3.org/TR/UNDERSTANDING-WCAG20/visual-audio-contrast-contrast.html
[29]: https://www.smashingmagazine.com/2014/10/color-contrast-tips-and-tools-for-accessibility/
[30]: http://a11yproject.com/posts/what-is-color-contrast/
[31]: http://jxnblk.com/colorable/
[32]: http://webaim.org/resources/contrastchecker/
[33]: https://www.paciellogroup.com/resources/contrastanalyser/
[34]: https://github.com/evcohen/eslint-plugin-jsx-a11y
[35]: https://github.com/facebookincubator/create-react-app
[36]: https://www.deque.com/products/axe-core/
[37]: https://www.deque.com/products/axe/
[38]: https://github.com/dylanb/react-axe
[39]: http://wave.webaim.org/extension/
[40]: https://www.paciellogroup.com/blog/2015/01/the-browser-accessibility-tree/
[41]: https://gist.github.com/marcysutton/0a42f815878c159517a55e6652e3b23a
[42]: https://developer.apple.com/library/content/documentation/Accessibility/Conceptual/AccessibilityMacOSX/OSXAXTestingApps.html
[43]: https://www.nvaccess.org/
[44]: http://webaim.org/articles/nvda/
[45]: https://dequeuniversity.com/screenreaders/nvda-keyboard-shortcuts
[46]: http://webaim.org/articles/voiceover/
[47]: https://dequeuniversity.com/screenreaders/voiceover-keyboard-shortcuts
[48]: https://dequeuniversity.com/screenreaders/voiceover-ios-shortcuts
[49]: http://www.freedomscientific.com/Products/Blindness/JAWS
[50]: http://webaim.org/articles/jaws/
[51]: https://dequeuniversity.com/screenreaders/jaws-keyboard-shortcuts








