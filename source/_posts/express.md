---
title: express
date: 2019-05-15 17:20:03
tags: node
categories: 后台
---

### 创建一个服务
```
  const express = require('express')

  let server = express()
  server.listen(8080)

  //处理一个get 请求
  server.get('/a',(req,res,next)=>{
     res.send('这是a')
         next()
     //如果要给下一步传参
       req.xxx = 5
  
  })
  //同一个请求可以多次执行
    server.get('/a',(req,res,next)=>{
          //打印上一步传过来的参数
          console.log(req.xxx)


       res.send({a:1,b:2})
     
  })
```

**res.send()**:原生res.write()只支持buffer或者原生字符串，而express的send是什么都能发
**next**:执行下一个请求

//出现get请求后根据url走get
get('url',fn)
post('url',fn)
use('url',fn)

//出现get请求后直接走get，use是任何方法任何地址都可以
get(fn)   
post(fn)
use(fn)

**也可以用中间件**
//中间件，会找static里面的文件，用的时候放在最后面，避免覆盖其他接口。
server.use(express.static('./static/'))

---

### 数据
GET     req.query
POST    需要用 body-parser 中间件获取数据


```
//安装中间件
  npm i body-parser -D

//引入中间件
  const body = require('body-parser')

 //添加路由之前，先添加中间件，先解析数据，完了再执行路由 
  server.use(body.urlencoded(
      extended:false  //是否开启扩展模式，没啥用，还会增加性能负担
  ))
 //用法
 server.post('/reg',(req,res)=>{
     //用了中间件后，req会多一个body属性，这是用了中间件后它加上去的
     console.log(req.body)
 })
```

### 上传文件中间件
  命令: npm i multer -D

   ```
    const express = require('express');
    const multer = require('multer');

    let server = express()
    server.listen(8080)

    //1. 
    let obj = multer({dest:'文件上传到哪'})
    //什么文件都要
    server.use(obj.any()) 

    server.post('/reg',(req,res)=>{
        console.log(req.files)
    })
   ```

---

### 操作cookie、操作session
 
 **cookie**
 安装中间件： npm i cookie-parser -D

 ```
   const express = require('express')
   const cookieParser = require('cookie-parser')

   let server = express()
   server.listen(8080)
    
    //为了保证安全，要给cookie设置签名，签名一般只保护sessionId，因为它本身也会占用一定的空间
   server.use(cookieParser(
       'sadsadnubcisaidnwqopqmvieof' //给个随机的字符串，也就是秘钥，不能让别人知道
   ))
    
    //获取cookie
   server.get('/a',(req,res)=>{
       //未签名的cookie
       console.log(req.cookies) 
       //签名的cookie
       console.log(req.signedCookies)

        res.send('ok')
   })

   //发送cookie
   res.cookie('price',998,{
       domain:'aaa.com' ,  //设置主域名
       path:'/',
       httpOnly:true       //设置cookie只能由服务器来操作，可以增加一些安全性
       maxAge: 14*86400*1000 //设置过期时间，两个礼拜
       secure:true,        //只有https的时候才能使用cookie
       signed:true        //就是在这里发送的这个cookie ，是不是需要签名的
   })
 ```

  **session**
  安装中间件：cookie-session
  *cookie-session是天然强制加密的*
  *session可以保存在数据库里，也可以保存在redis里，文件和内存也可以。*
  ```
    const express = require('express')
    const cookieSession = require('cookie-session')

    let server = express()
    server.listen(8080)

    server.use(cookieSession({
        keys:['asoidjaodbxz','asdoiadsnowneoiwefbefw','diaonasodndubzxcoiqwe'] ,   //循环秘钥,如果只有一个秘钥很有可能被猜出来
        maxAge:20*60*1000 //设置有效期，最好不要太长,20分钟可以
    }))

    server.get('/a',(req,res)=>{
        console.log(req.session)
        //种一个cookie
        if(req.session['view']){
            req.session['view']=1
        }else{
             req.session['view']++
        }
   
        req.session['amount']=998 //这边设置的session前端是看不到的，相对要安全一些
        res.send(`欢迎你第${req.session['view']}次到访本站,你的余额是:${req.session['amount']}`)
    })
  ```

