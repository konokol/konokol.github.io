---

date: 2019-01-24 20:04:05
title: 使用ConstraintLayout构建响应式布局
tags: Android
category: tech
description: 使用ContraintLayout能够使用平铺的布局方式（没有层级嵌套）创建大而复杂的布局。和RelativeLayout类似，ConstraintLayout所有的View都在同一层级中，但是它比RelativeLayout使用起来更加灵活，结合Android Studio的布局编辑器，使用起来也更加简单。
---


## 1 简介
> 使用ContraintLayout能够使用平铺的布局方式（没有层级嵌套）创建大而复杂的布局。和RelativeLayout类似，ConstraintLayout所有的View都在同一层级中，但是它比RelativeLayout使用起来更加灵活，结合Android Studio的布局编辑器，使用起来也更加简单。
> 
> ContraintLayout的API和Android Studio的布局编辑器的图形化工具是相辅相成的，通过布局编辑器的图形化工具，可以直接使用ConstraintLayout的全部特性。在使用ConstraintLayout时，可以直接拖拽进行布局，完全不用编辑xml。
> 
> ConstraintLayout支持Android 2.3(API 9)以上，本文是在Android Studio 3.0及以上版本中使用ConstraintLayout的教程，如果想要了解Android Studio中更多关于布局编辑器的内容，可以参考官网[Build a UI with Layout Editor](https://developer.android.com/studio/write/layout-editor.html)。
> 
> 如果想要看ConstraintLayout的Demo，可以去GitHub上围观：[https://github.com/googlesamples/android-ConstraintLayoutExamples](https://github.com/googlesamples/android-ConstraintLayoutExamples)。

## 2 约束预览

> 要确定一个View在ConstraintLayout中的位置，至少需要指定一个水平或者垂直的约束。每一个约束都代表了一个View和其它的View，其父View或者不可见的基准线之间的联系或者对齐关系。每一个约束都定义了View相对于横轴和纵轴的位置，因此，确定一个View的位置必须至少相对于横轴和纵轴各指定一个约束，通常来说，我们需要指定的约束会有多个。
> 
> 在Android Studio中，当把一个View拖到布局编辑器中的时候，这个View一般会显示在你拖动的位置，但是这仅仅是为了便于我们编辑，如果一个View一个约束也没有，那它会显示在坐标[0, 0]的位置，即左上角。

比如下面的两种情况：

<center>
![图1 A和C没有约束](http://ww1.sinaimg.cn/large/afdaace3ly1fnpf0fiigij20cw068dfr.jpg)</br>
图1 A和C没有约束
</center>

<center>
![图2 A和C有约束](http://ww1.sinaimg.cn/large/afdaace3ly1fnpf10sjdgj20cw068dfu.jpg)</br>
图2 A和C有约束
</center>

图1中，A和C没有约束关系，尽管在预览时会看到C在A的下方，实际运行时，C会浮到屏幕的最顶端。图2中，加上了A和C的约束(红圈所示)之后，C就会位于A的下方。

## 3 配置ConstraintLayout

1. maven仓库的配置

 在最新版本的gradle中，Google迁移了maven仓库，所以需要在项目根目录的build.gradle中添加：

 ```
 repositories {
    maven {
        url 'https://maven.google.com'
    }
 }
 ```
 或者是
 
 ```
 repositories {
        jcenter()
        google()
    }
 ```

2. 添加gradle最新版本的依赖，目前稳定版本是1.0.2，1.1.0还是beta版。

 ```
 dependencies {
     compile 'com.android.support.constraint:constraint-layout:1.0.2'
 }
 ```
3. 同步gradle

## 4 在项目中使用ConstraintLayout

在Android Studio 3.0+中，可以把旧的布局转换成ConstraintLayout，也可以在新建Constraint
Layout。如果没做特别的配置，使用Android Studio的Activity模版新建Activity时，布局文件根布局会默认是ConstraintLayout。

将普通布局文件转换成ConstraintLayout：
切到Design面板->打开Component Tree->右键选中一个Layout，选择Convert XXXLayout to ConstraintLayout；
<center>
![](http://ww1.sinaimg.cn/large/afdaace3ly1fnqks0e1w4j20n20i5q38.jpg)</br>
图3 将普通布局转换成Constraint
Layout</center>

## 5 实战练习

### 5.1 添加相对于父布局的位置约束

新建一个布局文件，根布局选择为ConstraintLayout，切换到Design面板
<center>
![](http://ww1.sinaimg.cn/large/afdaace3ly1fnqluzldu4j21ue19sn4v.jpg)</br>
图4 新建一个的ConstraintLayout
</center>

拖动一个控件到面板中，此时没有任何约束，会看到控件上报了一个错误，提示没有约束，如果直接运行，这个TextView会显示到屏幕的左上角

<center>
![](http://ww1.sinaimg.cn/large/afdaace3ly1fnqlxou6m8j21u817ewm2.jpg)
图5 未添加约束时的警告
</center>

添加相对于父布局的约束，在一个方向上可以添加一个或者两个约束
<center>
![](http://ww1.sinaimg.cn/large/afdaace3ly1fnqm9npfx3g20pk0g4172.gif)</br>
图6 添加左侧的约束
![](http://ww1.sinaimg.cn/large/afdaace3ly1fnqmn6pigmg20pk0g4atu.gif)</br>
图7 添加右侧的约束
</center>

### 5.2 添加相对于兄弟View的位置约束

添加相对于其它View的约束，下图中B、C只添加了对A的约束，运行起来后，B贴在屏幕顶部且在A的右边，C贴在屏幕的左侧且位于A的下方。
<center>
![](http://ww1.sinaimg.cn/large/afdaace3ly1fnqmvdktk7j215q0yctb7.jpg)
</br>
图7 相对于其它View的约束

![](http://ww1.sinaimg.cn/large/afdaace3ly1fnqswt7oohj20ko1240yx.jpg)
图8 运行的实际效果
</center>

### 5.3 添加对齐方式

在两个View的同一个方向(水平或垂直)上，把两个View的两条边连接起来即可添加对其约束。添加对其约束之后，laytou_marginXXX等属性仍然会生效。
<center>
![](http://ww1.sinaimg.cn/large/afdaace3ly1fnqnp0oot8j215o0y6mzp.jpg)
</br>
图9 C与A的左侧对其，B与A的顶部对其且向下偏移16dp
</center>

### 5.4 删除约束
删除约束很简单，选中一个View，鼠标hover在View的一个约束上时，约束会变红，点按即可删除。
<center>
![](http://ww1.sinaimg.cn/large/afdaace3ly1fnqnhkw90vj215q0yedir.jpg)
</br>
图10 删除一个约束
</center>

### 5.5 相对于guideline的约束

除了相对于父布局和兄弟View，ConstraintLayout中还可以在水平方向和垂直方向上添加guide，ConstraintLayout内的View可以添加相对于guideline的约束。

<center>
![](http://ww1.sinaimg.cn/large/afdaace3ly1fnqojv5ddcj20r4072my3.jpg)
</br>
图11 添加guideline
</center>

添加的guideline在xml布局中表现和View一样。guideline可以像View一样添加约束。
<center>
![](http://ww1.sinaimg.cn/large/afdaace3ly1fnqoogvy5jj20rg09ctax.jpg)
</br>
图12 guideline在xml中的表示
</center>

添加完guideline之后，选中guideline，在布局的边缘可以看到一个小三角或者百分号，点击可以切换guideline相对于布局编辑的位置，显示为小三角的方向表示guideline距布局边界的绝对位置（单位是dp），百分号则表示相对位置

<center>
![](http://ww1.sinaimg.cn/large/afdaace3ly1fnqr0whwlmj21ts1acqbo.jpg)
</br>
图13 guideline在屏幕垂直方向上20%的位置，A位于guideline的下方
</center>

### 5.6 添加相对于View的barrie
barrier是ConstraintLayout 1.1中新加的特性，barrier和guideline类似，从字面意思可以理解，它相当于一个屏障，包含在其中的View不能越过它的位置。举个例子，图14的布局中，B位于A的右侧，当C位于A的下方，当C的文字比较少时，B会在C的右侧，当C的文字比较多时，C就会被B盖住，但是B左侧的约束又只能有一个。

<center>
![](http://ww1.sinaimg.cn/large/afdaace3ly1fnqt7ah8toj21a8124mzz.jpg)
</br>
图13 C的文字比较少

![](http://ww1.sinaimg.cn/large/afdaace3ly1fnqtars5vij21aa124n0c.jpg)
</br>
图13 C的文字比较多时，B盖住了C
</center>


此时就可以添加一个barrier，在B的左侧添加一个到barrier的约束，使B的位置随着A、C的内容左右移动而不用担心B的内容盖住A。

<center>
![](http://ww1.sinaimg.cn/large/afdaace3ly1fnqt7ah8toj21a8124mzz.jpg)
</br>
图13 C的文字比较少

![](http://ww1.sinaimg.cn/large/afdaace3ly1fnqtjpiz20j21ae124adv.jpg)
</br>
图14 B添加了相对于barrier的约束
</center>

添加barrier的方法：

> 1. Click Guidelines ![](http://ww1.sinaimg.cn/large/afdaace3ly1fnqtmbpbvij200w00w047.jpg)in the toolbar, and then click Add Vertical Barrier or Add Horizontal Barrier.
> 2. In the Component Tree window, select the views you want inside the barrier and drag them into the barrier component.
> 3. Select the barrier from the Component Tree, open the Attributes window, and then set the barrierDirection.

1. 点击工具栏添加guideline的位置，添加垂直或者水平的barrier；
2. 在Component Tree视图中，把想要限制的View拖进barrier中；
3. 在Component Tree中选中barrier，打开右侧的属性窗口，设置barrierDirection属性。

### 5.7 调节约束的位置

在前面，给View水平方向上添加2个约束时，会发现View自动水平居中了。此时，可以调节ConstraintLayout的bias，使其在屏幕一定比例的位置上，而不是居中。

<center>
![](http://ww1.sinaimg.cn/large/afdaace3ly1fnrnk39ur1g21g80p0x6r.gif)
</br>
图15 调节约束的位置
</center>

### 5.8 调节View的大小

ConstraintLayout的子View中，除了可以通过正常的layout_width和layout_height来限制View的宽高为，在Design面板的右侧，可以很方便调整View的大小。

<center>
![](http://ww1.sinaimg.cn/large/afdaace3ly1fnrnm43y7lj20fr0j9jte.jpg)
</br>
图16 调节View的大小
</center>
图中的编号中，

1. 调节View的宽高比
2. 删除某个方向上的约束
3. View的宽高模式，Fixed（宽高写死），Wrap Content，Match Constraints（和match parent一样）
4. View的margin值
5. bias

### 5.9 链式线性分组

在一个链式线性分组内的View会在一个方向上形成一个双向约束，互相约束。