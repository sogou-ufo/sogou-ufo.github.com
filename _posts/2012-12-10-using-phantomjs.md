---
layout: post
title: "using phantomjs"
description: "phantomjs"
category:
tags: [javascript]
---
####1.神马是phantomjs？
Phantom JS是一个服务器端的JavaScript API的WebKit。其支持各种Web标准:DOM处理、CSS选择器、JSON、Canvas和SVG - <a target="_blank" href="http://phantomjs.org">read more</a>
####2.和nodejs有神马不同？
同为SSJS，phantomjs是一个WebKit，也就是说其具备了一个浏览器的完整功能——当然木有UI界面，我们可以通过phantomjs修改和操作页面，断言，注入脚本，以及页面截图。
另外不同于nodejs的单进程，phantomjs是可以多进程的【其实这比较没啥意义，nodejs更偏向server等功能，而phantomjs就是一个浏览器】
####3.下载安装
phantomjs官网已经有编译好的windows和linux版本，可以直接下载下来试用，当然也有源码，感兴趣的话可以下载下来自己编译 
linux下只需要直接将编译好的文件,放在bin或者sbin目录下，或者建别名等方式配置好，运行 phantomjs xxx.js就ok了
- <a target="_blank" href="http://phantomjs.org">read more</a>
####4.用phantomjs做截图例子
    //  引入系统模块，和nodejs一致
    var system = require('system');
    //    默认参数
    var args = [0, 0, 1024];
    for(var i = 1, len = args.length; i < len; i++) {
        args[i] = system.args[i] || args[i]；
    };
    function draw(){
        // 实例化一个Webpage对象
        var page = new WebPage();
        // 指定viewport的宽度，如果不指定，截图宽度会是页面的最小宽度
        page.viewportSize = { width: args[2]};
        // 抓取并渲染一个页面
        page.open('http://www.example.com/page.php'), function(){
            // 后缀名可以指定截取图片的类型，.png,.jpg,.gif...
            var imgname = './img/example.img.png';
            // 保存截图
            page.render(imgname);
            // 释放页面对象
            page.release();
            // 退出进程
            phantom.exit();
        });
    };
    // id必需
    if(!args[1]){
        console.log('file name is required');
        phantom.exit();
    }
    draw();
把上面的代码保存为shot.JS，然后运行
    phantomjs shot.js
<a target="_blank" href="http://phantomjs.org">read more</a>    
####5.需要注意的问题：一般服务器都是无图形界面的linux，一般也是缺少中文字体的，所以截取中文页面会出现乱码 - 安装字体么？！lz发觉这是个很蛋疼的事情，如果能用带图形界面的linux或者windows来做截图当然最好了
{% include JB/setup %}
