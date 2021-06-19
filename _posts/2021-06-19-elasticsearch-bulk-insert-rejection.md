---
layout:     post
title:      "Elasticsearch Bulk Index Request Rejection"
subtitle:   "" 
date:       2021-06-19 12:00:00
author:     "ChenRiang"
header-style: text
catalog: true
tags:
    - Elasticsearch
---



Yesterday, I obtained the `es_rejected_execution_exception` error response when trying to perform bulk indexing request.

```json
{
    "error": {
        "root_cause": [
            {
                "type": "es_rejected_execution_exception",
                "reason": "rejected execution of coordinating operation [coordinating_and_primary_bytes=0, 	 replica_bytes=0, all_bytes=0, coordinating_operation_bytes=61790813, max_coordinating_and_primary_bytes=53687091]"
            }
        ],
        "type": "es_rejected_execution_exception",
        "reason": "rejected execution of coordinating operation [coordinating_and_primary_bytes=0, replica_bytes=0, all_bytes=0, coordinating_operation_bytes=61790813, max_coordinating_and_primary_bytes=53687091]"
    },
    "status": 429
}
```



**<u>Root Cause</u>**

After I did a bit of googling about this error. I realized this is actually a feature that ES implemented to protect the system from over-loaded by indexing pressure.



In ES, each of the document will go through 3 indexing stage which is coordinating, primary, and replica. At the beginning of each indexing stage, ES will accumulate the bytes consumed by the request and will only clear the counter at the end of indexing. For example, the consumed bytes accumulated in coordinating request will not be clear until it passed through primary and replica stage. 



The node will start reject future requests when the consumed bytes more than the configured limit which default configured as 10% of the heap size.

See more, [here](https://www.elastic.co/guide/en/elasticsearch/reference/current/index-modules-indexing-pressure.html).

--



From the error obtained, I understand that my bulk index request was being rejected during coordinating stage. 

<br>



**<u>Solution</u>**

The solution for this is simple, we can either **reduce the request size** or **allocate more memory to ES**. 



In my case, I allocated more memory to ES. 



*P.S: Lower down the `indexing_pressure.memory.limit` is something not recommenced by ES.* 



 





