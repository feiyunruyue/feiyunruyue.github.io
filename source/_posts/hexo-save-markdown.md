title: hexo 保存markdown源码
date: 2015-01-31 11:34:52
tags: hexo
---

刚开始接触[hexo](http://hexo.io)的时候就有个疑问，**README.md**放到哪里？写文章用到的**markdown**源码如何保存，难不成换电脑之后我还得拷贝一份之前的目录？这些问题困扰了我好久，让我觉得不是很爽。我也想到过其他一些方法，如新建一个repository，然后将这些源码保存到这里。但是觉得这样不够**优雅**（知乎看多了）。

以前一直用我的Ubuntu 14.04，08年买的本，2G内存，开个Android studio卡的不行，于是换上了我女朋友的本。昨天晚上想写点东西（现在一耽误，我都忘了要写啥了），把Ubuntu上的源文件*拷*到了现在的本上。我觉得不能再拷来拷去了！于是在网上搜了一下，找到了简书上的[一篇文章](http://www.jianshu.com/p/e6b2fbcfa05e)，看完之后，深受启发。答案就是新建一个分支，将这些源码文件放到这个分支里，源码和展现，各占一个分支（**要注意的是html文件必须在master分支上**）。git杀手锏，我竟然没有想到， orz！<!--more-->

具体来说，就是分以下几步：

 1. 将自己的github pages clone一份到本地
     > git clone https://github.com/feiyunruyue/feiyunruyue.github.io

 2. 新建一个空的分支(orphan也是一个新的知识点啊)
	 > git checkout --orphan blog_source

 3. 这时候自己目录下应该是空的，添加blog源码，
	 > git add .
	 > git commit -m "xxxxxxx"

 4. 添加远程分支
	 > git push origin blog_source:blog_source

 5. 最后一步，更完美一点点。在github 项目上选择settings->defaul branch

**大功告成！**

 