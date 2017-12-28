---
layout: post
title: angular2 模板驱动型表单
tags:
  - angular2
---

要使用模板驱动型表单，我们必须先导入```FormsModule```模块。
```
@NgModule({
  imports: [
    FormsModule,    //<==
    CommonModule
  ],
  ...
})
```
### 构造表单获取表单数据 ###
*template.component.html*
```
<form #form="ngForm" (ngSubmit)="onSubmit(form.value)">
  <div>
    <label for="">Name: </label>
    <input type="text" name="name" ngModel>
  </div>
  <div>
    <label for="">Password: </label>
    <input type="password" name="password" ngModel>
  </div>
  <button type="submit">Submit</button>
</form>
<div>
  {{form.value | json}}
</div>
```
*template.component.ts*

```
import { Component, OnInit } from '@angular/core';

@Component({
  selector: 'app-template',
  templateUrl: './template.component.html'
})
export class TemplateComponent{
  constructor() { }
 
  onSubmit(value) {
    console.log('submit', value);
  }
}
```
几点说明：
1. `angular2` 会自动为表单添加 `ngForm` 指令，我们不需要再单独写```<form ngForm #form="ngForm" (ngSubmit)="onSubmit(form.value)">```，我们可以通过模板引用变量```#form="ngForm"```引用该指令，获取表单的验证信息和数据。
2. 当给表单内元素添加 `ngModel`指令后，该元素会被自动添加到`ngForm`中，我们可以在`form.controls`上找到它。要注意的是，使用 `ngModel`指令时，必须同时给该元素添加 `name` 属性（`name="name"`），否则控制台会报错。
3. 如果我们确实想使用`ngModel`，又不想给其添加`name`属性，我们可以设置 `[ngModelOptions]="{standalone: true}"`，这样控制台不会报错，但该表单元素不会被添加到`ngForm`上。
4. 在提交表单时我们可以通过`(ngSubmit)="onSubmit(form.value)"`获取表单内添加了 `ngModel` 指令同时设置了 `name` 属性的表单元素的值。

### 添加表单验证显示错误信息 ###
假设`name`和`password`都是必填的，`password`长度须在6到12位之间。

*template.component.html*

```
<form #form="ngForm" (ngSubmit)="onSubmit(form)">
  <div>
    <label for="">Name: </label>
    <input type="text" name="name" ngModel required #name="ngModel">
    <div [style.color]="'red'" *ngIf="name.invalid && (name.dirty || name.touched || form.submitted)">
      <p *ngIf="name.errors.required">name can not be empty!</p>
    </div>
  </div>
  <div>
    <label for="">Password: </label>
    <input type="password" name="password" ngModel required minlength="6" maxlength="12" #password="ngModel">
    <div [style.color]="'red'" *ngIf="password.invalid && (password.dirty || password.touched || form.submitted)">
      <p *ngIf="password.errors.required">password can not be empty!</p>
      <p *ngIf="password.errors.maxlength">the maxlength of password is 12!</p>
      <p *ngIf="password.errors.minlength">the minlength of password is 6!</p>
    </div>
  </div>
  <button type="submit">Submit</button>
</form>
```
几点说明：
1. `angular2`内置了一些常见的验证器，```required```(必填)，```maxlength```(最大长度),```minlength```(最小长度),```max```(最大值),```min```(最小值),```pattern```(正则表达式)...等等，我们可以直接使用它。
2. 我们可以通过模板引用变量```#name="ngModel"```引用```ngModel```指令，来获该元素的验证信息，从而做相应的验证提示。
3. ```*ngIf="name.invalid && (name.dirty || name.touched || form.submitted)"```，这里表示当```name```为脏值或者已经被访问过或者表单已经提交了，```name```值仍然无效时，显示错误信息，详细的状态说明可参考[通过 ngModel 跟踪修改状态与有效性验证][1]。
4. 当元素验证失败时，我们可以通过```errors```属性获取具体是那一项验证没有通过，如：```*ngIf="name.errors.required"```，表示如果`errors`上存在`required`属性，表明用户没有输入。

