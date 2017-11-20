---
title: STOMP-over-WebSocket文档
date: 2016-08-17 15:40:36
categories: 前端开发
tags: [Stomp, websocket, javascript, html5]
---

## 前言
前两天整理了`websocket`的资料，今天就把上次没说完的`Stomp.js`好好说一说~  
Stomp Over Webscoket参考文档：http://jmesnil.net/stomp-websocket/doc/  
本文为参考文档的部分翻译，技术不佳，如有失误请指正。

## 什么是Stomp
> STOMP即Simple (or Streaming) Text Orientated Messaging Protocol，简单(流)文本定向消息协议，它提供了一个可互操作的连接格式，允许STOMP客户端与任意STOMP消息代理（Broker）进行交互。STOMP协议由于设计简单，易于开发客户端，因此在多种语言和多种平台上得到广泛地应用。

## 协议支持
该库支持多种版本的STOMP协议：
* [STOMP 1.0][1]
* [STOMP 1.1][2](包含 heart-beating)

## 下载STOMP.JS
你可以下载 [stomp.js][3] 并在你自己的WEB应用程序中使用。
提供了[多种版本][4]也可以直接用于生产。
这个js文件由CoffeeScript文件构建，请查看[Contribute][5]部分下载源码或浏览 [annote source code][6]

## 服务端要求
这个库不是单纯的Stomp 客户端。它旨在WebSockets上运行而不是TCP。基本上，WebSocket协议需要在浏览器客户端和服务端之间进行握手，确保浏览器的“same-origin”（同源）安全模型仍然有效。

这意味着该库不能连接常规的STOMP 代理，因为Websocket初始化的握手不是STOMP协议的一部分，他们不能理解从而会拒绝连接。

有一些正在进行的工作添加了WebSocket支持STOMP代理，从而他们可以在WebSocket协议上接受STOMP连接。

### HornetQ
HornetQ是由Red Hat and JBoss创立的开源消息系统.

要使HornetQ支持STOMP Over WebSocket，下载最新版本并按照下列步骤执行：

    $ cd hornetq-x.y.z/examples/jms/stomp-websockets
    $ mvn clean install
    ...
    INFO: HQ221020: Started Netty Acceptor version 3.6.2.Final-c0d783c         localhost:61614 for STOMP_WS protocol
    Apr 15, 2013 1:15:33 PM org.hornetq.core.server.impl.HornetQServerImpl$SharedStoreLiveActivation run
    INFO: HQ221007: Server is now live
    Apr 15, 2013 1:15:33 PM org.hornetq.core.server.impl.HornetQServerImpl start
    INFO: HQ221001: HornetQ Server version 2.3.0.CR2 (black'n'yellow2, 123) [c9e29e45-a5bd-11e2-976a-b3fef7ceb5df]
    
此时HornetQ已经开启了，并且61614在端口监听STOMP over WebSocket
它从URL为ws://localhost:61614/stomp 接受WebSocket的连接。

[配置文档][7]

### ActiveMQ
[配置文档][8]

### ActiveMQ Apollo
[配置文档][9]

### RabbitMQ
[配置文档][10]

### Stilts & Torquebox
[Stilts][11] 是一个STOMP原生的消息框架。

[TorqueBox][12] 使用Stilts去提供它的[Websockets and STOMP stack][13]。

## Stomp API
### STOMP 帧（Frame）
STOMP Over WebSocket 提供了一个直接从`Stomp Frame`映射到 Javascript 对象的方式。  
`Stomp Frame`帧格式如下：

|Property|Type|Notes
| ----------- |:--------:| -----:|
|command|String	|name of the frame ("CONNECT", "SEND", etc.)
|headers|JavaScript object	
|body|String|	

`command`和`headers`属性始终会被定义，不过当这个`frame`没有头部时，`headers`可以为空。若这个`frame`没有`body`，`body`的值可以为`null`。

### 创建`STOMP`客户端
#### 在web浏览器中使用普通的Web Socket
STOMP javascript 客户端会使用`ws://`的URL与STOMP 服务端进行交互。

