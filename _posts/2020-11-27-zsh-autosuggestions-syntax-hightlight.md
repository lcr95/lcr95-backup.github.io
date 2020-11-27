---
layout:     post
title:      "Supercharge ZSH console with plugins"
subtitle:   ""
date:       2020-11-27 12:00:00
author:     "ChenRiang"
catalog: true
header-img: "img/in-post/header-post-zsh.png"
tags:
    - ZSH
---

ZSH enable you to use plugin and it will supercharge your productivity and terminal experience by 10X. 

In this article, we will be adding another 2 plugin for our ZSH:
- Auto Suggestion
- Syntax Highlighting

# Auto Suggestion 
1. Clone the plugin repo to a directory(`~/.zsh/zsh-autosuggestions`).
    ```shell script
    git clone https://github.com/zsh-users/zsh-autosuggestions.git ~/.zsh/zsh-autosuggestions
    ```

2. Autostart the plugin when the terminal is launching.
    ```shell script
    echo 'source ~/.zsh/zsh-autosuggestions/zsh-autosuggestions.zsh' >> ~/.zshrc
    ```
3. Reload the terminal with new changes.
    ```shell script
    source ~/.zshrc
    ```

# Syntax Highlighting
1. Clone the plugin repo to a directory(`~/.zsh/zsh-autosuggestions`).
    ```shell script
    git clone https://github.com/zsh-users/zsh-syntax-highlighting.git ~/.zsh/zsh-syntax-highlighting
    ```

2. Autostart the plugin when the terminal is launching.
    ```shell script
    echo 'source ~/.zsh/zsh-syntax-highlighting/zsh-syntax-highlighting.zsh' >> ~/.zshrc
    ```

3. Reload the terminal with new changes.
    ```shell script
    source ~/.zshrc
    ```



