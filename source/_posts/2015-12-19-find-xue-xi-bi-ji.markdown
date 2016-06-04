---
layout: post
title: "Find 学习笔记"
date: 2015-12-19 19:30:56 +0800
comments: true
categories: Tools
---
##### 版权声明：本文所有内容都是原创，如有转载请注明出处，谢谢。


### 1. 一个简单的find命令，找到当前目录下所有的java文件
```
find . -name '*.java'
find . -name '*.java' -print 
```
上面两个命令是一样的

### 2. find 命令的基本格式:
```
find [-H | -L | -P] [-EXdsx] [-f path] path ... [expression]
```
其中，E 是使用extended regular expression, d使用depth first search

### 3. find 使用多个文件夹
```
find dir1 dir2 dir3 -name '*.java' -print
```

### 4. 删除找到的文件或者是文件夹
```
find . -name '*.tmp'
find . -name '*.tmp' -delete
```
这个命令是很强大的，不要轻易使用，在使用之前先确认找到的文件都是自己想删除的

### 5. 找到深度相对于目前目录是5的所有java文件
```
find . -name '*.java' -depth 5
```

### 6. 寻找深度最多为5层的所有文件
```
find . -name '*.java' -maxdepth 5
```

### 7. 找到所有不是java的文件
```
find . -not \( -name '*.java' \) -maxdepth 5
```
还可以使用 \!, -or, -and, -false, -true

### 8. 使用-exec， 执行其他的命令
```
find . -name '*.java' -maxdepth 5 -exec ls -lh {} ";"
find . -name '*.java' -maxdepth 5 -exec ls -lh {} \;
```
\ 和 "" 的目的是问了让;不被shell吃掉，而变成一个string成为参数的一部分

### 9.  找到所有类型的文件
```
find . -type f 普通文件
find . -type d 目录
```
s socket, p FIFO, l symbolic link, c character special, b block special 

### 10. 特别用户名的文件
```
find . -uid / find . -user
```

### 11. 找到当前文件夹中大于1M的文件
```
find . -size +1M
find . -size n[ckMGTP] 
```
"+" 表示大于, "-" 表示小于

### 12. 一些关于时间的命令
```
find . -atime n[smhdw] -ctime n[smhdw] -mtime n[smhdw]
```
- s       second
- m       minute (60 seconds)
- h       hour (60 minutes)
- d       day (24 hours)
- w       week (7 days)

好吧暂时先写这么多，很简单的命令，确实很好用。我用的mac系统，有时可能会和Linux有点差别。
