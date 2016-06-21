---
title: Webstorm10无法调试5版nodejs
date: 2016-06-19 15:33:31
tags: 
- 环境
categories: 
- 学习
- bug & error
---

在Webstorm10中调试node.js，会出现以下错误

```
Cannot stop on breakpoint due to internal error:   
org.jetbrains.v8.V8CommandProcessor$1: TypeError: f is 
not a function at Function.t.getScopes (eval at 
undefined, :217:15) at t.describeFrame (eval at 
undefined, :213:33) at t.getFrames (eval at 
undefined, :114:89) at 
DebugCommandProcessor.r.processDebugJSONRequest (eval 
at undefined, :348:15) ...
```

百度一番后，发现是因为node.js升级为5版以后，和之前的Webstorm不兼容导致的，至于是否可通过升级Webstorm来解决还没尝试。

可先通过以下方法解决：  

1. 找到 `/Applications/WebStorm.app/Contents/bin/webstorm.vmoptions` 这个文件  
2. 打开 `/Users/somename/Library/Preferences/WebStorm10` ，将 **webstorm.vmoptions** 复制过来在。**注意：**这里的 `somename` 要对应自己电脑用户名  
3. 用文本编辑器打开 ** *webstorm.vmoptions** ，添加一行命令 `-Dnodejs.debugger.use.jb.support=false`，最终效果如下：

```
-Xms128m
-Xmx1024m
-XX:MaxPermSize=350m
-XX:ReservedCodeCacheSize=225m
-XX:+UseCompressedOops
-Dnodejs.debugger.use.jb.support=false
```

参考
===

1. [http://stackoverflow.com/questions/33515777/node-v5-breaks-webstorms-debugger](http://stackoverflow.com/questions/33515777/node-v5-breaks-webstorms-debugger)