为了创建一个STOMP客户端js对象，你需要使用`Stomp.client(url)`，而这个URL连接着服务端的WebSocket的代理：

    var url = "ws://localhost:61614/stomp";
    var client = Stomp.client(url);

`Stomp.client(url, protocols)`也可以用来覆盖默认的`subprotocols`。第二个参数可以是一个字符串或一个字符串数组去指定多个`subprotocols`。

#### 在web浏览器中使用定制的WebSocket
浏览器提供了不同的WebSocket的协议，一些老的浏览器不支持WebSocket的脚本或者使用别的名字。默认下，`stomp.js`会使用浏览器原生的`WebSocket class`去创建WebSocket。

但是利用`Stomp.over(ws)`这个方法可以使用其他类型的WebSockets。这个方法得到一个满足WebSocket定义的对象。

例如，可以使用由`SockJS`实现的Websocket：

    <script src="http://cdn.sockjs.org/sockjs-0.3.min.js"></script>
    <script>
        // use SockJS implementation instead of the browser's native implementation
        var ws = new SockJS(url);
        var client = Stomp.over(ws);
        [...]
    </script>

如果使用原生的Websockets就使用`Stomp.client(url)`，如果需要使用其他类型的Websocket（例如由SockJS包装的Websocket）就使用`Stomp.over(ws)`。

除了初始化有差别，Stomp API在这两种方式下是相同的。

#### 在`node.js`程序中
通过`stompjs npm package`同样也可以在`node.js`程序中使用这个库。

    $ npm install stompjs

在node.js`app`中, `require`这个模块:

    var Stomp = require('stompjs');

为了与建立在TCP socket的STOMP-broker连接，使用`Stomp.overTCP(host, port)`方法。

    var client = Stomp.overTCP('localhost', 61613);

为了与建立在Web Socket的STOMP broker连接，使用`Stomp.overWS(url)`方法。

    var client = Stomp.overWS('ws://localhost:61614/stomp');

除了初始化不同，无论是浏览器还是node.js环境下，Stomp API都是相同的。

### 连接服务端
一旦Stomp 客户端建立了，必须调用它的`connect()`方法去连接，从而Stomp服务端进行验证。这个方法需要两个参数，用户的登录和密码凭证。

这种情况下，客户端会使用Websocket打开连接，并发送一个`CONNECT frame`。

这个连接是异步进行的：你不能保证当这个方法返回时是有效连接的。为了知道连接的结果，你需要一个回调函数。

    var connect_callback = function() {
        // called back after the client is connected and authenticated to the STOMP server
    };

但是如果连接失败会发生什么呢？`connect()`方法接受一个可选的参数(`error_callback`)，当客户端不能连接上服务端时，这个回调函数`error_callback`会被调用，该函数的参数为对应的错误对象。

    var error_callback = function(error) {
        // display the error's message header:
        alert(error.headers.message);
    };
    
在大多数情况下，`connect()`方法可接受不同数量的参数来提供简单的API：

    client.connect(login, passcode, connectCallback);
    client.connect(login, passcode, connectCallback, errorCallback);
    client.connect(login, passcode, connectCallback, errorCallback, host);

`login`和`passcode`是strings，`connectCallback`和`errorCallback`则是functions。（有些brokers（代理）还需要传递一个`host`（String类型）参数。）

如果你需要附加一个`headers`头部，`connect`方法还接受其他两种形式的参数：

    client.connect(headers, connectCallback);
    client.connect(headers, connectCallback, errorCallback);
    
`header`是`map`形式，`connectCallback`和`errorCallback`为functions。

需要注意：如果你使用上述这种方式，你需要自行在`headers`添加`login`,`passcode`（甚至`host`）：

    var headers = {
        login: 'mylogin',
        passcode: 'mypasscode',
        // additional header
        'client-id': 'my-client-id'
    };
    client.connect(headers, connectCallback);

