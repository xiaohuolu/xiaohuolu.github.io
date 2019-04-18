---
title: （通信相关）跨域、http、表单、ajax
date: 2019-04-11 10:24:42
tags: js
categories: 前端
---
### 数据交互
1.表单               最基本最简单的 http数据请求其实都是表单 
2.ajax              不用刷新，ajax 可以跨域，但性能低一些，单向，跨域麻烦
3.jsonp             跨域，安全性太差，只能发起get请求
4.websocket         快，双向，跨域，性能高，双向（双工），直接跨域

**对于服务器来说，除了websocket其他都是表单，它区分不出来到底是form还是ajax还是jsonp**

---
### 跨域
注意：跨域是有风险的，如xss

举例：www.baidu.com/a.html里有一段js，要读取www.360.cn/1.txt 里的内容，就跨域了。
xss（跨站脚本攻击），其实就是利用了你的懒，从其他域名下读取了什么东西过来，也不做校验就直接执行了，有可能对方就发现你读他东西了，给你植入一些不好的东西，大概是这样。

跨域的常见场景：
1.很多网站都有很多域名，如360buy.com - jd.com ，  t.sina.cn - weibo.com  你知道他们是一个网站下的域名，但浏览器不知道。
2.第三方数据交互的需求，如一些网站有qq登录，微信登录，微博登录这种，一般是你发一个校验请求给第三方，他来帮你校验，这种时候必须要跨域。

---

### http，https 
**http**    容易被攻击

- http1.0   
   - 一次性连接
- http1.1   
   - 保持连接  
   - 性能提升
- http2.0(草案)     
   - 强制https
   - 自带双向通信
   - 多路复用

 
  **浏览器与服务器建立通信的过程，三次握手：**
  客户端（浏览器）向服务器发送连接请求，服务器接受请求（也可以拒绝，如http协议不支持，或者服务器负载太高没有空闲资源），然后客户端发送请求，服务器返回数据并关闭连接。 
   
  
**https**    安全 
https s就是ssl协议                 


  **TCP、UDP**
    
  - TCP——传输控制协议      文件下载、聊天
     - 1.丢失重传，保证到达
     - 2.错误重发，保证质量
     - 3.保证顺序  

  - UDP——用户数据报协议    对质量没有绝对要求，对延迟有很高要求。  ip电话 、视频直播
     - 1.不保证到达
     - 2.不保证质量
     - 3.不保证顺序

---
  **osi七层交换参考模型**
  
  1.物理层                               物理学家、通信工程--材料、电压
  2.链路层                               内网寻址    ARP、ICMP
  3.网络层                               外网寻址    IP
  4.传输层                               通信稳定性  tcpv
  5.表现层（被传输层取代了）                统一各个网络的结构
  6.会话层（有其他方式可以记录，比如cookie）  希望可以记录状态，不希望无状态
  7.应用层                               HTTP、FTP、SMTP、POP3

  **五层模型**
  1.物理层
  2.链路层
  3.网络层
  4.传输层
  5.应用层
  
  **无状态通信**
  正常情况下，咱们的绝大多数通信都是无状态的。
   
   简单说就是你向服务器发一个请求，它给你个东西，你就走了。下回再请求它不记得你，它会认为你是第一次来。


---

### 表单
```
  <form action="http:localhost/a.php" method="GET">
    用户名:<input type="text" name="user" value="" /><br>
    密码:<input type="password" name="pass" />
    <input type="submit" value="提交" />
  </form>
```

- 1.属性
   - action 提交到哪
   - method 方式——GET、POST、PUT、HEADER、DELETE、也可以自定义
   - name   必须要加，可以重复。服务器靠name才知道你传的是什么
   - submit 提交按钮
- 2.数据提交方法（GET和POST安全性完全一样，https才是真安全）
   - GET 
   - POST   
    **get** ：   1.数据放在URL里面，看得见 2.容量有限（32k），实际容量小于32k。 3.有缓存（不适合读取经常变的数据，股票新闻等。可以想办法去掉缓存）。 适合分享、收藏
    **POST**：  1.看不见 2.数据放在http-body里。容量很大（1g）3.没缓存。  没法分享、收藏
- 3.校验


表单重复提交处理：
1.开始提交的时候禁用submit
2.完成后（成功、失败）启用submit

---
### ajax
表单提交比较稳定，尤其在网络很差的时候，ajax更偏向用户体验。

**http 状态码：**
- 1xx   消息
- 2xx   成功
- 3xx   重定向
   - 301 永久重定向——浏览器永远不会请求老的地址
   - 302 临时重定向——浏览器下回还会请求老的地址（一般用临时）
   - 304 缓存 
- 4xx   请求错误（客户端）
- 5xx   服务端错误
- 6xx   扩展

**content-type有哪些类型，各是什么意思**
1. text/plain（纯文本）
    纯文本               
2. application/x-www-form-urlencoded（简单数据）
    k=v & k=v & k=v这种方式
3. multipart/form-data（上传文件）
   定界符分割各个数据 

**原生ajax怎么写**
```
  let xhr = new XMLHttpRequest()

  //连接
    //true异步
    //false同步
    xhr.open("GET","1.php",true)

  //发送
    xhr.send()

  //接收
    xhr.onreadystatechange=function(){
        //0 初始化——刚刚创建
        //1 已连接
        //2 已发送
        //3 已接收——头
        //4 已接收——body

        if(xhr.readyState==4){
            //成功——2xx，304
            if( xhr.status >= 200 && xhr.status <= 300 || xhr.status == 304){
                //xhr.responseText 文本
             console.log(“成功” + xhr.responseText)
            }else{
             console.log(“失败”)
            }
        }
    }
```
 

  **简单模拟jquery封装ajax**
 ```
   function ajax(options){
       options=options||{};

       options.type=options.type||'get';
       options.data=options.data||{};
       options.dataType=options.dataType||'text';
       
       let xhr=XMLHttpRequest();
        //数据组装
       let arr=[];
       for(let name in options.data){
           arr.push(`${name}=${options.data[name]}`);
       }
       let strData=arr.join("&")；

       if(options.type=="post"){
         xhr.open("post",options.url,true)
         xhr.setRequestHeader("content-type","application/x-www-form-urlencoded");
         xhr.send(strData)
       }else{
           xhr.open("GET",options.url+"?"+strData,true);
           xhr.send();
       }

       //接收
       xhr.onreadystatechange=function(){
           //4——完事
           if(xhr.readyState==4){
               let data=xhr.responseText;
               switch(options.dataType){
                   case 'json':
                   if(window.JSON && JSON.parse){
                     JSON.parse(data);
                   }else{
                    data=eval('('+str+')')
                   }
                    
                    break;
                   case 'xml':
                    data=xhr.responseXML;
                    break;
               }
               // 成功——2xx，304
               if(xhr.status>=200 && xhr.status<300 || xhr.status==304){
                   options.success && options.success();
               }else{
                   options.error && options.error()
               }
           }
       }
   }

   ajax({
       url:url,
       data:{a:12,b:5},
       dataType:'json',
       success(str){
        console.log(str)
       }
   })
 ```


**同步**
串行，一个一个来，前一个操作如果没完事，后面的等着
**异步**
并行，一堆一块进行，前面完没完事无所谓，后面都能继续开始。

