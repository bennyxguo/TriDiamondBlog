---
title: "IONIC 2 - 确认密码"
abstract: 开发过程中一般在注册或者修改密码都要写一个密码确认的表格, 这篇文章就是记录怎么在Ionic2里面使用FormBuilder和Validators判断密码是否一致.
header_image: /assets/images/ionic2-banner.png
date: 2017/03/19
categories:
  - Ionic 2
tags:
  - Ionic 2
  - Angular 2
  - Hybird App
---

开发过程中一般在注册或者修改密码都要写一个密码确认的表格, 这篇文章就是记录怎么在Ionic2里面使用**FormBuilder**和**Validators**判断密码是否一致.

首先这篇文章是默认您已经了解怎么使用**Angualr2**的**FormBuilder**和**Validator**, 如果你还没了解这两个类的用法可以前去看[Ionic2的文档](https://ionicframework.com/docs/v2/resources/forms/)

# 实例一个`FormGroup`

> 第一步首先我们需要实例了FormBuilder的一个FormGroup

+ 这里我们定义了passwordForm的这个表格里面的input.
+ 在最后我们加入了自定义认证方法 { validator: AdvanceValidator.matchingPasswords('password', 'rePassword') }
+ 这里我们把password, 和rePassword 传给了 AdvanceValidator 方法, 这里传的是密码和确认密码在FormGroup里面定义的名字.
+ 现在我们看看这个password.ts怎么写.

```typescript
import { FormBuilder, Validators, FormGroup } from '@angular/forms';
import { AdvanceValidator } from '../../validators/advance-validator';

@Component({
  selector: 'page-password',
  templateUrl: 'password.html'
})

//密码修改页
export class PasswordPage {
  passwordForm: FormGroup;

  constructor(private fb: FormBuilder) 
  {
    
    this.passwordForm = fb.group({
        username: ['', Validators.required],
        password: ['', Validators.compose([Validators.maxLength(30), Validators.minLength(7), Validators.required])],
        rePassword: ['', Validators.compose([Validators.maxLength(30), Validators.minLength(7), Validators.required])],
    }, { validator: AdvanceValidator.matchingPasswords('password', 'rePassword') });
    
  }
}
```

# 创建自定义认证器

![](https://github.com/Bennygx/bennygx.github.io/blob/master/assets/images/advance-validator-screenshot.png?raw=true)

+ 我们首先在项目根目录创建 **validators** 的文件夹
+ 然后在里面创建 **advance-validators.ts** 的ts文件
+ 在 **advance-validators.ts** 里面编以下代码

```typescript
import { FormGroup } from '@angular/forms';

export class AdvanceValidator {

	static matchingPasswords(passwordKey: string, rePasswordKey: string) {
    return (group: FormGroup) => {
      let password = group.controls[passwordKey]; //获取密码值
      let rePassword = group.controls[rePasswordKey]; //获取确认密码值

      if(password.value !== rePassword.value) {
        //如果密码和确认密码的值不一致就返回给FormBuild rePassword有错误
        return rePassword.setErrors({notEquivalent: true}) 
      }
    }
  }
}
```

# 前端表格示例

```html
<form [formGroup] = "passwordForm">
  <ion-list inset>

    <ion-item>
      <ion-input type="tel" placeholder="用户名" formControlName="username"></ion-input>
      <div item-right *ngIf="!passwordForm.controls.username.valid  && (passwordForm.controls.username.dirty)">
        <ion-icon name="alert"></ion-icon> 用户名必填
      </div>
    </ion-item>

    <ion-item>
      <ion-input type="password" placeholder="新密码" formControlName="password"></ion-input>
      <div item-right *ngIf="!passwordForm.controls.password.valid  && (passwordForm.controls.password.dirty)" >
        <ion-icon name="alert"></ion-icon> 密码必须7个字以上
      </div>
    </ion-item>

    <ion-item>
      <ion-input type="password" placeholder="确认密码" formControlName="rePassword"></ion-input>
      <div item-right *ngIf="!passwordForm.controls.rePassword.valid  && (passwordForm.controls.rePassword.dirty)" >
        <ion-icon name="alert"></ion-icon> 密码必须一致
      </div>
    </ion-item>

  </ion-list>
</form>
```
