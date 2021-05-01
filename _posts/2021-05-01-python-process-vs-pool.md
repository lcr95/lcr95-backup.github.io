---
layout:     post
title:      "Process vs Pool"
subtitle:   "Python Multiprocessing" 
date:       2021-05-01 12:00:00
author:     "ChenRiang"
header-style: text
catalog: true
tags:
    - Python
---


# TL;DR
- Use **Process** when the task 
    - amount to execute is small  
    - has long IO wait 
    
- Use **Pool** when the task 
    - amount to execute is big
    - has short IO wait 
    - consumed a lot of memory, (you required to do resources saving)
    

# Background 
In Python, the multithreading capability was disable by GIL Lock. Thus, multiprocessing is the only way library for us to achieve parallelism.
I noticed that Python has two classes that provide multiprocessing capability: Process and Pool. <br/>


Let's have a look on the differences between these two classes.



# Process Scheduler 
**Both Process and Pool used FIFO (First In First Out) scheduler** which the first task assigned to core will get executed first.


# Memory
**Process will keep all the memory** whereas **Pool will only keep those that are under execution**. 
When you have tons of tasks required a lot of memory, Process might not be a wise choice as it might waste a lot of memory.


However, using Pool is not a silver bullet for saving memory as well. In the case when you only have little number of tasks, using pool will create even more overhead.  


# I/O Operation
**Pool will not schedule another process till the task's IO operation is complete**. 
While **Process** on the other hand,  it will **halt the current process if it is running IO Operation**.  


Let's try a small experiment for this behavior. 


In the code snippet below, we will execute 2 IO tasks (`IOTask`) with Process and Pool.
Then, we will observe the time required for both process and pool.
```python
from multiprocessing import Process, Pool
import time

def IO_Task(file_name):
    f = open(file_name, "w")
    f.write("hello world")
    f.write("hello world")
    f.write("hello world")
    f.write("hello world")
    f.write("hello world")
    f.close()


process_time = time.time()

p1 = Process(target=IO_Task, args=("myIOFile1.txt",))
p2 = Process(target=IO_Task, args=("myIOFile2.txt",))
p1.start()
p2.start()
p1.join()
p2.join()
print("Time taken for Process: ", (time.time() - process_time)*1000, " ms")

pool_time = time.time()

pool = Pool()
po1 = pool.apply_async(IO_Task, args=("myIOFile3.txt",))
po2 = pool.apply_async(IO_Task, args=("myIOFile4.txt",))
po1.wait()
po2.wait()
print("Time taken for Pool   : ", (time.time() - pool_time)*1000, " ms")

```

**Output**
```text
Time taken for Process:  5.030632019042969  ms
Time taken for Pool   :  25.95376968383789  ms
```

From the output we can observe that, **Process only used 5ms** whereas **Pool used 25ms**.