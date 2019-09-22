---
title: "IONIC 3 使用sliders轮播时拖动后autoplay失效解决方法"
abstract: 使用sliders轮播时拖动后autoplay失效解决方法
header_image: /assets/images/ionic2-banner.png
date: 2017/04/20
categories:
  - Frontend
  - 前端
tags:
  - Ionic 3
  - Angular 4
  - Hybird App
cover: https://s2.ax1x.com/2019/09/18/n7E8Wd.png
---

#### 我们先了解一下Ionic的Sliders

> 首先Ionic里面的sliders是用[Swiper.js](http://idangero.us/swiper/api/#.WPhzbPB96Uk)的第三方插件实现的, Ionic官方Sliders的文档里面只描述了可以直接写入html标签内的属性, 有很多高级属性是没有写在文档里面的. 

#### 要怎么改变sliders的其他属性呢?

> 那如果我们要用到Swiper的其他属性怎么办呢? Ionic 2.x 的时候我们是可以在options里面传入的, 但是升级Ionic 3.x.x 后sliders的options属性被移除了. 现在要改变sliders的属性我们要用到sliders类. 

#### 如何拖动轮播图后不让autoplay失效呢?

> 首先我们要引入```viewChild```和```Sliders```

```typescript
import { ViewChild } from '@angular/core';
import { Slides } from 'ionic-angular';

```

> 然后使用```ionViewWillEnter```在进入页面前改变sliders的```autoplayDisableOnInteraction```属性

```typescript
import { ViewChild } from '@angular/core';
import { Slides } from 'ionic-angular';

class MyPage {
  @ViewChild(Slides) slides: Slides;

  ionViewWillEnter() {
    this.slides.autoplayDisableOnInteraction = false; //禁止slider拖动后autoPlay失效
  }
}
```
