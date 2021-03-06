#iOS开发中的一些tips
持续更新自己开发中遇到的一些小问题，把这些记录下来

目录：

1. 设置`UITextView`的`placeHolder`
2. 键盘弹出和消失时让工具栏紧贴键盘
3. 让`Status Bar`上显示一个旋转的`ActivityIndicator`
4. `[NSDate date]`和实际时间相差8小时
5. `UICollectionView`中加入`UIRefreshControl`
6. 判断`UITableView`的滑动方向，向下滑动就隐藏`UITabBar`，向上滑动就显示`UITabBar`
7. 把`textView`文字长按出现的选项从英文改为中文
8. 开启`finder`显示隐藏文件

### 1. 设置UITextView的placeHolder

```objective-c
⁃	(void)textViewDidBeginEditing:(UITextView *)textView { 
	if ([textView.text isEqualToString:@"placeHolder here..."]) { 
		textView.text = @""; 
		textView.textColor = [UIColor blackColor]; 
	} 
	[textView becomeFirstResponder]; 
} 

⁃	(void)textViewDidEndEditing:(UITextView *)textView { 
	if ([textView.text isEqualToString:@""]) { 
		textView.text = @"placeHolder here..."; 
		textView.textColor = [UIColor lightGrayColor]; 
	} 
	[textView resignFirstResponder]; 
}
```
### 2. 键盘弹出和消失时让工具栏紧贴键盘
先写通知

```objective-c
⁃	(void)viewDidAppear:(BOOL)animated { 
	[super viewDidAppear:animated]; 

	[[NSNotificationCenter defaultCenter] addObserver:self selector:@selector(keyboardWillShow:) name:UIKeyboardWillShowNotification object:nil]; 
	[[NSNotificationCenter defaultCenter] addObserver:self selector:@selector(keyboardDidShow:) name:UIKeyboardDidShowNotification object:nil]; 
	[[NSNotificationCenter defaultCenter] addObserver:self selector:@selector(keyboardWillHide:) name:UIKeyboardWillHideNotification object:nil]; 
} 

⁃	(void)viewDidDisappear:(BOOL)animated { 
	[super viewDidDisappear:animated]; 

	[[NSNotificationCenter defaultCenter] removeObserver:self name:UIKeyboardWillShowNotification object:nil]; 
	[[NSNotificationCenter defaultCenter] removeObserver:self name:UIKeyboardDidShowNotification object:nil]; 
	[[NSNotificationCenter defaultCenter] removeObserver:self name:UIKeyboardWillHideNotification object:nil]; 
} 

```
方法具体写

```objective-c
- (void)keyboardWillShow:(NSNotification *)notification {

    self.hideKeyboardButton.hidden = NO;

    NSDictionary *userInfo = notification.userInfo;

    // 取到键盘的bounds
    CGRect keyboardBounds = [[userInfo objectForKey:UIKeyboardFrameEndUserInfoKey] CGRectValue];
    CGFloat keyboardHeight = keyboardBounds.size.height;
    self.bottomView.transform = CGAffineTransformMakeTranslation(0, -keyboardHeight);

}

- (void)keyboardDidShow:(NSNotification *)notification {

    NSDictionary *userInfo = notification.userInfo;
    CGRect keyboardBounds = [[userInfo objectForKey:UIKeyboardFrameEndUserInfoKey] CGRectValue];
    CGFloat keyboardHeight = keyboardBounds.size.height;
    // 屏幕高-(导航栏高+状态栏高)-键盘高-键盘上bottomView高
    CGFloat textViewHeight = kScreenHeight - (kNavigationBarHeight + kStatusHeight) - keyboardHeight - 30;
    self.textViewHeight.constant = textViewHeight;

}

- (void)keyboardWillHide:(NSNotification *)notification {

    self.hideKeyboardButton.hidden = YES;

    CGFloat textViewHeight = kScreenHeight - (kNavigationBarHeight + kStatusHeight) - 30;
    self.textViewHeight.constant = textViewHeight;

    self.bottomView.transform = CGAffineTransformIdentity;

}
```

### 3. 让Status Bar上显示一个旋转的`ActivityIndicator`

```objective-c
[UIApplication sharedApplication].networkActivityIndicatorVisible = YES;
```

### 4. `[NSDate date]`和实际时间相差8小时

