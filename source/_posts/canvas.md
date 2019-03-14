---
title: canvas 基础用法
date: 2019-02-20 09:52:36
tags: h5
categories: 前端
---



### canvas使用方法 

```
//获取canvas画布对象
 var canvas = document.querySelector("canvas")
//获取绘图上下文
 var ctx = canvas.getContext("2d")
 //落笔
 ctx.moveTo(100,100)
 //连线
 ctx.lineTo(300,100)
 //描边

 //带颜色的线
 ctx.moveTo(100,150)
 ctx.lineTo(300,150)
 ctx.lineWidth="20“
 ctx.strokeStyle="red"
 ctx.stroke()
```

[查看更多canvas方法](http://www.w3school.com.cn/tags/html_ref_canvas.asp)

---
### 绘制步骤

 1.先落笔 
 2.连线 
 3.描边 

 三个步骤对应三个方法 

---
### 总结: 
 **1.** canvas的画布大小需要指定在标签属性上，如果在css中设置，只是改变了canvas标签大小，实际绘图区域还是300*150。
 **2.** ctx.stroke() 一次就会重新渲染一次全部画布，so，只写一个stroke，或者开启一个新图层，方法是 ctx.beginPath()


