---
title: koa
date: 2019-05-17 10:06:05
tags: node
categories: 后台
---

### koa与express
区别： 本身区别不是特别大，但写法上不太一样
express: 回调
koa: 
v1  generator
v2  过渡板 generator&async  
v3  async/await

---

### koa安装
```
  1.
   npm init -y 
  2.
   npm i koa -D
```

---

### 路由

express 自带路由
```
  server.get('/a')
  server.post('/a')
```
koa 不带路由,需要用**koa-router**中间件

```
  //安装
  npm i koa-router -D
  
  //用法
  const Router = require('koa-router');
   
   //1.首先要创建一个路由
  let router = new Router()
   //2.给router加东西
   router.get('/a',async (ctx,next)=>{

   })
   //2.由于next在koa里用的不多，所以你见过的大多数的koa处理函数都是长这样的
   router.get('/a',async ctx=>{
       // 你想返回什么，直接往ctx的body里扔
       ctx.body = '我是aaa'
   })
   //3 需要把router使用到服务上，否则router不起作用,写法是作者的锅。
   server.use(router.routes())
    
```

**嵌套路由**
  安装koa-router

格式：
/user
  /company
  /admin
/news
  /sport
  /woman
/cart

栗子：
新建一个路由主目录routers,里面有一个管理用户的路由目录user，里面建三个文件，admin，company，index，分别是管理员，企业，还有连接这两个路由的index

/node_modules
/routers
  /user
    admin.js
    company.js
    index.js
server.js
package.json

```
  //admin.js
  const Router = require('koa-router')
  let router = new Router()

  router.get('/a',async ctx=>{
    ctx.body = '管理员的a'
  })

  module.exports = router.routes()


  //company.js
  const Router = require('koa-router')
  let router = new Router()

  router.get('/a',async ctx=>{
    ctx.body = '企业的a'
  })

  module.exports = router.routes()


  //index.js
  const Router = require('koa-router')

  let router = new Router()

  router.use('/company',require('./company'))
  router.use('/admin',require('./admin'))

  module.exports = router.routes()


  //server.js
  const koa = require('koa')
  const Router = require('koa-router')

  let server = new koa()
  server.listen(8081)

  let router = new Router()
 
  router.use('/user',require('./routers/user')
  
  server.use(router.routes())
```

### 带参路由
还是嵌套路由里那几个页面。稍微对server.js进行一点修改

```
  const Koa = require('koa')
  const Router = require('koa-router')

  let server = new Koa()
  server.listen(8081)

  let router = new Router()
  
   //这里的:id就是news页面的参数
  router.get('/news/:id/',async ctx=>{
    // 拿到新闻页面的参数，就是上面的 :id
    let {id} = ctx.params

    console.log(id)

    ctx.body='新闻'
  })
    //多个参数
   router.get('/news/:id/:id2/:id3',async ctx=>{
    let {id,id2,id3} = ctx.params

    console.log(id,id2,id3)

    ctx.body='新闻'
  })
  

  //query传参,localhost:8080/news?38291
    router.get('/news/',async ctx=>{
        let id = ctx.query
        console.log(ctx.query)

        ctx.body='bbb'
    })

  server.use(router.routes())


```

**用?id=xxx传参和/:id有什么区别？倾向于用/:id吗？**

urlencoded    http://aaa.com/user?a=12&b=5
顺序灵活 可省略 不利于SEO

params        http://aaa.com/user/12/5
顺序死的 不可省略 利于SEO

---

### ctx

**先来了解一些server.context**
server.context  相当于ctx的prototype

栗子：
```
  const koa = require('koa')
  const Router = require('koa-router')

  let server = new koa()
  server.listen(8080)
  
   //在server.context里添加属性a=12
  //server.context.a=12
    //常见用法,一些公用的东西几乎都是往这个上面加
  server.context.db = mysql.createPool()

  let router = new Router
  router.get('/news',async ctx=>{
      //可以直接在ctx.a里拿到
      //ctx.body = 'bbb'+ctx.a
    
      //然后在这里拿到
     console.log( ctx.db)
  })  
```

**ctx**
ctx.request，ctx.response
原始的request和response对象，一般用不到


*信息：*

ctx.method
请求方法

ctx.url
请求的具体地址

ctx.path
请求的路径部分，不包括域名，不包括query

ctx.query
get 数据

ctx.ip
客户端的ip

ctx.headers
请求头


*方法：*

ctx.throw(code,msg)
报错并且退出
```
  ctx.throw(400,'user and password is required')
```

