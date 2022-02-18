---
layout: post
title: "Kubernetes Lens show blank screen"
subtitle: ""
date: 2022-02-18
author: "ChenRiang"
header-style: text
tags:
    - Kubernetes
---

[Lens ](https://k8slens.dev/) show blank screen when it connected (green dot) to an active Kubernetes cluster. 

{% include image.html src="lens-blank-screen.png" data="group" title="" %}



**Solution**

Remove `lens-local-storage` folder solve my issue.

```bash
rm -r /home/chenriang/.config/Lens/lens-local-storage
```

*Restart Lens after the folder is deleted.



<br/>

**Reference**

[Blank Screen when accessing one (out of three) clusters · Issue #4018 · lensapp/lens · GitHub](https://github.com/lensapp/lens/issues/4018#issuecomment-942885854)
