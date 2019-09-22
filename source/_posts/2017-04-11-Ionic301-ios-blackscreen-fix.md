---
title: "修复 Ionic 3.0.1 在IOS点击Tabs多次出现黑屏问题方法"
abstract: Ionic 3.0.1 在IOS存在的一个严重的BUG，在IOS下多次重复点击TAB的时候页面会出现黑屏问题。
date: 2017/04/11
categories:
  - Ionic 3
tags:
  - Ionic 3
  - Angular 4
  - Hybird App
cover: https://s2.ax1x.com/2019/09/18/n7E8Wd.png
---

> Ionic 3.0.1 在IOS存在的一个严重的BUG，在IOS下多次重复点击TAB的时候页面会出现黑屏问题。

> 好消息是目前有一个暂时的修复方法。但是这个方法涉及修改Ionic核心代码，所以如果你们正在使用Ionic3发布APP，可以暂时使用以下办法修复问题。

> 在```node_modules/ionic-angular/components/tabs/tabs.js```找到一下代码 (468行)

```typescript
getComponent(this._linker, tab.root).then(function (viewController) {
    if (viewController !== active.component) {
        // Otherwise, if the page we're on is not our real root
        // reset it to our default root type
        return tab.setRoot(tab.root);
    }
}).catch(function () {
    (void 0) /* console.debug */;
});
```

> 把以上代码改为

```typescript
getComponent(this._linker, tab.root).then(function (viewController) {
    if (viewController.component !== active.component) {
        // Otherwise, if the page we're on is not our real root
        // reset it to our default root type
        return tab.setRoot(tab.root);
    }
}).catch(function () {
    (void 0) /* console.debug */;
});
```

> 以上解决办法来自于github上面的一个大神 https://github.com/driftyco/ionic/pull/11084



