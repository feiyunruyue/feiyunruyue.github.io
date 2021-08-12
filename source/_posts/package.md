title: package.json ～和^
date: 2014-11-22 21:07:12
tags:
---
node package.json 中的~符号类似**约等于**，它会比较主版本号和中间的版本号，比如`～1.2.3`会匹配所有的1.2.X，但不会匹配1.3.0（之前一直理解错了，以为只有1.2.4到1.3.0才会被匹配）

^符号匹配的较宽松，它将比较主版本号，即第一位版本，如`1.2.3`会匹配1.2.4、1.3.0，但是不会匹配2.0.0.

通过下面这段代码，我们做个测试。首先新建一个空文件夹，在该文件夹内新建文件package.json，将如下代码写入：
<!--more-->
    {
     "name": "hello",
     "version": "1.0.0",
     "dependencies": {
       "async": "~0.2.9"
      }
    }

执行npm install,我们会发现npm帮我们下载了0.2.10版本的async。
将版本号改为`0.2.8`，你会发现执行上述命令，npm仍帮我们下载了0.2.10版本的async。