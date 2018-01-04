---
layout: post
title: flex布局和angular flxlayout
tags:
  - angular2
  - flexlayout
  - flex
---


## flex布局的几个基本概念

### 与容器相关的几个属性

#### flex-direction
`flex-direction`用于设置主轴的方向，主轴的方向即是容器中元素的排列方向。

它可能有4个值：
+ row（默认值）：主轴为水平方向，起点在左端。
+ row-reverse：主轴为水平方向，起点在右端。
+ column：主轴为垂直方向，起点在上沿。
+ column-reverse：主轴为垂直方向，起点在下沿。

#### flex-wrap
`flex-wrap`用于设置当元素在一行排列不下时，超出部分的元素该如何换行。

它可能有三个值：
+ nowrap（默认）：不换行。
+ wrap：换行，第一行在上方。
+ wrap-reverse：换行，第一行在下方。

#### flex-flow
`flex-flow`是`flex-direction`和`flex-wrap`的简写。默认值为`row nowrap`。第一个值是`flex-direction`，第二个值是`flex-wrap`。

#### justify-content
`justify-content`用于定义元素在主轴上的对其方式。

它可能取5个值，具体对齐方式与轴的方向有关。下面假设主轴为从左到右。
+ flex-start（默认值）：左对齐
+ flex-end：右对齐
+ center： 居中
+ space-between：两端对齐，项目之间的间隔都相等。
+ space-around：每个项目两侧的间隔相等。所以，项目之间的间隔比项目与边框的间隔大一倍。

#### align-items
`align-items` 用于定义元素在交叉轴上的对齐方式。

它可能取5个值。具体的对齐方式与交叉轴的方向有关，下面假设交叉轴从上到下。
+ flex-start：交叉轴的起点对齐。
+ flex-end：交叉轴的终点对齐。
+ center：交叉轴的中点对齐。
+ baseline: 项目的第一行文字的基线对齐。
+ stretch（默认值）：如果项目未设置高度或设为auto，将占满整个容器的高度。

### 与元素相关的几个属性

#### order
`order`属性定义项目的排列顺序。数值越小，排列越靠前，默认为0。

#### flex-grow
`flex-grow`属性定义项目的放大比例，默认为0，即如果存在剩余空间，也不放大。
如果所有项目的`flex-grow`属性都为1，则它们将等分剩余空间（如果有的话）。如果一个项目的`flex-grow`属性为2，其他项目都为1，则前者占据的剩余空间将比其他项多一倍。

#### flex-shrink
`flex-shrink`属性定义了项目的缩小比例，默认为1，即如果空间不足，该项目将缩小。
如果所有项目的`flex-shrink`属性都为1，当空间不足时，都将等比例缩小。如果一个项目的`flex-shrink`属性为0，其他项目都为1，则空间不足时，前者不缩小。

负值对该属性无效。