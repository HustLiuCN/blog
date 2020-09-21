---
title: 思维导图
date: 2020-06-05 21:50:27
tags: canvas, 可视化, 前端
---

数据平台日常有许多类思维导图的场景，看了一下身边的同事基本都是为这样一个小的需求点引入一个庞大的可视化库，可谓是杀鸡用牛刀，浪费是一回事，显得很耿直是另一回事
于是闲暇之余撸了一个思维导图生成工具，支持基本的日常交互需求，例如**放大、缩小、拖拽，连线支持弧线和折线**

以之前做的一个[**机器学习平台的架构图为demo**](https://stillbold.com/demos/dag-editor/demos/mind.html)

**附: [项目地址](https://github.com/HustLiuCN/dag-editor)**

<!--more-->

大概阐述一下基本思路，相对比较简单

## 源数据

思维导图，如无意外都是单向的数据链路，用最常见的`树形结构`来描述最为合适，这样也降低了工具的使用门槛，增加适用范围有许多三方库需要用户去适配工具，对源数据做很多修饰和转换，个人不是很喜欢这种方式

```javascript
const node = {
	text: '节点内容',
	children: [
		{ ...node },
		// ...nore node
	],
}
```

## 排版布局

这里我们以单侧展开的图为例，两侧或多侧展开只需要将`深度为1的子节点`特殊排版处理即可，本质上是一样 的

### 初始化图形信息

首先初始化定义每个节点的基本图形信息

```javascript
const info = {
	height: 26,	// 节点的绘图高度, 单位 px, 所有节点内容统一为单行排列
	minWidth: 80, // 节点最小宽度
	horizontalGap: 100,	// 节点之间的纵向距离
	verticalHeight: 36,	// 节点的实际占位高度, 可以理解为节点是一个盒模型, height 是 content, verticalHeight 是包括了 padding
}
```

### 绘制

广度优先的遍历树

**注意，根节点在初次遍历的时候额外绘制，为坐标系的(0, 0)。本质上每次遍历是绘制当前节点`node`到其所有子节点`child`的`连线`，以及所有`child`，即当前节点的绘制是在其父节点的遍历循环中完成的，根节点除外**
<small>这部分逻辑略为有些绕，详细代码可参考**[code](https://github.com/HustLiuCN/dag-editor/blob/master/src/mind.js#L144)**</small>

理解上述逻辑，接下来的绘制便比较容易。
绘制所有`child`节点以及连线的过程为

- 计算所有兄弟`child`的数量`n`
- 以父节点`node`的坐标`(x0, y0)`作为原点，利用`n`和每个`child`的顺序`i`上下对称的计算`child`的坐标`(x1, y1)`<small>这里利用到`height`，`horizontalGap`和`verticalHeight`</small>
- 以`(x1, y1)`绘制`child`
- 以`(x0, y0)`和`(x1, y1)`作为顶点，`((x0 + x1)/2, y0)`和`(x0, y1)`作为两个控制点，绘制**[贝赛尔曲线](https://github.com/hujiulong/blog/issues/1)**作为连线。这里控制点的选择可以自行调试

如果这里选择的是**折线**作为连线，则绘制过程为

- `(x0, y0)` -> `((x0 + x1)/2, y0)` -> `((x0 + x1)/2, y1)` -> `(x1, y1)`

### 缩放 & 移动

这里的缩放和移动均是针对**画布**

缩放利用`CanvasRenderingContext2D.transform(sc, 0, 0, sc, x, y)`方法而非`scale()`，其中`sc`为缩放比例。这样可以避免许多计算上的问题，可参考[MDN文档](https://developer.mozilla.org/zh-CN/docs/Web/API/CanvasRenderingContext2D/transform)

移动则利用`CanvasRenderingContext2D.translate(diffX, diffY)`，通过鼠标事件来计算`diff`，同时需要考虑到`scale`系数


**思维导图整体来说还是比较简单的，所有代码均可在[GitHub](https://github.com/HustLiuCN/dag-editor/blob/master/src/mind.js)查看**
