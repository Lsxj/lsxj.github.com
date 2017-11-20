---
title: websocket资料总结
date: 2016-08-14 19:29:43
categories: 前端开发
tags: [websocket, html5, javascript]
---
## 前言
之前的项目中使用了`websocket`、`socketJS`，趁此机会将一些知识点及资料进行整理总结。正所谓温故而知新~

## websocket是什么
按照惯例，在使用之前，先了解一下概念。
> `websocket`是在`html5`中提供了一种浏览器和服务器间进行**全双工**通讯的网络技术。

浏览器向服务端发送一个请求，通过报文头部`Upgrade`来表明需要从`HTTP`切换至`Websocket`协议。

    GET ws://echo.websocket.org/?encoding=text HTTP/1.1
    Origin: http://websocket.org
    Cookie: __utma=99as
    Connection: Upgrade
    Host: echo.websocket.org
    Sec-WebSocket-Key: uRovscZjNol/umbTt5uKmw==
    Upgrade: websocket
    Sec-WebSocket-Version: 13

如果服务端理解websocket协议，它也是通过报文`Upgrade`从HTTP转换为Websocket协议。

    HTTP/1.1 101 WebSocket Protocol Handshake
    Date: Fri, 10 Feb 2012 17:38:18 GMT
    Connection: Upgrade
    Server: Kaazing Gateway
    Upgrade: WebSocket
    Access-Control-Allow-Origin: http://websocket.org
    Access-Control-Allow-Credentials: true
    Sec-WebSocket-Accept: rLHCkw/SKsO9GAH/ZSFhBATDKrU=
    Access-Control-Allow-Headers: content-type

这个时候就建立起了websocket连接，基于TCP/IP。使用端口与HTTP(80)和HTTPS(443)一样。
    
## 为什么要用websocket？
知道了什么是websocket，那么为什么要使用websocket呢？除了websocket之外，浏览器进行即时通信的方式还有以下几种：

* 定期查询
每隔一个时间段就向服务器发送一个请求，请求服务器的最新数据再进行更新。但这样做的后果就是浪费大量流量，对服务端造成了巨大压力。

* Comet
基于http长连接的“服务器推”技术。客户端与服务器端保持一个长连接，只有客户端需要的数据更新时，服务器才主动将数据推送给客户端。有两种形式：
    * 基于`Ajax`的长轮询（long-polling）方式
    浏览器发出XMLHttpRequest请求，服务器端接收到请求后，会阻塞请求直到有数据或者超时才返回，浏览器在处理请求返回信息（超时或有效数据）后再次发出请求，重新建立连接。在此期间服务器端可能已经有新的数据到达，服务器会选择把数据保存，直到重新建立连接，浏览器会把所有数据一次性取回。
    * 基于`Iframe`及`htmlfile`的流（http streaming）方式
    通常的做法是在页面中嵌入一个隐藏的iframe,然后让这个iframe的src属性指向我们请求的一个服务端地址，并且为了数据更新，我们将页面上数据更新操作封装为一个js函数，将函数名当做参数传递到这个地址当中。服务端收到请求后解析地址取出参数（客户端js函数调用名），每当有数据更新的时候，返回对客户端函数的调用，并且将要跟新的数据以js函数的参数填入到返回内容当中，例如返回“<script type="text/javascript">update("data")</script>”这样一个字符串，意味着以data为参数调用客户端update函数进行客户端view更新。
当应用程序有高吞吐量的需求，Comet的长轮询就不适合了。

* SSE
SSE(服务端推送事件)是一种允许服务端向客户端推送新数据的HTML5技术。与websocket相比，WebSocket相较SSE最大的优势在于它是双向交流的，这意味向服务端发送数据就像从服务端接收数据一样简单。用SSE时，一般通过一个独立的Ajax请求从客户端向服务端传送数据。相对于WebSocket，这样使用Ajax会增加开销，但也就多一点点而已。

相比于间断的轮询或长轮询来模拟全双工连接的解决方式，Websocket极大的减少了不必要的网络流量和延迟。除此之外，Websocket-based的应用减轻了服务器的负担，让现有的机器能支持更多的并发连接。如下图所示：

