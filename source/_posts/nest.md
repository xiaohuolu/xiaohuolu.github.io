---
title: nest
date: 2019-11-17 18:05:03
tags: node
categories: 后台
---

**安装**

```
sudo cnpm install nestjs/cli --global
```

**新建项目**

```
nest new 文件名
```

**启动项目**
```
npm run start:dev // 这样修改项目后就不需要手动重启服务了
```

**创建控制器**

处理请求用的是应用里的控制器，路由的功能就是把请求指派到特定的控制器上面处理

```
nest generate controller 控制器名字
```

**get请求访问**
```
@Controller('posts')  //让PostsController变成一个可以处理请求的控制器
export class PostsController {
    @Get()     //表示 用get访问时会用index方法去处理
    index(){
        return "posts"
    }
}
```