---
title: "IONIC 2 升级 3 教程"
abstract: 这一次的升级Ionic 3 换成了使用最新的Angular 4.0, 最新的TypeScript, 添加了懒加载和修复了一些组件的bug.
header_image: /assets/images/ionic2-banner.png
date: 2017/04/10
categories:
  - Ionic 3
tags:
  - Ionic 3
  - Angular 4
  - Hybird App
---

## 升级步骤

> 这一次的升级Ionic 3 换成了使用最新的Angular 4.0, 最新的TypeScript, 添加了懒加载和修复了一些组件的bug.

1. 首先更新```package.json```, 按照以下的代码相应替换你package.json里面的代码, 并且把你项目根目录下的```node_modules```文件夹删除掉, 然后运行```npm install``` (如果你是用淘宝镜像可以运行 ```cnpm install```)

```json
"dependencies": {
    "@angular/common": "4.0.0",
    "@angular/compiler": "4.0.0",
    "@angular/compiler-cli": "4.0.0",
    "@angular/core": "4.0.0",
    "@angular/forms": "4.0.0",
    "@angular/http": "4.0.0",
    "@angular/platform-browser": "4.0.0",
    "@angular/platform-browser-dynamic": "4.0.0",
    "@ionic-native/core": "3.4.2",
    "@ionic-native/splash-screen": "3.4.2",
    "@ionic-native/status-bar": "3.4.2",
    "@ionic/storage": "2.0.1",
    "ionic-angular": "3.0.1",
    "ionicons": "3.0.0",
    "rxjs": "5.1.1",
    "sw-toolbox": "3.4.0",
    "zone.js": "^0.8.4"
},
"devDependencies": {
  "@ionic/app-scripts": "1.3.0",
  "typescript": "~2.2.1"
}
```

2. 第二步你需要在```app/app.module.ts```文件里面引入```BrowserModule```和```HttpModule```

> 首先需要在头部引入这两个module (如果你的APP不使用HTTP可以不引入```HttpModule```)

```typescript
import { BrowserModule } from '@angular/platform-browser';
import { HttpModule } from '@angular/http';
```

> 在同一个文件里面找到```imports```并且加入```BrowserModule```和```HttpModule```

```typescript
imports: [
  BrowserModule,
  HttpModule,
  IonicModule.forRoot(MyApp)
],
```

3. 如果你升级到 Ionic Native 3.x, 就是CLI3. 使用 Ionic Native 3.x的话, APP打包出来会更小. 因为Ionic Native的原生插件都不自带有了, 你使用一个就要安装一个. 如果你原有的Ionic 2 项目有引入原生插件, 你就要做以下操作.

> 这里用```Camera```和```Geolocation```这个两个原生插件作为例子, 你其他的插件都需要使用相同的方式做修改

> 注意您使用的所有插件都必须要在```app/app.module.ts```里面先引用了, 而且要在```app/app.modules.ts```里面的providers里面声明, 如果没有这样配置就会出现```No provider for XXXXX```这样的报错了!

#### Camera插件

```typescript
// 在app/app.module.ts文件里面
import { Camera } from '@ionic-native/camera';

...

@NgModule({
  ...

  providers: [
    ...
    Camera
    ...
  ]
  ...
})
export class AppModule { }
```

#### Geolocation插件

```typescript
// 在app/app.module.ts文件里面
import { Geolocation } from '@ionic-native/geolocation';
import { Platform } from 'ionic-angular';

import { NgZone } from '@angular/core';

@Component({ ... })
export class MyComponent {

  constructor(private geolocation: Geolocation, private platform: Platform, private ngZone: NgZone) {

    platform.ready().then(() => {

      // get position
      geolocation.getCurrentPosition().then(pos => {

        console.log(`lat: ${pos.coords.latitude}, lon: ${pos.coords.longitude}`)

      });


      // watch position
      const watch = geolocation.watchPosition().subscribe(pos => {

        console.log(`lat: ${pos.coords.latitude}, lon: ${pos.coords.longitude}`)

        // Currently, observables from Ionic Native plugins
        // need to run inside of zone to trigger change detection
        ngZone.run(() => {
          this.position = pos;
        })

      });

      // to stop watching
      watch.unsubscribe();

    });

  }

}
```

> 更详细的文档可以参考官方的修改日记 https://github.com/driftyco/ionic-native/blob/master/README.md

## 组件`Component`更变

#### 新网格

> 旧的网格体系已经废除, 新的网格组件请参考官方文档 http://blog.ionic.io/build-awesome-desktop-apps-with-ionics-new-responsive-grid/

#### 标签的`color`属性更变

> 以下标签的```color```属性在新的版本里面会不起效果了, 现在必须要使用```ion-text```才会起效果, 详细说明请看官方文档 http://ionicframework.com/docs/api/components/typography/Typography/

```css
h1[color], h2[color], h3[color], h4[color], h5[color], h6[color], a[color]:not([ion-button]):not([ion-item]):not([ion-fab]), p[color], span[color], b[color], i[color], strong[color], em[color], small[color], sub[color], sup[color]
```

#### Slides组件更变

> 以下的Slides属性和方法已经正式在新版本里面移除了

+ Slides的input的```options```属性已经废除, 请使用标签的属性;
+ Slide的事件```ionWillChange```方法已经废除, 请使用```ionSlideWillChange```;
+ Slide的事件```ionDidChange```方法已经废除, 请使用```ionSlideDidChange```;
+ Slide的事件```ionDrag```方法已经废除, 请使用```ionSlideDrag```;
+ Slides的```getSlider()```方法已经废除, 请使用```ion-slides```实例;

