---
layout: post
title: angular2路由
tags:
  - angular2
---

## 配置
### 基础配置
#### 声明`Route`数组
```
const routes:Route [] = [
    {
        path: 'home',
        component: HomeComponent
    },
    {
        path: '',
        redirectTo: '/home',
        pathMatch: 'full'
    },
    {
        path: '**',
        component: NotFoundComponent
    }
]
```

#### 引入`RouterModule`

```
imports: [
    RouterModule.forRoot(routes, {
        enableTracing: true
    })
],
```

#### 在模板中添加`router-outlet`

```
<div style="text-align:center">
  <p>App</p>
  <router-outlet></router-outlet>
</div>
```

几点说明：
1. 最基础的路由只需配置 `path` 和 `component` 两个属性即可，分别表示路由路径和该路径所对应的组件。
2. `path` 值为 `''` 时，表示当匹配到路径为空时匹配该路由，结合`pathMatch`和`redirectTo`属性，我们可以用它来配置默认路由。
    > `pathMatch`可用来设置路由的匹配规则，有两个可选值`prefix`和`full`，默认值是：`prefix`。  
    > `prefix` 表示只匹配前缀，当`path`值为`home`时，`/home`, `/home/123`, 都能匹配到该路由。  
    > `full` 表示要进行完整的匹配，当`path`值为`home`时，只有`/home`能匹配到该路由，, `/home/123`不能匹配到该路由。   
    > 当路由配置项中有`{path: '',redirectTo: '/home'}`， 但没有设置`pathMatch`为`full`时，`angular`会报错，因为`pathMatch`默认值是`prefix`，因为所有的路由前缀都能匹配到`''`。  
3. `redirectTo` 不具有传递性。 如在下面的路由中，当访问`/`路径时，只能跳转到`/home`, 跳转到`/home`后，会直接匹配到`**`路由。
    ```
    const routes:Route [] = [
        {
            path: 'home',
            redirectTo: 'home/all',
            pathMatch: 'full'
        },
        {
            path: 'home/:id',
            component: HomeComponent,
            pathMatch: 'full'
        },
        {
            path: '',
            redirectTo: '/home',
            pathMatch: 'full'
        },
        {
            path: '**',
            component: NotFoundComponent
        }
    ]
    ```
