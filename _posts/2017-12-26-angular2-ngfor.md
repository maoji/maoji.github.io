---
layout: post
title: angular2中的ngFor
tags:
  - angular2
---

## 准备
我们先写一个简单的`service`,可生成一个格式如下的随机数列。

```
[{id:3},{id:5},{id:8}...]
```

```
import { Injectable } from '@angular/core';

@Injectable()
export class DataService {
    base = 0;

    getList(len) {
        const data = [];
        for (let i = 0; i < len; i++) {
            const item = this.getItem(3);
            data.push(item);
            this.base = item.id;
        }
        return data;
    }

    getItem(delta) {
        const id = this.base + Math.ceil(Math.random() * delta) + 1;
        return {
            id
        };
    }

    reset() {
        this.base = 0;
    }
}
```
## 一般用法

### 微语法
```
<ul class="one-list">
  <li *ngFor="let item of list;i = index;odd=odd;even=even;first=first;last=last">
    编号：{{i}},奇数：{{odd}},偶数：{{even}},第一个：{{first}},最后一个：{{last}}
    <one-item [data]="item"></one-item>
  </li>
</ul>
```

### 原始语法
```
<ul class="one-list">
  <ng-template ngFor let-item [ngForOf]="list" let-i="index" let-odd="odd" let-even="even" let-first="first" let-last="last">
    <li>
      编号：{{i}},奇数：{{odd}},偶数：{{even}},第一个：{{first}},最后一个：{{last}}
      <one-item [data]="item"></one-item>
    </li>
  </ng-template>
</ul>
```

## trackBy
`angular2`使用列表中对象引用来跟踪列表中的插入和删除操作。下面这些操作均不会造成列表大面积的更新，这里大面积的更新是指先全部销毁然后全部重建。
### 插入一个元素
```
    list.push(item);
    // 或
    list = [...list, item];

    list.unshift(item)
    // 或
    list = [item, ...list];
```

### 移除一个元素
```
    list.splice(index,1)
    list.pop();
```

### 列表排序
```
    list.reverse()
```
### 追加一个新的列表
```
    list = [...list, ...data];
```


上面这些操作有些没有改变`list`的引用，如：`push`、`pop`、`splice`、`unshift`、`reverse`等等，有些改变了`list`的引用，如：`...`。
不管改变了列表的引用，还是没有改变列表的引用，这些操作都有一个共同的特点，就是列表内元素的引用没有大面积的更改。`angular2`会比较元素的引用，如果元素的引用没有改变，那么该元素在位置改变或者列表引用改变的时候不会销毁重建。

当列表中元素的引用全部更改时，如：从服务器获取的数据。此时列表中的元素会全部重新销毁并重建。如果旧的列表和新的列表，有很多元素数据是相同的，虽然他们的引用变了，但实际是不需要销毁重建的。

这个时候，我们可以考虑使用`trackBy`来优化。`trackBy`会根据我们指定的规则来判断列表中的元素是否需要重建，我们可以使用某些字段比如`id`来告诉`angular2`，如果新的列表和旧的列表有两个元素`id`相同，那么该元素是不需要销毁重建的。


```
<ul class="one-list">
  <li *ngFor="let item of list;let i = index;let odd=odd;let even=even;let first=first;let last=last;trackBy:trackByFn">
    编号：{{i}},奇数：{{odd}},偶数：{{even}},第一个：{{first}},最后一个：{{last}}
    <p>
      <button (click)="insert(i)">Insert</button>
      <button (click)="remove(i)">Remove</button>
    </p>
    <one-item [data]="item"></one-item>
  </li>
</ul>
```

```
 trackByFn(index, item) {
    return item.id;
  }
```

我们给`Item`组件添加两个生命周期的函数`ngOnInit`、`ngOnDestroy`，以便我们能观察到元素的初始化与销毁过程。
```
  ngOnInit() {
    console.log(`ngOnInit ${this.data.id}`);
  }

  ngOnDestroy() {
    console.log(`ngOnDestroy ${this.data.id}`);
  }
```

### 效果:
#### 不使用`trackBy`
从打印出的记录可以看出，每次更新列表中的元素都会销毁重建。对于`id=6`的元素,我们认为是不需要销毁重建的。
```
// 第一次初始化
ngOnInit 4  
ngOnInit 6          // id 6 初始化
ngOnInit 8
ngOnInit 10
ngOnInit 13
// 全部销毁
ngOnDestroy 4
ngOnDestroy 6      // id 6 销毁
ngOnDestroy 8
ngOnDestroy 10
ngOnDestroy 13
// 初始化新的数据
ngOnInit 2
ngOnInit 6         // id 6 重新初始化
ngOnInit 9
ngOnInit 11
ngOnInit 15
```

#### 使用`trackBy`
从打印出的记录可以看到，第二次更新列表时，列表只是部分销毁，`id=2`的元素没有销毁重建。
```
// 第一次初始化
ngOnInit 2
ngOnInit 6
ngOnInit 8
ngOnInit 11
ngOnInit 13
// 销毁
ngOnDestroy 6
ngOnDestroy 8
ngOnDestroy 11
ngOnDestroy 13
// 初始化新的列表
ngOnInit 4
ngOnInit 7
ngOnInit 9
ngOnInit 12
```

