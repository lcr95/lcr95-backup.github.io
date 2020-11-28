---
layout:     post
title:      "Setup Java like a Pro"
subtitle:   ""
date:       2020-11-28 12:00:00
author:     "ChenRiang"
header-style: text
catalog: true
tags:
    - Java
---

{% include image.html src="post-sdkman.png" data="group" title="SDKMAN!" %}

**SDKMAN!** (formerly known as GVM- Groovy enVironment Manager) is a tool for managing parallel versions of multiple SDK on Unix environment.
It will become extreme handy when you need to test your Java application with different Java version and distribution.
<br/>

Although I don't need an environment with multiple Java version, but being a lazy developer, setting up java environment just too much work for me. 
{% include meme.html src="meme-way-too-much-work.png"  title="too much" %}



We are going to set up our development in WSL with:
- Java 1.8
- Maven 3.6.3

# Install SDKMAN!
1. Launch wsl terminal  and execute following command to install SDKMAN!.
    
    ```shell
    curl -s "https://get.sdkman.io" | bash
    ```


2. Open a new terminal or refresh the terminal with following command.

    ```shell
    source ~/.bashrc
    ```
3. Check SDKMAN! version to validate.
    ```shell
    sdk v
    ```

# Install Java
1. List all the available Java in SDKMAN!.
    ```shell
    sdk l java
    ```  
    {% include image.html src="post-sdkman-install-java.png" data="group" title="List all available Java" %}

2. Install the Java. *(In my case I will use 8.0.275.hs-adpt)*
    ```shell
    sdk i java 8.0.275.hs-adpt
    ```
3. Done.

# Install Maven
1. List all the available Maven in SDKMAN!.
    ```shell
    sdk l maven
    ```  
2. Install Maven.
    ```shell
    sdk i maven 3.6.3
    ```

3. Done.