4. `path` 值为 `**` 时，表示当所有路径都匹配不到时匹配该路由，这通常被用来设置 `404` 页面。
5.  `routes`里面`Route`的顺序很重要，解析的时候是从上往下解析的，遇到匹配的`Route`即停止，所以通配符路由一般放在最后。
6. 我们可以通过`enableTracing: true`，开启调试模式。
7. [参考](http://vsavkin.tumblr.com/post/146722301646/angular-router-empty-paths-componentless-routes)

### 子路由
子路由的配置和上面的配置基本一致，只不过是添加到父路由的`children`属性上。

## 参数
### 必要参数
必要参数指在配置 `path` 时指定的参数，会影响路由的匹配，如`/home`, `/home/123`, 是两个完全不同的路由。
下面的路由能匹配到: `/home/123`, 我们能通过`ActivateRoute`取得`id`的值, 访问`/home`时，会跳转到`**`路由。
```
const routes:Route [] = [
    {
        path: 'home/:id',
        component: HomeComponent,
    },
    {
        path: '',
        redirectTo: '/home/1',
        pathMatch: 'full'
    },
    {
        path: '**',
        component: NotFoundComponent
    }
]
```
如果我们希望`/home`也能使用`HomeComponent`组件，我们可能会这样设置：
```
const routes:Route [] = [
    {
        path: 'home',
        component: HomeComponent,
    },
    {
        path: 'home/:id',
        component: HomeComponent,
    },
    {
        path: '',
        redirectTo: '/home/123',
        pathMatch: 'full'
    },
    {
        path: '**',
        component: NotFoundComponent
    }
]
```
此时，访问`/home`能正常显示，访问`/home/123`时，控制台会报错，这是因为，`{path: 'home',component: HomeComponent},` 是能匹配到`/home/123`的，但他不能处理后面的`/123`参数。   
要使这两个路由都能正常工作，可设置`pathMatch`属性为`full`即可，这样`path: 'home'`就只会匹配到`/home`,  `path:'home/:id'`就只会匹配到`/home/123`。
```
const routes:Route [] = [
    {
        path: 'home',
        component: HomeComponent,
        pathMatch: 'full'
    },
    {
        path: 'home/:id',
        component: HomeComponent,
        pathMatch: 'full'
    },
    {
        path: '',
        redirectTo: '/home',
        pathMatch: 'full'
    },
    {
        path: '**',
        component: NotFoundComponent
    }
]
```
### 可选参数
可选参数是指在`url`后面添加的参数，参数以`;`开始，参数对的格式为`key=value`，多个参数对以`;`分割。格式类似于：
```
/home/456;id=123;name=jack
```
### 参数的获取
必要参数和可选参数的获取都可以从`ActivatedRoute`上的`paramMap`上获取，`paramMap`是一个`Observable`对象，我们可以这样获取参数的值:
```
    this.activatedRoute.paramMap.subscribe((paramMap:ParamMap) => {
      console.log(paramMap.keys);
    })
```
> `params`是老的获取必要参数和可选参数的方法，在新版本中仍然可以使用，但最好使用`paramMap`。
> 可选参数的值会覆盖必要参数的值，`path:'home/:id'`，匹配到`/home/456;id=123;name=jack`，获取到的参数对象是：`{id:'123',name:'jack'}`

### snapshot
`ActivatedRoute`有一个属性`snapshot`，上面保存了当前路由信息的一个快照，我们不需要订阅就可以直接使用它们。
```
this.activatedRoute.snapshot.paramMap.keys();
```

### 为什么要使用`Observable`?
在一些复杂的场景下，我们可能会碰到路由参数改变了，但组件没有重新初始化的情况。
假设这样一种场景，我们有一个用户列表，点击列表中的某个数据，我们需要能在同一界面显示用户详情。这里我们会用到子路由，路由配置项如下，
当我们第一次点击用户列表时，`UserDetailComponent`会初始化，我们一般在`ngOnInit`中实现获取用户详情的操作，当我们后面再点击用户列表时，出于组件复用的考虑，`angular`并不会销毁`UserDetailComponent`组件，而是会触发`ngDoCheck`, 这样`ngOnInit`就不会再执行了。如果我们直接从快照上获取，就会造成路由更新了，用户信息但没有更新的情况。
> 那我们为什么不把初始化放在`ngDoCheck`中呢？会触发`ngDoCheck`的行为有很多，这样会频繁的去更新用户信息。  
> 使用`Observable`时，我们不需要专门去取消订阅。ActivateRoute及其可观察对象都是由Router本身负责管理的。 Router会在不再需要时销毁这个路由组件，而注入进去的ActivateRoute也随之销毁了。
```
    {
        path: 'user',
        component: UserComponent,
        children: [
            {
                path: 'detail/:id',
                component: UserDetailComponent
            },
            {
                path: '',
                component: UserEmptyComponent
            },
        ]
    },
```

## router-outlet
路由的一个好处就是刷新后，能保持之前的界面。有时候，我们希望某一个打开的弹窗能在界面刷新后不消失，我们就需要用到二级路由了。二级路由允许我们创建一个或多个与当前路由平级的带名字的`router-outlet`，我们能往里面添加组件，也可随意移除。

### 创建步骤
1. 在模板中添加`router-outlet`, 必须要添加`name`属性，一个模板中只允许有一个不带名字的`router-outlet`。

```
<router-outlet name="popup"></router-outlet>
```

2. 在路由配置项中添加配置项，`path`为路由路径，`outlet`为在模板中指定的`router-outlet`名，`component`为弹出的组件。

```
[
    ...
     {
        path: 'compose',
        outlet: 'popup',
        component: PopupComponent
    },
    ...
]
```

3. 跳转到`outlet`

```
this.router.navigate([{ outlets: { popup: ['compose'] }}]);
```

4. 关闭
```
this.router.navigate([{ outlets: { popup: null }}]);
```

## Router, NavigationExtras, routerLink
使用`Router`和`NavigationExtras`，我们可以用代码实现路由跳转。
使用`routerLink`指令可以在模板中实现路由跳转。

1. 跳转到指定url

```
// router
this.router.navigate(['/home']);
// routerLink
<a routerLink="/home">Home</a>
```

2. 添加必要参数

```
// router
this.router.navigate(['/user/detail/1']);
// routerLink
<a routerLink="/user/detail/1">User</a>
```

3. 添加可选参数

```
// router
this.router.navigate(['/user/detail/1', {test:123}]);
// routerLink
<a [routerLink]="['/user/detail/1', {test:123}]">User</a>
```

4. 弹出outlet

```
// router
this.router.navigate(['/user/detail/1', {test:123}, { outlets: { ad: ['user-ad'] }}]);
// routerLink
<a [routerLink]="['/user/detail/1', {test:123}, { outlets: { ad: ['user-ad'] }}]">User</a>
```

5. 关闭outlet

```
// router
this.router.navigate(['/user/detail/1', {test:123}, { outlets: { ad: null }}]);
// routerLink
<a [routerLink]="['/user/detail/1', {test:123}, { outlets: { ad: null }}]">User</a>
```

6. 相对路径

`routerLink`默认使用相对路由。`./`,`../`,`/`
`Router`默认使用绝对路由。要使用相对路由，可设置：
```
 let navigationExtras: NavigationExtras = {
    relativeTo:this.state,
    ...
};
```

7. 查询参数

```
// router
 let navigationExtras: NavigationExtras = {
    queryParams: { debug: true },
    fragment: 'education'
};
this.router.navigate(['/user', {test:123,test2:345}], navigationExtras);

// routerLink
<a [routerLink]="['/user/bob']" [queryParams]="{debug: true}" fragment="education">
  link to user component
</a>

// 保存之前的参数
// router
let navigationExtras: NavigationExtras = {
    relativeTo:this.state,
    queryParamsHandling: 'preserve',
    preserveFragment: true
};
this.router.navigate([`detail/${user.id}`], navigationExtras)
// routerLink
<a [routerLink]="['/user/bob']" preserveQueryParams preserveFragment>
  link to user component
</a>
```

## 路由模块
将路由配置提取出来放到单独的模块，能精简主模块的配置项，方便我们对代码进行管理。
路由模块应该是纯净的，不包含任何组件或服务的声明，仅仅负责路由的配置。
路由模块引入的时候，应该放在主模块路由模块的前面，因为一般来说，通配符路由和空路由都是在主模块路由里面配置的，路由解析时是按引入模块的顺序解析的。如果主模块放在前面，会导致其他路由被覆盖。

```
import { RouterModule,Route } from '@angular/router';
import { NgModule } from '@angular/core';
import { AuthorityComponent } from './authority/authority.component';
import { PasswordComponent } from './password/password.component';

const routes:Route []= [
    {
        path:'authority',
        component: AuthorityComponent,
        children: [
            {
                path: '',
                pathMatch: 'full',
                redirectTo: 'password'
            },
            {
                path: 'password',
                component: PasswordComponent
            }
        ]
    }
]

@NgModule({
    imports:[
        RouterModule.forRoot(routes)
    ],
    exports:[
        RouterModule
    ]
})
export class AuthorityRoutingModule{

}
```

## 路由守卫
+ 用`CanActivate`来处理导航到某路由的情况。
+ 用`CanActivateChild`来处理导航到某子路由的情况。
+ 用`CanDeactivate`来处理从当前路由离开的情况.
+ 用`Resolve`在路由激活之前获取路由数据。
+ 用`CanLoad`来处理异步导航到某特性模块的情况。