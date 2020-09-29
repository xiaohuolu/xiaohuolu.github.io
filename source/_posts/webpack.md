---
title: webpack
date: 2020-07-14 14:13:05
tags: utils
categories: 前端
---


---

**1.webpackb唯一的功能：打包**
**2.webpack默认只能处理js文件，loader使webpack可以做更多事情**

---


### webpack.config.js 
一般来讲就是一个大模块，最基本的就是三个必有的，其他是可选的

```

  module.exports={
      mode: 'none' //什么都不做，只是打包 , 'production' //能压多大压多大，尽量压 , 'development //保留必要的，方便开发，有错就报',
      entry: '入口',
      output:{  //出口
          path, 
          filename
      },
      module:{
          //放各种loader和插件的地方
          rules:{ //规则
           {test:/\.css$/,use:'xxx-loader'}
          }
      },
      plugins:[ //插件
         reuqire('xxxx')
      ]
  }

```

### 常用loader

---

loader —— 帮助webpack处理js以外的文件
css-loader —— 加载css文件，让样式文件成为js文件的一部分，让webpack可以认识css文件
style-loader —— 可以让样式变成页面中的style标签
file-loader  ——（可用url——loader取代） —— 全能loader
sass-loader —— 编译sass
dev-server —— 起服务 
eslint —— 代码质量
jest —— 测试

---