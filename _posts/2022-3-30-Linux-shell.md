---
layout: post
title: 把写的脚本变为linux/unix中的系统命令
subtitle: bash命令
date: 2022-03-30
author: Levi
header-img: img/post-bg-linux.jpeg
catalog: true
top: true
tags:
  - Linux
  - Bash
  - unix
---





1. 先给脚本权限

   ```shell
   chmod 777 脚本文件.bash
   ```

2. 然后给脚本创建符号链接

   ```shell
   ln -s ln -s wakeUpNas.bash  wake
   ```

3. 复制符号链接wake到/usr/bin/中

   ```shell
   cp wake /usr/bin/
   ```

   