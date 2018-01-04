---
layout: post
title: angular2路由懒加载
tags:
  - angular2
---

## 懒加载的目的
当使用`ng-cli`发布`angular2`项目时，我们的代码一般会打包到一个名为`main.***.bundle.js`的文件中，里面包含了所有我们自己开发的模块的代码以及我们项目中所依赖的第三方模块的代码。随着我们的项目越来越复杂，我们自己开发的模块会越来越多，第三方依赖也越来越多，打包生成的`main.***.bundle.js`文件会越来越大，这会使我们的项目首次加载的时间越来越长。

这个时候我们可以使用路由懒加载来优化我们的项目。其思路是：我们将一些不常用的模块不打包到`main.***.bundle.js`中，只有当用户主动访问该模块时，才去请求该模块的代码，从而减少`main.***.bundle.js`的大小，加快加载首页的时间。

## 懒加载配置
### 1. 将子路由模块`message`的`path`设为`''`
old:
```
const routes:Route[] = [
    {
        path: 'message',
        pathMatch: 'full',
        component: MessageListComponent,
        canActivate: [AuthService],
        resolve: {
            message: MessageService
        }
    }
];
```
new:
```
const routes:Route[] = [
    {
        path: '',
        ...
    }
];
```
> 注意懒加载的路由模块注册路由时需使用`forChild`

### 2. 在主路由模块中添加一个`path`为`message`的路由，并将`loadChildren`设为`MessageModule`所在的路径。
```
const routes:Route[] = [
    {
        path: 'message',
        loadChildren: 'app/modules/message/message.module#MessageModule'
    },
    ...
]
```

### 3. 在`AppModule`中移除所有`MessageModule`的引用。

### 4. 观察`ng serve`控制台，我们会发现新生成了一个名为`message.module.chunk.js`的文件。


## 懒加载策略
我们可以在主路由模块中使用`preloadingStrategy`配置懒加载策略。
```
@NgModule({
    imports:[
        RouterModule.forRoot(routes, {
            useHash: true,
            preloadingStrategy: PreloadAllModules
        })
    ],
    exports:[
        RouterModule
    ]
})
```
`angular2`有两种懒加载策略：
+ `NoPreloading`, 默认值，不自动加载配置了懒加载的模块，只有当用户主动访问该模块时，才加载该模块的文件。
+ `PreloadAllModules`, 在主模块加载成功后，立马加载所有的配置了懒加载的模块。

> `canLoad`能阻止模块的懒加载，其优先级高于`canActivate`。

## 自定义路由加载策略
`angular2`内置的两种懒加载策略，要么在主模块加载后不加载所有模块，要么在主模块加载后加载所有模块，通过自定义懒加载策略我们可以实现在主模块加载后，根据我们的配置项加载指定的模块。
### 1. 给需要懒加载的模块，添加一个特别的属性。
这里我们在`data`上添加一个`preload`属性，主模块加载后，我们希望只加载`preload`为`true`的模块。
```
const routes:Route[] = [
    {
        path: 'message',
        loadChildren: 'app/modules/message/message.module#MessageModule',
        data: {
            preload:true
        }
    },
    {
        path: 'user/:loginname',
        loadChildren: 'app/modules/user/user.module#UserModule'
    },
    ...
]
```

### 2. 新建一个文件实现`PreloadingStrategy`接口
实现`PreloadingStrategy`接口，我们需要实现`preload`函数，`preload`函数有两个参数，一个是当前路由`route`，一个是`load`函数，我们可以从`route`上取得自定义的数据，然后判断是否需要加载，需要加载只需调用`load`函数即可，如果不需要加载，我们需要返回一个空的`Observable`对象。
```
import 'rxjs/add/observable/of';
import { Injectable } from '@angular/core';
import { PreloadingStrategy, Route } from '@angular/router';
import { Observable } from 'rxjs/Observable';

@Injectable()
export class SelectivePreloadingStrategy implements PreloadingStrategy {
    preloadedModules: string[] = [];

    preload(route: Route, load: () => Observable<any>): Observable<any> {
        if (route.data && route.data['preload']) {
            // add the route path to the preloaded module array
            this.preloadedModules.push(route.path);

            // log the route path to the console
            console.log('Preloaded: ' + route.path);

            return load();
        } else {
            return Observable.of(null);
        }
    }
}
```

### 3. 在主模块的`providers`中注册`SelectivePreloadingStrategy`服务，然后将主路由模块中的`preloadingStrategy`的值设为`SelectivePreloadingStrategy`
```
providers: [
    SelectivePreloadingStrategy,
    ...
  ],
```
```
imports:[
    RouterModule.forRoot(routes, {
        useHash: true,
        preloadingStrategy: SelectivePreloadingStrategy
    })
],
```