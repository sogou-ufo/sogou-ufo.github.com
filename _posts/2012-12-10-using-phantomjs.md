---
layout: post
title: "using phantomjs"
description: "phantomjs"
category:
tags: []
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
####4.phantomjs入门
#####a.运行时命令行参数
    phantomjs --options  xxx.js
    
    --cookies-file=/path/to/cookies.txt 指定存储cookie的文件.
    --disk-cache=[yes|no] 开启disk cache，默认否.
    --help or -h 帮助.
    --ignore-ssl-errors=[yes|no] 忽略ssl错误，默认否.
    --load-images=[yes|no] 是否加载页面里的图片，默认是.
    --local-to-remote-url-access=[yes|no] allows local content to access remote URL (default is no).
    --max-disk-cache-size=size 限定disk cache的大小(in KB).
    --output-encoding=encoding 终端输出编码 (default is utf8).
    --proxy=address:port 代理，这个比较有用，可以通过代理翻墙或者通过代理模拟跨网、异地访问 (e.g. +proxy=192.168.1.42:8080).
    --proxy-type=[http|socks5|none] 代理服务器类型 (default is http).
    --script-encoding=encoding 指定脚本编码(default is utf8).
    --version or -v 版本信息.
    --web-security=[yes|no] 安全限制，禁止跨域XHR (default is yes).
也可以将所有配置信息都写到一个文件内
例如：

    // config.json
    {
        // Same as: --ignore-ssl-errors=yes
        "ignoreSslErrors": true,

        // Same as: --max-disk-cache-size=1000
        "maxDiskCacheSize": 1000,

        // Same as: --output-encoding=utf8
        "outputEncoding": "utf8"

        // etc.
    }
    
    // then
    phantomjs --config=/path/to/config.json
    
- <a target="_blank" href="http://phantomjs.org">read more</a>
#####b.phantom object常用的几个属性和方法
    // 关闭进程，抛出returnValue
    phantom.exit(returnValue);
    
    // 错误处理，这个可以用来监控错误，生成日志
    phantom.onError = function(msg, trace) {
    }

    // 引入自定义脚本，不是注入到页面，不像nodejs，自定义的module可以require进来
    phantom.injectJs('./test.js');
- <a target="_blank" href="http://phantomjs.org">read more</a>
#####c.模块
phantomjs的模块引用是符合commonjs规范的，这点跟nodejs一样【attention:phantomjs的自定义模块则是不遵循common.js规范的…】

1\.webpage - 最有用的最常用的
webpage模块提供了创建、操作和销毁网页的接口    

    // 也可以page = new WebPage()，创建一个web page对象

    var page = require('webpage').create();

a.配置属性    

    // 如果只需要渲染页面的某一个区域，可以通过clipRect属性实现，例如
    page.clipRect = { top: 14, left: 3, width: 400, height: 300 };
    
    // 设置布局的viewport大小
    page.viewportSize = { width: 480, height: 800 };
    
    // 一般请求一个页面是需要携带一些请求头信息的，以及cookie之类
    page.customHeaders = {
        'X-Test': 'foo',
        'DNT': '1'
    };
    
    // 缩放比率，将页面缩小到25%
    page.zoomFactor = 0.25;
    
    // 甚至还可以设置页面滚动条的位置，比如让滚动条向下滚动到100px的位置
    page.scrollPosition = { top: 100, left: 0 };
    
b.方法接口

    // 一般请求一个页面是需要携带一些请求头信息的，以及cookie之类，有addCookie，必然有deleteCookie,deleteCookies
    page.addCookie({
        'name': 'Added-Cookie-Name',
        'value': 'Added-Cookie-Value'
    });
    
    // 打开一个页面
    page.open('http://m.bing.com', function(status) {
        phantom.exit();
    });
    
    // 在页面内的上下文环境以沙盒形式【页面不能访问和操作phantom对象及webpage对象的配置】执行指定的函数，可以用来做检测，判断页面的元素状态，evaluateAsync是evaluate的异步版
    page.open('http://m.bing.com', function(status) {
        var title = page.evaluate(function(s) {
            return document.querySelector(s).innerText;
        }, 'title');
        phantom.exit();
    });
    
    // 向页面内注入代码，带回调函数
    page.includeJs('http://ajax.googleapis.com/ajax/libs/jquery/1.8.2/jquery.min.js', function() {
    });
    
    // 向页面注入代码，和includeJs的区别：没有回调函数
    page.injectJs('test.js');
    
    // 释放内存
    page.release();
    
    // 一个相当有用的接口，将页面保存成图片，碉堡了，page.renderBase64('png|gif|jpeg') - 将页面存为base64码
    page.render('filename.(png|gif|jpeg|pdf)')

