---
title: WebSocket
date: 2019-04-28 16:54:26
tags: js
categories: 前端
---

**WebSsocket和Ajax**
打个比方，时不时要去服务器查看一下有没有新的留言。

**用ajax**
只能搞个定时器，时不时去请求一下看看有没有新的留言，也就是轮询。这种方式缺点是
1.浪费服务器资源，因为要不停的连接，断开，请求。
2.浪费带宽资源，如果是移动端的应用，流量是要钱的，更死人了。
所以ajax不适合现在移动互联网的需求。

**用websocket**
1. 性能高，比普通ajax大概高2-10倍左右 
   普通的http通信是基于字符的通信，所谓超文本，还是文本。而websocket一开始是文本，但一旦连接建立了，协议升级完成了，会是一个二进制的一个协议，无需对数据做什么转换之类的，这是性能高的原因之一。
2. 双向通信
   不需要你去请求，只要连上了等着就行，等有数据了服务器会主动找你。这是websocket最大的优点。还是基于http，在建立连接的阶段，还是通过普通的http协议来交换信息。
3. 直接跨域

---
### socket.io

cnpm i socket.io -D

1. 简单方便
2. 兼容1e5
3. 自动帮助你对数据进行解析

```
  //html
  <!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>Document</title>
    
    <script src='http://localhost:8080/socket.io/socket.io.js'></script>
    <script>
      let sock = io.connect('ws://localhost:8081/'); 

    //   sock.emit('aaa',12,'socket成功')
      sock.on('aaa',function(a,b){
          console.log(a,b)
      })
    </script>
</head>
<body>
    
</body>
</html>

//js
const http = require('http');
const io = require('socket.io');

//1. 建立普通http
let server = http.createServer((req,res)=>{
  
})
server.listen(8081);

//2. 建立ws
 let wsServer = io.listen(server); //监听http服务，如果发现是找ws的，它就会接手，把连接抢过来，它来负责做下面的交互。
 //当建立连接了
 wsServer.on('connection',sock=>{
     //发送数据
   sock.emit('aaa',12,'哈哈');
     //接收数据           
//   sock.on('aaa',function(a,b){
//       console.log(a,b);
//   })
 }); 
```

### 原生websocket
network里websocket请求头里会有这几个东西
```
//一个扩展信息，可忽略
Sec-WebSocket-Extensions: permessage-deflate; client_max_window_bits
// 验证是不是websocket
Sec-WebSocket-Key: ydOb9ivgNMpZk0zNiobpfQ==
// websocket版本,不同版本验证是有差异的
Sec-WebSocket-Version: 13
//协议升级
Upgrade: websocket  
```

连接建立阶段实现：
```
 //html
   <!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>Document</title>
    <script>
      let ws = new WebSocket('ws://localhost:8081/');
      
      //并不是连接上就会触发，交换数据后会触发
      ws.onopen=function(){
          console.log('连接已建立'); 
      };
      //当有消息
      ws.onmessage=function(){};
      //当连接关闭
      ws.onclose=function(){};
      //当异常关闭
      ws.onerror=function(){};
    </script>
</head>
<body>
    
</body>
</html>


// js
  
//二进制，实际上websocket就是tcp连接，所以需要net
const net = require('net');
const crypto = require('crypto');

function parseHeader(str){
    let arr = str.split('\r\n').filter(line=>line);
    arr.shift();

    let headers={}
    arr.forEach(line => {
       let [name,value]=line.split(':');
       name=name.replace(/^\s+|\s+$/g,'').toLowerCase();
       value=value.replace(/^\s+|\s+$/g,'');

       headers[name]=value;
    });
    return headers;
}

 //参数是一个原始的sock对象
let server = net.createServer(sock=>{
    //第一次握手接到的数据
   sock.once('data',buffer=>{
    let str = buffer.toString();
    let headers=parseHeader(str);

    if(headers['upgrade']!='websocket'){
        console.log('no upgrade');
        sock.end();
    }else if(headers['sec-websocket-version']!='13'){
        console.log('no 13');
        sock.end();
    }else{
        //1.key : 从http的头里来
        //2.uuid : '258EAFA5-E914-47DA-95CA-C5AB0DC85B11' //作者定的，固定的。 
        //3.result => base64(sha1(key + uuid))
        let key =  headers['sec-websocket-key'];
        let uuid = '258EAFA5-E914-47DA-95CA-C5AB0DC85B11';
         //创建一个hash对象，签名算法是sha1
        let hash = crypto.createHash('sha1');
        hash.update(key + uuid);
        let key2 = hash.digest('base64');

        sock.write(`HTTP/1.1 101 Switching Protocols\r\nUpgrade:websocket\r\nConnection:upgrade\r\nSec-Websocket-Accept:${key2}\r\n\r\n`);
    }
   })
 

   sock.on('end',()=>{
   
   })
});
server.listen(8081);
```