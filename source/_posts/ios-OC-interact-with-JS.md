---
title: OC 与 JS 通信的几种方式
date: 2017-06-23 22:57:25
tags: iOS
---

1. 在代理方法中拦截协议
2. 使用 JavaScriptCore
3. WKWebView 的 WKScriptMessagehandler
4. 使用 NSURLProtocol 拦截请求
5. 使用第三方库 WebViewJavascriptBridge
6. 使用 WebSocket

这里只介绍第 6 种，其它的相关资料网上有很多。

使用 WebSocket 的方式需要在应用内起一个 websocket server 服务（有很多第三方的 websocket server 库），html 页面通过 Websocket 连接到服务，接着就是发送消息了，剩下的就跟代理方法拦截协议类似。

```
// OC code, 以 PocketSocket 这个库为例
_socketServer = [PSWebSocketServer serverWithHost:nil port:9001];
_socketServer.delegate = self;
_socketServer.delegateQueue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);
[_socketServer start];
    
#pragma mark - PSWebSocketServerDelegate
- (void)serverDidStart:(PSWebSocketServer *)server 
{
    NSLog(@"Server did start…");
}

- (void)serverDidStop:(PSWebSocketServer *)server 
{
    NSLog(@"Server did stop…");
}

- (BOOL)server:(PSWebSocketServer *)server acceptWebSocketWithRequest:(NSURLRequest *)request 
{
    NSLog(@"Server should accept request: %@", request);
    return YES;
}

- (void)server:(PSWebSocketServer *)server webSocket:(PSWebSocket *)webSocket didReceiveMessage:(id)message 
{
    // 在这里拦截
    NSLog(@"Server websocket did receive message: %@", message);
    NSString *text = message;
    NSURL *url = [NSURL URLWithString:text];
    if ([url.scheme isEqualToString:@"camera"]) {
        ......
    }
}

- (void)server:(PSWebSocketServer *)server webSocketDidOpen:(PSWebSocket *)webSocket
{
    NSLog(@"Server websocket did open");
}

- (void)server:(PSWebSocketServer *)server didFailWithError:(NSError *)error
{
    NSLog(@"Server did fail with error: %@", error);
}

- (void)server:(PSWebSocketServer *)server webSocket:(PSWebSocket *)webSocket didCloseWithCode:(NSInteger)code reason:(NSString *)reason wasClean:(BOOL)wasClean 
{
    NSLog(@"Server websocket did close with code: %@, reason: %@, wasClean: %@", @(code), reason, @(wasClean));
}

- (void)server:(PSWebSocketServer *)server webSocket:(PSWebSocket *)webSocket didFailWithError:(NSError *)error 
{
    NSLog(@"Server websocket did fail with error: %@", error);
}	
```


```
// JS code 
var wsServer = 'ws://localhost:9001'; 
var websocket = new WebSocket(wsServer); 
websocket.onopen = function (evt) { onOpen(evt) }; 
websocket.onclose = function (evt) { onClose(evt) }; 
websocket.onmessage = function (evt) { onMessage(evt) }; 
websocket.onerror = function (evt) { onError(evt) };

function onOpen(evt) { 
	console.log("Connected to WebSocket server."); 
}
function onClose(evt) { 
	console.log("Disconnected"); 
} 
function onMessage(evt) {
	console.log("Recieve data: " + evt.data); 
} 
function onError(evt) { 
	console.log('Error occured: ' + evt.data); 
}

// 发送消息
websocket.send("camera://openCamera?index=1&quality=high&callback=callbackFunction);
function callbackFunction(data){
	
}
```

[javascriptcore](https://www.raywenderlich.com/124075/javascriptcore-tutorial)

[WKWebView](http://nshipster.com/wkwebkit/)

[自定义 NSURLProtocol](https://developer.apple.com/library/content/samplecode/CustomHTTPProtocol/Introduction/Intro.html)

[WebViewJavascriptBridge](https://github.com/marcuswestin/WebViewJavascriptBridge)

[WebSocket](https://zh.wikipedia.org/wiki/WebSocket)
