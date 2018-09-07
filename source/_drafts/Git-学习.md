title: Git 学习
date: 2018-08-31 16:35:01
categories: 编程
tags: git
---
本科四年，一直没有接触到版本控制，直到研究生期间跟一个公司合作，才知道还有SVN这种好用的东西，实验室里的几个人再也不用来回拷代码了。版本控制是个解放生产力的好东西。另外，学校的教育好像跟社会有点脱节了，教的都是些基础的理论知识，只是告诉你这些有用，但是如何用，在哪里用，没人知道，也没人告诉你这些基础虽然很重要，但是真正工作的时候这些理论往往派不上用场（当然有牛逼的人能用上）。

公司去年将版本控制由SVN迁到了Git上。刚开始有点不习惯，稍微熟悉了之后，才发现Git更好用啊。来一个需求拉一个分支，分分钟的事儿。

Git是Linux的发明者Linus缔造的。Git发明之前，Linus一直在用商业软件BitKeeper，2005年，Samba协议的发明者想要逆向BitKeeper，被发现，BitKeeper宣布停止对Linus支持，几周后Git诞生。用的不爽了，就自己写一个，彪悍的人生。改天要写下Linus这个人。

第一个版本的Git代码只有1300行左右，可以先用着。

>The first version of git was just ~1300 lines of code, and I have reason 
to believe that I started it at or around April 3rd. The reason: I made 
the last BK release on that day, and I also remember aiming for having 
something usable in two weeks.


Git更像一个小型文件系统，每次都保存一个完整的文件快照，SVN则只记录每次的变更。Git每次提交都会生成一个由40个十六进制字符组成的字符串。

Git分成工作目录、暂存区、Git仓库。工作目录是你自己当前工作的文件夹；暂存区是一个文件，保存了下次将提交的文件列表信息；Git仓库用来保存项目元数据和对象数据库。


git pull命令大多数情况下相当于git fetch 加上git merge