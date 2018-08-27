---
layout: post
title: NodeJS内存泄露分析之heapdump!
---

NodeJS内存泄露分析常用工具

* node-heapdump  
  对V8堆内存抓取快照，用于事后分析。<br>
  安装node-heapdump：  
  `npm install heapdump`  
  如果没有安装python会报错，因为heapdump依赖于python(版本Python >= v2.5.0 & < 3.0.0.)
  1\.如果报错gyp，则需要安装node-gyp  
  安装node-gyp(本地插件生成工具)  
  `npm install node-gyp -g --save`  
  2\.报错MSBUILD : <span style="color:red;">error MSB4132: 无法识别工具版本“2.0”。可用的工具版本为 "4.0"。</span>  
  参考`https://www.cnblogs.com/iTlijun/p/8193588.html`  
  测试：  

  启动一个测试服务，多次请求之后，发送信号  
  `$ kill -USR2 <pid>`  
  快照文件存在于chrome控制台-Profilesmi面板  
  <pre>
    <code>
      var leakArray = [];
      var leak = function () {
        leakArray.push("leak" + Math.random());
      };
      http.createServer(function (req, res) {
        leak();
        res.writeHead(200, {'Content-Type': 'text/plain'});
        res.end('Hello World\n');
      }).listen(1337);
      console.log('Server running at http://127.0.0.1:1337/'); 
    </code>
  </pre>

