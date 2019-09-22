---
title: "IONIC 2 实现首页双击退出APP"
abstract: 在ICONIC 2 快速实现双击退出APP的功能
header_image: /assets/images/ionic2-banner.png
date: 2017/04/24
categories:
  - Ionic 2
tags:
  - Ionic 2
  - Angular 4
  - Hybird App
cover: https://s2.ax1x.com/2019/09/18/n7E8Wd.png
---

## 添加绑定值
> 首先在```app/app.html```下加入```#myNav```, 这个是用于绑定当前页面的导航标签

```html
 <ion-nav #myNav [root]="rootPage"></ion-nav>
```

## 代码实现
> 然后在```app.component.ts```做相对的改动

```typescript
import { Component, ViewChild } from '@angular/core';
import { Platform, ToastController, Nav, App } from 'ionic-angular';
import { StatusBar } from '@ionic-native/status-bar';
import { SplashScreen } from '@ionic-native/splash-screen';


@Component({
  templateUrl: 'app.html',
  providers: [SplashScreen, StatusBar]
})
export class MyApp {
  rootPage = 'TabsPage';
  backButtonPressed: boolean = false;  //用于判断返回键是否触发
  @ViewChild('myNav') nav: Nav;

  constructor(public platform: Platform, private splashScreen: SplashScreen, private statusBar: StatusBar, public app: App, public toastCtrl: ToastController) {
    platform.ready().then(() => {
      // Okay, so the platform is ready and our plugins are available.
      // Here you can do any higher level native things you might need.
      this.statusBar.styleDefault();
      this.statusBar.backgroundColorByHexString('#661F22');
      this.splashScreen.hide();
      this.registerBackButtonAction(); //运行这个方法绑定返回按钮
    });
  }

  registerBackButtonAction() {
    this.platform.registerBackButtonAction(() => {
      //如果想点击返回按钮隐藏toast或loading或Overlay就把下面加上
      // this.ionicApp._toastPortal.getActive() || this.ionicApp._loadingPortal.getActive() || this.ionicApp._overlayPortal.getActive();
      let nav = this.app.getActiveNav();
      if (nav.canGoBack()){ //是否已经到了首页
        nav.pop();
      }else{
        this.showExit()
      }
    }, 1);
  }

  //双击退出提示框
  showExit() {
    if (this.backButtonPressed) { //当触发标志为true时，即2秒内双击返回按键则退出APP
      this.platform.exitApp();
    } else {
     this.toastCtrl.create({
      message: '再按一次退出应用',
      duration: 2000,
      position: 'middle'
     }).present();
     this.backButtonPressed = true;
     setTimeout(() => this.backButtonPressed = false, 2000);//2秒内没有再次点击返回则将触发标志标记为false
    }
  }
}
```


