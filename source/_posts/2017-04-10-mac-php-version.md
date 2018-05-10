---
title: 更新Mac的PHP默认版本
date: 2017-04-10 01:07:38
tags: [mac, php, brew]
category: php
 
---

#### 知识点

类*nx系统，系统默认软件的安装位置

```shell
/usr/bin
```

非系统默认，brew安装的软件安装位置

```shell
/usr/local/bin
```

#### brew安装指定版本php后，执行一下命令，设置新版本为默认php版本

```shell
export PATH="$(brew --prefix php56)/bin:$PATH"
export PATH="$(brew --prefix php56)/sbin:$PATH"
export PATH="/usr/local/bin:/usr/local/sbin:$PATH"
```

