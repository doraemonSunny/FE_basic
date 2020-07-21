# CSS相关

#### 一、浏览器渲染过程



![webkit渲染过程](https://segmentfault.com/img/remote/1460000017329983?w=624&h=289)



1. 解析HTML，生成DOM树，解析CSS，生成CSSOM树
2. 将DOM树和CSSOM树结合，生成渲染树(Render Tree)
3. Layout(回流):根据生成的渲染树，进行回流(Layout)，得到节点的几何信息（位置，大小）
4. Painting(重绘):根据渲染树以及回流得到的几何信息，得到节点的绝对像素
5. Display:将像素发送给GPU，展示在页面上。

---

#### 二、重绘与回流

当元素的样式发生变化时，浏览器需要触发更新，重新绘制元素。这个过程中，有两种类型的操作，即重绘与回流。

- **重绘(repaint)**: 当元素样式的改变不影响布局时，浏览器将使用重绘对元素进行更新，此时由于只需要UI层面的重新像素绘制，因此 **损耗较少**
- **回流(reflow)**: 当元素的尺寸、结构或触发某些属性时，浏览器会重新渲染页面，称为回流。此时，浏览器需要重新经过计算，计算后还需要重新页面布局，因此是较重的操作。会触发回流的操作:
  - 页面初次渲染
  - 浏览器窗口大小改变
  - 元素尺寸、位置、内容发生改变
  - 元素字体大小变化
  - 添加或者删除可见的 dom 元素
  - 激活 CSS 伪类（例如：:hover）
  - 获取布局信息或调用某些方法
    - clientWidth、clientHeight、clientTop、clientLeft
    - offsetWidth、offsetHeight、offsetTop、offsetLeft
    - scrollWidth、scrollHeight、scrollTop、scrollLeft
    - getComputedStyle()
    - getBoundingClientRect()
    - scrollTo()

*回流必定触发重绘，重绘不一定触发回流。重绘的开销较小，回流的代价较高。*

**浏览器的自动优化：**

浏览器会把多次修改”保存“起来，大多数浏览器通过队列化修改并批量执行来优化回流过程，一次完成。但是有些操作会强制刷新队列并立即执行，比如以上获取布局信息的操作。

**最佳实践:**

- css
  - 避免使用`table`布局
  - 将动画效果应用到`position`属性为`absolute`或`fixed`的元素上
- javascript
  - 避免频繁操作样式，可汇总后统一 **一次修改**
  - 尽量使用`class`进行样式修改
  - 减少`dom`的增删次数，可使用 **字符串** 或者 `documentFragment` 一次性插入
  - 极限优化时，修改样式可将其`display: none`后修改
  - 避免多次触发上面提到的那些会触发回流的方法，可以的话尽量用 **变量存住**

---

#### 三、浏览器的主线程和合成线程

**主线程的主要任务：**

* 运行JavaScript
* 计算HTML元素的CSS样式
* layout（relayout）
* 将页面元素绘制成一张或多张位图
* 将位图发送给合成线程

**合成线程的主要任务：**

- 利用GPU将位图会知道屏幕上
- 让主线程将可见的或即将可见的位图发给自己
- 计算哪部分页面是可见的
- 计算哪部分页面是即将可见的（当你的滚动页面的时候）
- 在你滚动时移动部分页面

在很长的一段时间内，主线程都在忙于运行Javascript和绘制元素。

例如，当用户滚动一个页面时，合成线程会让主线程提供最新的可见部分的页面位图。然而主线程不能及时的响应。这时合成线程不会等待，它会绘制已有的页面位图。对于没有的部分则绘制白屏。

**GPU擅长的领域：**

- 绘制东西到屏幕上
- 一次次绘制同一张位图到屏幕上
- 绘制同一张位图到不同的位置、旋转角度和缩放比例

**可以使用GPU加速的CSS3属性：**

- CSS transform
- CSS opacity
- CSS filter

CSS的最终表现分为以下四步：`Recalculate Style` -> `Layout` -> `Paint Setup and Paint` -> `Composite Layers`，以上属性位于`Composite Layers`（组合层）。

---

#### 四、怎么清除无用的css样式

[PurgeCSS](https://github.com/FullHuman/purgecss)

---

#### 五、层叠上下文 Stacking Context

文档中的层叠上下文由满足以下任意一个条件的元素形成：

- 文档根元素（`<html>`）；
- [`position`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/position) 值为 `absolute`（绝对定位）或 `relative`（相对定位）且 [`z-index`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/z-index) 值不为 `auto` 的元素；
- [`position`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/position) 值为 `fixed`（固定定位）或 `sticky`（粘滞定位）的元素（沾滞定位适配所有移动设备上的浏览器，但老的桌面浏览器不支持）；
- flex ([`flexbox`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/flexbox)) 容器的子元素，且 [`z-index`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/z-index) 值不为 `auto`；
- grid ([`grid`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/grid)) 容器的子元素，且 [`z-index`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/z-index) 值不为 `auto`；
- [`opacity`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/opacity) 属性值小于 `1` 的元素（参见 [the specification for opacity](http://www.w3.org/TR/css3-color/#transparency)）；
- [`mix-blend-mode`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/mix-blend-mode) 属性值不为 `normal` 的元素；
- 以下任意属性值不为none的元素：
  - [`transform`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/transform)
  - [`filter`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/filter)
  - [`perspective`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/perspective)
  - [`clip-path`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/clip-path)
  - [`mask`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/mask) / [`mask-image`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/mask-image) / [`mask-border`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/mask-border)
- [`isolation`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/isolation) 属性值为 `isolate` 的元素；
- [`-webkit-overflow-scrolling`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/-webkit-overflow-scrolling) 属性值为 `touch` 的元素；
- [`will-change`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/will-change) 值设定了任一属性而该属性在 non-initial 值时会创建层叠上下文的元素（参考[这篇文章](http://dev.opera.com/articles/css-will-change-property/)）；
- [`contain`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/contain) 属性值为 `layout`、`paint` 或包含它们其中之一的合成值（比如 `contain: strict`、`contain: content`）的元素。

在层叠上下文中，子元素同样也按照上面解释的规则进行层叠。 重要的是，其子级层叠上下文的 `z-index` 值只在父级中才有意义。子级层叠上下文被自动视为父级层叠上下文的一个独立单元。

总结:

- 层叠上下文可以包含在其他层叠上下文中，并且一起创建一个层叠上下文的层级。
- 每个层叠上下文都完全独立于它的兄弟元素：当处理层叠时只考虑子元素。
- 每个层叠上下文都是自包含的：当一个元素的内容发生层叠后，该元素将被作为整体在父级层叠上下文中按顺序进行层叠。

**7阶层叠**

![image-20200421104603251](/Users/wangjia/Library/Application Support/typora-user-images/image-20200421104603251.png)

---

#### 六、JS判断浏览器是否支持CSS3

各主流浏览器都定义了私有属性，以便让用户体验 CSS3 的新特性。例如，

- Webkit 类型浏览器（如 Safari、Chrome）的私有属性是以`-webkit-`前缀开始，
- Gecko 类型的浏览器（如 Firefox）的私有属性是以`-moz-`前缀开始，
- Konqueror 类型的浏览器的私有属性是以`-khtml-`前缀开始，
- Opera 浏览器的私有属性是以`-o-`前缀开始，
- 而 Internet Explorer 浏览器的私有属性是以`-ms-`前缀开始（目前只有 IE 8+ 支持`-ms-`前缀）。

```javascript
function supportCss3(style) {  
    let prefix = ['webkit', 'Moz', 'ms', 'o'],  
    i,  
    humpString = [],  
    htmlStyle = document.documentElement.style,  
    _toHumb = function (string) {  
      return string.replace(/-(\w)/g, function ($0, $1) {  
      	return $1.toUpperCase();  
      });  
    };  
       
    for (i in prefix)  
    humpString.push(_toHumb(prefix[i] + '-' + style));  
       
    humpString.push(_toHumb(style));  
       
    for (i in humpString)  
    if (humpString[i] in htmlStyle) return true;  
       
    return false;  
}  
```

---

#### 七、grid布局

兼容性：Chrome: 57+    Firefox: 52+   IE: 10+   Safari: 10.1+   Opera: 44+    ios: 10.3+    Android: 5+

```css
.container {
  display: grid;
  grid-template-columns: repeat(3, 33.33%);
  grid-template-rows: repeat(3, 33.33%);
  grid-row-gap: 20px;
  grid-column-gap: 20px;
  /* grid-gap: <grid-row-gap> <grid-column-gap>; */
  grid-auto-flow: column;
}
```

---

#### 八、link 与 @import 区别

- `link`功能较多，可以定义 RSS，定义 Rel 等作用，而`@import`只能用于加载 css
- 当解析到`link`时，页面会同步加载所引的 css，而`@import`所引用的 css 会等到页面加载完才被加载
- `@import`需要 IE5 以上才能使用
- `link`可以使用 js 动态引入，`@import`不行