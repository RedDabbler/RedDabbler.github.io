---
title: source-code-java-Path
permalink: source-code-java-Path
date: 2018-03-21 17:49:22
tags:
categories:
---

Path接口是1.7引入的，在java.nio.file 包下，是用来定位在文件系统中文件的位置，通常代表一个文件的系统文件的路径。这个方法继承Watchable接口，因此，被path路径来定位的文件夹可以被WatchService注册，并且监控文件夹内的条目。

File类 有toPath方法可以获得Path对象，Path类也有toFile方法获取File对象。
resolve(Path)
