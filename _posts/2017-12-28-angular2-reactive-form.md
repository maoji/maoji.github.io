---
layout: post
title: angular2 响应式表单
tags:
  - angular2
---

要使用响应式表单，我们必须先导入```ReactiveFormsModule```模块。
```
@NgModule({
  imports: [
    ReactiveFormsModule,    //<==
    CommonModule
  ],
  ...
})
```

## 用FormControl控制表单元素 ##

*login-form.component.html*

```
<div>
  <label>Name:</label>
  <input type="text" [formControl]="name">
</div>
<div>
  <label>Password:</label>
  <input type="password" [formControl]="password">
</div>
<button type="submit" (click)="onSubmit()">Submit</button>
```
*login-form.component.ts*

```
import { FormControl } from '@angular/forms';
import { Component } from '@angular/core';

@Component({
  selector: 'app-login-form',
  templateUrl: './login-form.component.html',
  styleUrls: ['./login-form.component.css']
})
export class LoginFormComponent {
  name:FormControl = new FormControl('jack');
  password:FormControl = new FormControl();

  onSubmit() {
    console.log(this.name.value, this.password.value);
  }
}
```
几点说明：
1. ```formControl```是一个指令，它接收一个```FormControl```的实例，来管理表单元素。
2. 构造```FormControl```的实例时，第一个参数是表单元素的初始值，也可传入一个对象```{value:'jack', disabled:true}```，里面可以包含表单元素的状态，如是否禁用等。
3. 在提交时，可以通过```this.name.value```获取表单元素的值。
4. 有时候我们需要在元素值改变时做一些事情，可以订阅```valueChanges```。
 ```
this.name.valueChanges.subscribe(() => {
      // do something...
      console.log(this.name);
});
```

## 给FormControl添加验证器 ##

*password.validator.ts*
```
export const PasswordValidator = (password?:RegExp): ValidatorFn => {
    return (control:AbstractControl): {[key: string]: any}  => {
        let error = null;
        if (password && !password.test(control.value)) {
            error = {
                password:control.value
            }
        }
        return error;
    }
}
```
*login-form.component.html*
```
<div>
  <label>Password:</label>
  <input type="password" [formControl]="password">
  <div [style.color]="'red'" *ngIf="password.invalid && (password.touched || password.dirty)">
    <p *ngIf="password.errors.required">password can not be empty!</p>
    <p *ngIf="password.errors.password">password is illegal!</p>
  </div>
</div>
```
*login-form.component.ts*
```
password:FormControl = new FormControl('', [Validators.required, PasswordValidator()]);
```
1. ```FormControl```的第二个参数可以传入一个```ValidatorFn```对象或者一个包含```ValidatorFn```对象的数组。
2. `angular2`内置的验证器可以通过```Validators```取到，如；```Validators.required```。
3. 自定义的验证器只需引用我们在模板驱动型表单中构造的验证器工厂函数(```PasswordValidator```)生成一个```ValidatorFn```对象的实例即可。
4. ```FormControl```上包含了表单的验证信息，在模板中可以直接使用它。这里可看到，之前我们在模版驱动型表单中通过模板引用变量```#password="ngModel"```引用的```ngModel```实际就是一个```FormControl```实例。

## 给FormControl添加异步验证器 ##

*name.validator.ts*
```
const userList = [
    'jack',
    'mary',
    'jimi',
    'tom'
];

export const checkNameValidator = () => {
    return (control:AbstractControl) => new Promise(
        (resolve, reject) => {
            setTimeout(() => {
                if (userList.indexOf(control.value) >= 0) {
                    resolve({
                        'checkName':control.value
                    });
                } else {
                    resolve(null);
                }
            }, 2000)
        }
    );
}
```
*login-form.component.html*
```
<label>Name:</label>
  <input type="text" [formControl]="name">
  <div [style.color]="'red'" *ngIf="name.invalid && (name.touched || name.dirty)">
    <p *ngIf="name.errors.required">Name can not be empty!</p>
    <p *ngIf="name.errors.checkName">Name is exist!</p>
  </div>
```
*login-form.component.ts*
```
name:FormControl = new FormControl('jack', Validators.required, checkNameValidator());
```
1. ```FormControl```的第三个参数可以传入一个```AsyncValidatorFn```对象或者一个包含```AsyncValidatorFn```对象的数组。
2. 异步验证器只需引用我们在模板驱动型表单中构造的验证器工厂函数(```checkNameValidator ```)生成一个```AsyncValidatorFn```对象的实例即可。

