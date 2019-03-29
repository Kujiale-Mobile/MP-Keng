# 小程序踩坑记

希望这个 Repo 能尽量记录下小程序的那些坑，避免开发者们浪费自己的生命来定位到底是自己代码导致的还是啥神秘的字节跳变原因。欢迎大家踊跃 PR 。



## 前记

小程序大多数坑是同一套代码在不同平台上表现不一致导致的，微信开发者工具，Android，iOS。千万不要以为自己写的代码在模拟器上跑过了就完事了，一定需要在 Android 和 iOS 真机上再测一遍，否则后果不堪设想。之所以有如此坑爹的情况，也是由于小程序在不同系统下使用的底层框架不一致导致的，如下图所示。 

![](https://raw.github.com/Kujiale-Mobile/MP-Keng/master/img/0.jpeg)

另：小程序的视图层和 js 代码运行在不同的地方，相当于小程序的视图层运行在浏览器中，而 js 运行在原生的 js 解析引擎上。这也是为啥小程序的 js 无法获得 dom 和 bom 的原因。



## 坑们

### 1，onShareAppMessage 中设置封面在 Android 和 iOS 上展示策略不一致

**相关文档**

https://developers.weixin.qq.com/miniprogram/dev/api/share.html#onshareappmessageoptions

**现象**

当设置的 imageUrl 不是 5:4 的图片时，比如 5:3 时，就会出现下列情况，左边是在 Android 上的展示，右边是 iOS上的。两者对分享消息的封面展示策略并不一样。

![](https://raw.github.com/Kujiale-Mobile/MP-Keng/master/img/1.png)

**版本**

明显 iOS 上更合理一点，目前 Android 微信版本 6.6.7 依然会发生这种情况。

**解决**

像这种非 5:4 的图片，尽量也弄成 5:4 的，然后多余部分用白色进行填充。

---

### 2，wx.showToast 在 iOS 和 Android 上行为不一致

 **相关文档**

https://developers.weixin.qq.com/miniprogram/dev/api/api-react.html#wxshowtoastobject

**现象**

当 showToast 后，马上返回上一个页面，在 Android 上，toast 依然存在，但 iOS 上 toast 无法显示。感觉是因为在 Android 上这个 toast 是全局的，而 iOS 上 toast 只是依附于某个页面。

**解决**

如果想在这种情况下，Android 和 iOS 都能显示出 toast，那只能加个 timeout 延时了，比如 1s 后再 执行 `wx.navigateBack();` 的操作。

---

### 3，小程序的 web-view 中页面跳转后，点击 Android 手机上的物理返回按钮会返回前一个页面。而点击左上角的返回按钮，会直接关闭整个 web-view。

---

### 4，原生组件之坑

小程序可能为了实现方便，其中几个组件使用了原生组件，canvas，textarea，map，camera，video，live-player，live-pusher。而原生组件的层级最高，无法通过 z-index 来控制。而且也无法在 scroll-view、swiper、picker-view、movable-view 中使用这几个原生组件，否则效果可谓是惨不忍睹。



**解决**

使用 cover-view 和 cover-image 来覆盖在原生组件之上

相关文档：https://developers.weixin.qq.com/miniprogram/dev/component/cover-view.html

---

### 5，video 组件的视频源地址不能有中文，否则在 iOS 上无法加载 

**相关文档**

https://developers.weixin.qq.com/miniprogram/dev/component/video.html

**现象**

视频源地址设置中文后，Android 和模拟器上是可以正常播放的，但 iOS 上无法播放。

### 6，wx.showToast 文字最多可显示两行，多出部分会被截断，所以注意控制自己提示的内容。

### 7，wx.showModal 的 title 在 iOS 上只能显示一行，而在 Android 上可以显示两行。多出会 ...。

### 8，wx.showActionSheet 在 Android 和 iOS 上面行为差异过大。
https://developers.weixin.qq.com/miniprogram/dev/api/ui/interaction/wx.showActionSheet.html

iOS 会从下部滑动上来，Android 会弹一个中间列表弹框。而且 iOS 的列表中会多一个 ”取消“ 的选项，更坑爹的时，后来一个版本竟然把 Android 的点击外部取消功能给干掉了，导致 Android 上只能按返回键才能取消 ActionSheet。

我也不知道腾讯写的这个注意文档到底是为了啥。
```
注意
Android 6.7.2 以下版本，点击取消或蒙层时，回调 fail, errMsg 为 "fail cancel"；
Android 6.7.2 及以上版本 和 iOS 点击蒙层不会关闭模态弹窗，所以尽量避免使用「取消」分支中实现业务逻辑
```

### 9，在 iOS 上两个 Image 重合摆放，但更高层级上的 Image 无法显示问题

微信小程序图片在同一父标签下重叠排布的时候会产生问题，具体表现为无论如何设定z-index，悬浮在上的图片一直会被沉在下方的图片遮盖。这个问题仅仅反映在iOS上，在模拟器和 Android 上都没有问题。
在真机调试中发现，是小程序 image 标签，自带的 `overflow:hidden`；产生了影响，在取消该属性后就正常显示了，但是替换为 overflow： visble；等方式未起作用。解决办法有两种：

1.使用cover-image代替上浮图片的image标签，这样在ios和android上都显示正常，但是在模拟器上似乎有些问题。
2.使用一个 view 包裹上浮的图片。这样在三端表现一致且能解决上述问题。

### 10，有关 web-view 中有背景音乐，后台后无法关闭的问题
https://developers.weixin.qq.com/community/develop/doc/c75139c842a40c67cade23d3f66e7992

```
var hiddenProperty = 'hidden' in document ? 'hidden' :
    'webkitHidden' in document ? 'webkitHidden' :
    'mozHidden' in document ? 'mozHidden' :
     null;



if (hiddenProperty) {
    var visibilityChangeEvent = hiddenProperty.replace(/hidden/i, 'visibilitychange');

    var onVisibilityChange = function() {

    if (document[hiddenProperty]) {

        !MpMovie.video.paused && MpMovie.video.pause();

    }

};



document.addEventListener(visibilityChangeEvent, onVisibilityChange);
```

---
### 11，web-view 中调用如 openLocation 需要鉴权问题
https://mp.weixin.qq.com/wiki?t=resource/res_main&id=mp1421141115
此处鉴权必须得使用一个公众号的 appId 和 appsecret 才行。

**不能用对应小程序的**

**千万不能用对应小程序的**


### 12，wx.showActionSheet 在 Android 和 iOS 上面行为差异过大。 

# Canvas 篇

因 Canvas 坑实在太多，把 Canvas 的坑单独列出来

### 1，wx.canvasToTempFilePath 在模拟器和真机上行为不一致

**相关文档**

https://developers.weixin.qq.com/miniprogram/dev/api/canvas/temp-file.html

**现象**

使用 Canvas 绘图成功后，直接调用该方法生成图片，在 IDE 上没有问题，但在真机上会出现生成的图片不完整的情况。

**解决**

需要加个神秘的 timeout （300ms，再少就不行了）时间后，再调用该方法，这样生成的图片才完整。下面是社区里面的一篇帖子。

https://developers.weixin.qq.com/blogdetail?action=get_post_info&lang=zh_CN&token=&docid=708aea205622cc8518cfeea3c154a5b7

---

### 2，canvasContext.font 的几个坑

**相关文档**

https://developers.weixin.qq.com/miniprogram/dev/api/canvas/font.html

- **size 不能使用小数**

如果设置 font 中字体大小部分包含小数，则会导致整个 font 设置无效。

- **style 设置 italic 问题**

如果设置为 italic，在 Android 机上只有同时设置 weight 为 bold 才能生效。在 iOS 上，不管设不设置都不生效。而真机上是正常的。

---

### 3，canvasContext.drawImage 的坑

**相关文档**

https://developers.weixin.qq.com/miniprogram/dev/api/canvas/draw-image.html

- **在 IDE 中可以直接设置网络图片进行绘制，但在真机上设置网络图片无用**

**解决**

使用 painter 库进行绘制，https://github.com/Kujiale-Mobile/Painter 。Painter 在进行图片绘制前，会先进行图片下载操作，并且还会把下载的图片存储本地，进行 LRU 管理，让后续绘制同样图片时，节省下载时间。

- **在 IDE 中可设置 base64 的图片数据进行绘制，但真机上无用**

**解决**

先把 base64 转成 Uint8ClampedArray 格式。然后再通过 [wx.canvasPutImageData(OBJECT, this)](https://developers.weixin.qq.com/miniprogram/dev/api/canvas/put-image-data.html) 绘制到画布上，然后把画布导出为图片。

---


### 4，canvasContext.clip 的一些坑

**相关文档**

https://developers.weixin.qq.com/miniprogram/dev/api/canvas/clip.html

- **canvasContext.clip 方法在 iOS 设备上的微信 6.6.6 版本及以下会导致该 clip 下面使用的的 restore 方法失效。**
- **在 clip 之前如果设置了，setFillStyle 为透明，在 Android 上会直接导致所裁图片无法显示。而 IDE 和 iOS 上无此问题。**

**解决**

如果想实现圆角，还是使用 Painter 库来解决。https://github.com/Kujiale-Mobile/Painter 。


