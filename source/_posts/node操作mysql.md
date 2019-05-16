---
title: node操作mysql
date: 2019-05-07 17:14:25
tags: node
categories: 后台
---

##1.mysql模块##
1. npm init -y
2. npm i mysql -D

##2.连接池##
建立连接池：createPool
```
const mysql = require('mysql');
let db = mysql.createPool({
    connectionLimit:10  //最大连接数，默认10
    host: 'localhost', port: 3306, user: 'root', password: '12345678', database: 'newCreate' 
})
```
好处：加快速度，减轻负载，也不会轻易被卡住

比如我一次跟服务器建立五十条或者一百条连接，需要请求的时候，我就从这个池子（其实就是数组）里面拿出一个空闲的连接来用，用完了再把它放回去。


##3.异步co-mysql模块##
它本身不能连接数据库，就相当于一个包装器,可以把普通mysql变成一个异步的东西。如果不用co-mysql的话，就不能用async/await
  1. npm i co-mysql -D 

---

**一个简单的注册登录栗子：**

html:
```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>Document</title>
    <script src="http://libs.baidu.com/jquery/2.0.0/jquery.min.js"></script>
</head>
<body>
    
   用户:<input type='text' id='user'/></br> 
   密码:<input type='password' id='pass'/></br> 
    <input type='button' value='注册' id='reg' />
    <input type='button' value='登录' id='login' />
</body>
 <script>
    $("#reg").click(async ()=>{
      let data=await $.ajax({
          url:'http://localhost:8080/reg',
          data:{
              username:$('#user').val(),
              password:$('#pass').val()
          }
      })
      alert(data)
    })

    $("#login").click(async ()=>{
      let data=await $.ajax({
          url:'http://localhost:8080/login',
          data:{
              username:$('#user').val(),
              password:$('#pass').val()
          }
      })
      alert(data)
    })
 </script>
</html>
```

validator:
```
  module.exports = {
    username(username) {
        if (!username) {
            return '用户名不能为空'
        } else if (username.length > 32) {
            return '用户名最长32位'
        } else if (!/^\w{3,32}$/.test(username)) {
            return '格式不对'
        } else {
            return null;
        }
    },
    password(password) { 
        if (!password) {
            return '密码不能为空'
        } else if (password.length > 32) {
            return '密码最长32位'
        } else {
            return null;
        }
    }
}
```

node
```
const http = require('http');
const mysql = require('mysql');
const co = require('co-mysql');
const url = require('url');
const fs = require('fs');
const validator = require('./libs/validator');

//1.连接到服务器
// let db = mysql.createConnection({ host: 'localhost', port: 3306, user: 'root', password: '12345678', database: 'newCreate' });

//1.建立连接池
let conn = mysql.createConnection({ 
    host: 'localhost', 
    port: 3306, 
    user: 'root', 
    password: '12345678', 
    database: 'newCreate' 
    });
let db = co(conn);

// 操作数据库栗子
// db.query('SELECT * FROM user', (err, data) => {
//     if (err) {
//         console.log('错了', err);
//     } else {
//         console.log(JSON.stringify(data))
//     }
// })

// let username='mc';
// let password='666666';
// db.query(`INSERT INTO user (username,password) VALUES('${username}','${password}')`,(err,data)=>{
//     if(err){
//         console.log('错了',err);
//     }else{
//         console.log(data)
//     }
// })

// 2.跟http配合
http.createServer(async (req, res) => {
    const { pathname, query } = url.parse(req.url,true);

    if (pathname == '/reg') {
        let {username,password}=query
        //1.参数是否正确
        let err = validator.username(query.username);
        if(err){
            res.write(err);
        }else{
          let err = validator.password(query.password);
          if(err){
              res.write(err);
          }else{
            try{
                let data=await db.query(`SELECT ID FROM user WHERE username='${username}'`)
                if(data.length>0){
                    res.write('此用户名已被占用');
                }else{
                        //3.检查通过插入进去
                    await db.query(`INSERT INTO user (username,password) VALUES('${username}','${password}')`)
                    res.write('注册成功');
                }
             }catch(e){
                res.write('数据库出错');
             }
          }
        }
        res.end();
    } else if (pathname == '/login') {
        //1.检查用户名是否存在
        //2.检查密码对不对
        //3.返回
        let {username,password}=query
        let err=validator.username(query.username);
        if(err){
            res.write(err);
        }else{
           try{
            let err = validator.password(query.password);
            if(err){
                res.write(err);
            }else{
              let data = await db.query(`SELECT ID,password FROM user WHERE username='${username}'`);
              if(data.length==0){
                  res.write('用户名不存在');
              }else if(data[0].password!=password){
                  res.write('密码不对')
              }else{
                  res.write('成功')
              }
            }
           }catch(e){
               console.log(e)
               res.write('数据库出错');
           }
        }
      res.end();
    } else { 
        //请求文件
        fs.readFile('www' + pathname, (err, buffer) => {
            if (err) {
                res.writeHeader(404);
                res.write('not found');
            } else {
                res.write(buffer);
            }
            res.end();
        })
    }
}).listen(8080);
```

### 启动器 forever
 安装：npm i forever -g
 用法：
 1. 启动服务  forever start xxx.js
 2. 列出正在运行的服务 forever list
 3. 重启服务  forever restart xxx.js 
 4. 关闭某个服务  forever stop xxx.js
 5. 关闭全部服务  forever stopall

 修饰符:
 1. -l 日志输出到哪  forever start xxx.js -l c:/a.log
 2. -o 普通输出到哪
 3. -e 错误输出到哪
 4. -a 不要清楚之前的日志，往后追加
