---
layout: post
title: "Merge ZSH History files"
subtitle: ""
date: 2023-02-26
author: "ChenRiang"
header-style: text
tags:
    - ZSH
---

Zsh is a powerful shell that offers extensive customization options. One of its useful features is the ability to maintain a history of executed commands. However, if you use multiple terminals, you might end up with multiple history files. In this post we will look into how to merge the history files into one. 

<br/>

**Solution**

In ZSH it provide a build-in command `fc`. We will use this command to merge the history file.

```bash
    builtin fc -R -I "<history file path>"
    builtin fc -R -I "<another history file path>"

    # write the loaded history to new histroy file
    builtin fc -W "<new histroy file path>"
    mv "<new histroy file path>" ~/.zsh_history
```