### 自定义同步验证器 ###
假设我们需要一个可以指定验证规则的密码验证器。
*password.validator.ts*

```
import { AbstractControl, NG_VALIDATORS, Validator } from '@angular/forms';
import {Directive, Input} from '@angular/core'

@Directive({
    selector:'[password]',
    providers: [{provide: NG_VALIDATORS, useExisting: PasswrodDirective, multi: true}]
})
export class PasswrodDirective implements Validator{
    @Input()
    password:string;
    
    validate(control:AbstractControl) {
        let error = null;
        if (this.password) {
            const reg = new RegExp(this.password, 'i');
            if (!reg.test(control.value)) {
                error = {
                    password:control.value
                }
            }
        }
        return error;
    }
}
```
*在模板中使用*
```
<input type="password" name="password" ngModel password="^[0-9]{6,12}$" #password="ngModel">
    <div class="error" [style.color]="'red'" *ngIf="password.invalid && (password.dirty || password.touched || form.submitted)">
      <p *ngIf="password.errors.password">password is illegal!</p>
    </div>
```
几点说明：
1. 验证器的本质就是一个属性型指令。
2. `angular2`内置的验证器都是绑定在`NG_VALIDATORS`上的，我们可以通过设置`providers`属性对其进行扩展。
3. 我们通过`@Input()`给指令添加了一个输入参数，这样我们可以指定验证规则了。
4. 该指令的类实现了```Validator```接口，需要实现```validate```这个方法。```validate```方法接收一个```AbstractControl```类型的参数```control```,我们可以通过```control.value```获取到需要校验的值。```validate```方法返回```null```表示验证成功，返回对象（这个例子中是：```{password:'test'}```）表示验证失败，返回的错误对象的属性会被添加到```errors```对象中。

### 自定义异步验证器 ###
假设我们需要一个可以验证用户名是否重复的验证器。
```name.validator.ts```

```
import { AbstractControl, AsyncValidator, NG_ASYNC_VALIDATORS } from '@angular/forms';
import { Directive } from '@angular/core';

const userList = [
    'jack',
    'mary',
    'jimi',
    'tom'
];

@Directive({
    selector:'[checkName]',
    providers: [{provide: NG_ASYNC_VALIDATORS, useExisting: CheckNameDirective, multi: true}]
})
export class CheckNameDirective implements AsyncValidator{

    validate(control:AbstractControl) {
        console.log(control);
        return new Promise((resolve, reject) => {
            setTimeout(() => {
                if (userList.indexOf(control.value) >= 0) {
                    resolve({
                        'checkName':control.value
                    });
                } else {
                    resolve(null);
                }
            }, 2000)
        });
    }
}
```
*在模板中使用*
```
<input type="text" name="name" ngModel required #name="ngModel" checkName>
    <div class="error" [style.color]="'red'" *ngIf="name.invalid && (name.dirty || name.touched || form.submitted)">
      <p *ngIf="name.errors.required">name can not be empty!</p>
      <p *ngIf="name.errors.checkName">this name is exist!</p>
    </div>
```
几点说明：
1. 异步验证器的本质也是一个属性型指令。
2. 我们可以通过设置`providers: [{provide: NG_ASYNC_VALIDATORS, useExisting: CheckNameDirective, multi: true}]`对`NG_ASYNC_VALIDATORS`进行扩展。
3. 该指令的类实现了```AsyncValidator```接口，需要实现```validate```这个方法。```validate```方法接收一个```AbstractControl```类型的参数```control```,我们可以通过```control.value```获取到需要校验的值。与同步验证器不同的是，异步验证器的```validate```方法返回一个```Promise```对象，当延时操作执行完以后，```resolve(null)```表示验证成功，```resolve({'checkName':control.value})```表示验证失败，```resolve```传递的数据会被附加到```errors```对象中。
4. 当所有的同步验证器验证通过后，才会开始异步验证。


  [1]: https://angular.cn/guide/forms