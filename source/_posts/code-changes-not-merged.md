---
title: change没有被合并
date: 2017-06-09 20:02:00
permalink: code-changes-not-merged
tags: git
categories: git
---

git 在合并分支时偶然碰到的奇葩问题
<!--more-->
### 问题现象
   dev 合同master5.6分支,检查文件，发现master5.6的修改并没有合并到dev

 ### 可能的问题原因
  master5.6_bug 之前合并到dev分支，master5.6_bug代码曾经回退过代码，dev分支并没有回退，导致master5.6合并时，涉及被回退的文件有问题

### 处理方案
    1.  确认暂时没人提交代码，约束大家稍缓提交
    2.  本地从master5.6 checkout一个临时分支
    3.  临时分支 rebase dev  这样就保证 以dev分支为主干，dev分会分别执行master5.6每次提交分支

### 知识点

   git rebase
    rebase 的工作原理，注意弄清楚那个分支会作为基准分支
          [参考文档](http://weizhifeng.net/git-rebase.html)


### 总结
   1. 开发测试上线要按照git工作流走，尽可能要有**唯一**的主分支
   2. 代码的回退要慎重，回退前要考虑好影响到的分支
   3. 分支合并尽可能合并稳定的分支，如线上分支，不断开发的分支要慎重合并
