---
layout: default
title: 使用git当作备份工具
---

### 说明

备份项目方式, 通常用 tar+gz 做全量备份.
如果使用 git 可对目录做增量备份:

- 减少备份附件大小
- 速度快

### 操作方式

1. 初始化 git

```
cd /path/to/project
git init
```

2. crontab 加个定时备份命令

```
0 0 * * *  cd /path/to/project && git add . --all && git commit -m "backup"
```
