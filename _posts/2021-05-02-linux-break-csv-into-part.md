---
layout:     post
title:      "Break Large CSV into parts"
subtitle:   "" 
date:       2021-05-02 12:00:00
author:     "ChenRiang"
header-style: text
catalog: true
tags:
    - Linux
---

Often time, process or read a large CSV file is quite troublesome. With the concept of parallelism, we can break the
file into smaller chunk and create multiple processes to process each of these smaller chunk of record simultaneously.


But, how do we to chunk the csv equally according to our needs?


e.g.<br/>
We have a CSV file named `application.csv` which consist of 20,000 records. 
We need to chunk it into part files which have 10,000 each.


We can achieve this by creating a linux function. 

1. Paste the following command into your terminal.
    ```bash
    splitCsv() {
        HEADER=$(head -1 $1)
        if [ -n "$2" ]; then
            CHUNK=$2
        else 
            CHUNK=1000
        fi
        tail -n +2 $1 | split -l $CHUNK - $1_split_
        for i in $1_split_*; do
            sed -i -e "1i$HEADER" "$i"
        done
    }
    ```

2. Run the following command.
    ```bash
    # e.g. splitCsv application.csv 10000
    splitCsv <csv file> <chunk size>
    ```

3. You will notice the part files created with the following format:
    ```text
    <csv file>_split_<part>
    
    e.g.
    application.csv_split_aa
    application.csv_split_ab
    ```

<br/><br/>

**Reference**


[StackOverflow](https://stackoverflow.com/a/20721203/2985850)