## 用FormGroup将表单元素组合起来 ##

当表单变得复杂时，组件中的```FormControl```对象会很多，我们可以用```FormGroup```对其进行统一管理。
*login-form.component.html*
```
<div [formGroup]="info">
  <div>
    <label>Name:</label>
    <input type="text" formControlName="name">
    <div [style.color]="'red'" *ngIf="info.get('name').invalid && (info.get('name').touched || info.get('name').dirty)">
      <p *ngIf="info.get('name').errors.required">Name can not be empty!</p>
      <p *ngIf="info.get('name').errors.checkName">Name is exist!</p>
    </div>
  </div>
  <div>
    <label>Password:</label>
    <input type="password" formControlName="password">
    <div [style.color]="'red'" *ngIf="info.get('password').invalid && (info.get('password').touched || info.get('password').dirty)">
      <p *ngIf="info.get('password').errors.required">password can not be empty!</p>
      <p *ngIf="info.get('password').errors.password">password is illegal!</p>
    </div>
  </div>
  <button type="submit" (click)="onSubmit()">Submit</button>
</div>
```
*login-form.component.ts*
```
import { checkNameValidator } from '../validators/name.validator';
import { PasswordValidator } from '../validators/password.validator';
import { FormControl, FormGroup, Validator, Validators } from '@angular/forms';
import { Component } from '@angular/core';

@Component({
  selector: 'app-login-form',
  templateUrl: './login-form.component.html',
  styleUrls: ['./login-form.component.css']
})
export class LoginFormComponent {
  info:FormGroup = new FormGroup({
    name:new FormControl('jack', Validators.required, checkNameValidator()),
    password:new FormControl('', [Validators.required, PasswordValidator()])
  });

  onSubmit() {
    console.log(this.info.value);
  }
}
```
1. 构造```FormGroup```需传入一个包含有```FormControl```实例的对象。
2. 在模板中可以通过```[formGroup]="info"```指令，管理一个表单组。
3. 在包含有```formGroup```指令的元素的子节点上，可以使用```formControlName="name"```代替```[formControl]="name"```来控制表单元素，```formControlName```指令只能在```formGroup```指令内部使用。
4. 在```FormGroup```上可以通过```info.get('name')```获取```FormControl```实例。
5. 在```FormGroup```上可以通过```this.info.value```获取表单组的值。
6. 在```FormGroup```上可以通过```this.info.setValue(newValue)```或```this.info.patchValue(newValue)```设置```FormGroup```的值，```setValue```是修改整个```FormGroup```的值，传入的值必须包含全部的属性，缺少任意一个属性控制台会报错，用```patchValue```时，可修改部分属性的值。
7. ```FormGroup```内部可以嵌套```FormGroup```，如`Name`可能又包含`firstName`和`lastName`，此时我们可以构造嵌套的```FormGroup```。

 ```
<div formGroupName="name">
    <input type="text" formControlName="first">
    <input type="text" formControlName="last">
</div>
info:FormGroup = new FormGroup({
    name: new FormGroup({
      first:new FormControl(),
      last:new FormControl()
    })
});
```

## 用FormBuilder简化FormGroup的构造 ##

*login-form.component.ts*
```
import { checkNameValidator } from '../validators/name.validator';
import { PasswordValidator } from '../validators/password.validator';
import { FormBuilder, FormControl, FormGroup, Validator, Validators } from '@angular/forms';
import { Component } from '@angular/core';

@Component({
  selector: 'app-login-form',
  templateUrl: './login-form.component.html',
  styleUrls: ['./login-form.component.css']
})
export class LoginFormComponent {
  info:FormGroup;

  constructor(private fb:FormBuilder){
    this.createForm();
  }

  createForm() {
    this.info = this.fb.group({
      name: ['jack', Validators.required, checkNameValidator()],
      password: ['', [Validators.required, PasswordValidator()]]
    });
  }

  onSubmit() {
    console.log(this.info.value);
  }
}
```
1. 我们需要先在构造函数中注入```FormBuilder```, 最好新建一个方法来初始化表单，这里我们在`createForm`方法中初始化表单。
2. 借助```FormBuilder```我们可以通过参数来构建表单组，参数的结构与使用`new`基本一致，就不需要重复的写```new FormGroup(...)```、```new FormControl(...)```了。


