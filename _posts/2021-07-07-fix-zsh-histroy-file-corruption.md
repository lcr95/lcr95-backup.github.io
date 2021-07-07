---
layout: post
title: "Fix ZSH histroy file corruption"
subtitle: ""
date: 2021-07-07
author: "ChenRiang"
header-style: text
tags:
    - ZSH
---



Today after a force reboot, I get a ZSH history file corruption error message.

```text
zsh: corrupt history file /home/chenriang/.zsh_history
```

<br>





# Solution

Solution to solve this error message is relatively easy.



```shell
cd ~
mv .zsh_history .zsh_history_bak
strings -eS .zsh_history_bak > .zsh_history
fc -R .zsh_history
```





