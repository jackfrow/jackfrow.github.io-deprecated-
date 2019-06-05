---
title: MBProgressHUD源码解析
date: 2019-5-23 22:30:00
tag: 源码阅读
categories: 源码阅读


typora-root-url: ../assets
---

​    在所有程序员提高的方法中，几乎所有人都提到了要多读优秀的源码。所以在学习的路途中便开始了源码阅读的计划，至于从哪个源码开始呢，首先肯定是要自己经常用的，并且比较优秀的。所以最终确定为从MBProgressHUD开始进行源码计划。

### 功能

> MBProgressHUD是一个为iOS添加悬浮层的框架，主要用在网络请求，弹窗提示等场景。

### 框架结构

> ##### MBProgressHUD包含了4个类，MBProgressHUD，MBRoundProgressView，MBBarProgressView，MBBackgroundView.

### API结构

#### MBProgressHUD

##### 方法

~~~objective-c
- (void)showAnimated:(BOOL)animated;
~~~



```objective-c
+ (instancetype)showHUDAddedTo:(UIView *)view animated:(BOOL)animated;
+ (BOOL)hideHUDForView:(UIView *)view animated:(BOOL)animated;
+ (nullable MBProgressHUD *)HUDForView:(UIView *)view;
- (instancetype)initWithView:(UIView *)view;
- (void)showAnimated:(BOOL)animated;
- (void)hideAnimated:(BOOL)animated;
- (void)hideAnimated:(BOOL)animated afterDelay:(NSTimeInterval)delay;
```

##### 属性

```objective-c
@property (weak, nonatomic) id<MBProgressHUDDelegate> delegate;
@property (copy, nullable) MBProgressHUDCompletionBlock completionBlock;
@property (assign, nonatomic) NSTimeInterval graceTime;
@property (assign, nonatomic) NSTimeInterval minShowTime;
@property (assign, nonatomic) BOOL removeFromSuperViewOnHide;
@property (assign, nonatomic) MBProgressHUDMode mode;
@property (strong, nonatomic, nullable) UIColor *contentColor UI_APPEARANCE_SELECTOR;
@property (assign, nonatomic) MBProgressHUDAnimation animationType UI_APPEARANCE_SELECTOR;
@property (assign, nonatomic) CGPoint offset UI_APPEARANCE_SELECTOR;
@property (assign, nonatomic) CGFloat margin UI_APPEARANCE_SELECTOR;
@property (assign, nonatomic) CGSize minSize UI_APPEARANCE_SELECTOR;
@property (assign, nonatomic, getter = isSquare) BOOL square UI_APPEARANCE_SELECTOR;
@property (assign, nonatomic, getter=areDefaultMotionEffectsEnabled) BOOL defaultMotionEffectsEnabled UI_APPEARANCE_SELECTOR;
@property (assign, nonatomic) float progress;
@property (strong, nonatomic, nullable) NSProgress *progressObject;
@property (strong, nonatomic, readonly) MBBackgroundView *bezelView;
@property (strong, nonatomic, readonly) MBBackgroundView *backgroundView;
@property (strong, nonatomic, nullable) UIView *customView;
@property (strong, nonatomic, readonly) UILabel *label;
@property (strong, nonatomic, readonly) UILabel *detailsLabel;
@property (strong, nonatomic, readonly) UIButton *button;
```

#### MBRoundProgressView

```objective-c
@property (nonatomic, assign) float progress;
@property (nonatomic, strong) UIColor *progressTintColor;
@property (nonatomic, strong) UIColor *backgroundTintColor;
@property (nonatomic, assign, getter = isAnnular) BOOL annular;
```

#### MBBarProgressView

```objective-c
@property (nonatomic, assign) float progress;
@property (nonatomic, strong) UIColor *lineColor;
@property (nonatomic, strong) UIColor *progressRemainingColor;
@property (nonatomic, strong) UIColor *progressColor;
```

#### MBBackgroundView

```objective-c
@property (nonatomic) MBProgressHUDBackgroundStyle style;
@property (nonatomic) UIBlurEffectStyle blurEffectStyle;
@property (nonatomic, strong) UIColor *color;
```

### 调用流程

外部暴露的API主要分为show和hide两个类，不论是show还是hide，方法的最终调用都会走到`animateIn:withType: completion:`，下面是方法调用流程图。

