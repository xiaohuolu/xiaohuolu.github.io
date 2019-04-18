---
title: node系统模块
date: 2019-04-15 16:22:32
tags: node
categories: 后台
---
**1.node严重依赖模块，想做什么功能都有相应的模块来配合，对模块的熟悉程度就反应了你node的熟练程度**

**2.node里所有api操作里都有一个sync版的同步方法，但一般不用**

---
### 常用系统模块
1. http
   HTTPS
   HTTP/2
2. 断言——assert
3. fs模块
4. c++ addons (用c给node写插件以提高node性能)
5. 多进程
   child Processes
   Cluster
   Process
6. 加密——Crypto
7. 系统信息——OS
8. 处理路径——path
9. 事件队列——Events
10. 查询字符串——Query Strings
11. 网络
    TCP——稳定，保证质量，保证顺序，保证到达。 
    UDP——快，不保证质量，不保证顺序，不保证到达。
    NET——NET是node里实现的TCP协议。
    UDP/Datagram——node里实现的UDP协议。
12. 域名解析(把域名解析成ip地址)——DNS、domain 
13. 流操作——Stream
14. 加密、安全——TLS/SSL (https就是基于SSL的,普通的http加上了ssl后就变成了https)
15. 压缩——ZLIB(gz)
---

**http模块:**
http.createServer 创建一个服务器对象

```
const http = require("http"); //首先要引入http模块

//创建一个服务器对象，参数是一个回调，每当有浏览器请求时，就会执行这个回调
let server = http.createServer(()=>{
  console.log("有人执行我了")
}); 
// 所有服务器都要监听端口号，包括node
// 端口号作用是一个服务器可以运行很多程序，如果有一个客户端要连接服务器，首先得告诉服务器你连接的是哪个程序，这就靠端口号来区分了。
// 默认端口号： http:80  ftp:21  mysql:3306 
server.listen(8080);
// 这时访问localhost:8080 可以看到console里打印的了
```


上边的已经看到打印了，但什么都没输出，现在看一下怎么给浏览器输出：
```
const http = require("http"); 
let server = http.createServer((req,res)=>{
    //req  请求  对服务器来说就是 接收的数据(输入)
    //res  响应  对服务器来说接收 发送的数据(输出)
    console.log(req.method)  // 请求的方法
    console.log(req.url)     //请求的地址

    res.write('hello world')  //写到页面
    res.end() // 只有end了后浏览器才知道服务器输出完毕了
}); 
server.listen(8080);
// 重启服务，再次访问可以看到页面上的hello world了
```
目前的写到页面是死的，谁访问看到的都是hello world，我们写活一点
```
 const http = require("http"); 
 const fs = require("fs");

 let server=http.createServer((req,res)=>{
   fs.readFile(`www${req.url}`,(err,data)=>{
     if(err){
       res.write("404")
     }else{
       res.write("data")
     }
     res.end()
   })
 })

 server.listen(8080)
```
---

**断言——assert**
 语法： assert(条件,"一段话")
 作用：断定某个条件必须为真，如果不为真就直接把程序弄死

*tips: 断言最常用的地方就是在函数里做参数的检查*

```
 const assert=require("assert");
  
 
 function sum(a,b){
   assert(arguments.length == 2,"必须传2个参数");
   assert(typeof a == "number","第一个参数必须为数字");
   assert(typeof b == "number","第二个参数必须为数字");

   return a+b
 }

 console.log(sum(12,7)); //正常运行
 console.log(sum(2));    //直接报错  必须传2个参数
```
---

**二进制——Buffer和File System(fs模块)**

*Buffer是文本文件的话可以通过toString方法查看内容,其他格式toString后文件会损坏*

**fs**
语法:
fs.readFile(读谁,(err,data)=>{})
fs.writeFile(写谁,"写什么",err=>{})

```

  const fs=require('fs');
  
  //读数据
  fs.readFile('1.txt',(err,data)=>{
    if(err){
        console.log('读取错误')
    }else{
        console.log(data)
        //读出来的文件为Buffer
    }
  })

  //写数据
  fs.writeFile('1.txt','哈哈',err=>{
     if(err){
         console.log(err)
     }else{
         console.log("写入成功")
     }
  })

```
---

**Crypto(加密)**
*md5和sha1是单向散列算法，网上的破解只是通过存储最常用的md5值，查询时只是去库里找一下，这不是破解，只是库足够大*
*绝大多数网站都没有找回密码，是因为加密后存到库里，它自己都不知道你的密码，所以大部分都只有修改密码*
*目前应用最广泛安全的加密是**RSA(非对称加密)** *

```
 const crypto=require("crypto");
 
 // 如果用sha1只需在这里换上sha1
 let obj=crypto.createHash("md5");

  //直接写也行，分多次写也行
 // obj.update("123456");
 obj.update("123");
 obj.update("4");
 obj.update("56");
 
 // 以16进制显示出来
 console.log(obj.digest("hex"))
```

双层md5：
```
 const crypto=require("crypto");

 function md5(str){
 let obj=crypto.createHash("md5");
 obj.update(str);
 return obj.digest("hex")
 }

 // console.log(md5(md5('123456')))
 // 可以通过加入随机字符串来增加安全度,只要保证字符串不被人知道就可以了
  console.log(md5(md5('123456')+'ss21f43sax'))
```
---

**Events(事件队列)**
*Events与函数调用的最大区别在于可以解耦*

```
  //EventEmitter翻译过来为事件触发器
 const Event=require('events').EventEmitter;
 //new 一个事件队列对象
 let ev=new Event();
 
 //1.监听(接收)
 ev.on('msg',function(a,b,c){
    console.log('收到了msg事件',a,b,c)
 });
   

 //2.派发(发送)
 ev.emit('msg',12,5,88)
```
---

**Query Strings**
*解析问号后面的那一部分*

```
 const querystring=require('querystring');
  
 let obj=querystring.parse(一串querystring)

 console.log(obj)
```

**url**
一般url用的会比较多一点，url是解析整段url的

```
  const url=require('url');
  
   //不加true不会解析问号后面的
  let obj=url.parse(url,true)
  
  console.log(obj)
```
---

**DNS**
*假设一个客户端，想访问某个网站，是分两步走的。1.把这个网站地址，解析成ip(类似120.54.26.38这种,一个地址可以有多个ip) 2.有了ip后才能根据这个ip找到对应的服务器*

谁来解析这个地址？
答案是**根DNS服务器**
据说全世界只有五台，如果忙不过来怎么办？它下面会有一些缓存，分级的，不重要，反正是找它要ip地址，它其实就是一个超大型的数据库，里面大概就是每个域名对应的ip是多少，每个域名对应的ip是多少，这种。根dns服务器会定期广播同步给下面的分dns服务器，以便下面的分dns服务器解析
```
 // 这里的dns操作其实就是去找dns服务器，去要这个ip

 const dns=require('dns');
 
 dns.resolve('baidu.com',(err,res)=>{
     if(err){
         console.log('解析失败')
     }else{
         console.log(res)
     }
 });
```
---

**Stream**
*连续数据都是流，视频流、网络流、文件流、语音流*