---
layout: post
title: angular2 HttpClient
tags:
  - angular2
---

## Get和Post 一般用法
### getTest
```
  getTest() {
    this.httpClient.get(`${this.base_url}/topics`)
    .subscribe(data => {
      console.log(data);
    }, error => {
      console.log(error);
    })
  }
```

### postTest
```
  postTest() {
    const accesstoken = this.accesstoken;
    this.httpClient.post(`${this.base_url}/accesstoken`, {
      accesstoken
    }).subscribe(data => {
      console.log(data);
    }, error => {
      console.log(error);
    })
  }
```

## 几个常见的配置项

### params
通过`params`可配置`get`参数。

```
const params = new HttpParams();
params.append('page', '1');
params.append('tab', 'ask');
params.append('limit ', '30');
params.append('mdrender ', 'false');
this.httpClient.get(`${this.base_url}/topics`, {
    params
})
```

### responseType
通过`responseType`可配置以那种格式解析响应内容，默认是`json`。

```
const responseType = 'text';

this.httpClient.get(`${this.base_url}/topics`, {
    responseType
})
```

### observe
通过`observe`可配置订阅的时候是否获取完整的响应，如果值为`response`，会返回完整的响应体，包括响应状态，响应头等等。如果值为`body`，只会返回响应内容。默认值是`body`。

```
const observe = 'response';

this.httpClient.get(`${this.base_url}/topics`, {
    observe
})
```

### headers
通过`headers`可配置自定义的响应头。
```
let headers = new HttpHeaders();
headers = headers.set('Authorization', 'my-auth-token');
this.httpClient.get(`${this.base_url}/topics`, {
    headers
})
```

## 拦截器
要实现拦截器，需新建一个类实现`HttpInterceptor`接口，实现`HttpInterceptor`接口，需要实现`intercept`函数，我们可以在里面对请求和响应做统一处理。

```
import { Injectable } from '@angular/core';
import { HttpEvent, HttpInterceptor, HttpHandler, HttpRequest, HttpResponse } from '@angular/common/http';
import { Observable } from 'rxjs/Observable';
import 'rxjs/add/operator/map';

@Injectable()
export class NoopInterceptor implements HttpInterceptor {
    intercept(req: HttpRequest<any>, next: HttpHandler): Observable<HttpEvent<any>> {
        // 更改请求信息
        const temp = req.clone({
            headers: req.headers.set('common', 'commonheader')
        });
        return next.handle(temp).map(event => {
            // 更改响应信息
            if (event instanceof HttpResponse) {
                return event.clone({
                    body: `${event.body}test`
                });
            } else {
                return event;
            }
        });
    }
}
```

拦截器类实现好后，需要在主模块中注册。`HTTP_INTERCEPTORS`是一个`InjectionToken`，`multi`为`true`表示，通过该`token`取到的注入值是一个数组。我们可以用自己的拦截器类对其进行扩展。如果有多个拦截器，拦截器按注册的顺序执行。

```
@NgModule({
  ...
  providers: [
    {
      provide: HTTP_INTERCEPTORS,
      useClass: NoopInterceptor,
      multi: true
    }
  ],
  ...
})
export class HttpModule { }
```

## 流程控制

### 取消请求
在订阅一个请求后我们会得到一个`subscriber`,调用其`unsubscribe()`方法可以取消订阅,从而取消请求.
```
const subscriber = this.httpClient.get(`${this.base_url}/topics`)
.subscribe(data => {
    console.log(data);
}, error => {
    console.log(error);
})
subscriber.unsubscribe()
```

### 在一个请求执行完成之后,用该请求的数据,开始另外一个请求.
思路:先使用`map`操作符将第一个请求的响应数据转换成另外一个请求对象,这样得到的是一个高阶`Observable`,然后再用`mergeAll`将高阶`Observable`转换成低阶`Observable`.
```
const req1 = this.httpClient.get(`${this.base_url}/topics`);
const req2 = this.httpClient.get(`${this.base_url}/topics`);
req1.map(data => req2).mergeAll().subscribe(data => {
    console.log(data);
});
```
使用`mergeMap`操作符简写.
```
const req1 = this.httpClient.get(`${this.base_url}/topics`);
const req2 = this.httpClient.get(`${this.base_url}/topics`);
req1.mergeMap(data => req2).subscribe(data => {
    console.log(data);
});
```

### 在一个请求执行完之后,执行另外一个请求
思路:`Observable`的静态方法`concat`会确保传入的`Observable`按顺序执行,并依次`emit`每一个请求的数据.
```
const req1 = this.httpClient.get(`${this.base_url}/topics`);
const req2 = this.httpClient.get(`${this.base_url}/topics`);
Observable.concat(req1, req2).subscribe(data => {
    console.log(data);
})
```

### 在两个请求执行完成之后做一些操作.
思路:`Observable`的静态方法`forkJoin`会等待所有的`Observable`按执行完成后,一次`emit`所有数据.
```
const req1 = this.httpClient.get(`${this.base_url}/topics`);
const req2 = this.httpClient.get(`${this.base_url}/topics`);
Observable.forkJoin(req1, req2).subscribe(data => {
    console.log(data);
})
```

### 避免请求重发
思路: `switchMap`会在接收到一个新的`Observable`时,取消所有之前未完成的`Observable`.
```
const req1 = this.httpClient.get(`${this.base_url}/topics`);
this.subject = new Subject().switchMap(() => req1);
this.subject.subscribe(data => {
    console.log(data);
});
this.subject.next();
```