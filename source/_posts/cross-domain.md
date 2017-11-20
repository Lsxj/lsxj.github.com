---
title: 关于跨域
date: 2016-09-14 17:57:56
categories: 前端开发
tags: [javascript, 跨域]
---

# 前言
转眼就是秋招季啦。经历了几场笔试面试，屡次被问到关于如何实现跨域。老实说，之前都是纸上谈兵，也没有项目需要跨域，甚至觉得这个东西没什么意义。直到今天项目中遇到了跨域问题，看了不少资料才理解跨域的普遍性和意义。特写此篇文章整理自己所得。


# 什么是跨域
一般来说，如果你在开发中需要进行跨域操作（从一个非同源网站发送请求获取数据），一般而言，你在浏览器控制台看到的结果为：

`XMLHttpRequest cannot load http://external-domain/service. No ‘Access-Control-Allow-Origin’ header is present on the requested resource. Origin ‘http://my-domain’ is therefore not allowed access.`


## 同源策略
说到跨域就不得不提“同源策略”。   
同源策略是Web浏览器针对恶意的代码所进行的措施，为了防止世界被破坏，为了保护世界的和平，Web浏览器，采取了同源策略，只允许脚本读取和所属文档来源相同的窗口和文档的属性。
那么，怎么判断文档来源是否相同呢？很简单，看三个部分： **协议**、**主机**、**端口号**。只要其中一个部分不同，则不同源。

## 跨域的应用场景
1. 来自 `home.example.com` 的文档里的脚本读取 `developer.example.com`载入的文档的属性。
2. 来自 `home.example.com` 的文档里的脚本读取 `text.segmentfault.com`载入的文档的属性

# 如何跨域
## 设置`domain`属性
针对上述应用场景的第一种情况，可以设置`Document`对象的`domain`属性。但是设置时使用的字符串必须具有有效的域前缀或者它本身。   
PS: `domain`值中必须有一个点号。   
PS: `domain`不能由松散的变为紧绷的。

    //初始值 "home.example.com" 
    document.domain = "example.com"; //OK
    document.domain = "home.example.com"; //NO,不能由松散变紧绷
    document.domain = "example"; //NO，必须有一个点号
    document.domain = "another.com"; //NO， 必须是有效域前缀或其本身
    
## JSONP
JSONP由两部分组成： 回调函数和数据。
原理：通过动态`<script>`元素来使用，可以通过`src`属性指定一个跨域URL。

    function handler(data){
        console.log(data);
    }
    
    var script = document.createElement("script");
    script.src = "https://segmentfault.com/json/?callback=handler";
    document.body.insertBefore(script, document.body.firstChild);
    
除此之外，还可以利用`jQuery`来实现。

	function jsonCallback(json){
	  console.log(json);
	}
	
	$.ajax({
	  url: "http://run.plnkr.co/plunks/v8xyYN64V4nqCshgjKms/data-2.json",
	  dataType: "jsonp"
	});

