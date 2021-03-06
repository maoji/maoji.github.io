---
layout: post
title: angular2管道
tags:
  - angular2
---

## 自定义管道
`angular2`中自定义管道很容易，只需用`@Pipe`装饰，然后实现`PipeTransform`接口即可。

实现`PipeTransform`接口需实现`transform`函数，`transform`函数接收两个参数，第一个参数是要转换的值，第二个参数是组件传给管道的参数。

下面是一个可以转换`tab`的管道。
```
import { Pipe, PipeTransform } from '@angular/core';

const TAB_DATA = [
  {
    tab: 'all',
    name: '全部'
  },
  {
    tab: 'share',
    name: '分享'
  },
  {
    tab: 'good',
    name: '精华'
  },
  {
    tab: 'ask',
    name: '问答'
  },
  {
    tab: 'job',
    name: '招聘'
  },
];

@Pipe({
  name: 'tab'
})
export class TabPipe implements PipeTransform {

  transform(value: any, args?: any): any {
    const filter = TAB_DATA.filter(item => {
      if (item.name === value) {
        return item;
      }
    });
    return filter.length > 0 ? filter[0].name : '未知';
  }

}

```

### 纯管道与非纯管道
`Pipe`除了可以设置`name`属性外，还可以设置`pure`属性，当`pure`为`true`时，为纯管道，当`pure`为`false`时，为非纯管道。默认情况下，管道都是纯的。

纯管道和非纯管道的主要区别在于调用管道的时机。
> `Angular`只有在它检测到输入值发生了纯变更时才会执行纯管道。 纯变更是指对原始类型值(String、Number、Boolean、Symbol)的更改或者对对象引用(Date、Array、Function、Object)的更改。

> 非纯管道在`Angular`的每一次变更检测中都会执行。每一次JavaScript事件之后都可能会运行变更检测：每次按键、鼠标移动、定时器以及服务器的响应。 非纯管道如果不加限制的话，其调用会变得非常频繁。

