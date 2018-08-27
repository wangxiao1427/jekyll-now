---
layout: post
title: NodeJS单机多核原理!
---

NodeJS单机多核原理

* 进程间通信，传递服务器句柄
<pre>
  <code>
    // parent.js
    var child = require('child_process').fork('child.js');
    var server = require('net').createServer(); 
      server.on('connection', function (socket) {
      socket.end('handled by parent\n');
      });
    server.listen(1337, function () {
      child.send('server', server);
    }); 
  </code>
</pre>

<pre>
  <code>
    // child.js
    process.on('message', function (m, server) {
      if (m === 'server') {
        server.on('connection', function (socket) {
        socket.end('handled by child\n');
        });
      }
    }); 
  </code>
</pre>

启动服务器，监听1337端口 
<pre>
  <code>
  \$node parent.js  
  \$curl 127.0.0.1:1337  
  handled by parent  
  \$curl 127.0.0.1:1337  
  handled by child  
  \$curl 127.0.0.1:1337  
  handled by parent  
  </code>
</pre>
进程间传递的句柄外表看起来像是直接传递了服务器对象，其实是经过序列化之后的数据。
每次请求的处理者都是随机的。  
如果在传递完句柄后就把主进程关掉，让子进程处理请求，那么需要修改代码：  
<pre>
  <code>
  // parent.js 
  var cp = require('child_process'); 
  var child1 = cp.fork('child.js');
  var child2 = cp.fork('child.js');
  // Open up the server object and send the handle
  var server = require('net').createServer();
  server.listen(1337, function () {
    child1.send('server', server);
    child2.send('server', server);
    // 关闭
    server.close();
  }); 
  </code>
</pre>

<pre>
  <code>
  // child.js
  var http = require('http');
  var server = http.createServer(function (req, res) {
    res.writeHead(200, {'Content-Type': 'text/plain'});
    res.end('handled by child, pid is ' + process.pid + '\n');
  });
  process.on('message', function (m, tcp) {
    if (m === 'server') {
      tcp.on('connection', function (socket) {
      server.emit('connection', socket);
      });
    }
  }); 
  </code>
</pre>
主进程在发送句柄给子进程之后关闭，之后所有的请求全都由fork出来的多个子进程处理。  
并且多个子进程监听的是同一个端口。  

1.句柄发送与还原  
2.端口共同监听  

* NodeJS单机多核集群  
使用cluster创建nodejs单机多核集群(多进程)  
<pre>
  <code>
  var cluster = require("cluster");
  var http = require("http");
  var numCPUs = require("os").cpus().length;
  var port = parseInt(process.argv[2]);

  if (cluster.isMaster) {
    for (var i = 0; i < numCPUs; i++) {
      cluster.fork();
    }

    cluster.on("exit", function (worker, code, signal) {
      cluster.fork();
    });
  } else {
    http.createServer(function (request, response) {
      console.log("Request for:  " + request.url);
      response.writeHead(200);
      response.end("hello world\n" + process.pid);
    }).listen(port);
  }
  </code>
</pre>
如上，cluster是通过isMaster来区分父子进程，父进程中处理创建逻辑:根据 CPU的核数，创建相应数量的子进程；子进程中运行具体的 Server 创建代码  