```objective-c
NSDate *date = [NSDate date];
NSTimeZone *zone = [NSTimeZone systemTimeZone];
NSInteger interval = [zone secondsFromGMTForDate:date];
NSDate *localDate = [date dateByAddingTimeInterval:interval];
NSLog(@"localDate: %@", localDate);
```

### 5. `UICollectionView`中加入`UIRefreshControl`
先把`UIRefreshControl`初始化出来，然后`addSubview`上去

```objective-c
UIRefreshControl *refreshControl = [[UIRefreshControl alloc] init];
[refreshControl addTarget:self action:@selector(refresh:) forControlEvents:UIControlEventValueChanged];
[collectionView addSubview:refreshControl];
```

设置`collectionView`的`alwaysBouncesVertical`为`YES`

```objective-c
collectionView.alwaysBounceVertical = YES;
```

### 6. 判断`UITableView`的滑动方向，向下滑动就隐藏`UITabBar`，向上滑动就显示`UITabBar`

```objective-c
// 先在上面定义一个CGFloat类型的_oldPanOffsetY用于记录上一次滑动的y

- (void)scrollViewWillBeginDragging:(UIScrollView *)scrollView{
    if (scrollView == self.tableView) {
        _oldPanOffsetY = [scrollView.panGestureRecognizer translationInView:scrollView.superview].y;
    }
}

- (void)scrollViewDidEndDragging:(UIScrollView *)scrollView willDecelerate:(BOOL)decelerate{
    _oldPanOffsetY = 0;
}

- (void)scrollViewDidScroll:(UIScrollView *)scrollView{
    if (scrollView.contentSize.height <= CGRectGetHeight(scrollView.bounds)-50) {
        [self hideTabBar:NO];
        return;
    }else if (scrollView.panGestureRecognizer.state == UIGestureRecognizerStateChanged){
        CGFloat nowPanOffsetY = [scrollView.panGestureRecognizer translationInView:scrollView.superview].y;
        CGFloat diffPanOffsetY = nowPanOffsetY - _oldPanOffsetY;
        CGFloat contentOffsetY = scrollView.contentOffset.y;
        if (ABS(diffPanOffsetY) > 50.f) {
            [self hideTabBar:(diffPanOffsetY < 0.f && contentOffsetY > 0)];
            _oldPanOffsetY = nowPanOffsetY;
        }
    }
}

- (void)hideTabBar:(BOOL)hide {
    if (hide) {
        [self setTabBarHidden:YES animated:YES];
    } else {
        [self setTabBarHidden:NO animated:YES];
    }
}

- (void)setTabBarHidden:(BOOL)hidden animated:(BOOL)animated {

    void (^block)() = ^{
        CGSize viewSize = self.tabBarController.view.bounds.size;
        CGFloat tabBarStartingY = viewSize.height;
        CGFloat contentViewHeight = viewSize.height;
        CGFloat tabBarHeight = CGRectGetHeight([[self.tabBarController tabBar] frame]);
        if (!tabBarHeight) {
            tabBarHeight = 49;
        }

        if (!hidden) {
            tabBarStartingY = viewSize.height - tabBarHeight;
            if (![[self.tabBarController tabBar] isTranslucent]) {
                contentViewHeight -= 49;
            }
            [[self.tabBarController tabBar] setHidden:NO];
        }

        [[self.tabBarController tabBar] setFrame:CGRectMake(0, tabBarStartingY, viewSize.width, tabBarHeight)];
    };

    void (^completion)(BOOL) = ^(BOOL finished){
        if (hidden) {
            [[self.tabBarController tabBar] setHidden:YES];
        }
    };

    if (animated) {
        [UIView animateWithDuration:0.10 animations:block completion:completion];
    } else {
        block();
        completion(YES);
    }
}
```

### 7. 把`textView`文字长按出现的选项从英文改为中文

> 在info.plist中加上一个Localization的Array，然后把Array中的item0改为Chinese(Simplified)就可以实现

![示意图](http://7xr0k3.com1.z0.glb.clouddn.com/iOS-Develop-Tips/Snip20160419_5.png)

### 8. 开启`finder`显示隐藏文件

```shell

$ defaults write com.apple.finder AppleShowAllFiles -bool true     // 开启查看隐藏文件

$ defaults write com.apple.finder AppleShowAllFiles -bool false     // 关闭查看隐藏文件

```
