---
title: 命令使用学习笔记之head和tail
date: '2020-11-01 11:36:11'
updated: '2020-11-07 10:37:05'
categories:
  - 2 Shell
---
# 命令使用学习笔记之head和tail

```sh
# 打印前5行
head -n 5 <文件名>
# 打印后5行
tail -n 5 <文件名>
# 跟踪日志，并且保持读取追加数据
tail -f <日志文件名>
# 打印后5行，跟踪日志，并且保持读取追加数据
tail -n 5 -f <日志文件名>
```
