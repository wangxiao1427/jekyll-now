---
layout: post
title: NodeJS之内存泄漏与分析!
---

V8引擎的内存限制为0.7GB(64位为1.4GB)，这种情况下是无法操作大文件的；并且不经意的内存泄漏(代码规范问题)会导致可用内存越来越小，性能下降。  
本文主要介绍NodeJS中的内存分析工具heapdump安装和使用。  

首先需要安装必要环境(又是因为windows系统的node-gyp安装异常，下面都是以centOS为例)
+ 安装heapdump  
`npm install heapdump`  

可能会报的错：  
  - gyp ERR! stack Error:Can't find executable "python"  
  解决方案：安装python2.7.*(指定版本)，最好设置环境变量  
+ 安装node-gyp  
`npm install node-gyp -g`  
+ 配置python命令  
`npm config set python python2.7`  
如果本机安装过多个python版本，上述命令结尾应该是指向python bin目录下的指定版本的可执行文件  
本人机器：`npm config set python python` 
+ c/c++编译环境  
如果安装heapdump过程中，编译报错，需要安装gcc  
+ 最后安装heapdump  
`npm install heapdump`  

<h3>如上，所有环境安装成功之后，开始进入正题</h3>  
先在centOS中跑一个简单的内存泄漏的web服务，代码如下： 

  var http = require('http');
  var leakArray = [];
  var leak = function () {
    leakArray.push("leak" + Math.random());
  };
  http.createServer(function (req, res) {
    leak();
    let temp = 1;
    res.writeHead(200, {'Content-Type': 'text/plain'});
    res.end('Hello World\n');
  }).listen(1337); 
  console.log('Server running at http://127.0.0.1:1337/'); 

此段代码，多次访问接口之后，leakArray一直在占用内存没有释放。  

接下来分析一下内存飙升的因素：  
  * 缓存，写代码的时候为了查询更快，喜欢创建全局的缓存变量，使用之后不及时清空  
  * 闭包，作用域没有释放；闭包滥用；  
  * 生产者和消费者存在速度差，比如数据库忙不过来，Query 队列堆积  
V8内存分为两部分，new space和old space。new space大小为8M，old space大小为0.7G。  
如果内存占用过高不能及时被回收，这些对象最终会被从new space转到old space，GC会不停的扫描old space，这些没有释放的对象增加了GC负担。  

如上代码中leakArray最后始终会在old space中被持续扫描；temp会在new space中被GC回收。  

<h3>接下来可以看一下如何使用heapdump分析内存泄漏</h3>  
* 在启动程序时添加`--trace_gc` 参数，GC在垃圾回收的时候会输出信息到控制台。  
![_config.yml]({{ site.baseurl }}/images/nodejs_trace_gc.png) 
[还有很多命令](http://nodejs.cn/api/cli.html)  

通过`node --prof test.js`命令记录日志，会在当前目录下生成一个*-v8.log文件，可以使用tick分析结果。  
使用日志分析工具 tick需要安装`sudo npm install tick -g`  
完成之后执行`node-tick-processor *-v8.log`，如果找不到node-tick-processor命令，可以找到安装目录通过绝对路径执行，如：
 `~/node-v8.11.4-linux-x64/bin/node-tick-processor *-v8.log`  
![_config.yml]({{ site.baseurl }}/images/nodejs_processor.png)  

<h3>另外可以使用pm2生成内存快照，导入到chrome->控制台->memory中</h3>
[参考文档](https://www.colabug.com/110317.html)