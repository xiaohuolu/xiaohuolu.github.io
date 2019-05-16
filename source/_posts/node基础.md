---
title: node基础
date: 2019-04-20 16:22:32
tags: node
categories: 后台
---
**1.node严重依赖模块，想做什么功能都有相应的模块来配合，对模块的熟悉程度就反应了你node的熟练程度**

**2.node里所有api操作里都有一个sync版的同步方法，但一般不用**

---

### 适用场景
1. 中间层。
  - 提高安全性
  - 提高性能
  - 降低主服务器复杂度
2. 小型服务
3. 工具
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

### node.js的模块系统
**1.module 批量导出**
  ```
    //导出json
    module.exports={
      a:12,b:5
    }
    //导出函数
    module.exports=function(){
      console.log(1)
    }
    //导出class
    module.exports=class {
      constructor(name){
        this.name=name
      }
      show(){
        console.log(this.name)
      }
    }
  ```
**2.exports**
```
  exports.a=12
  exports.b='hehe'
```
**3.require**
```
  const http = require('http')
```
1. 如果带有路径
   - 去路径下面找
2. 如果没有路径
   - node_modules文件夹找
   - 系统的node_modules文件夹找

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
    //正确的写法应该用流，这里写法是为了理解
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
全部读取完才耗费时间和资源，正确的是加入流操作，一边读一边写入
```
  const http=require('http');
  const fs=require('fs');

  let server=http.createServer((req,res)=>{
    let rs=fs.createReadStream(req.url);

    rs.pipe(res);
    
     //处理流读取错误
    rs.on('error',err=>{
      res.writeHeader(404);
      res.write('Not Found'); 

      res.end();
    }) 
  });
  server.listen(8080);
```
写入的时候加入压缩操作，减小体积
```
  const http=require('http');
  const fs=require('fs');
  const zlib=require('zlib');

  let server=http.createServer((req,res)=>{
    let rs=fs.createReadStream(req.url);
    // 这步是告诉浏览器，这是个gzip文件，需要压缩后才能用，不要当成普通文件来用。不加这步打开网页后会直接下载
    res.setHeader('content-encoding','gzip');
    //创建压缩对象
    let gz=zlib.createGzip();
    //传入gz，然后再由gz压缩后传入res
    rs.pipe(gz).pipe(res);

    res.on('error',err=>{
      res.writeHeader(404);
      res.write('Not Found');

      res.end();
    })
  });
  server.listen(8080);
```

**数据交互**
 获取数据,又能处理get，又能处理post
```
   const http = require("http"); 
   const url = require("url"); 
   const querystring = require("querystring"); 

   let server=http.createServer((req,res)=>{
     //GET
     let {pathname,query}=url.parse(req.url,true);

     //POST
     let arr=[]
      //有一段到达了
     req.on("data",data=>{
      arr.push(data)
     })
     //结束了
     res.on("end",()=>{
      let data=Buffer.concat(arr)
      console.log(data)

       res.end();
     }) 


   })
   server.listen(8080);
```

---

**断言——assert**
 语法： 
 1. assert(条件,msg)
  作用：断定某个条件必须为真，如果不为真就直接把程序弄死
 2. assert.deepEqual(变量,预期值,msg)
  作用：深度比较，比方说两个数组，我不仅要它们都是数组，我还要他们里面的成员也一样，相当于js里的双等。
 3. assert.deepStrictEqual(变量,预期值,msg)
  作用：相当于三等，不光比较成员，还会比较类型。


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

**File System(fs模块)和二进制——Buffer**

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

**buffer处理**
文件上传 node原生处理:
```
  const http=require('http');
  const common=require('./libs/common');
  const fs=require('fs');

  let server=http.createServer((req,res)=>{
    let arr=[];

    req.on('data',data=>{
      arr.push(data)
    })
    req.end('end',()=>{
      let data=Buffer.concat(arr);
      
      //准备两个容器
      let post={};  //装普通数据
      let files={}; //装文件数据
      //解析二进制文件上传的数据
      //怎么知道是不是文件上传 可以从req.headers里的content-type里判断类型
        //如果有东西说明是文件上传 
     if(req.headers['content-type']){
      let str=req.headers['content-type'].split('; ')[1];

       if(str){
         let boundary='--'+str.split('=')[1];

         // 1.用分隔符来切分整个数据
          data.split(boundary);
        
         // 2. 丢弃头尾的无用数据
          arr.shift();
          arr.pop();
         
         // 3. 丢弃掉每个头尾的\r\n
         arr=arr.map(buffer=>buffer.slice(2,buffer.length-2));

         // 4. 每个数据在第一个\r\n\r\n处切成两半，一半是描述部分，一半是值的部分
         arr.forEach(buffer=>{
           let n=buffer.indexOf('\r\n\r\n');
   
           //描述部分
           let disposition=buffer.slice(0,n);
           //内容部分
           let content=buffer.slice(n+4);
           
           //描述部分肯定是文字，所以可以toString
           disposition=disposition.toString();

            // ==-1说明只有一行描述
           if(disposition.indexOf('\r\n')==-1){
              //普通数据
              //Content-Disposition:form-data; name='user'

              content=content.toString();
              //取name
              let name=disposition.split('; ')[1].split('=')[1];
              //去掉name分号
              name=name.substring(1,name.length-1);

              post[name]=content;
           }else{
              //文件数据
              /*Content-Disposition:form-data; name='f1'; filename='a.txt'\r\n Content-type:text/plain*/

              let [line1,line2]=disposition.split('\r\n'); 
              let [,name,filename]=line1.split('; ');
              let type=line2.split(': ')[1];

              name=name.split('=')[1];
              name=name.substring(1,name.length-1);

              filename=filename.split('=')[1];
              filename=filename.substring(1,filename.length-1);

              fs.writeFile('')
           }
         })
       }
     }
      
      res.end();
    })
  })
  server.listen(8080)
```
文件上传：node模块multiparty处理
```
const http = require('http');
const multiparty = require('multiparty');

http.createServer((req,res)=>{
    let form=new multiparty.form({
        uploadDir:'./upload'  //上传到哪
    })
    form.parse(req);

    form.on('field',(name,value)=>{
        console.log(name,value);
    })
    form.on('file',(name,file)=>{
        console.log(name)
        console.log(file)
    })
    form.on('close',()=>{
        console.log('表单解析完成');
    })
}).listen(8080);
```
---

