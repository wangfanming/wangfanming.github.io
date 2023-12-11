#Angularjs的使用笔记

使用步骤：

1、创建app对象：

    1.1 创建base.js   
``` JavaScript
    var app=angular.module('dit',[]);
```
    

##细节总结

在controller中引入$location，才能使用其相关方法。

location.href="http://www.wangfanming.top/html/index.html#?name=wfm",用于页面带参数跳转

在跳转后的index.html页面中则可以使用`$location.search()['name'];`获取传递过来的参数。