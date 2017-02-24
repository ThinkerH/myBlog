---
layout: post
title: iOS 下拉刷新实现原理
tags: 学习
---

最近研究iOS的下拉刷新，想自己封装个下拉刷新控件，所以首先研究了下实现的原理。
这里主要涉及到两个知识点，如下：

__1.KVO__

__2.`UIScrollView`的下面三个属性__

``` Objective-C
	@property(nonatomic)         CGPoint                      contentOffset;                  // default CGPointZero
	@property(nonatomic)         CGSize                       contentSize;                    // default CGSizeZero
	@property(nonatomic)         UIEdgeInsets                 contentInset;                   // default UIEdgeInsetsZero. add
```

---

## 1.KVO

KVO是iOS中观察者模式的一种实现方式，具体实现机制这里不赘述，可参考这篇文章[《谈谈 KVO》](http://www.jianshu.com/p/cfd553f250f9)。
下拉刷新控件要实现首先就要监控`UIScrollView`(`UITableVeiw`继承自`UIScrollView)`的`contentOffset`属性值，这里需要定义一个`UIScrollView`的扩展类(catagray)，在扩展类的实例方法中通过KVO监控`UIScrollView`下拉是的`contentOffset`属性值变化，具体代码如下：

``` Objective-C
@implementation UIScrollView (RefreshView)

- (void)addRefreshWithRefreshViewType:(HLRefreshViewType)refreshViewType 	refreshingBlock:(void (^)())block
{
   HLRefresh *refresh = [HLRefresh refreshWithRefreshViewType:refreshViewType refreshingBlock:block];
    refresh.scrollView = self;
    
    [self addObserver:refresh forKeyPath:@"contentOffset" options:NSKeyValueObservingOptionNew context:nil];
    [self addSubview:refresh];
}
//这里的`HLRefresh`是下拉刷新时显示刷新状态的头View
```

然后在KVO的Observer——`refresh`对象的`HLRefresh`类中实现如下：

``` Objective-C
- (void)observeValueForKeyPath:(NSString *)keyPath ofObject:(id)object change:(NSDictionary *)change context:(void *)context
{
    HLOG(@"=============")
    //如果当前状态是 HLRefreshHeaderStateRefreshing 不再做检测处理
    if (_headerState ==  HLRefreshHeaderStateRefreshing) {
        return;
    }
    [self adjustStateWithContentOffset];
}

- (void)adjustStateWithContentOffset
{
    //记录 scrollView原始的上边距.  方便刷新之后,把 scrollView 的contentInset改回这个位置.
    _edgeInsetsTop = self.scrollView.contentInset.top;
    
    if (_headerState == HLRefreshHeaderStateRefreshing) {
        return;
    }
    
    //当前的偏移量
    CGFloat contentOffsetY = self.scrollView.contentOffset.y;
    //scrollView左上角 原始偏移量(默认是0),在有导航栏的情况下可能会被调整为64.
    CGFloat happenOffsetY = -self.scrollView.contentInset.top;
    
    //如果往上滑动,直接 return
    if (contentOffsetY >= happenOffsetY) return;
    //header 完全出现时的contentOffset.y
    CGFloat headerCompleteDisplayContentOffsetY = happenOffsetY - kHeaderHeight;
//    NSLog(@"%f  %f  %f",contentOffsetY,happenOffsetY,headerCompleteDisplayContentOffsetY);
    if (self.scrollView.isDragging == YES) {//如果正在拖拽
        //如果当前状态是 HLRefreshStateIdle(闲置状态或者叫正常状态) && header 已经全部显示
        if (_headerState == HLRefreshHeaderStateIdle && contentOffsetY < headerCompleteDisplayContentOffsetY) {
            //将状态设置为  松开就可以进行刷新的状态
            self.headerState = HLRefreshHeaderStatePulling;
//            NSLog(@"下拉状态");
        }else if (_headerState == HLRefreshHeaderStatePulling && contentOffsetY > headerCompleteDisplayContentOffsetY){//如果当前状态是 HLRefreshStatePulling(松开就可以进行刷新的状态) && header只显示了一部分(用户往上滑动了)
            self.headerState = HLRefreshHeaderStateIdle;
//            NSLog(@"常态");
        }
    }else{//如果松开了手
        if (_headerState == HLRefreshHeaderStatePulling) {//如果状态是1,下拉状态.让它进入刷新状态
            self.headerState = HLRefreshHeaderStateRefreshing;
//            NSLog(@"刷新中");
        }
    }
}
```

即可监测添加下拉刷新控件的`UIScrollView`的`contentOffset`从而判断刷新状态，这里的
`HLRefreshHeaderStateIdle`、`HLRefreshHeaderStatePulling `、`HLRefreshHeaderStateRefreshing `是枚举值，定义如下：

``` Objective-C
	typedef NS_ENUM(NSUInteger, HLRefreshHeaderState){
	    HLRefreshHeaderStateIdle,         //闲置状态
	    HLRefreshHeaderStatePulling,      //松开就可以进行刷新的状态
	    HLRefreshHeaderStateRefreshing    //正在刷新中的状态
	};
```

## 2.`UIScrollView`的三个属性`contentOffset `、`contentSize `、`contentInset `

``` Objective-C
	@property(nonatomic)  CGPoint   contentOffset;   // default CGPointZero；//contentSize是scrollview可以滚动的区域，比如frame = (0 ,0 ,320 ,480) contentSize = (320 ,960)，代表你的scrollview可以上下滚动，滚动区域为frame大小的两倍
	@property(nonatomic)  CGSize    contentSize;      // default CGSizeZero；//contentOffset是scrollview当前显示区域顶点相对于frame顶点的偏移量，比如上个例子你拉到最下面，contentoffset就是(0 ,480)，也就是y偏移了480；
	@property(nonatomic)  UIEdgeInsets   contentInset;     // default UIEdgeInsetsZero. add ；//contentInset是scrollview的contentview的顶点相对于scrollview的位置，例如你的contentInset = (0 ,100)，那么你的contentview就是从scrollview的(0 ,100)开始显示；
```

这里重点要用的就是`contentOffset `、`contentInset `

>`contentOffset `是监测下拉位移以判断刷新状态;
>`contentInset `是用来在显示刷新的头View(例如上面的`refresh`)时将UIScrollView的
>`contentview`下移一个头View的高度以显示头View，刷新结束后恢复原先的`contentInset `；
>__在整个过程中，头View(`refresh`)一直是贴在UIScrollView的`contentview`顶部，只是`contentInset `为初始状态是在屏幕外，当`contentview`下移一个头View的高度后则显示出来。__


__理解上面两点后，基本就可以 去实现自己的下拉刷新控件了，而上拉加载更多的实现原理也基本一样。__