断开连接时，调用`disconnect`方法，这个方法也是异步的，当断开成功后会接收一个额外的回调函数的参数。如下所示。
    client.disconnect(function() {
        alert("See you next time!");
    };

当客户端与服务端断开连接，就不会再发送或接收消息了。

### Heart-beating
如果STOMP broker(代理)接收STOMP 1.1版本的帧，`heart-beating`是默认启用的。`heart-beating`也就是频率，`incoming`是接收频率，`outgoing`是发送频率。

通过改变`incoming`和`outgoing`可以更改客户端的`heart-beating`(默认为10000ms)：

    client.heartbeat.outgoing = 20000; 
    // client will send heartbeats every 20000ms
    client.heartbeat.incoming = 0;
    // client does not want to receive heartbeats
    // from the server
    

`heart-beating`是利用`window.setInterval()`去规律地发送`heart-beats`或者检查服务端的`heart-beats`。

### 发送消息

当客户端与服务端连接成功后，可以调用`send()`来发送STOMP消息。这个方法必须有一个参数，用来描述对应的STOMP的目的地。另外可以有两个可选的参数：`headers`，`object`类型包含额外的信息头部；`body`，一个String类型的参数。

    client.send("/queue/test", {priority: 9}, "Hello, STOMP");

client会发送一个STOMP发送帧给`/queue/test`，这个帧包含一个设置了`priority`为9的`header`和内容为“Hello, STOMP”的`body`。

如果你想发送一个有`body`的信息，也必须传递`headers`参数。如果没有`headers`需要传递，那么就传`{}`即可，如下所示：

    client.send(destination, {}, body);
    

### 订阅（Subscribe）和接收（receive）消息
为了在浏览器中接收消息，STOMP客户端必须先订阅一个目的地`destination`。

你可以使用`subscribe()`去订阅。这个方法有2个必需的参数：目的地(`destination`)，回调函数(`callback`)；还有一个可选的参数`headers`。其中`destination`是String类型，对应目的地，回调函数是伴随着一个参数的`function`类型。

    var subscription = client.subscribe("/queue/test", callback);

`subscribe()`方法返回一个`object`，这个`object`包含一个`id`属性，对应这个这个客户端的订阅ID。而`unsubscribe()`可以用来取消客户端对这个目的地`destination`的订阅。

默认情况下，如果没有在`headers`额外添加，这个库会默认构建一个独一无二的`ID`。在传递`headers`这个参数时，可以使用你自己的`ID`:

    var mysubid = '...';
    var subscription = client.subscribe(destination, callback, { id: mysubid });

这个客户端会向服务端发送一个STOMP订阅帧（`SUBSCRIBE frame`）并注册回调事件。每次服务端向客户端发送消息时，客户端都会轮流调用回调函数，参数为对应消息的STOMP帧对象（`Frame object`）。如下所示：

    callback = function(message) {
        // called when the client receives a STOMP message from the server
        if (message.body) {
            alert("got message with body " + message.body)
        } else {
            alert("got empty message");
        }
    });
    
`subscribe()`方法，接受一个可选的`headers`参数用来标识附加的头部。

    var headers = {ack: 'client', 'selector': "location = 'Europe'"};
  
    client.subscribe("/queue/test", message_callback, headers);


这个客户端指定了它会确认接收的信息，只接收符合这个`selector : location = 'Europe'`的消息。

如果想让客户端订阅多个目的地，你可以在接收所有信息的时候调用相同的回调函数：

    onmessage = function(message) {
        // called every time the client receives a message
    }
    var sub1 = client.subscribe("queue/test", onmessage);
    var sub2 = client.subscribe("queue/another", onmessage)


如果要中止接收消息，客户端可以在`subscribe()`返回的`object`对象调用`unsubscribe()`来结束接收。

    var subscription = client.subscribe(...);
  
    ...
  
    subscription.unsubscribe();

