# Weex 快速上手记录 #

> **weex**简单来说是一个将遵循**Weex规则**的JS Bundler转换成native应用的方案，JS使用`requireModule`加载并使用native方法。

## weex规则 ##
### CSS ###
- 只支持`border-box`特性的盒子模型
- 只支持`flexbox`作为布局模型
- 只支持两种颜色的线性渐变
- 不支持相对长度单位，如`%`、`em`、`rem`，只支持像素值`px`，
- 不支持CSS的简写形式，如：
```CSS
/* regular css */
margin:0 1px 2px 3px;
border-style: dotted;
```
```
/* weex css */
margin-top:0;
margin-right:1px;
margin-bottom:2px;
margin-left:3px;
border-top-style:dotted;
border-right-style:dotted;
border-bottom-style:dotted;
border-left-style:dotted;
```
- 不支持`z-index`设置元素层级
- 不支持`transform`的3D系变换
- 不支持`display:none`
- 不支持除了`active`, `focus`, `disabled`, `enabled`以外的伪类，后三种仅`input`和`textarea`支持，除`:disabled:focus`和`:disabled:enabled`互斥外，其它的伪类均可互联，如：`input:active:enabled`
- 不支持颜色的`HSL`, `HSLA`, `currentColor`, 8字符表示法，而`rgb`, `rgba`会比其它表示法的性能差很多
- `lines: {number=0}` 只用于`<text>`，限制文本行数
- `font-weight`不支持`lighter`, `bolder`这种相对值
- `font-style`可选值：**`normal`** | `italic`
- `text-decoration`可选值：**`none`** | `underline` | `line-through`
- `text-align`可选值：**`left`** | `center` | `right`
- `text-overflow`可选值：`clip` | `ellipsis`
- `position`可设置为`relative` | `absolute` | `fixed` | `sticky`,`sticky`用于实现元素滚动超出页面之外时的固定行为
#### Android 差异 ####
- 当父view满足下列条件时，父view会clip子view，即形成`overflow:hidden`的效果：
	- 父view为`div`, `a`, `cell`, `refresh`, `loading`时
	- 父view没有`background-image`
	- android >= 4.3
	- android >= 5.0
	- android != 7.0
- 不支持`box-shadow`
- 不支持`overflow: visible`
- `font-weight`仅支持400和700，即`normal`和`bold`

### weex组件 ###
> 在weex中实现了一些标准HTML元素，称为weex组件，weex也扩展了一些常用功能的组件，如`<slider>`、`<waterfall>`等，以下列举一些weex组件与HTML元素不同的地方防止踩坑，对HTML元素的具体实现参见：http://weex.apache.org/cn/references/web-standards.html

- 只有`<text>`组件可以包含文本字面量
```HTML
<!-- regular html -->
<a href="/foo">text</a>
```
```HTML
<!-- weex "html" -->
<a href="/foo"><text>text</text></a>
```
- 使用`<image>`渲染图片，而不是`<img>`，而且必须指定`width`及`height`，否则无法显示
- `<div>`的嵌套层级官方建议应当控制在10层内，否则会有性能问题，可能是因flex布局的解析生成了太多的UIView
- 使用`<list>`与`cell`代替`<ul>`和`<li>`
- `<input>`仅用于文本输入，其`type`属性可以使用`password` | `url` | `email` | `tel`
- 页面根结点只允许`<div>`, `<list>`, `<scroller>`

### 通用事件 ###
除了通用事件外，weex组件有其特别事件，这里就不列举了
- `click`事件并非被所有组件所支持，如`<input>`和`<switch>`不支持，在某些组件中可能会有意外表现，如：`<a>`同时设置`click`事件与`href`属性，其后果不可意料
- `longpress` 长按时触发
- `appear` 可滚动区域内组件在屏幕上可见时触发
- `disappear` 可滚动区域内组件在屏幕上不可见时触发
- `viewappear`在页面显示或页面动画执行前触发，仅用于页面根组件
- `viewdisappear`在页面就要关闭时触发，仅用于页面根组件

### DOM API ###
- Node

```
Node {
	constructor()	// 构造器
	destroy()		// 销毁结点

	ref: string,	// 结点id，用于查找节点
	nextSibling: Node?		// 下一个相邻结点
	previousSibling: Node?	// 上一个相邻结点
	parentNode: Node?		// 父结点
	children: Node[]		// 子结点
	pureChildren: Node[]	// 仅Element，不包含Comment
}
```
- Element

```
Element extends Node {
  constructor(type: string, props: Object?)	// 构造器

  appendChild(node:Node) // 顺序插入
  insertBefore(node: Node, before: Node?) // 后插入
  insertAfter(node: Node, after: Node?) // 前插入

  removeChild(node: Node, preserved: boolean?) // 删除子结点node，preserved=true时从内存中删除，不暂时保留
  clear() // 清除所有子结点

  setAttr(key: string, value: string, silent: boolean?) // 设置属性
  setStyle(key: string, value: string, silent: boolean?) // 设置样式

  addEvent(type: string, handler: Function) // 绑定事件
  removeEvent(type: string)	// 解绑事件
  fireEvent(type: string, e: any) // 触发事件

  nodeType: number // 永远是数字1
  type: string // 来自构造函数
  attr: object // 属性键值对，禁止直接修改，使用setAttr()
  style: object // 样式键值对，禁止直接修改，使用setStyle()
  event: object // 所有事件绑定，禁止直接修改，使用addEvent()/removeEvent()
  toJSON(): object // 将Element序列化为JSON
}
```
- Document

```
Document {
  constructor(id: string, url: string?) // 构造器
  destroy() // 销毁页面

  createElement(tagName: string, props: Object?): Element // 创建Element
  createComment(text: string): Comment // 创建Comment
  createBody(tagName: string, props: Object?): Element // 创建主体结点
  fireEvent(el: Element, type: string, e: Object?, domChanges: Object?) //触发事件

  id: string // 实例id
  URL: string? // JS bundle URL
  body: Element // 主体结点
  documentElement: Element // 文档主体结点的父结点
  getRef(id): Nod // 使用结点id查找结点
}
```
除此之外还有个Common结点，非常简单就此不表
