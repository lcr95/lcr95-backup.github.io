---
layout:     post
title:      "How to start Jekyll server in local"
subtitle:   ""
date:       2020-11-10 12:00:00
author:     "ChenRiang"
header-style: text
catalog: true
tags: 
    - Jekyll
---

This article will be going through on how can we launch this blog page(Jekyll Server) in local machine. 

-> We will be using WSL. 


## Install prerequisite Gem and Bundler
```
# 1. install ruby
sudo apt install ruby-full

#2. install bundler
gem install bundler

```

## Install Dependecies 
```
bundle install 
```

## Serve the webpage
```
bundle exec jekyll serve
```