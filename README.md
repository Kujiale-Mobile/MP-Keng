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

**绕坑**

像这种非 5:4 的图片，尽量也弄成 5:4 的，然后多余部分用白色进行填充。



### 2，wx.showToast 在 iOS 和 Android 上行为不一致

 **相关文档**

https://developers.weixin.qq.com/miniprogram/dev/api/api-react.html#wxshowtoastobject

**现象**

当 showToast 后，马上返回上一个页面，在 Android 上，toast 依然存在，但 iOS 上 toast 无法显示。感觉是因为在 Android 上这个 toast 是全局的，而 iOS 上 toast 只是依附于某个页面。

**绕坑淫巧**

如果想在这种情况下，Android 和 iOS 都能显示出 toast，那只能加个 timeout 延时了，比如 1s 后再 执行 `wx.navigateBack();` 的操作。



### 3，wx.canvasToTempFilePath 在模拟器和真机上行为不一致

**相关文档**

https://developers.weixin.qq.com/miniprogram/dev/api/canvas/temp-file.html

**现象**

使用 Canvas 绘图成功后，直接调用该方法生成图片，在 IDE 上没有问题，但在真机上会出现生成的图片不完整的情况。

**绕坑淫巧**

需要加个神秘的 timeout （300ms，再少就不行了）时间后，再调用该方法，这样生成的图片才完整。下面是社区里面的一篇帖子。

https://developers.weixin.qq.com/blogdetail?action=get_post_info&lang=zh_CN&token=&docid=708aea205622cc8518cfeea3c154a5b7