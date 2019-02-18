---
title: video多浏览器支持
date: 2019-02-18 16:46:57
tags: h5
categories: 前端
---


### video视频兼容所有浏览器

viedo支持的视频格式为 mp4 ogg 和webM ，如果想在所有浏览器都能播放视频的话，只需要如下写法。

```
  <video>
    <source src=“xxx.mp4”></source>
    <source src=“xxx.ogg”></source>
    <source src=“xxx.webM”></source>
  </video>
```

#### 播放规则是浏览器依次查询是否支持某个格式的视频，如果支持则播放，不去查询下面的。