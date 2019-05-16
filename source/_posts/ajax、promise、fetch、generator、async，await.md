---
title: ajax、fetch、promise、generator、async，await
date: 2019-03-15 16:10:12
tags: js
categories: 前端
---
### ajax
写法:
```
  $.ajax({
      url:'1.txt',
      dataType:json,
      success(res){
      console.log(res)
      },
      error(err){
      console.log(err)
      }
  })

  //写法2
    $.ajax({
      url:'1.txt',
      dataType:json 
  }).then(res=>{
      console.log(res)
  },err=>{
      console.log(err)
  })
```

### fetch
用来替代原生xhr
1.读取
2.解析

``` 
  //文本变量
  let oBtn=document.getElementById('btn');
  //图片变量
  let oImg=document.getElementById('img');
  oBtn.onclick = async function(){
    //解析文本
    //请求
    let res = await fetch('data/1.json');
    //解析
    let json = await res.json();


    //解析图片，音频，视频等
    //请求
    let res = await fetch('data/1.json');
    let data = await res.blob();
    //浏览器帮你处理二进制数据
    let url = URL.createObjectURL(data);
    oImg.src = url
    console.log(json);
  }


```

### promise
1. 解决回调地狱
2. 有局限，对带逻辑的异步操作比较麻烦
--- 
Promise.all() 所有的都成功
Promise.race() 只要有一个完成

---

基础用法:

```
  let p = new Promise((resolve,reject)=>{
      $.ajax({
          url:'1.txt',
          dataType:'json',
          success(json){
            resolve(json)
          },
          error(err){
              reject(err)
          }
      })
  })
  p.then(json=>{
    console.log(json)
  },err=>{
    console.log(err)
  })

```

```
 //promise.all

  //语法
  Promise.all(数组).then(arr=>{
    let [a,b,c] = arr
  })

  //栗子
  Promise.all([
      $.ajax({url:'1.txt',dataType:'json'}),
      $.ajax({url:'2.txt',dataType:'json'}),
      $.ajax({url:'3.txt',dataType:'json'}),
  ]).then(arr=>{
       let [a,b,c]=arr
       console.log(a,b,c)
  },err=>{
      alert('失败')
  })
```

### generator(生成器函数)
yield: 暂停
next : next一下走一步 ，一般可以等数据请求完了再next
```
//yield传参

  function *show(){
        alert("a")

     let a=yield ;
     console.log(a)
 
        alert("b")
  }
    let gen=show()

    gen.next()

    
    gen.next(12)

//yield 返回

    function *show(){
        alert("a")

        yield 55;
 
        alert("b")
  }
    let gen=show()

    let res1=gen.next() 

      console.log(res1)

    let res2=gen.next()

      console.log(res2)
```

### async，await 
基本用法：

```
  async:function(){
    //同步操作
    //同步操作

     //当有异步操作时
     let 结果=await 异步操作-promise、generator、另一个async函数

    //同步操作
    //同步操作
  }
```

加入错误处理，多个处理可以多个try catch
```
   async function show(){
     try{
       let data= await $.ajax({url:url,dataType:'json'})

       console.log(data)
     }catch(e){
       console.log(e)
     }
   }
```