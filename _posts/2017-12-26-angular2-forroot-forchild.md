---
layout: post
title: angular2 forRoot和forChild
tags:
  - angular2
---


## 怎么实现 `forRoot` 和 `forChild` ?
`forRoot`和`forChild`一般作为模块的静态方法，我们可以直接使用，如：`RouterModule.forRoot()`。要实现这个方法，我们一般需要返回一个`ModuleWithProviders`对象，其包括两个属性：`ngModule`一般就是当前模块的引用，`providers`一般是一组`provider`。

`ModuleWithProviders`接口

```
export interface ModuleWithProviders {
    ngModule: Type<any>;
    providers?: Provider[];
}
```

实现

```
@NgModule({

})
export class XXXModule{
    static forRoot():ModuleWithProviders {
        return {
            ngModule: XXXModule
            providers: []   // 一组provider
        }
    }
}
```

## 为什么要用 `forRoot` 和 `forChild` ？
引入模块我们一般有3个方法:
#### 直接导入
```
import { XXXModule } from 'xxx.module'

@NgModule({
    imports: [
        XXXModule
    ]
})
```

#### 如果`XXXModule`实现了`forRoot`方法，我们可以：
```
import { XXXModule } from 'xxx.module'

@NgModule({
    imports: [
        XXXModule.forRoot()
    ]
})
```

#### 如果`XXXModule`实现了`forChild`方法，我们可以：
```
import { XXXModule } from 'xxx.module'

@NgModule({
    imports: [
        XXXModule.forChild()
    ]
})
```

在`angular`中，对于一些公共的组件，指令，管道，当我们需要在某个模块中使用时，我们就需要在模块的`imports`数组中导入一次。

但公共的服务是只需要引入一次的，对于一些公共的服务，我们一般只需要在主模块中引入一次，然后就可以在其他模块中使用了。

当我们直接导入`XXXModule`时，`angular`会为我们注册该模块上面的所有服务，如果我们在多个模块中导入了`XXXModule`模块，`XXXModule`模块上面的服务是不会重复注册的，因为`angular`会确保服务都是单例的，也就是说一个应用里只会有一个该服务的实例。这能确保我们用服务共享数据时，不同模块取到的数据是一致的。

但有一种情况`angular`处理不了了，当我们懒加载某一个模块时，该模块上面的服务会重新注册，如果该服务已经被注册过了，就会造成应用中一个服务有多个实例的情况。

怎么能确保完全的单例呢？这个时候我们就需要借助`forRoot`和`forChild`了。

观察`ModuleWithProviders`，我们可以看到`providers`属性是可选的，他们的区别也是在`providers`上。我们可以将模块的`providers`从`@NgModule`的配置项中移除掉，然后通过`forRoot`方法去返回`providers`。这样我们在根模块中注册模块时就使用`forRoot`方法，在子模块中注册时，就使用直接导入模块的方法，这样既能实现组件共享，也能确保服务是完全单例的。

当我们在使用了懒加载的模块中使用`RouterModule.forRoot()`时，`angular`会报错，提示我们不要使用`forRoot`。但我们自己开发的模块一般不会有提示。服务是否是完全单例一般情况下对我们的系统可能没有影响，如果我们的系统中使用了服务来共享数据，而且这个数据是可变的，这个时候就需要确保服务完全单例了。
