# Android非正常关闭 #
## Activity的异常关闭 ##
###异常关闭情况1###
####资源内存不足导致低优先级Activity被杀死####
#####Activity优先级#####

1. 前台Activity——正在和用户交互的Activity，优先级最高
1. 可见但非前台Activity——Activity中弹出的对话框导致Activity可见但无法交互
1. 后台Activity——已经被暂停的Activity，优先级最低

系统内存不足是，会按照以上顺序杀死Activity，并通过onSaveInstanceState和onRestoreInstanceState这两个方法来存储和恢复数据。

比如有些手机本身内存不足时调用拍照会导致整个app被杀掉。这时我们就要用到手动保存数据。

如果当你不想activity重新创建可以在AndroidMainfest中给Activity指定configChanges属性，如

    android:configChanges="orientation"

更多属性请看[官方文档](http://developer.android.com/intl/zh-cn/guide/topics/manifest/activity-element.html#config)

Android官方给出的onSaveInstanceState和onRestroeInstanceState调用顺序图
![](http://i.imgur.com/A9xcX1u.png)

但是要注意onActivityResult要调用到网络上返回的数据（虽然会在oncreat（执行在onActivityResult之前）中重新获取，但是还是有一定延迟的），反正oncreat中请求的就是最新数据，我们只要把oncreat的异常给try-catch掉


## Fragment的奇葩情况 ##
最让我头疼的莫过于fragment了。
fragment的onSaveInstanceState的调用并非是fragment异常销毁的情况下。只要fragment一到后台就是被调用（fuck，网上都是说异常销毁时，你们自己打下log看看吧）

##顺便附上完整的fragment和activity的生命周期##
> 转自：https://github.com/xxv/android-lifecycle
![](http://i.imgur.com/Jk3UkmA.png)