### 支持JSON
STOMP消息的`body`必须为字符串。如果你需要发送/接收`JSON`对象，你可以使用`JSON.stringify()`和`JSON.parse()`去转换JSON对象。

    var quote = {symbol: 'APPL', value: 195.46};
    client.send("/topic/stocks", {}, JSON.stringify(quote));

    client.subcribe("/topic/stocks", function(message) {
        var quote = JSON.parse(message.body);
        alert(quote.symbol + " is at " + quote.value);
    };

### Acknowledgment(确认)
默认情况，在消息发送给客户端之前，服务端会自动确认（`acknowledged`）。

客户端可以选择通过订阅一个目的地时设置一个`ack header`为`client`或`client-individual`来处理消息确认。

在下面这个例子，客户端必须调用`message.ack()`来通知客户端它已经接收了消息。

    var subscription = client.subscribe("/queue/test",
        function(message) {
            // do something with the message
            ...
            // and acknowledge it
            message.ack();
        },
        {ack: 'client'}
    );

`ack()`接受`headers`参数用来附加确认消息。例如，将消息作为事务(transaction)的一部分，当要求接收消息时其实代理（broker）已经将`ACK STOMP frame`处理了。

    var tx = client.begin();
    message.ack({ transaction: tx.id, receipt: 'my-receipt' });
    tx.commit();

`nack()`也可以用来通知STOMP 1.1.brokers（代理）：客户端不能消费这个消息。与`ack()`方法的参数相同。

### 事务(Transactions)
可以在将消息的发送和确认接收放在一个事务中。

客户端调用自身的`begin()`方法就可以开始启动事务了，`begin()`有一个可选的参数`transaction`，一个唯一的可标识事务的字符串。如果没有传递这个参数，那么库会自动构建一个。

这个方法会返回一个object。这个对象有一个`id`属性对应这个事务的ID，还有两个方法：
`commit()`提交事务
`abort()`中止事务

在一个事务中，客户端可以在发送/接受消息时指定transaction id来设置transaction。

    // start the transaction
    var tx = client.begin();
    // send the message in a transaction
    client.send("/queue/test", {transaction: tx.id}, "message in a transaction");
    // commit the transaction to effectively send the message
    tx.commit();

如果你在调用`send()`方法发送消息的时候忘记添加transction header，那么这不会称为事务的一部分，这个消息会直接发送，不会等到事务完成后才发送。

    var txid = "unique_transaction_identifier";
    // start the transaction
    var tx = client.begin();
    // oops! send the message outside the transaction
    client.send("/queue/test", {}, "I thought I was in a transaction!");
    tx.abort(); // Too late! the message has been sent
  
### 调试（Debug）

有一些测试代码能有助于你知道库发送或接收的是什么，从而来调试程序。

客户端可以将其`debug`属性设置为一个函数，传递一个字符串参数去观察库所有的debug语句。

    client.debug = function(str) {
        // append the debug log to a #debug div somewhere in the page using JQuery:
        $("#debug").append(str + "\n");
    };

默认情况，debug消息会被记录在在浏览器的控制台。


  [1]: http://stomp.github.io/stomp-specification-1.0.html
  [2]: http://stomp.github.io/stomp-specification-1.1.html
  [3]: https://raw.githubusercontent.com/jmesnil/stomp-websocket/master/lib/stomp.js
  [4]: https://raw.githubusercontent.com/jmesnil/stomp-websocket/master/lib/stomp.min.js
  [5]: http://jmesnil.net/stomp-websocket/doc/#contribute
  [6]: http://jmesnil.net/stomp-websocket/doc/stomp.html
  [7]: http://docs.jboss.org/hornetq/2.3.0.CR2/docs/user-manual/html/interoperability.html#stomp.websockets
  [8]: http://activemq.apache.org/websockets.html
  [9]: http://activemq.apache.org/apollo/documentation/user-manual.html#WebSocket_Transports
  [10]: RabbitMQ
  [11]: http://stilts.projectodd.org/
  [12]: http://torquebox.org/
  [13]: http://torquebox.org/documentation/2.1.2/stomp.html