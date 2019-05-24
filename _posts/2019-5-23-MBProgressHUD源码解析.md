---
title: MBProgressHUD源码解析
date: 2019-5-23 22:30:00
tag: 源码阅读
categories: 源码阅读
---

​    在所有程序员提高的方法中，几乎所有人都提到了要多读优秀的源码。所以在学习的路途中便开始了源码阅读的计划，至于从哪个源码开始呢，首先肯定是要自己经常用的，并且比较优秀的。所以最终确定为从MBProgressHUD开始进行源码计划。

### 功能

> MBProgressHUD是一个为iOS添加悬浮层的框架，主要用在网络请求，弹窗提示等场景。

### 框架结构

> ##### MBProgressHUD包含了4个类，MBProgressHUD，MBRoundProgressView，MBBarProgressView，MBBackgroundView.

### API结构

#### MBProgressHUD

##### 方法

```objective-c
+ (instancetype)showHUDAddedTo:(UIView *)view animated:(BOOL)animated;
+ (BOOL)hideHUDForView:(UIView *)view animated:(BOOL)animated;
+ (nullable MBProgressHUD *)HUDForView:(UIView *)view;
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