## 用FormArray处理动态表单 ##

有时候我们可能需要动态表单，如一个用户可能有多个地址，```FormArray```可以帮助我们处理这种需求。
*address.component.html*
```
<div [formGroup]="info">
  <div>
    <div>Address</div>
    <div><button (click)="addAddress()">Add</button></div>
    <div *ngFor="let address of info.get('address').controls;let index=index">
      <p>
        Provience:<input [formControl]="address.get('provience')">
        City:<input [formControl]="address.get('city')">
        <button (click)="removeAddress(index)">Remove</button>
      </p>
    </div>
  </div>
</div>
<button (click)="getInfo()">Get Info</button>
```
*address.component.ts*

```
import { FormArray, FormBuilder, FormControl, FormGroup } from '@angular/forms';
import { Component, OnInit } from '@angular/core';

@Component({
  selector: 'app-address',
  templateUrl: './address.component.html',
  styleUrls: ['./address.component.css']
})
export class AddressComponent{
  address:FormArray;
  info:FormGroup;

  constructor(private fb:FormBuilder) {
    this.createForm();
  }

  /**
   * 初始化表单
   */
  createForm() {
    this.address = this.fb.array([]);
    this.info = this.fb.group({
      address:this.address
    });
  }

  /**
   * 添加地址
   */
  addAddress() {
    this.address.push(new FormGroup({
      provience:new FormControl(),
      city:new FormControl()
    }));
  }

  /**
   * 移除地址
   * @param index 
   */
  removeAddress(index) {
    this.address.removeAt(index);
  }

  /**
   * 获取表单值
   */
  getInfo() {
    console.log(this.info.value);
  }
}


```
几点说明：
1. 构造```FormArray```时需传入一个数组，数组里面可以包含```FormControl```，```FormGroup```，```FormArray```。
2. 可以通过```push(...)```方法往```FormArray```中添加值。
3. 可以通过```removeAt(index)```移除```FormArray```中指定位置的值。
4. 这里是借助```FormBuilder.array```来构建```FormArray```，同样我们也可以使用```new FormArray([...])```来构建表单。

## 构建 form-error 组件来简化验证信息的展示 ##

![clipboard.png](/img/bVWXZ3)
从上图可以看到表单里的错误信息的展示结构基本类似，我们需要通过```info.get('name')```获取到```FormControl```的实例，然后把```FormControl```上的验证信息来展示到页面上。当验证项较多时，书写修改都变得很繁琐。我们可以考虑将错误信息的展示提取成一个组件。
假设组件需要两个参数：
1. ```control```是一个```FormControl```实例，我们可以通过他获取表单元素的有效性和详细的错误信息。
2. ```errors```是一个数组，包含每一个验证项的名称以及对应的提示信息。格式如下：

```
[
    {
        name: 'required',
        msg: 'Name can not be empty!',
    },
    {
        name: 'checkName',
        msg: 'Name is exist!'
    }
    ],
```

组件代码：

*form-error.component.html*

```
<div [style.color]="'red'" *ngIf="control.invalid && (control.touched || control.dirty)">
    <div *ngFor="let error of errors">
        <p *ngIf="control.errors[error.name]">{{ error.msg }}</p>
    </div>
</div>
```

*form-error.component.ts*

```
import { FormControl } from '@angular/forms';
import { Component, Input, OnInit } from '@angular/core';

@Component({
  selector: 'app-form-error',
  templateUrl: './form-error.component.html',
  styleUrls: ['./form-error.component.css']
})
export class FormErrorComponent{

  // 表单元素的FormControl实例
  @Input()
  control:FormControl;

  // 包含错误信息的数组
  @Input()
  errors:any [];
}

```
*使用`form-error`组件简化后的模板*
*login-form.component.html*

```
<div [formGroup]="info">
  <div>
    <label>Name:</label>
    <input type="text" formControlName="name">
    <app-form-error [control]="info.get('name')" [errors]="errors.name"></app-form-error>
  </div>
  <div>
    <label>Password:</label>
    <input type="password" formControlName="password">
    <app-form-error [control]="info.get('password')" [errors]="errors.password"></app-form-error>
  </div>
  <button type="submit" (click)="onSubmit()">Submit</button>
  <button type="button" (click)="setInfo()">SetInfo</button>
</div>

```

*login-form.component.ts*

