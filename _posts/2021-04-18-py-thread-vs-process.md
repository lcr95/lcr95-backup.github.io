---
layout:     post
title:      "Python Multithreading vs Multiprocessing"
subtitle:   "" 
date:       2021-04-18 12:00:00
author:     "ChenRiang"
header-style: text
catalog: true
tags:
    - Python
---

My first impression towards python multithreading and multiprocessing is that both of them work pretty much the same.
However, this is wrong and in this article we will be looking at the differences of multithreading and multiprocessing.

# Multithreading vs Multiprocessing

Let's look at the code snippet below to understand more.

```python
import threading
import time
import multiprocessing


def hello_func(thread_no):
    time.sleep(2)
    print("greeting from - ", thread_no)

t1 = threading.Thread(target=hello_func, args=("t1",))
t2 = threading.Thread(target=hello_func, args=("t2",))

# start thread
t1.start()
t2.start()

# wait for thread to complete 
t1.join()
t2.join()

p1 = multiprocessing.Process(target=hello_func, args=("p1",))
p2 = multiprocessing.Process(target=hello_func, args=("p2",))

# start process
p1.start()
p2.start()

# wait for process to complete
p1.join()
p2.join()

```

In the code snippet above, we will spawn 2 thread and process that execute method `hello_func`. 

Result:
```text
greeting from -  t1
greeting from -  t2
greeting from -  p1
greeting from -  p2
```
From the result above, **multithreading seem like doing the exact same thing as multiprocessing**.


To understand the differences, let's look at the code snippet below:

```python
import threading
import time
import multiprocessing


def heavy_calculation(n, name):
    count = 0
    for i in range(n):
        count += i
    print(name, " done calculation")


t_time = time.time()
t1 = threading.Thread(target=heavy_calculation, args=(10000000, "t1",))
t2 = threading.Thread(target=heavy_calculation, args=(10000000, "t2",))

t1.start()
t2.start()

t1.join()
t2.join()
print("computation time for multithreading : ", (time.time() - t_time))

p_time = time.time()
p1 = multiprocessing.Process(target=heavy_calculation, args=(10000000, "p1",))
p2 = multiprocessing.Process(target=heavy_calculation, args=(10000000, "p2",))

p1.start()
p2.start()

p1.join()
p2.join()

print("computation time for multiprocessing : ", (time.time() - p_time))
```
In the code snippet above, we spawn 2 thread and process to execute a method `heavy_calculation` which will run a CPU intensive computation logic.

Result:
```text
t2  done calculation
t1  done calculation
computation time for multithreading :  0.8291773796081543
p2  done calculation
p1  done calculation
computation time for multiprocessing :  0.4506714344024658
```

**Multithreading take 0.82 second** to run the method but **multiprocessing only used 0.45 second**.
<br/><br/>

But, why?

Multiprocessing is a true parallelism but multithreading is not because the Python's global interpreter lock (GIL) will 
assure that there is will be only **one** thread running each time in a process. Click [here](https://realpython.com/python-gil/) for more info.

# When to use?
In a simple short answer, use 
- **multithreading** for I/O intensive task 
- **multiprocessing** for CPU intensive task


Threads in Python are best use for IO task because they can share the result easily with each other in a process
while for Processes it they need to pickle and combine their results (which takes time).
However, due to GIL thread provide no benefit in parallelism and CPU intensive task.   