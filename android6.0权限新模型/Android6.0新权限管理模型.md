# Android6.0新权限管理模型 #

> 原文：
http://www.w2bc.com/article/android-6-0-runtime-permission

### 运行时权限处理 ###
Android6.0系统默认为targetSdkVersion小于23的应用默认授予了所申请的**所有**权限，所以如果你以前的APP设置的targetSdkVersion低于23，在运行时也不会崩溃，但这也只是一个临时的救急策略，用户还是可以在设置中取消授予的权限。

### 解决方案 ###

检查并申请权限
我们需要在用到权限的地方，每次都检查是否APP已经拥有权限，比如我们有一个下载功能，需要写SD卡的权限，我们在写入之前检查是否有WRITE_EXTERNAL_STORAGE权限，没有则申请权限

	  if (ContextCompat.checkSelfPermission(this, Manifest.permission.WRITE_EXTERNAL_STORAGE)
              != PackageManager.PERMISSION_GRANTED) {
          //申请WRITE_EXTERNAL_STORAGE权限
          ActivityCompat.requestPermissions(this, new String[]{Manifest.permission.WRITE_EXTERNAL_STORAGE},
                  WRITE_EXTERNAL_STORAGE_REQUEST_CODE);
      }

![](http://pic.w2bc.com/upload/201511/09/201511090020345755.png)

用户选择允许或需要后，会回调onRequestPermissionsResult方法, 该方法类似于onActivityResult

	  @Override
	  public void onRequestPermissionsResult(int requestCode, String[] permissions, int[] grantResults) {
	      super.onRequestPermissionsResult(requestCode, permissions, grantResults);
	      doNext(requestCode,grantResults);
	  }
我们接着需要根据requestCode和grantResults(授权结果)做相应的后续处理
	
	private void doNext(int requestCode, int[] grantResults) {
	      if (requestCode == WRITE_EXTERNAL_STORAGE_REQUEST_CODE) {
	          if (grantResults[0] == PackageManager.PERMISSION_GRANTED) {
	              // Permission Granted
	          } else {
	              // Permission Denied
	          }
	      }
	  }

###坑--> Fragment中运行时权限的特殊处理 ###
在Fragment中申请权限，**不要使用**ActivityCompat.requestPermissions, 直接使用Fragment的requestPermissions方法，否则会回调到Activity的 onRequestPermissionsResult
如果在Fragment中嵌套Fragment，在子Fragment中使用requestPermissions方 法，onRequestPermissionsResult不会回调回来，建议使用 **getParentFragment**().requestPermissions方法，这个方法会回调到父Fragment中的onRequestPermissionsResult，加入以下代码可以把回调透传到子Fragment

	  @Override
	  public void onRequestPermissionsResult(int requestCode, String[] permissions, int[] grantResults) {
	      super.onRequestPermissionsResult(requestCode, permissions, grantResults);
	      List<Fragment> fragments = getChildFragmentManager().getFragments();
	      if (fragments != null) {
	          for (Fragment fragment : fragments) {
	              if (fragment != null) {
	                  fragment.onRequestPermissionsResult(requestCode,permissions,grantResults);
	              }
	          }
	      }
	  }


### 第三方解决方案 ###

相关开源项目
[PermissionsDispatcher](https://github.com/hotchemi/PermissionsDispatcher)

使用标注的方式，动态生成类处理运行时权限，目前还不支持嵌套Fragment。
[RxPermissions](https://github.com/tbruyelle/RxPermissions)

基于RxJava的运行时权限检测框架
[Grant](https://github.com/anthonycr/Grant)

简化运行时权限的处理，比较灵活
[android-RuntimePermissions](https://github.com/googlesamples/android-RuntimePermissions)
Google官方的例子