```
createForm() {
    this.info = this.fb.group({
      name: ['jack', Validators.required, checkNameValidator()],
      password: ['', [Validators.required, PasswordValidator()]]
    });

    this.errors = {
      name: [{
        name: 'required',
        msg: 'Name can not be empty!',
      },{
        name: 'checkName',
        msg: 'Name is exist!'
      }],
      password: [
        {
          name: 'required',
          msg: 'Password can not be empty!'
        },
        {
          name: 'password',
          msg: 'Password is illegal!'
        }
      ]
    }
  }
```

## 构建 `form-widget` 组件来整合每一个表单项 ##

![clipboard.png](/img/bVWYaV)

观察使用`form-error`组件整合后的表单，我们可以看到每一个表单项的结构也是基本类似的，我们需要一个表单控件`(Input, Select..)`和一个`form-error`组件。
我们可以考虑通过构建一个通用的表单控件组件来循环输出每一个表单元素，这样我们的模板将会进一步精简。
构建`form-widget`组件，我们依赖一些数据：
1. 需要一个`FormControl`对象，来帮助我们控制表单元素，这可以通过`FormGroup`的`get(...)`方法拿到。
2. 需要一个表示组件类型变量，如：`text,password,checkbox,radio,select`等等，用它来确定表单元素的模板。
3. 由于`form-error`组件是`form-widget`组件的子组件，我们也需要能得到`errors`参数。

组件代码：

*form-widget.component.html*

```
<div [ngSwitch]="options.type">
  <div>
    {{ options.name }}:
  </div>
  <div *ngSwitchCase="'text'">
    <input type="text" [formControl]="control">
  </div>
  <div *ngSwitchCase="'password'">
      <input type="password" [formControl]="control">
  </div>
  <app-form-error [control]="control" [errors]="options.errors"></app-form-error>
</div>
```

*form-widget.component.ts*

```
import { FormControl } from '@angular/forms';
import { Component, Input } from '@angular/core';

@Component({
  selector: 'app-form-widget',
  templateUrl: './form-widget.component.html',
  styleUrls: ['./form-widget.component.css']
})
export class FormWidgetComponent {
  /**
   * 表单控制项
   */
  @Input()
  control;

  /**
   * 组件配置项
   * {
   *    name: '...' // 配置项名称
   *    type: '...' // 表单类型
   *    errors: '...' // 错误提示信息
   * }
   */
  @Input()
  options:{};
}

```

我们把`LoginComponent`的配置项调整成如下格式：

 *login-form.component.ts*
```
  /**
   * 创建表单
   */
  createForm() {
    this.info = this.fb.group({
      name: ['jack', Validators.required, checkNameValidator()],
      password: ['', [Validators.required, PasswordValidator()]]
    });

    this.options = [
      {
        name: 'name',       // 表单控制项的名字，便于我们从formGroup上找到formControl
        type: 'text',       // 表单控制项的类型
        errors: [{          // 错误提示信息
          name: 'required',
          msg: 'Name can not be empty!',
        },{
          name: 'checkName',
          msg: 'Name is exist!'
        }]
      },
      {
        name: 'password',
        type: 'password',
        errors:  [
          {
            name: 'required',
            msg: 'Password can not be empty!'
          },
          {
            name: 'password',
            msg: 'Password is illegal!'
          }
        ]
      }
    ];
  }
```

*login-form.component.html*

```
<div [formGroup]="info">
  <app-form-widget *ngFor="let item of options" [control]="info.get(item.name)" [options]="item"></app-form-widget>
  <button type="submit" (click)="onSubmit()">Submit</button>
  <button type="button" (click)="setInfo()">SetInfo</button>
</div>

```
这里我们给`form-widget`组件设置了两个输入参数`control`和`options`, `options`上面包含了控件名称（`name`）,控件类型（`type`）,控件错误提示信息（`errors`）等属性。在模板中我们通过`*ngSwitch`指令来处理不同的控件类型。

上面调整后的模板已经简化了很多了，接下来我们考虑这样一个问题：
现在我们的模板只支持`input`和`password`两种类型，怎么样让它支持更多类型呢？

## 使用动态加载组件让form-widget支持更多的控件类型 ##