**Crypto加密**
*md5和sha1是单向散列算法，网上的破解只是通过存储最常用的md5值，查询时只是去库里找一下，这不是破解，只是库足够大*
*绝大多数网站都没有找回密码，是因为加密后存到库里，它自己都不知道你的密码，所以大部分都只有修改密码*
*目前应用最广泛安全的加密是**RSA(非对称加密)***

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

### path
常用方法：
1. path.dirname(str)  目录名
2. path.extname(str)  扩展名
3. path.basename(str) 文件名
4. path.resolve('/root/a/b','../c','build','..','strict')             路径拼接，解析出来绝对路径
   解释：root/a/b下，返回上一级进入c,找build,返回上一级，找strict。console出来的路径就是/root/a/c/strict。(__dirname被称为魔术变量，不需要声明，是由编译器给的一个地址，在不同文件里值不一样，指的就是当前目录)
   常用：当需要当前目录的绝对路径的时候，就需要这个。path.resolve(__dirname,文件);
---

### process
判断开发环境和生产环境的简单方法之一：

*取得process.env.OS的值，通过判断是不是Window_NT，来确定是开发环境还是生产环境,true就是dev*

```
  const process = require('process');


```

**Events事件队列**
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
  const url=require(url);
  
   // 加true会把query解析成json
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
原理：
生产者/消费者模式

node中三种流:
1. 读取流   fs.createReadStream、req
2. 写入流   fs.createWriteStream、res
3. 读写流   压缩、加密

```
 const fs=require('fs');

 let rs=fs.createReadStream('1.png');
 let ws=fs.createWriteStream('2.png');
 
  //管道
 rs.pipe(ws)

 rs.on('error',err=>{
   console.log(err)
 })

 ws.on('finish',()=>{
   console.log('完成')
 })
```

---

### ZLIB压缩

```
 const zlib=require('zlib');
 const fs=require('fs');

 let rs=fs.createReadStream('jquery.js');
 let ws=fs.createWriteStream('jquery.js.gz');
  
  //创建压缩对象
 let gz=zlib.createGzip();
  //给压缩对象里传入东西，压缩后再传入ws里
 rs.pipe(gz).pipe(ws);

 ws.on('finish',()=>{
   console.log('完成')
 })

```
---
### 缓存

**缓存实现过程**
C客户端
S服务端
 
第一次：
请求的时候S需要告诉C一个 Last-Modified:Sat,02 Dec 2017 04:03:14 GMT(文件修改的日期)
第二次：
1.C->S: If-Modified-Since: Sat,02 Dec 2017 04:03:14 GMT （文件最后修改的时间）
2.S->C: 200 || 304

代码:
```
const http = require('http');
const fs = require('fs');
const url = require('url'); // 有可能有数据有可能没数据，需要url解析一下

http.createServer((req, res) => {
    let { pathname } = url.parse(req.url);
    //获取文件日期
    fs.stat(`www${pathname}`, (err, stat) => {
        if (err) {
            res.writeHeader(404);
            res.write('Not Found');
            res.end()
        } else {
            if (req.headers['if-modified-since']) {
                let oDate = new Date(req.headers['if-modified-since']);
                let time_client = Math.floor(oDate.getTime() / 1000); //客户端时间

                let time_server = Math.floor(stat.mtime.getTime() / 1000); //服务端时间
                
                if(time_server>time_client){         //服务器的文件时间大于客户端手里的版本，说明服务器的比较新，就直接发送
                    sendFileToClient()    
                }else{
                    res.writeHeader(304);
                    res.write('Not Modified');
                    res.end();
                }
            }else{
                sendFileToClient();
            }
            function sendFileToClient() {
                let rs = fs.createReadStream(`www${pathname}`);
              
                res.setHeader('Last-Modified', stat.mtime.toGMTString());
                //输出
                rs.pipe(res);

                res.on('error', err => {
                    res.writeHeader(404);
                    res.write('Not Found');
                    res.end()
                })
            }
        }

    })

}).listen(8080);

    //  stat对象

    // dev: 16777218,                               设备号
    // mode: 33188,                                 模式
    // nlink: 1,                                    连接数
    // uid: 501,
    // gid: 20,
    // rdev: 0,
    // blksize: 4096,
    // ino: 5220372,
    // size: 277,                                   大小
    // blocks: 8,
    // atimeMs: 1555987201000,
    // mtimeMs: 1555985692000,                      
    // ctimeMs: 1555985692000,
    // birthtimeMs: 1555984506000,
    // atime: 2019-04-23T02:40:01.000Z,
    // mtime: 2019-04-23T02:14:52.000Z,             修改时间Modified-time
    // ctime: 2019-04-23T02:14:52.000Z,
    // birthtime: 2019-04-23T01:55:06.000Z }
```
