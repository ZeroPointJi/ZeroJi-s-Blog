---
title:  "iOS 之 Crawer"
date:   2016-03-28 20:35:00 +8:00
categories: 
  - iOS
tags: 
  - Objective-C
---
## Crawer 抽屉效果

`iOS` 编程中经常使用的一种动画效果. 利用 `UIView` 动画, 改变视图的 `frame` 来达到像抽屉推进抽出的效果.

根据传入的数值创建视图控制器

```objc
- (void)createViewController:(NSInteger)index
{
    BaseViewController *baseVC = nil;
        
    if (index == 0) {
        baseVC = [[ReadViewController alloc] init];
    } else if (index == 1) {
        baseVC = [[RadioViewController alloc] init];
    } else if (index == 2) {
        baseVC = [[TopicViewController alloc] init];
    } else if (index == 3) {
        baseVC = [[ProductViewController alloc] init];
    }
        
    self.naVC = [[UINavigationController alloc] initWithRootViewController:baseVC];
    baseVC.navigationItem.leftBarButtonItem = [[UIBarButtonItem alloc] initWithTitle:@"三" style:UIBarButtonItemStyleDone target:self action:@selector(pushLeft)];
    baseVC.navigationItem.title = self.rootViewNameArr[index];
    [self.view addSubview:self.naVC.view];
    //    self.rootView = baseVC.view;
    //[self createNvButtonAndAddToView:self.naVC.view];
    //[self addChildViewController:baseVC];
}
```

推出左视图

```objc
- (void)pushLeft
{
    if (_showLeft) {
            [UIView animateWithDuration:0.3 animations:^{
            [self changeXTo:0];
        } completion:^(BOOL finished) {
            _showLeft = NO;
        }];
    } else {
        [UIView animateWithDuration:0.3 animations:^{
            [self changeXTo:kLeftWidth];
        } completion:^(BOOL finished) {
            _showLeft = YES;
        }];
    }
}
```

改变当前显示视图的x轴坐标并且根据坐标判断是否响应轻拍事件

```objc
- (void)changeXTo:(NSInteger)x
{
    if (x) {
        _tap.enabled = YES;
    } else {
        _tap.enabled = NO;
    }
    CGRect frame = self.naVC.view.frame;
    frame.origin.x = x;
    self.naVC.view.frame = frame;
}
```

视图加载

```objc
- (void)viewDidLoad {
    [super viewDidLoad];
    
    _showLeft = NO;
    self.leftTableView.delegate = self;
    self.leftTableView.dataSource = self;
    
    [self createViewController:0];
    
    self.tap = [[UITapGestureRecognizer alloc ]initWithTarget:self action:@selector(pushLeft)];
    [self.view addGestureRecognizer:self.tap];
    _tap.enabled = NO;
    self.tap.delegate = self;
}
```

更改轻拍方法作用域

```objc
- (BOOL)gestureRecognizerShouldBegin:(UIGestureRecognizer *)gestureRecognizer
{
   return CGRectContainsPoint(self.naVC.view.frame, [gestureRecognizer locationInView:self.view]);
}
```

当点击 `cell` 时更换视图

```objc
- (void)tableView:(UITableView *)tableView didSelectRowAtIndexPath:(NSIndexPath *)indexPath
{
    [self.naVC.view removeFromSuperview];
    self.naVC = nil;
    [self createViewController:indexPath.row];
    
    [self changeXTo:kLeftWidth];
    [UIView animateWithDuration:0.3 animations:^{
        [self changeXTo:0];
    } completion:^(BOOL finished) {
        _showLeft = NO;
    }];
}
```

> 注意, 一定要接受代理并且修改 `UITapGestureRecognizer` 的作用域, 否则会覆盖 `cell` 的点击.

边缘拖拽还没实现, 未完待续.
