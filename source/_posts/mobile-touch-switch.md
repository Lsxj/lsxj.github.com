---
title: 移动端上下滑动切换屏幕的实现
date: 2016-07-21 21:42:01
categories: 前端开发
tags: [html5, css, javascript]
---
### 前言
看过了不少移动端上下滑动切换屏幕的效果，于是自己拿简历试着实现了一下。效果可看:[滑动切换屏幕效果][1]或者扫描二维码
![lsxj615.com/cv](http://oao50r2ex.bkt.clouddn.com/imagcv.png)


## 实现过程

* ### 大体思路

页面的切换主要是采用`css3`动画效果`transform:translateY()`来实现。事件监听主要以`touchstart`,`touchmove`,`touchend`。

* ### HTML结构

		<div class="container"> 
		    <section id = "intro">...</section>   
		    <section id = "project">...</section> 
		    <section id = "profession">...</section>
		    <section id = "education">...</section>
		    <section id = "contact">...</section>  
		</div>

* ### CSS样式如下：

		//css样式
	    html,body {
	        width: 100%;
	        height: 100%;
	        padding: 0;
	        margin: 0;
	        overflow: hidden; 
	    }

	    .container {
	        position: absolute; 
	        width: 100%;
	        height: 100%;
	        -webkit-transform: translate3d(0, 0, 0);
	        -moz-transform: translate3d(0, 0, 0);
	        -o-transform: translate3d(0, 0, 0);
	        -ms-transform: translate3d(0, 0, 0);
	        transform: translate3d(0, 0, 0);
	        -webkit-backface-visibility: hidden;
	        -webkit-transition: all 0.5s ease;
	        -moz-transition: all 0.5s ease;
	        -o-transition: all 0.5s ease;
	        -ms-transition: all 0.5s ease;
	        transition: all 0.5s ease; 
	    }
	    
	    .container > section {
	        position: relative;
	        width: 100%;
	        height: 100%; 
	    }
    
   其中`-webkit-backface-visibility:hidden`是为了隐藏被旋转的元素的背面。关于动画效果，由于我采用的`SCSS`，在更改时只需要更改$n即container内section的个数，较为便捷。

	    //以下为SCSS部分：
	    $n:5;
	    @function getContentTopOffset($i) {
	        @return -100%*$i+100%;
	    }
	    @for $i from 1 through $n {
	        .slide_to_#{$i} {
	            transform: translateY(getContentTopOffset($i));
	        }
	    }
	    //CSS的效果如下：
	    .slide_to_1 {
			transform: translateY(0%); }
	    
	    .slide_to_2 {
			transform: translateY(-100%); }
	    
	    .slide_to_3 {
			transform: translateY(-200%); }
	    
	    .slide_to_4 {
			transform: translateY(-300%); }
	    
	    .slide_to_5 {
			transform: translateY(-400%); }
    
    
   切换的效果示意图如下：container相当于红色边框矩形，内容部分就相当于白色部分。下图为最初效果，红色矩形内为可见的。如图1.当container执行translateY(-100%)时，就相当于内容部分向上移动了100%，即显示了下一部分的页面。如图2所示。   
图1：最初效果 ![](http://oao50r2ex.bkt.clouddn.com/image/blog0721_1.png)   
图2：移动后的效果![](http://oao50r2ex.bkt.clouddn.com/image/blog0721_2.png)
    
* ### js实现
知道如何切换页面之后就好办了。

	    var cur_page = 1, //当前页数
	        slide_range = 30, 
	        page_count = 5,//共有5页
	        obj_container = '.container';//需要切换页面的元素

	    $(document).on("touchstart",obj_container, function(e){
	        touchFirst_obj = {
	            startY: e.touches[0].clientY
	        };
	    });//记录起始触屏位置
	    
	    $(document).on("touchmove",obj_container, function(e){
	        e.preventDefault();//阻止浏览器默认事件
	        touchLast_obj = e.touches[0];
			moveY = touchLast_obj.clientY - touchFirst_obj.startY;
	    });
	    
	    $(document).on("touchstart",obj_container, function(e){
	        if(moveY > 0) { //向上滑
				moveUp();
			} else if(moveY < 0) { //向下滑
				moveDown();
			}
			if(moveY != 0) {//如果有滑动，则需要切换页面
				switching();
			}
	    });
	    
	    function moveDown() { //向下滑
			if(cur_page == +page_count) {
				return;
			}
			cur_page++;
		}
	
	    function moveUp() { //向上滑
			if(cur_page == 1) return;
			cur_page--;
		}
	
	    function switching() {//切换页面
			$(obj_selector).attr('class', obj_container + ' slide_to_' + cur_page);
		}


   技术不精，若有Bug欢迎指正！原创作品，若有转载请告知。谢谢！
    
  [1]: http://lsxj615.com/cv
