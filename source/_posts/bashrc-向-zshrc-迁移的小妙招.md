---
title: .bashrc 向 .zshrc 迁移的小妙招
author: Dynamic_Pigeon
avatar: https://s3.bmp.ovh/imgs/2024/10/08/ca40e9bf152dbbf0.jpg
authorLink: chy669086.github.io
authorAbout: 一个好奇的人
authorDesc: 一个好奇的人
categories: 技术
comments: true
date: 2024-10-11 17:45:00
updated: 2024-10-11 17:45:00
tags:
    - linux
    - bash
    - zsh
keywords:
description:
photos: https://s3.bmp.ovh/imgs/2024/10/11/5610094626aebd1d.png
---
昨天兴致突发安装了 zsh 并使用了 powerlevel10k 主题。有一说一 zsh 使用体验比 bash 好太多了，但是今天我发现了一个重大的问题：之前的所有配置都在 .bashrc，这些不会自动继承到 .zshrc，这可咋办？而且 zsh 和 bash 命令并不兼容，不能简单在 .zshrc 加一句 `source $HOME/.bashrc` 来解决。

直接 `source $HOME/.bashrc` 会出现以下报错

```shell
/usr/share/bash-completion/bash_completion:45: command not found: shopt
/usr/share/bash-completion/bash_completion:1596: parse error near `|'
```

一种方法是把所有配置文件都重新迁移到 .zshrc，~~但是我懒~~，所以我找到了些小妙招。

发现在 bash 直接运行 zsh 会让 zsh 继承 bash 的所有环境变量，所以只要想办法让 zsh 启动的时候启动 bash 再用 bash 启动 zsh 就好了。

## 解决方法

### 手动解决

一种方法是手动解决

```shell
# 在 zsh 中执行
exec bash
# 在 bash 中执行
exec zsh
```

### 改写配置文件

另一种方法是改写 .zshrc 和 .bashrc。

首先，我们在 .zshrc **头部**加入

```shell
if [[ -v __USE_ZSH ]]; then
  unset __USE_ZSH
else
  echo "ENTER BASH!"
  export __USE_ZSH=1
  exec bash
fi
```

然后在 .bashrc **底部**加入

```shell
if [[ $__USE_ZSH -eq 1 ]]; then
	echo "ENTER ZSH!"
	exec zsh
fi
```

这两个命令大概就是：进入 zsh 先判断有没有 `__USE_ZSH` 这变量，有，就 unset 掉，否则创建 `__USE_ZSH=1` 然后进入 bash，bash 中如果发现 `__USE_ZSH==1`，就会进入 zsh，就很妙的把 bash 的环境变量偷过来了。

## 参考资料

https://blog.csdn.net/qq_49030008/article/details/137921205