![](http://ww2.sinaimg.cn/large/006tNc79ly1g3g2vm1hh6j314e0irdis.jpg)

### 内部实现

#### show

```objective-c
+ (instancetype)showHUDAddedTo:(UIView *)view animated:(BOOL)animated {
    MBProgressHUD *hud = [[self alloc] initWithView:view];
    hud.removeFromSuperViewOnHide = YES;
    [view addSubview:hud];
    [hud showAnimated:animated];
    return hud;
}

- (void)showAnimated:(BOOL)animated {
    MBMainThreadAssert();
    [self.minShowTimer invalidate];
    self.useAnimation = animated;
    self.finished = NO;
    // If the grace time is set, postpone the HUD display
    if (self.graceTime > 0.0) {
        NSTimer *timer = [NSTimer timerWithTimeInterval:self.graceTime target:self selector:@selector(handleGraceTimer:) userInfo:nil repeats:NO];
        [[NSRunLoop currentRunLoop] addTimer:timer forMode:NSRunLoopCommonModes];
        self.graceTimer = timer;
    } 
    // ... otherwise show the HUD immediately
    else {
        [self showUsingAnimation:self.useAnimation];
    }
}

- (void)showUsingAnimation:(BOOL)animated {
    // Cancel any previous animations
    [self.bezelView.layer removeAllAnimations];
    [self.backgroundView.layer removeAllAnimations];

    // Cancel any scheduled hideAnimated:afterDelay: calls
    [self.hideDelayTimer invalidate];

    self.showStarted = [NSDate date];
    self.alpha = 1.f;

    // Needed in case we hide and re-show with the same NSProgress object attached.
    [self setNSProgressDisplayLinkEnabled:YES];

    if (animated) {
        [self animateIn:YES withType:self.animationType completion:NULL];
    } else {
        self.bezelView.alpha = 1.f;
        self.backgroundView.alpha = 1.f;
    }
}

```

#### hide

```objective-c
+ (BOOL)hideHUDForView:(UIView *)view animated:(BOOL)animated {
    MBProgressHUD *hud = [self HUDForView:view];
    if (hud != nil) {
        hud.removeFromSuperViewOnHide = YES;
        [hud hideAnimated:animated];
        return YES;
    }
    return NO;
}

+ (MBProgressHUD *)HUDForView:(UIView *)view {
    NSEnumerator *subviewsEnum = [view.subviews reverseObjectEnumerator];
    for (UIView *subview in subviewsEnum) {
        if ([subview isKindOfClass:self]) {
            MBProgressHUD *hud = (MBProgressHUD *)subview;
            if (hud.hasFinished == NO) {
                return hud;
            }
        }
    }
    return nil;
}

 (void)hideAnimated:(BOOL)animated {
    MBMainThreadAssert();
    [self.graceTimer invalidate];
    self.useAnimation = animated;
    self.finished = YES;
    // If the minShow time is set, calculate how long the HUD was shown,
    // and postpone the hiding operation if necessary
    if (self.minShowTime > 0.0 && self.showStarted) {
        NSTimeInterval interv = [[NSDate date] timeIntervalSinceDate:self.showStarted];
        if (interv < self.minShowTime) {
            NSTimer *timer = [NSTimer timerWithTimeInterval:(self.minShowTime - interv) target:self selector:@selector(handleMinShowTimer:) userInfo:nil repeats:NO];
            [[NSRunLoop currentRunLoop] addTimer:timer forMode:NSRunLoopCommonModes];
            self.minShowTimer = timer;
            return;
        } 
    }
    // ... otherwise hide the HUD immediately
    [self hideUsingAnimation:self.useAnimation];
}

- (void)hideUsingAnimation:(BOOL)animated {
    // Cancel any scheduled hideAnimated:afterDelay: calls.
    // This needs to happen here instead of in done,
    // to avoid races if another hideAnimated:afterDelay:
    // call comes in while the HUD is animating out.
    [self.hideDelayTimer invalidate];

    if (animated && self.showStarted) {
        self.showStarted = nil;
        [self animateIn:NO withType:self.animationType completion:^(BOOL finished) {
            [self done];
        }];
    } else {
        self.showStarted = nil;
        self.bezelView.alpha = 0.f;
        self.backgroundView.alpha = 1.f;
        [self done];
    }
}
```

这里注意一下在拿当前hud时的方法.

```objective-c
+ (MBProgressHUD *)HUDForView:(UIView *)view {
    NSEnumerator *subviewsEnum = [view.subviews reverseObjectEnumerator];
    for (UIView *subview in subviewsEnum) {
        if ([subview isKindOfClass:self]) {
            MBProgressHUD *hud = (MBProgressHUD *)subview;
            if (hud.hasFinished == NO) {
                return hud;
            }
        }
    }
    return nil;
}
```

因为hud通常是最后放上去的，所以这里在拿hud的时候使用反向枚举器，可以减少循环次数。



而无论是`show`方法，还是`hide`方法，在设定animated属性为YES的前提下，最终都会走到`animateIn: withType: completion:`方法：

```objective-c
- (void)animateIn:(BOOL)animatingIn withType:(MBProgressHUDAnimation)type completion:(void(^)(BOOL finished))completion {
    // Automatically determine the correct zoom animation type
    if (type == MBProgressHUDAnimationZoom) {
        type = animatingIn ? MBProgressHUDAnimationZoomIn : MBProgressHUDAnimationZoomOut;
    }

    CGAffineTransform small = CGAffineTransformMakeScale(0.5f, 0.5f);
    CGAffineTransform large = CGAffineTransformMakeScale(1.5f, 1.5f);

    // Set starting state
    UIView *bezelView = self.bezelView;
    if (animatingIn && bezelView.alpha == 0.f && type == MBProgressHUDAnimationZoomIn) {
        bezelView.transform = small;
    } else if (animatingIn && bezelView.alpha == 0.f && type == MBProgressHUDAnimationZoomOut) {
        bezelView.transform = large;
    }

    // Perform animations
    dispatch_block_t animations = ^{
        if (animatingIn) {
            bezelView.transform = CGAffineTransformIdentity;
        } else if (!animatingIn && type == MBProgressHUDAnimationZoomIn) {
            bezelView.transform = large;
        } else if (!animatingIn && type == MBProgressHUDAnimationZoomOut) {
            bezelView.transform = small;
        }
        CGFloat alpha = animatingIn ? 1.f : 0.f;
        bezelView.alpha = alpha;
        self.backgroundView.alpha = alpha;
    };
    [UIView animateWithDuration:0.3 delay:0. usingSpringWithDamping:1.f initialSpringVelocity:0.f options:UIViewAnimationOptionBeginFromCurrentState animations:animations completion:completion];
}

```

### 收获

1. 暴露出来的API最终都会走到同一个私有方法里。
2. 使用CADisplayLink来刷新更新频率可能很高的view。
3. 使用循环时，注意条件可以减少循环次数，进而对程序进行优化。
4. 在添加hud时可以增加最小时间和延迟执行时间，避免一闪而过。