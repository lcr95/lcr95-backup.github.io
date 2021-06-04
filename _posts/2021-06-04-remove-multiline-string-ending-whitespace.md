---
layout:     post
title:      "Remove multi-line line end whitespace"
subtitle:   "" 
date:       2021-06-04 12:00:00
author:     "ChenRiang"
header-style: text
catalog: true
tags:
    - Java
---



Today, I bump into a problem when fixing some failed unit test. 

Long story short, I need to compare returned multiline string from `class A` and for some reason it randomly added a whitespace in each line end. This caused the unit test failed. 





**<u>Solution</u>**

Found this solution which stripe out the line end whitespace using regrex.

```java
private final Pattern LINE_END_SPACES = Pattern.compile(" +\$", Pattern.MULTILINE);

public String stripLineEndSpaces(String str) {
    return LINE_END_SPACES.matcher(str).replaceAll("");
}

@Test
public void testAbc()
    String testObject  = "123 \n abc ";
    assertEquals("123\n abc", stripLineEndSpaces(testObject));
}
```





<br>

<u>Reference</u>

[stackoverflow](https://stackoverflow.com/a/25072335/2985850)