在`form-widget`组件中，我们使用了`ngSwitch`指令和`ngSwitchCase`指令，根据配置项的`type`值来处理不同的控件类型，对于一些简单的控件，如：`select, checkbox, radio`等等，我们只需要添加几个`ngSwitchCase`匹配项就好了，这里很好的完成了我们的需求，但是有几个弊端：
1. 我们难免会有一些复杂的控件，全部加到`form-widget`组件模板中会使模板变得臃肿。
2. 每次扩展类型，我们都需要修改`form-widget`组件模板，这是很繁琐的一件事，多人合作开发时也很容易造成冲突。

我们能不能将每一种类型的控件都提取成组件，扩展控件时只需写好组件，`form-widget`组件根据配置项里的控件类型动态加载组件呢？
是可以的，动态加载组件的详细的介绍可以参考[angular2文档][1]，[Angular 4.x 动态创建组件][2]。

控件代码：
### 控件基类 ###
*widget.base.component.ts*

```
import { Component, Input } from '@angular/core';

@Component({
    selector: 'form-widget-base',
    template: ''
})
export class FormWidgetBaseComponent{
    @Input()
    controls;

    @Input()
    options;
}
```
### 密码控件 ###
*input.password.component.ts*

```
import { FormWidgetBaseComponent } from '../base/widget.base.component';
import { Component, Input } from '@angular/core';

@Component({
    selector: 'form-widget-password',
    template: `<div *ngIf="control && options">
                    <label>{{options.name}}:</label>
                    <input type="password" [formControl]="control">
                </div>`
})
export class FormWidgetPassword extends FormWidgetBaseComponent{
    
}
```
### 文本控件 ###
*input.text.component.ts*

```
import { FormWidgetBaseComponent } from '../base/widget.base.component';
import { Component, Input } from '@angular/core';

@Component({
    selector: 'form-widget-text',
    template: `<div *ngIf="control && options">
                    <label>{{options.name}}:</label>
                    <input type="text" [formControl]="control">
                </div>`
})
export class FormWidgetText extends FormWidgetBaseComponent{

}
```
### 更新后的form-widget组件 ###
*form-widget.component.ts*
```
import { FormWidgetPassword } from './widgets/password/input.password.component';
import { FormWidgetText } from './widgets/text/input.text.component';
import { FormErrorComponent } from '../form-error/form-error.component';
import { FormControl } from '@angular/forms';
import { Component, ComponentFactoryResolver, Input, ViewChild, ViewContainerRef } from '@angular/core';


@Component({
  selector: 'app-form-widget',
  templateUrl: './form-widget.component.html',
  styleUrls: ['./form-widget.component.css'],
  entryComponents: [FormWidgetText,FormWidgetPassword,FormErrorComponent]
})
export class FormWidgetComponent {
  /**
   * 表单控制项
   */
  @Input()
  control;

  /**
   * 组件配置项
   * {
   *    name: '...' // 配置项名称
   *    type: '...' // 表单类型
   *    errors: '...' // 错误提示信息
   * }
   */
  @Input()
  options:{};

  /**
   * 组件视图容器
   */
  @ViewChild('widget', {read: ViewContainerRef})
  widget;

  /**
   * 表单控件组件的引用
   */
  controlRef;
  
  /**
   * 错误组件的引用
   */
  errorsRef;

  constructor(private componentFactoryResolver: ComponentFactoryResolver) {
  }

  ngAfterViewInit() {
    setTimeout(() => {
      this.loadComponent();
    }, 0);
  }

  loadComponent() {
    // 加载表单控件
    const controlComponentFactory = this.componentFactoryResolver.resolveComponentFactory(this.options['type']);
    this.controlRef = this.widget.createComponent(controlComponentFactory);
    this.controlRef.instance.control = this.control;
    this.controlRef.instance.options = this.options;
    // 加载错误组件
    const errorsComponentFactory = this.componentFactoryResolver.resolveComponentFactory(FormErrorComponent);
    this.errorsRef = this.widget.createComponent(errorsComponentFactory);
    this.errorsRef.instance.control = this.control;
    this.errorsRef.instance.errors = this.options['errors'];
  }

}

```
*form-widget.component.html*

```
<ng-template #widget></ng-template>
```
### 几点说明 ###
1. 在定义控件时，我们首先定义了一个`FormWidgetBaseComponent`基组件，里面包含了基础的配置项，我们可以使用组件继承来扩展不同的类型，在扩展的组件中就不需要重复定义配置项了，对于一些简单的模板，我们只需要继该组件，然后设置模板即可。关于组件的继承可参考[angular2文档][3]，[Angular 2 Component Inheritance][4]。
2. 之前配置项中的`type`是`text,password`等值。采用动态加载组件的方法，我们需要将`type`类型改为我们所定义的组件，如：`FormWidgetText`，`FormWidgetPassword`等等。
3. 扩展组件的时候，我们只需要实现好组件，然后将其加入到`entryComponents`即可。

