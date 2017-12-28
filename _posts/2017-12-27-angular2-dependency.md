---
layout: post
title: angular2依赖注入的几种提供商
tags:
  - angular2 
---



在`angular2`我们必须通过注册提供商`(provider)`来配置注入器，这些提供商为应用创建所需服务。所谓提供商其实就是我们在组件中注入该服务时取到的值。在`angular2`中有4种类型的提供商。

在模块中注册一个提供商需配置两个必要的属性：`provide`是提供商的名字，这个名字可以是一个字符串或者一个对象。还需要配置表示提供商值的属性，可以是`useClass`、`useValue`、`useFactory`、`useExisting`，下面我们分别介绍这四种值类型提供商的注册和使用方法。

>我们一般不直接使用普通的字符串作为提供商的名字，这样可能会引起冲突。如在某一个模块中注册了一个名为`'DataService'`的提供商，另外一个模块也注入了一个名为`'DataService'`的提供商，由于`'DataService' === 'DataService'`总是成立的，因此`angular2`就没办法区分这两个服务了。当使用字符串作为提供商时，我们一般用`InjectionToken`对字符串进行封装，如：`token1 = new InjectionToken('data service')`,`token2 = new InjectionToken('data service')`。虽然`token1`和`token2`传入的字符串是一样的，但每次`new`产生的对象的引用是不一样的，也就是`token1 === token2`是假的，这就确保了`provide`值的唯一性。


### 1. useClass
`useClass`类型的提供商，其值是一个类。

#### 新建一个`DataService`类，并导出。
`data.service.ts`  
```
import { Injectable } from "@angular/core";

@Injectable()
export class DataService {
    getRandom(max) {
        return Math.ceil(Math.random() * max);
    }
}

```

上面`DataService`就是一个普通的类，里面包含了一个可以生成`0-max`范围内随机数的方法。
注意到类的开头用了`@Injectable`装饰器进行装饰。当该类不依赖于其它服务时，`@Injectable`不是必须的。当该服务依赖于其它服务时，`@Injectable`是必须的。

我们给`DataService`增加了一个记录日志的服务`LoggerService`，此时`@Injectable`就不能省略了。通常情况下，不管服务是否依赖于其它服务，我们最好都用`@Injectable`进行装饰。
```
import { LoggerService } from './logger.service';
import { Injectable } from '@angular/core';

@Injectable()
export class DataService {

    constructor(private loggerService: LoggerService) {

    }

    getRandom(max) {
        const random = Math.ceil(Math.random() * max);
        this.loggerService.log(`get a random between 0 and ${max}: ${random}`);
        return random;
    }
}
```

#### 使用`useClass`在主模块中注册
这里我们使用`DataService`类的引用作为提供商的名字，其值使用`useClass`配置，也是`DataService`类的引用。
```
@NgModule({
  ...
  providers: [
    {
      provide: DataService,
      useClass: DataService
    },
    ...
  ],
  ...
})
export class ProvideModule { }
```

上面这种方法有一种简便写法，当我们直接往`providers`里面添加一个类时，其默认会使用`useClass`进行注册。
```
@NgModule({
  ...
  providers: [
    DataService,
    ...
  ],
  ...
})
export class ProvideModule { }
```

#### 在组件中使用
```
export class ProvideTestComponent implements OnInit {

  constructor(private dataService: DataService) {
    console.log(this.dataService.getRandom(10));
  }

  ngOnInit() {
  }

}
```

### 2. useValue
`useValue`类型的提供商，其值是一般是一个常量。

#### 准备好`token`和所需要的值
```
export const APP_CONFIG = new InjectionToken<any>('app config');
export const config = {
  base: 'http://127.0.0.1',
  name: '测试'
};
```

#### 在模块中注册
```
@NgModule({
  ...
  providers: [
    ...
    {
      provide: APP_CONFIG,
      useValue: config
    }
  ],
})
export class ProvideModule { }
```

#### 在组件中使用

```
import { DataService } from './../../services/data.service';
import { Component, OnInit, Inject } from '@angular/core';
import { APP_CONFIG } from '../../services/config';

@Component({
  selector: 'one-provide-test',
  templateUrl: './provide-test.component.html',
  styleUrls: ['./provide-test.component.less']
})
export class ProvideTestComponent implements OnInit {

  constructor(@Inject(APP_CONFIG) private config, private dataService: DataService) {
    console.log(this.dataService.getRandom(10), config);
  }

  ngOnInit() {
  }

}
```

### 3. useExisting
`useExisting`是别名提供商，一般用来升级旧的服务。当一个项目中已经大量使用的服务需要升级到一个新的服务时，我们能想到的直接的办法就是将注册一个新的服务，然后将旧的服务完全替换掉。但这样很容易出错，而且大量改动已经稳定的代码很容易引出问题。`angular2`为我们提供了一种简便的方法，使用`useExisting`提供商。

假设我们需要将之前的`LoggerService`升级到`NewLoggerService`，我们可以这样做：

```
@NgModule({
  imports: [
    CommonModule,
    ProvideRoutingModule
  ],
  providers: [
    NewLoggerService,           //注册新的服务
    {
      provide: LoggerService,
      useExisting: NewLoggerService //将旧的服务使用的`useClass`改为`useExisting`，其值改为新的服务名
    }
    ...
  ],
  declarations: [ProvideTestComponent]
})
export class ProvideModule { }
```

### 4. useFactory
`useFactory`是工厂提供商，它值一般是一个工厂函数，我们在组件中使用它时，得到的就是该工厂函数返回的值。

#### 注册方法
`APP_INIT`是一个`Injectiontoken`，其值是一个工厂函数，在组件中注入时，我们会得到一个函数，执行该函数会打印`123`。

```
{
    provide: APP_INIT,
    useFactory: () => {
        return () => {
            console.log('123');
        }
    },
},
```
#### 高级用法
上面的用法其实没有很强的代表性，如果仅仅是需要注入一个函数，我们完全可以用`useValue`实现，不需要搞得这么复杂。`useFactory`的特别之处在于工厂函数是可以注入其他服务的，我们需要额外再配置一个`deps`属性，表示工厂函数依赖的服务，像下面这样：
```
{
    provide: APP_INIT,
    useFactory: (dataService:DataService) => {
    return () => {
        // do something
        return dataService.getRandom(10);
    }
    },
    deps: [DataService]
},
```

在`angular2`中`useFactory`一个典型的用法是，结合`APP_INITIALIZER`在程序初始化时做一些准备工作。