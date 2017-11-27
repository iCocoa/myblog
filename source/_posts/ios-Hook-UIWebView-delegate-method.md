---
title: iOS Hook WebView 的代理方法
date: 2016-10-18 20:09:01
tags: iOS
---
最近在使用 DCloud 团队开发的 HTML5+ 来开发 Hybrid 应用。它们使用 iOS 系统原生 UIWebView 和 WKWebView 来加载资源。我们的应用有个需求，就是在 webview 加载完页面或者加载页面之前加入一些东西。比如：加载完页面后，根据 HTML 的 title 标签来设置导航栏标题。

原生想要插手页面加载周期，只能靠代理方法。但是因为没法修改源码，所以只能找其它办法。主要思路是：使用 Method Swizzle 找出代理对象然后再换掉代理方法实现。

以 UIWebView 为例，具体操作如下：

**第一步，通过交换 setDelegate 的实现，找到目标代理对象所属的类；**

```
UIWebView+Intercepter.m

- (void)p_setDelegate:(id<UIWebViewDelegate>)delegate
{
    [self p_setDelegate:delegate];
    Class delegateClass = [self.delegate class];
    // 进一步交换 delegateClass 的代理方法
    [UIWebViewDelegateHook exchangeUIWebViewDelegateMethod:delegateClass];
}

#pragma mark - Method Swizzling
+ (void)load {
    static dispatch_once_t onceToken;
    dispatch_once(&onceToken, ^{
        Class class = [super class];
        
        // When swizzling a class method, use the following:
        // Class class = object_getClass((id)self);
        
        SEL originalSelector = @selector(setDelegate:);
        SEL swizzledSelector = @selector(p_setDelegate:);
        
        Method originalMethod = class_getInstanceMethod(class, originalSelector);
        Method swizzledMethod = class_getInstanceMethod(class, swizzledSelector);
        
        BOOL didAddMethod =
        class_addMethod(class,
                        originalSelector,
                        method_getImplementation(swizzledMethod),
                        method_getTypeEncoding(swizzledMethod));
        
        if (didAddMethod) {
            class_replaceMethod(class,
                                swizzledSelector,
                                method_getImplementation(originalMethod),
                                method_getTypeEncoding(originalMethod));
        } else {
            method_exchangeImplementations(originalMethod, swizzledMethod);
        }
        
    });
}
```

**第二步，把目标代理对象所属类的代理方法实现换成我们自己写的方法实现。**

```
UIWebViewDelegateHook.m

+ (void)exchangeUIWebViewDelegateMethod:(Class)aClass
{
    p_exchangeMethod(aClass, @selector(webViewDidStartLoad:), [self class], @selector(replaced_webViewDidStartLoad:));
    p_exchangeMethod(aClass, @selector(webViewDidFinishLoad:), [self class], @selector(replaced_webViewDidFinishLoad:));
    p_exchangeMethod(aClass, @selector(webView:didFailLoadWithError:), [self class], @selector(replaced_webView:didFailLoadWithError:));
    p_exchangeMethod(aClass, @selector(webView:shouldStartLoadWithRequest:navigationType:), [self class], @selector(replaced_webView:shouldStartLoadWithRequest:navigationType:));
}

- (BOOL)replaced_webView:(UIWebView *)webView shouldStartLoadWithRequest:(NSURLRequest *)request navigationType:(UIWebViewNavigationType)navigationType
{
    NSLog(@"shouldStartLoadWithRequest");
    return [self replaced_webView:webView shouldStartLoadWithRequest:request navigationType:navigationType];
}

- (void)replaced_webViewDidStartLoad:(UIWebView *)webView
{
    NSLog(@"webViewDidStartLoad");
    [self replaced_webViewDidStartLoad:webView];
}

- (void)replaced_webViewDidFinishLoad:(UIWebView *)webView
{
    NSLog(@"webViewDidFinishLoad");
    [self replaced_webViewDidFinishLoad:webView];
    NSString *pageTitle = [webView stringByEvaluatingJavaScriptFromString:@"document.title"];
    [[NSNotificationCenter defaultCenter] postNotificationName:PAFWebViewPageTitleDidChange object:self userInfo:@{PAFPageTitle:pageTitle}];
}

- (void)replaced_webView:(UIWebView *)webView didFailLoadWithError:(NSError *)error{
    NSLog(@"didFailLoadWithError");
    [self replaced_webView:webView didFailLoadWithError:error];
}
```

**注意防止 setDelegate 被调用多次，这样会导致方法又被换掉。**


```
static void p_exchangeMethod(Class originalClass, SEL originalSel, Class replacedClass, SEL replacedSel)
{
    static NSMutableArray *classList = nil;
    if (classList == nil) {
        classList = [NSMutableArray array];
    }
    NSString *className = [NSString stringWithFormat:@"%@__%@", NSStringFromClass(originalClass), NSStringFromSelector(originalSel)];
    for (NSString *item in classList) {
        // 防止 setDelegate 方法被调用多次，导致代理方法又被换掉
        if ([className isEqualToString:item]) {
            return;
        }
    }
    
    [classList addObject:className];
    
    Method originalMethod = class_getInstanceMethod(originalClass, originalSel);
    assert(originalMethod);
    
    Method replacedMethod = class_getInstanceMethod(replacedClass, replacedSel);
    assert(replacedMethod);
    IMP replacedMethodIMP = method_getImplementation(replacedMethod);
    
    BOOL didAddMethod =
    class_addMethod(originalClass,
                    replacedSel,
                    replacedMethodIMP,
                    method_getTypeEncoding(replacedMethod));
    
    if (didAddMethod) {
        NSLog(@"class_addMethod failed --> (%@)", NSStringFromSelector(replacedSel));
    } else {
        NSLog(@"class_addMethod succeed --> (%@)", NSStringFromSelector(replacedSel));
    }
    
    Method newMethod = class_getInstanceMethod(originalClass, replacedSel);
    
    method_exchangeImplementations(originalMethod, newMethod);
}
```

参考 ： [Objective-C的hook方案（一）: Method Swizzling](http://blog.csdn.net/yiyaaixuexi/article/details/9374411)