## 构建dynamic-form组件，通过配置项生成FormGroup，简化表单构建 ##
将构建FormGroup的参数整合进配置项里能让我们根据一个配置项就能构建好表单，进一步简化表单的构建。

我们需要一个转换函数，我们将它定义到一个服务里。
*form.service.ts*

```
import { FormBuilder } from '@angular/forms';
import { Injectable } from '@angular/core';

@Injectable()
export class FormService {
    constructor(private fb:FormBuilder) {

    }
    
    /**
     * 根据配置项生成FormGroup
     * @param options 
     */
    transform(options:any []) {
        const group = {};
        options.forEach((option) => {
            if (option.controlType === 'group') {
                group[option.name] = this.transform(option.control);
            } else if (option.controlType === 'array') {
                group[option.name] = this.fb.array(option.control);
            } else {
                group[option.name] = option.control;
            }
        });
        return this.fb.group(group);
    }
}
```

组件代码：

*dynamic-form.component.ts*

```
import { FormService } from '../services/form.service';
import { FormGroup } from '@angular/forms';
import { Component, EventEmitter, Input, OnInit, Output } from '@angular/core';

@Component({
  selector: 'app-dynamic-form',
  templateUrl: './dynamic-form.component.html',
  styleUrls: ['./dynamic-form.component.css']
})
export class DynamicFormComponent {
  group:FormGroup;

  @Input()
  options:any [];

  @Output()
  submitEvent:EventEmitter<any> = new EventEmitter<any>();

  constructor(private formService:FormService) {
    
  }

  ngOnInit() {
    this.group = this.formService.transform(this.options);
  }

  onSubmit() {
    this.submitEvent.emit(this.group.value);
  }
}

```
*dynamic-form.component.html*

```
<div [formGroup]="group">
  <app-form-widget *ngFor="let item of options" [control]="group.get(item.name)" [options]="item"></app-form-widget>
  <button type="submit" (click)="onSubmit()">Submit</button>
</div>
```
login-form组件调用
*login-form.component.html*

```
<app-dynamic-form [options]="options" (submitEvent)="onSubmit($event)"></app-dynamic-form>
```
*login-form.component.ts*

```
/**
   * 创建表单
   */
  createForm() {

    // 表单配置项
    this.options = [
      {
        name: 'name',               // 表单控制项的名字，便于我们从formGroup上找到formControl
        control: ['jack', Validators.required, checkNameValidator()], // 表单配置项的构建参数
        type: FormWidgetText,       // 表单控制项的类型
        errors: [{                  // 错误提示信息
          name: 'required',
          msg: 'Name can not be empty!',
        },{
          name: 'checkName',
          msg: 'Name is exist!'
        }]
      },
      ...
    ];
  }
```
1. 在`dynamic-form`组件里我们通过`Output`暴露了一个提交事件的接口，如有需要可以通过这个方法暴露更多接口。
自此我们的组件已经基本完成了，我们可以通过一个配置项很快捷的构建表单了。

## 小结 ##
1. 借助响应式表单，我们能将使用代码实现表单初始化，表单验证，获取表单值等等操作。我们可以用```FormControl```控制单个表单元素，用```FormGroup```将表单元素分组，用```FormArray```处理需要动态添加的表单。在模板中我们只需要展示表单和验证信息即可，从而实现模板和业务逻辑的分离。
2. 通过```FormBuilder```，我们可以使用配置项来简化响应式表单的构造。
3. 对于结构类似的表单，我们可以考虑构建动态表单组件，来避免重复的劳动，提升开发效率。


  [1]: https://angular.cn/guide/dynamic-component-loader
  [2]: https://segmentfault.com/a/1190000009175508
  [3]: https://angular.cn/guide/dependency-injection-in-action#%E9%80%9A%E8%BF%87%E6%B3%A8%E5%85%A5%E6%9D%A5%E6%89%BE%E5%88%B0%E4%B8%80%E4%B8%AA%E7%88%B6%E7%BB%84%E4%BB%B6
  [4]: https://segmentfault.com/a/1190000008976996