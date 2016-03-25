#iOS开发中的一些tips
持续更新自己开发中遇到的一些小问题，把这些记录下来

目录：

1. 设置`UITextView`的`placeHolder`
2. 键盘弹出和消失时让工具栏紧贴键盘
3. 让`Status Bar`上显示一个旋转的`ActivityIndicator`
4. `[NSDate date]`和实际时间相差8小时
5. `UICollectionView`中加入`UIRefreshControl`

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