---
layout: post
title:  ReactiveCocoa在UIView上的运用
date:   2015-6-12 17:02:33
categories: iOS
---

FOR CM AND PLUSUB
---

> 最近项目非常繁忙...上星期本来想把和block内存管理有关的东西看一下再写一个有关block的博客的后来有关block内存管理的博客都找好了没时间实战也就没出。
写了有两周的项目，因为很多原因所以这近两周一直在写界面，但是在写界面的时候用到了ReactiveCocoa的东西，也是因为实践了所以写篇博客分享出来。

*`ReactiveCocoa（下文统称为RAC）`，具体我就不介绍了，可以去我的博客里那篇还没填完坑的  [mvvm开发模式](http://walkginkgo.com/ios/2015/05/03/ios-mvvm-1.html)  文章看一下。*

*个人以为，RAC可以以信号的方式来触发各种动作，从而让代码更加精炼。*

## 下面上代码吧。

##  `在UIButton上的使用`
{% highlight objc %}
[[self.testBtn rac_signalForControlEvents:UIControlEventTouchUpInside]
                            subscribeNext:^(id x) {
    TestViewController *testVC = [[TestViewController alloc] init];
    [self.navigationController pushViewController:testVC animated:YES];
}];
{% endhighlight %}
在这里，对uibutton添加了一个rac_signalForControlEvents的方式，就不用利用addtarget的方式来再写一个方法来进行对uibutton添加点击事件了。

##  `在UIAlertView的使用`
{% highlight objc %}
UIAlertView *chooseAlert = [[UIAlertView alloc] initWithTitle:@"选择图片上传" message:nil delegate:nil cancelButtonTitle:@"取消" otherButtonTitles:@"拍照上传", @"从相册选择", nil];
[chooseAlert show];
    
[[chooseAlert rac_buttonClickedSignal] subscribeNext:^(NSNumber *indexNumber) {
    if ([indexNumber intValue] == 1) {
        [self chooseFromCamera];
    } else if ([indexNumber intValue] == 2) {
        [self chooseFromAlbum];
    }
}];
{% endhighlight %}
这两个是[limeboy](http://limboy.me/)在博客里提到的。因为有了RAC，所以我这次放弃使用blockskit和alertview的Categories。

下面两个是我利用RAC的方式写的代码。

##  `在UITextfield的使用`
第一个是我封装了一个安卓风格的输入框，就是下面一条线，当选择这个框的时候，线会加粗变黑。
我把中间的两句核心代码贴出来。self是因为我继承了一个textfield先进行功能添加和封装。所以self就是代指一个textfield了。
{% highlight objc %}
[[self rac_signalForControlEvents:UIControlEventEditingDidBegin] 
                    subscribeNext:^(NSNumber *editing) {
    self.bottomBorder.backgroundColor = [UIColor blackColor].CGColor;
}];
[[self rac_signalForControlEvents:UIControlEventEditingDidEnd] 
                    subscribeNext:^(NSNumber *editing) {
    self.bottomBorder.backgroundColor = [UIColor grayColor].CGColor;
}];
{% endhighlight %}
其实这两句是可以用addtarget的方式来添加的，但是我选择了用RAC的方式，更为简单也更为容易理解。当textfield被选中的时候，下面的borderline会变成黑色，当结束选择的时候，变成灰色。(其实当选择的时候borderline应该还要加粗)。

##  `监控UIPagecontrol改变`
还有一个地方我利用到了RAC。因为我现在需要实现一个功能，图片轮播的时候，当图片切换，我需要相应的刷新下面的一个列表。因为我们的图片轮播是用到的一个开源控件，我实在是能力有限不知道怎么进一步修改这个控件来进一步封装从而实现新的功能，我就利用了RAC。
{% highlight objc %}
[RACObserve(self.imagePlayer.pageControl, currentPage) subscribeNext:^(id x) {
    [self refreshSlideContent:self.imagePlayer.pageControl.currentPage];
}];
{% endhighlight %}
上面的代码将pageControl和它的currentPage属性相绑定，当currentPage改变的时候就会触发下面的函数。然后我传了一个currentPage的参数进去，从而下面的列表可以进一步更新。但是开始加载的时候会调用3次…我不知道我哪个地方写坑了我还在寻找。各位见谅…

上面的几个代码片段就是我这两周在写纯界面的时候用到的一些和RAC有关的东西，因为本身就是写纯界面，所以没用到很多，也很简单，没有其他的功能。刚刚在查limeboy博客的时候看到NotificationCenter也可以利用RAC的方式写，我抽时间也会尝试，因为项目里没用到，所以就不贴代码了。

> BTW：最近有一大波考试，博客更新程度会大大降低...后面可能会一直做项目，等积累一部分才会继续写了。多谢各位的支持。

##  参考资料：
Limboy [说说ReactiveCocoa 2](http://limboy.me/ios/2013/12/27/reactivecocoa-2.html)



