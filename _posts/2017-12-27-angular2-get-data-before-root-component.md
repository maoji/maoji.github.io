---
layout: post
title: angular2 在应用初始化之前准备数据
tags:
  - angular2
---

## 场景
有时候我们需要在组件初始化之前就准备好一些数据。如果这些数据是固定的，我们可以将这些数据放在一个服务里，在组件初始化的时候我们就可以使用它了。如果数据是需要从后台获取的，在路由中，我们可以通过`resolve`守卫，确保组件在初始化之前已经获取到需要的数据。如果我们需要在根组件初始化之前就准备好数据呢？这个时候路由守卫就不起作用了，幸运的是`angular`提供了一些其他的解决办法。

## `APP_INITIALIZER`
`APP_INITIALIZER`是一个`InjectionToken`，它的值是一个工厂提供商`(useFactory)`。利用服务的`multi`特性，我们可以注册多个提供商。这样在应用初始化时通过`APP_INITIALIZER`注入得到的值会是一个函数数组，`angular`会依次执行每一个函数，如果函数返回了`Promise`，`angular`会确保所有返回的`Promise`执行完成也就是`resolve`之后才会认为是初始化完成了，接着再进行组件的初始化等等操作。

正是因为`APP_INITIALIZER`返回的函数支持返回`Promise`这个特性，如果我们应用初始化时做了一些异步操作，我们能确保在根组件开始初始化之前这些异步操作已经执行完了。


## 实现
如下面的例子，我们添加了一个工厂提供商，它依赖于`AuthService`，在工厂提供商返回的函数里，我们执行了`autoLogin()`方法，他返回一个`Promise`，这样应用初始化会等自动登陆的验证完成之后，再执行其他的操作。在根组件初始化之前，我们能确保已经得到了登陆状态。

#### 注册
```
@NgModule({
  ...
  providers: [
    ...
    {
      provide: APP_INITIALIZER,
      useFactory: (authService: AuthService) => {
        return () => {
          return authService.autoLogin();
        };
      },
      deps: [AuthService],
      multi: true
    },
    ...
  ],
  ...
})
export class AppModule { }
```

#### 异步操作
```
    autoLogin() {
        return Promise((resolve, reject) => {
            const authState = this.getAuthStateFromCache();
            if (authState) {
                this.store.dispatch(new AuthLogin(authState.accesstoken, authState.logininfo));
                this.store.dispatch(new GetNotReadCount());
            }
            resolve(true);
        })
    }
```