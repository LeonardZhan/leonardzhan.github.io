---
title: Linux awk 取回输出的最后一列
date: 2019-03-29 21:56:54
tags: Linux
---

在学习 `docker` 的时候创建了不少 `container` ：

```
[root@bogon ~]# docker ps -a
```



一个个删总觉得太麻烦，所以就想用`shell`脚本去批量处理！





### 方法一：取出第一列的 `CONTAINER ID`

一番 Google 之后，找到了 Linux  的一个取列的命令 `awk` :

```
[root@bogon ~]# docker ps -a | awk '{print $1}'
```



`$1` 代表了取第一列, 但是这样抓出来的第一列是包含了第一行的 title 的，因此要添加参数：

```
[root@bogon ~]# docker ps -a | awk 'NR>1 {print $1}'
```



`NR > 1` 是一个判断条件，指当行数大于 1 的时候才运行 `awk` 的命令并打印出结果，接着用 `for` 命令批量的删除：

```
[root@bogon ~]# for i in `docker ps -a | awk 'NR>1 {print $1}'`
> do
> docker rm $i
> done
```



在使用 `for` 的时候需要注意一下，将长命令括起来的不是单引号，而是一个重音符 "  `  "，作为小白的我就踩坑了。。。





### 方法二：取出最后一列的 `NAME`

实际上我在做这件事的时候就是取的最后一列，完全忘记了取第一列的 `CONTAINER ID`也是可以的，直到在写这篇文章的时候才意识到，给自己跪了。。。_(:3」∠❀)_



在刚开始的时候我直接数列数，第七列

```
[root@bogon ~]# docker ps -a | awk '{print $7}'
```



但是！`awk` 是默认用空格来进行分列的，于是就出现了这种情况：

```
[root@bogon nginx]# docker ps -a | awk '{print $7}'
PORTS
26
hour
ago
Exited
Exited
Exited
Exited
Exited
21
21
```



这完全不是我要的，原来是应为`docker ps -a ` 输出的结果里面有很多空格，用命令看看没一行有多少列：

```
[root@bogon nginx]# docker ps -a | awk '{print NF}'
8
15
15
14
12
12
12
12
12
15
15
```



因此没办法直接用数字指出最后一列，当中想过给 `awk` 设置指定数量的空格作为分割符，最后因为之间的空格并不相等，所以告吹，最后用谷歌翻了不少别人的 Blog，终于找到了取出最后一列的方式：

```
[root@bogon nginx]# docker ps -a | awk '{print $NF}'
```



 是的，没错，就是 `$NF` ，最后命令：

```
[root@bogon ~]# for i in `docker ps -a | awk 'NR>1 {print $NF}'`
> do
> docker rm $i
> done
```



