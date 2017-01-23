## 如丝般顺滑：使用 CSS3 实现 60 帧的动画

*2016-08-17* *yanni4night* [奇舞周刊](http://mp.weixin.qq.com/s?__biz=MzA4NjE3MDg4OQ==&mid=2650963599&idx=1&sn=bb28e6cf0e3146252313d22fe42cb885&scene=0##)

编者按：本文由yanni4night在众成翻译平台上翻译，英文原文：https://medium.com/outsystems-experts/how-to-achieve-60-fps-animations-with-css3-db7b98610108#.vfuqpb6vu

在移动端上实现动画很简单。

在移动端上正确地实现动画也比较简单...如果你采纳我们的建议的话。

虽然现在每个人都会使用 CSS3 实现动画，但许多人用的都不够恰当。很多应加以考虑的最佳实践常常被忽略，因为仍然有人不明白这些最佳实践的真正意义。

如今有这么多的设备规范，如果还不有针对性地优化你的代码，糟糕的用户体验将让你死无葬身之地。

记住：虽然市场上始终有一些高端的旗舰机在挑战性能极限，但你面对的仍将是和这些性能怪兽相比只是玩具一样的低端设备。

我们想帮助你正确地驾驭 CSS3。首先先要了解几件事。

### 了解时间轴

当渲染和处理 HTML 元素时，浏览器做了什么？这个时间轴叫做关键渲染路径。

![img](http://mmbiz.qpic.cn/mmbiz_png/d8tibSEfhMMr7rIhSASzztQWE86Ve2uVCRwQ0sclexZUp3vS6NodGDqDfqeOafs6XsvDuz3jV5Ymg8WVgD3bdHA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1)

想达到流畅的动画效果我们需要关注修改属性对合成（Composite）阶段的影响，而不是其它阶段。

#### 1. 样式

![img](http://mmbiz.qpic.cn/mmbiz_png/d8tibSEfhMMr7rIhSASzztQWE86Ve2uVCX7FzsLib6yq8ZJ0t8J7rbJWRiaMurvmT0XLb3XluFo30aOsKEFpfVbXw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1)

浏览器开始计算应用于元素的样式 —— 重计算样式。

#### 2. 布局

![img](http://mmbiz.qpic.cn/mmbiz_png/d8tibSEfhMMr7rIhSASzztQWE86Ve2uVCXibVibMZpquibw56k5FOT5PqDROib10niapqtbA5h2WCiaIWWm7beBhYGuRg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1)

下一步，浏览器开始为每个元素生成形状和位置 —— 布局。这是浏览器设置页面属性的地方，如 width 和 height，以及 margin 或者 left、top、right、bottom。

#### 3. 渲染

![img](http://mmbiz.qpic.cn/mmbiz_png/d8tibSEfhMMr7rIhSASzztQWE86Ve2uVCf6SMXPLlQH0zFQm7ETaIjBv0HlEcoYDjYeHGFNiblicsfv81hL4FElZw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1)

浏览器开始向层中填充像素。要使用的属性有 box-shadow，border-radius，color，background-color 等等。

#### 4. 合成

这就是你动手脚的地方了，因为浏览器开始把所有层画到屏幕上。

![img](http://mmbiz.qpic.cn/mmbiz_png/d8tibSEfhMMr7rIhSASzztQWE86Ve2uVCTVvNQ08uXYwljtEibibbLdibc9UmgyKsohrthDMke8bhniayUCRnZWg1dA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1)

现代浏览器能够通过使用 transform 和 opacity 完美运行 4 种样式。

- 位置 — transform: translateX(*n*) translateY(*n*) translateZ(*n*);
- 缩放 — transform: scale(*n*);
- 旋转 — transform: rotate(*n* deg);
- 透明 — opacity: *n*;

### 如何达到 60 帧每秒

想法有了，让我们撸起袖子开始干活吧。

我们从 HTML 开始，创建一个非常简单的结构，并把类名 app-menu 的元素放入类名 layout 的元素中。

```html
<div class="layout">
    <div class=”app-menu”></div>
    <div class=”header”></div>
</div>
```

![img](http://mmbiz.qpic.cn/mmbiz_gif/d8tibSEfhMMr7rIhSASzztQWE86Ve2uVCz78qgRzXVrSic3acWHXcqTbicNicOePSBzr7UptK992XVMpzHU3Cia6ghg/0?wx_fmt=gif&tp=webp&wxfrom=5&wx_lazy=1)

### 错误做法

```css
.app-menu {
  left: -300px;
  transition: left 300ms linear;
}

.app-menu-open .app-menu {
  left: 0px;
  transition: left 300ms linear;
}
```

看到我们更改的这些属性了吗？你应该避免在 transitions 中使用left、top、right、bottom 属性。它们的更新让浏览器每次都要创建布局，影响所有它们的子元素，进而难以实现流畅的动画。

结果是这样的：

![img](http://mmbiz.qpic.cn/mmbiz_gif/d8tibSEfhMMr7rIhSASzztQWE86Ve2uVCNsTicKZB2WooLGqItjF6Wk3QL4xRT3uIDJzNSwkia02pXverBasicISng/0?wx_fmt=gif&tp=webp&wxfrom=5&wx_lazy=1)

这个动画一点也不流畅。我们通过[开发者工具](undefined)检查背后到底发生了什么，请看：

![img](http://mmbiz.qpic.cn/mmbiz_png/d8tibSEfhMMr7rIhSASzztQWE86Ve2uVCU1Y3R5ZxdyY7pruWEWfVU4nMh4oF0sgWJUdQRQ10nuficnriaEBEiacvw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1)

能够清晰地看到 FPS 非常不规则，性能也就比较糟糕。

### 使用 Transform

```css
.app-menu {
  -webkit-transform: translateX(-100%);
  transform: translateX(-100%);
  transition: transform 300ms linear;
}


.app-menu-open .app-menu {
  -webkit-transform: none;
  transform: none;
  transition: transform 300ms linear;
}
```

transform 属性会影响合成阶段。在这里浏览器被告知层马上要被渲染，做好准备执行动画，因此动画渲染时卡顿更少。

![img](http://mmbiz.qpic.cn/mmbiz_gif/d8tibSEfhMMr7rIhSASzztQWE86Ve2uVCPI0qbJaneFVKXiaTkl1ym4pkz4tuR7RwwScIkkjhKXWCBGj6Wd5js1A/0?wx_fmt=gif&tp=webp&wxfrom=5&wx_lazy=1)

时间轴：

![img](http://mmbiz.qpic.cn/mmbiz_png/d8tibSEfhMMr7rIhSASzztQWE86Ve2uVCcMxPzSrQPbUHUsiaLBib17ZFHIzfFultmneCNvIu1fltXaLDbibicGNKVQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1)

开始有效果了，FPS 变得有规律了，此外，动画变得更流畅了。

FPS 开始优化，并且更加稳定，动画也更流畅。

### 使用 GPU 执行动画

百尺竿头更进一步。为了让动画“如丝般顺滑”，我们接下来使用 GPU 来渲染。

```css
.app-menu {
  -webkit-transform: translateX(-100%);
          transform: translateX(-100%);
  transition: transform 300ms linear;
  will-change: transform;
}
```

虽然一些浏览器仍然需要 translateZ() 或 translate3d() 作为备选方案，will-change 被广泛支持已经是势不可挡了。它的功能是把元素提升到另一个层中，这样浏览器就不必关心布局渲染或者绘制了。

![img](http://mmbiz.qpic.cn/mmbiz_gif/d8tibSEfhMMr7rIhSASzztQWE86Ve2uVCJz4S2CXe268Nc8niaFpou2NkXia6IRKN8yrU0cQ7icCkkiaKcVdCeEkItg/0?wx_fmt=gif&tp=webp&wxfrom=5&wx_lazy=1)

这样动画能流畅到什么程度？看时间轴：

![img](http://mmbiz.qpic.cn/mmbiz_png/d8tibSEfhMMr7rIhSASzztQWE86Ve2uVCnctUSx9o8Pnso8Mv6pNZy8gxX4xjuOYvNTHQzpTvo83VkEUg8Oz2bQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1)

动画的 FPS 更稳定了，渲染也更快了。但是有一帧仍然渲染得很久。在开始处还有一点点瓶颈。

记住刚开始创建的 HTML 结构吗？我们用 JavaScript 控制 app-menu div。

```javascript
function toggleClassMenu() {
  var layout = document.querySelector(".layout");
  if(!layout.classList.contains("app-menu-open")) {
    layout.classList.add("app-menu-open");
  } else {
    layout.classList.remove("app-menu-open");
  }
}

var oppMenu = document.querySelector(".menu-icon");

oppMenu.addEventListener("click", toggleClassMenu, false);
```

问题出在对 layout 元素增加类名上，我们使浏览器计算了不止一次样式 —— 这影响了渲染效果。

### 如丝般顺滑的 60 帧 FPS

如果我们在视窗之外创建菜单会如何？这种分离化的区域能够保证只有需要做动画的元素才会被影响。

因此，我们改进下面的 HTML 结构：

```css
<div class="menu">
    <div class="app-menu"></div>
</div>

<div class="layout">
    <div class="header"></div>
</div>
```

此外你也可以通过一种略有不同的方式去控制菜单的状态。我们通过 JavaScript 的transitionend 方法，在监控到动画结束时移除这个控制动画的 CSS 类。

```javascript
function toggleClassMenu() {
  myMenu.classList.add("menu--animatable");
  myMenu.classList.add("menu--visible");
  myMenu.addEventListener("transitionend", 
  OnTransitionEnd, false);
}

function OnTransitionEnd() {
  myMenu.classList.remove("menu--animatable");
}


var myMenu = document.querySelector(".menu");

var oppMenu = document.querySelector(".menu-icon");

oppMenu.addEventListener("click", toggleClassMenu, false);
```

现在把它们放到一起，然后看效果。

下面是完全正确的使用 CSS3 实现动画的例子：

```css
.menu {
  position: fixed;
  left: 0;
  top: 0;
  width: 100%;
  height: 100%;
  overflow: hidden;
  pointer-events: none;
  z-index: 150;
}

.menu—visible {
  pointer-events: auto;
}

.app-menu {
  background-color: #fff;
  color: #fff;
  position: relative;
  max-width: 400px;
  width: 90%;
  height: 100%;
  box-shadow: 0 2px 6px rgba(0, 0, 0, 0.5);
  -webkit-transform: translateX(-103%);
  transform: translateX(-103%);
  display: flex;
  flex-direction: column;
  will-change: transform;
  z-index: 160;
  pointer-events: auto;
}

.menu—-visible.app-menu {
  -webkit-transform: none;
  transform: none;
}

.menu-—animatable.app-menu {
  transition: all 130ms ease-in;
}

.menu--visible.menu—-animatable.app-menu {
  transition: all 330ms ease-out;

}

.menu:after {
  content: ‘’;
  display: block;
  position: absolute;
  left: 0;
  top: 0;
  width: 100%;
  height: 100%;
  background: rgba(0,0,0,0.4);
  opacity: 0;
  will-change: opacity;
  pointer-events: none;
  transition: opacity 0.3s cubic-bezier(0,0,0.3,1);
}

.menu.menu--visible:after{
  opacity: 1;
  pointer-events: auto;
}

```

![img](http://mmbiz.qpic.cn/mmbiz_gif/d8tibSEfhMMr7rIhSASzztQWE86Ve2uVCMibqmVZ70QgV0miaM4sRRxPV5ng8ZN1XCSeVRPgwGuU8Xa6rTenXrgag/0?wx_fmt=gif&tp=webp&wxfrom=5&wx_lazy=1)

时间轴是怎么样的？

![img](http://mmbiz.qpic.cn/mmbiz_png/d8tibSEfhMMr7rIhSASzztQWE86Ve2uVCoLNkyLPcQ1tThGUS5qovAFibpX7Uq6GWuRHL0ZZDJknUWrFsWjRZeWA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1)

如丝般地顺滑，是吧？



