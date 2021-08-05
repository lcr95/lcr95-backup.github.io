---
layout: post
title: "Running IntelliJ IDEA in WSLg (Window 11)"
subtitle: ""
date: 2021-08-05
author: "ChenRiang"
header-style: text
tags:
    - WSL
---



I have been using WSL for development for sometime and turn up 80% of my tools has moved to WSL instead of native Window. However, if you're a WSL heavy user like me, you will surely understand the pain not able to use Linux GUI application natively.  Recently, Microsoft has started a project called, [WSLg](https://github.com/microsoft/wslg) (*Windows Subsystem for Linux GUI*) to support WSL running Linux application. In this article, I'm going to talk about running IntelliJ IDEA IDE with WSLg.



# Pre-Requisite

As of the time of writing, we will have to join the Window Insider program to get the WSL update.  Once you're registered for the program, just click window update and let Microsoft take care of it :)



After you get the latest version of Window, just launch CMD or PowerShell and type the following :

```bat
wsl --update
```



Once the WSL updated, restart it with following command:

```bash
wsl --shutdown
```



## Launch Linux GUI

Now we can try install some Linux GUI application.

```bash
sudo apt install nautilus -y
```



After installation done, just simply hit `nautilus` in the WSL command line and you will notice the it will the application launched!

{% include image.html src="first-linux-application.png" data="group" title="" %}







# Install IntelliJ IDEA IDE

I'll be using Jetbrain's [Toolbox](https://www.jetbrains.com/toolbox-app/) to install the IDE.

1. Download the tarball **.tar.gz** from the official website.

2. Extract the folder out. E.g.

   ```bash
   sudo tar -xzf jetbrains-toolbox-1.17.7391.tar.gz -C /opt
   ```

   

3. Execute the application and click **Install** button on the version that you wish to installed.  

{% include image.html src="toolbox-install.png" data="group" title="" %}

(IntelliJ Community version installed)



If everything installed correctly you will notice there is shortcut added to the start menu. 

{% include image.html src="shortcut-added.png" data="group" title="" %}



**Question: No shortcut added to my start menu! What should I do?**

Check out this directory `~/.local/share/applications` there should be a file named `jetbrains-idea-ce.desktop` in it.  

```bash
ls ~/.local/share/applications
```



Move the file to `/usr/share/applications`.

```bash
cd ~/.local/share/applications
sudo cp *.desktop /usr/share/applications
```



Done. 

Refer similar issue, [here](https://github.com/microsoft/wslg/issues/4)