![clipboard.png](http://oao50r2ex.bkt.clouddn.com/image/blog0814_2.png)


## 如何使用websocket
【以下例子来源于http://www.websocket.org/aboutwebsocket.html】
只需要创建一个新的Websocket实例，提供一个URL，这个URL表示的是你希望连接的那个end-point。如下所示。
需要注意的是: `ws://`和`wss://`的前缀表示了Websokcet和安全协议的Websocket连接。
    
    var myWebsocket = new Websocket("ws://www.websocket.org");

在连接到一个端点发送消息之前，你可以将一系列的事件监听器来处理连接的生命周期的每个阶段。如下所示：

    myWebSocket.onopen = function(evt) { 
        alert("Connection open ..."); 
    };
    myWebSocket.onmessage = function(evt) { 
        alert( "Received Message: " + evt.data); 
    };
    myWebSocket.onclose = function(evt) { 
        alert("Connection closed."); 
    };

向服务端发送信息，只需要简单的`send`并提供你希望传递的内容。发送信息后，`close`终止连接。如下所示：
    
    myWebSocket.send("Hello WebSockets!");
    myWebSocket.close();

## socketJS
我们都知道，webscoket是HTML5的新玩意，那么兼容性方面，如下图所示：

![clipboard.png](http://oao50r2ex.bkt.clouddn.com/image/blog0814_1.png)

可以看出IE8以及Android 4.3是不支持的。这个时候，我们就可以来看看`socketJS`的优势了。

> SockJS is a browser JavaScript library that provides a WebSocket-like object. SockJS gives you a coherent, cross-browser, Javascript API which creates a low latency, full duplex, cross-domain communication channel between the browser and the web server.
`socketJS`的一大好处在于**提供了浏览器兼容性**。优先使用原生`websocket`，如果在不支持`websocket`的浏览器中，会自动降为轮询的方式。
除此之外，`spring`也对`socketJS`提供了支持。如果代码中添加了`withSockJS()`如下，服务器也会自动降级为轮询。

    registry.addEndpoint("/coordination").withSockJS();  

## 如何使用socketJS

    <script src="//cdn.jsdelivr.net/sockjs/1.0.0/sockjs.min.js"></script>

    var sock = new SockJS('/coordination');  
    sock.onopen = function() {
        console.log('open');
    };
    sock.onmessage = function(e) {
        console.log('message', e.data);
    };
    sock.onclose = function() {
        console.log('close');
    };
    sock.send('test');
    sock.close();

更多内容，可查看github地址：https://github.com/sockjs/sockjs-client

## 什么是Stomp
通过以上部分我们可以知道`websocket`的优势，兼容性的问题`socketJS`也帮我们解决了。不过这个时候，我还要安利一个好东西，也就是`Stomp`。

> STOMP is a simple text-orientated messaging protocol. It defines an interoperable wire format so that any of the available STOMP clients can communicate with any STOMP message broker to provide easy and widespread messaging interoperability among languages and platforms (the STOMP web site has a list of STOMP client and server implementations.

具体内容，可查看官网：http://jmesnil.net/stomp-websocket/doc/。或者等我下一篇文章详谈吧~

参考资料：
 1. [Spring WebSocket教程（一）][1]
 2. [WebSocket详解（一）：初步认识WebSocket技术][2]
 3. [STOMP Over WebSocket][3]
 4. [sockjs/sockjs-client][4]
 5. [Spring websocket 使用][5]
 6. [Web端即时通讯技术盘点：短轮询、Comet、Websocket、SSE][6]
 7. [websocket官网About HTML5 WebSocket][7]


  [1]: http://blog.csdn.net/xjyzxx/article/details/24182677
  [2]: http://www.52im.net/forum.php?mod=viewthread&tid=331&ctid=15
  [3]: http://jmesnil.net/stomp-websocket/doc/
  [4]: https://github.com/sockjs/sockjs-client
  [5]: http://blog.csdn.net/yxb19870428vv/article/details/41495543
  [6]: http://www.tuicool.com/articles/uINBfiZ
  [7]: http://www.websocket.org/aboutwebsocket.html