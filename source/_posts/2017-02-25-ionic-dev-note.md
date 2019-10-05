---
title: "IONIC 2 开发笔记"
abstract: 记录了所有IONIC 2开发时踩过的坑
header_image: /assets/images/ionic2-banner.png
date: 2017/02/25
categories:
  - Ionic
tags:
  - Ionic 2
  - Angular 2
  - Hybird App
cover: https://s2.ax1x.com/2019/09/18/n7E8Wd.png
---

因为IONIC2才刚刚开始进入正式版, 中文文档基本都是不全的, 我现在开发都是在看英文文档
在开发的过程中遇到了很多文档没有描述的问题, 而且开发的过程中遇到一些框架本身没有完善的功能和存在的BUG

我会在这个日记里面记录一些文档没有写的, 和我开发过程中遇到的一些坑和经验, 希望可以帮助那些刚刚接触IONIC2的程序猿们!

## APP配置

### 域名配置
因为跨域问题，在开发时如果要用到本地环境进行开发(ionic serve)，必须配置proxy
- Proxy位于项目根目录下的 **ionic.config.json**
+ 只需要把**proxyUrl**改为你本地环境的API地址

```javascript
{
  "name": "rlph",
  "app_id": "",
  "v2": true,
  "typescript": true,
  "proxies": [
    {
      "path": "/api",
      "proxyUrl": "http://api.dev"
    }
  ]
}
```

+ 然后把API地址的常量制定为**/api/**这个proxy
- 在根目录下**/src/config.ts**里面把**"API_SERVER"**的值改为**"/api/"**

```
  export let data = {
      "API_SERVER" : "/api/"
  }
```


### 上线APP配置
+ 首先你需要配置真是服务器API地址
配置API地址是在根目录下**/src/config.ts**里面把**"API_SERVER"**的值改为线上API地址

```
  export let data = {
      "API_SERVER" : "http://api.domain.com/"
  }
```

## 开发常见问题

### APP run 失败
当运行**ionic run android**的时候可能会遇到该报错：

```
Error: Failed to install apk to device: [  1%] /data/local/tmp/android-debug.apk
[  2%] /data/local/tmp/android-debug.apk
...
[100%] /data/local/tmp/android-debug.apk
        pkg: /data/local/tmp/android-debug.apk
Failure [INSTALL_FAILED_UPDATE_INCOMPATIBLE]
```

- 此问题是因为已有签名的APP存在手机上， 需要想删除该APP才能安装测试（debug）版的apk
- 在cmd运行以下代码即可解决问题：

```
adb uninstall my.package.id
```


### Ionic 2 自带的native文件上传(FILE TRANSFER)插件无法获取成功返回内容
这个是Ionic 2 核心代码里面的一个BUG, 在一下版本下是有问题的
**Ionic CLI Version: 2.2.1**
- 首先找到项目根目录下以下路径里面的**filetransfer.d.ts**文件

```
node_modules\ionic-native\dist\es5\plugins\filetransfer.d.ts
node_modules\ionic-native\dist\esm\plugins\filetransfer.d.ts
```

- 分别修改以上两个文件里面的代码

```
//把这一行:
upload(fileUrl: string, url: string, options?: FileUploadOptions, trustAllHosts?: boolean): Promise<FileUploadResult | FileTransferError>
//改为: 
upload(fileUrl: string, url: string, options?: FileUploadOptions, trustAllHosts?: boolean): Promise<FileUploadResult>
```


### 在安卓下使用相册选择时, 返回的图片路径不能再显示问题
- 首先在这个例子使用的是cordova-plugin-camera组件(Cordova的相机插件)
- 首先引入需要的类
- FilePath 这个类就是用来修复安卓上图片URI的问题的

```typescript
import { Camera, File, FilePath } from 'ionic-native';
```

- 使用 FilePath.resolveNativePath(imagePath) 这个方法来纠正图片URI

```typescript
Camera.getPicture(options).then((imagePath) => {
 // 特殊安卓图片库的处理
 this.nativeFilePath = imagePath;
 if (this.pl.is('android') && sourceType === Camera.PictureSourceType.PHOTOLIBRARY) {
   FilePath.resolveNativePath(imagePath)
   .then(filePath => {
       this.nativeFilePath = filePath;
       let currentName = imagePath.substring(imagePath.lastIndexOf('/') + 1, imagePath.lastIndexOf('?'));
   });
 } else {
   var currentName = imagePath.substr(imagePath.lastIndexOf('/') + 1);
 }
}, err => {
 // this.presentToast('选择图片失败.');
});
```

