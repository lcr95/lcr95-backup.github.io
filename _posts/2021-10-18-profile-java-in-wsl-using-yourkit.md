---
layout: post
title: "Profile Java(WSL) using Yourkit(Window)"
subtitle: ""
date: 2021-10-18
author: "ChenRiang"
header-style: text
tags:
    - Performance
    - Java
    - Yourkit
---



I have been using WSL(Window Subsystem Linux) for development for nearly a year now, if you ask me how's the experience using it? I love it, it makes my life as a developer so much easier e.g. software building time is faster(`npm install` / `mvn install`), the script I wrote locally on my machine (95%) will work and can be reused in the CICD pipeline but fore and foremost command line user looks cool :)



However, there is still plenty room of improvement for WSL. Today we will talk about one the pain point I faced when using WSL, the software profiling tools, YourKit that run in Windows (the GUI doesn't launch in WSL) is not able to detect the Java program that running in WSL. Even though we can use Window to access Linux file system and vise versa, in reality both are still running in their own world. The only way I can think of is using the remote profiling feature. Let's look into how we can set this up.



0. YourKit is installed in Window.

1. You will still need to download the Linux package form the [website](https://www.yourkit.com/java/profiler/download/) and unzip it.

2. Run the bash script `bin/integrate.sh` and answer 7 question to **obtain the JVM option**. 

   {% include image.html src="jvm-option.png" data="group" title="" %}

   

3. **Run the java program with the JVM option** that obtain from step 2. I will be running the sample java program in IDE. 

   {% include image.html src="apply-option.png" data="group" title="" %}

   

4. Obtain **WSL's IP address** by typing the command in WSL terminal, `ip addr | grep eth0`.

   {% include image.html src="ip-addr.png" data="group" title="" %}

   In my case, my WSL IP will be `192.168.117.155` and note that this IP will change from time to time by Window.

   <br/>

5. Back to Windows, launch Yourkit profiling tool.

6. Click on the " **+** " button.

   {% include image.html src="add-remote-profiling.png" data="group" title="" %}

   

7. Fill up **the port and IP address** that configure in step 3 and 4.

   {% include image.html src="add-remote-profiling2.png" data="group" title="" %}

   

8. Click "**OK**".

9. Done! You will noticed the sample java program is being detected by Yourkit and ready to be profiled. 

   {% include image.html src="done-config.png" data="group" title="" %}

   