c.和页面交互

    // 在页面内触发鼠标或者键盘事件，用来模拟有效浏览行为 - 可以用来做自动化测试之类的
    page.sendEvent(mouseEventType[, mouseX, mouseY, button='left'])
    page.sendEvent(keyboardEventType, keyOrKeys)
    
    // 模拟上传文件
    page.uploadFile(selector, filename)
  
d.事件响应  

    // 响应页面的alert,confirm,prompt
    page.onAlert = function(msg) {
        // alert msg
    }
    page.onConfirm = function(msg) {
        // 确定true，false取消
        return true;
    }
    page.onPrompt = function(msg, defaultVal) {
        if (msg === 'What's your name?') {
            return 'PhantomJS';
        }
        return defaultVal;
    };
    
    // 响应页面里的window.callPhantom(xxx)
    page.onCallback = function(data) {
        console.log('CALLBACK: ' + JSON.stringify(data));
    };
    
    // 关闭回调
    page.onClosing = function(closingPage) {
        console.log('The page is closing! URL: ' + closingPage.url);
    };
其他：onError，onPageCreated，onInitialized，onNavigationRequested，onLoadStarted，onResourceRequested，onResourceReceived，onUrlChanged，onLoadFinished……
<a target="_blank" href="http://phantomjs.org">read more</a>    
    

2\.system

    var system = require('system');

system包含pid【进程id】、platform【=“phantomjs”】、os、env、agrs【命令行参数】几个属性

3\.filesystem

    var fs = require('fs');

文件系统操作接口

4\.webserver
比nodejs的server功能弱，强壮性稳定性方面也不及

    var server = require('webserver');
    var service = server.listen(8080, function(request, response) {
        response.statusCode = 200;
        response.write('<html><body>Hello!</body></html>');
        response.close();
    });  

####5.项目一些应用
#####a.自动截图
    function draw() {
        var page = new WebPage();    
        page.viewportSize = { width: 1024};
        page.open('http://www.exaple.com/page.php'), function(){ 
            var imgname = './img/test.img.png';
            page.render(imgname);
            page.release();
            phantom.exit();
        });
    };
    draw();
上面是一个简单的截图的例子，但是有的时候可能行不通，例如页面有延迟加载，这个时候就使用需要evaluate等接口

    function draw() {
        var page = new WebPage();    
        page.viewportSize = { width: 1024};
        page.open('http://www.exaple.com/page.php'), function(){ 
            var imgname = './img/test.img.png';
            var timer = setInterval(function() {
                var fullyLoaded = age.evaluate(function(s) {
                    return document.querySelector(s) ? 1 : 0;
                }, 'div#lazyload');
                if(!fullyLoaded)return;
                clearInterval(timer);
                page.render(imgname);
                page.release();
                phantom.exit();
            }, 1000);
        });
    };

    draw();
    
#####b.模拟测速
将phantomjs部署在不同网络的机器上【或者通过phantomjs --proxy代理模拟】，访问同一个url，获取加载时间

    var t = 0, total;

    page.onNavigationRequested = function() {
        t = +(new Date());
    }

    page.onLoadFinished = function() {
        total = +(new Date()) - t;
    }

page object的onResourceRequested接口可以针对单个资源，因此可以用来分析页面性能的瓶颈是出在哪一个文件资源的加载上

#####c.自动化测试 
类似selenium，可以用脚本驱动phantomjs对页面进行点击、滚动、按键等各种操作，并且可以通过onError捕获错误信息，还能响应alert、confirm、prompt等
####6.需要注意的问题：

1,一般服务器都是无图形界面的linux，一般也是缺少中文字体的，所以截取中文页面会出现乱码 - 安装字体么？！lz发觉这是个很蛋疼的事情，如果能用带图形界面的linux或者windows来做截图当然最好了

2,phantomjs是支持多进程的，自动化任务的时候要注意限定进程数，防止因为进程数过多导致系统崩溃【当然如果linux非root用户，限定了可使用的资源，应该不会出现这个问题】
{% include JB/setup %}