运行结果如下：
![结果](http://oao50r2ex.bkt.clouddn.com/image/blog0918_1.png)

某些API(例如[Github API](https://developer.github.com/v3/#json-p-callbacks "Github API"))允许你定义一个回调函数，当请求返回时执行该函数。

	function logResults(json){
	  console.log(json);
	}
	
	$.ajax({
	  url: "https://api.github.com/users/jeresig",
	  dataType: "jsonp",
	  jsonpCallback: "logResults"
	});

运行结果如下：   
![结果](http://oao50r2ex.bkt.clouddn.com/image/blog0918_2.png)
    
优点:   
1. 兼容性强。   
2. 简单易用，能之间访问响应文本，支持浏览器与服务器之间双向通信。

不足：   
1. 只能用`GET`方法，不能使用`POST`方法   
2. 无法判断请求是否失败，没有错误处理。


## 跨域资源共享CORS
需要浏览器和服务器同时支持。   
原理：使用`"Origin:"`请求头和`"Access-Control-Allow-Origin"`响应头来扩展HTTP。其实就是利用新的HTTP头部来进行浏览器与服务器之间的沟通。

针对前端代码而言，变化的地方在于**相对路径需改为绝对路径**。

    //以前的方式
    var xhr = new XMLHttpRequest(); 
    xhr.open("GET", "/test", true); 
    xhr.send(); 
    //CORS方式
    var xhr = new XMLHttpRequest(); 
    xhr.open("GET", "http://segmentfault.com/test", true); 
    xhr.send(); 
    
针对服务器代码而言，需要设置`Access-Control-Allow-Origin`，显式地列出源或使用通配符来匹配所有源。

优点：   
1. CORS支持所有类型的HTTP请求。   
2. 使用CORS，开发者可以使用普通的`XMLHttpRequest`发起请求和获得数据   


不足：   
1. 不能发送和接收`cookie`   
**更新：**服务端可以通过设置`Access-Control-Allow-Credentials`该字段来表示是否允许发送Cookie。发送ajax请求时，需配置`withCredentials`属性。(感谢sf小伙伴@lloyd_zhou 指正)
具体可查看   [阮一峰大大的博客][6]。   
2. 不能使用`setRequestHeader()`设置自定义头部   
3. 兼容IE10+

## postMessage
`postMessge()`是HTML5新定义的通信机制。所有主流浏览器都已实现。该API定义在Window对象。

    otherWindow.postMessage(message, targetOrigin);

`message`: 要传递的消息。   
`targetOrigin`: 指定**目标窗口**的源。在发送消息的时候，如果目标窗口的协议、主机地址或端口这三者的任意一项不匹配targetOrigin提供的值，那么消息就不会被发送；只有三者完全匹配，消息才会被发送。这个机制用来控制消息可以发送到哪些窗口；

当源匹配时，调用`postMessage()`方法时，目标窗口的Window对象会触发一个`message`事件。在进行监听事件时，应先判断`origin`属性，忽略来自未知源的消息。

    //<http://example.com:8080>上的脚本:
    var popup = window.open(...popup details...);
    popup.postMessage("The user is 'bob' and the password is 'secret'",
                  "https://secure.example.net");  
    popup.postMessage("hello there!", "http://example.org");
    
    function receiveMessage(event)
    {
      if (event.origin !== "http://example.org")
        return;
      // event.source is popup
      // event.data is "hi there yourself!  the secret response is: rheeeeet!"【见下面一段代码可知】
    }
    window.addEventListener("message", receiveMessage, false);
    
针对上面的脚本进行接受数据的操作：

    /*
     * popup的脚本，运行在<http://example.org>:
     */
    
    //当postMessage后触发的监听事件
    function receiveMessage(event)
    {
      //先判断源
      if (event.origin !== "http://example.com:8080")
        return;
    
      // event.source：window.opener
      // event.data："hello there!"
    
      event.source.postMessage("hi there yourself!  the secret response " +
                               "is: rheeeeet!",
                               event.origin);
    }
    
    window.addEventListener("message", receiveMessage, false);     

# 后续
收到了很多小伙伴的建议和指正，不胜感激，我会慢慢丰富这篇文章的内容的。请多多指教~

# 参考文章
1. [前端跨域请求原理及实践][1]
2. [Window.postMessage()|MDN][2]
3. [老生常谈的跨域处理][3]
4. [JavaScript跨域问题总结][4]
5. [实现跨域的几种方式][5]
6. [jQuery’s JSONP Explained with Examples][7]


  [1]: http://www.tuicool.com/articles/YRFRrmb
  [2]: https://developer.mozilla.org/en-US/docs/Web/API/Window/postMessage
  [3]: http://www.tuicool.com/articles/zEZJzm2
  [4]: http://www.tuicool.com/articles/A7Ff2iV
  [5]: http://www.tuicool.com/articles/JrMFbiY
  [6]: http://www.ruanyifeng.com/blog/2016/04/cors.html
  [7]: https://www.sitepoint.com/jsonp-examples/