ctx.assert(条件,code,msg)
断言，相当于封装了一下的throw
```
  ctx.assert(ctx.query.user,400,'username is required')
  ctx.assert(ctx.query.pass,400,'password is required')
```

ctx.state=305
状态码

ctx.redirect('')
重定向，比如登录成功后重定向到某个页面

---

### 中间件
**1.koa-static**
安装：npm i koa-static -D
```
  server.use(static('./static',{
      maxage:86400*1000 ,//缓存，在这个时间内文件不会更新，可以降低服务器压力
      index: '1.html' ,  //默认文件
  }))

```
更多栗子：

``` 
   const Koa = require('koa')
   const Router = require('koa-router')
   const static = require('koa-static')

   let server = new Koa()
   server.listen(8080)

  let staticRouter = new Router()

  //图片文件缓存两个月
  staticRouter.all(/(\.jpg|\.png|\.gif)$/i,static('./static',{
      maxage:60*86400*1000  
  }))

   //css缓存1天
  staticRouter.all(/(\.css)$/i,static('./static',{
      maxage:1*86400*1000  
  }))
   //网页文件缓存20天
  staticRouter.all(/(\.html|\.htm|\.shtml)$/i,static('./static',{
      maxage:20*86400*1000
  }))
   //其他缓存30天
  staticRouter.all('*',static('./static',{
      maxage:30*86400*1000  
  }))

  server.use(staticRouter.route())
```

**2.koa-better-body**
安装 npm i koa-better-body -D


**cookie**
cookie是koa里自带的
```
  const Koa = require('koa')
  const Router = require('koa-router')

  let server = new Koa()
  server.listen(8080)
  
  //如果要防篡改，要加签名
   server.keys=[
       'nsibsidbfndoiqdaioxchiosd',
       'nsibsadzxctrgedwefercjiloi',
       'sdscsdcbrvewecscfrhnuykiu',
   ]
  server.use(async ctx=>{
      ctx.cookies.set('user','cxn',{
          maxAge:14*86400*1000,//有效期
          signed:true , //是否签名
      })
      ctx.cookies.get('user',{
          signed:true //如果设置的时候有签名，获取的时候也必须要有，否则可以获取到但是不会校验，这个cookie还是可以被更改的
      })
  })
```

**session**
安装 npm i koa-session -D
*session强制签名*

```
  const Koa = require('koa')
  const Router = require('koa-router')
  const session = require('koa-session')

  let server = new Koa()
  server.listen(8080)

  server.key=[
      'asdoiasndisdbioasdasd',
      'dsfsdscx12eergrthrtg',
      'asdsdgtyjytvcfeveve',
  ]

  server.use(session({
      maxAge: 20*60*1000 ,//session有效期,
      renew:true  //自动续期，只要用户有任何和服务器的交互，session的有效期都会自动刷新
  },server))

  server.use(async ctx=>{
     console.log(ctx.session)
     if(!ctx.session['view']){
         ctx.session['view']=0
     }
     ctx.session['view']++
     ctx.body = `欢迎你第${ctx.session.view}次来访`
  })
```

---

### 数据库
安装  npm i mysql co-mysql -D
```
  const Koa = require('koa')
  const Router = require('koa-router')
  const mysql = require('mysql')
  const co = require('co-mysql')

  let conn = mysql.createPool({
      host:'localhost',
      user:'root',
      password:'123456',
      database:'newCreate'
  })

  let db = co(conn) 

  let server = new Koa()
  server.listen(8080)

  server.context.db = db

  server.use(async ctx=>{
      let data = await ctx.db.query('SELECT * FROM user')

      ctx.body=data
  })
```

**对上面这段做一个封装:**
新建一个目录，libs，下面放一个文件database.js

database.js:
```
  const mysql = require('mysql')
  const co = require('co-mysql')

  let conn = mysql.createPool({
      host:'localhost',
      user:'root',
      password:'123456',
      database:'newCreate'
  })

    let db = co(conn) 
    
    module.exports=db
```

server.js:
```
  const Koa = require('koa')
  const Router = require('koa-router')

  let server = new Koa()
  server.listen(8080)

  server.context.db = require('./libs/database')
  

  //这是一个处理所有异步错误的偷懒办法,就不用在每个处理前写try，catch了
  server.use(async (ctx,next)=>{
      try{
          await next()
      }catch(e){
          ctx.body='错了'
      }
  })

  server.use(async ctx=>{
      try{
          let data=await ctx.db.query('SELECT * FROM user')
      }catch(e){
          ctx.throw(500,'database error')
      }
     

      ctx.body=data
  })
```




