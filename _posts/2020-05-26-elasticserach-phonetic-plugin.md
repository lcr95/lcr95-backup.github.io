---
layout:     post
title:      "Search with phonetic analysis plugin"
subtitle:   ""
date:       2020-05-26 12:00:00
author:     "ChenRiang"
catalog: true
header-style: text
tags:
    - Elasticsearch
---

**Elasticsearch version : 7.7**

In this article, we will look into how we search a keyword that has similar pronouncation (e.g. "write" and "right")  via Phonetic Analysis plugin in Elasticsearch. Checkout the [official documentation](https://www.elastic.co/guide/en/elasticsearch/plugins/6.1/analysis-phonetic.html) for more infomation.


# Plugin Installation
In this article, we will use docker as the enviroment.Checkout [Install plugin on Elasticsearch container]({% post_url 2020-05-20-docker-compose-elasticsearch-plugin %}) for more infomation. 

Add the following new inline in ``docker-entrypoint-es.sh`` :
```bash
#!/bin/bash
bin/elasticsearch-plugin install analysis-phonetic
...

```

# Plugin Enablement
In this example, we trying to stimulate a situation that we would enable plugin in a created index. 

1. Create an index called ``mytest``.
    ```
    PUT /mytest
    {
    "settings": {
        "index": {
        "number_of_replicas": 0,
        "number_of_shards": 1
        }
      }   
    }
    ```
    <br>

2. Close the index.
    ```
    POST /mytest/_close
    ```
    **Note : Index must be closed before analyzer and filter can be added.
    <br><br>

3. Add the plugin by defining analyzer and filter via [Setting API](https://www.elastic.co/guide/en/elasticsearch/reference/current/indices-update-settings.html).
    ```
    PUT /mytest/_settings
    
    {
    "analysis": {
        "analyzer": {
            "my_phonetic_analyzer": {
                "tokenizer": "standard",
                "filter": [
                    "lowercase",
                    "my_phonetic_filter"
                ]
            }
        },
        "filter": {
            "my_phonetic_filter": {
                "type": "phonetic",
                "encoder": "metaphone",
                "replace": false
            }
        }
      }
    }
    
    ```
    <br>

4. Re-open the index.
    ```
    POST /mytest/_open
    ```
    <br>

5. Define mapping that will trigger ``my_phonetic_filter``.
    ```
    PUT /mytest/_mapping
    {
      "properties": {
          "phonetic": {
          "type": "text",
          "analyzer": "my_phonetic_analyzer"
           }
        }
    }
    ```

# Plugin Validation

1. Index a document with content in a JSON field called ``phonetic``.

    ```
    POST /mytest/_doc
    {
    "phonetic": "I couldn't remember the right answer."
    }
    ```
2. Perform a match query to search for the word ``right`` with the word that have similar pronunciation - ``write``

    ```
    GET /_search?
    {
    "query": {
        "match": {
        "phonetic": "write"
         }
      }
    }
    ```
    You will noticed the plugin would recognize **"right"** and **"write"** is similar pronunciation and return the matched result.
    <br>
    POSTMAN:
    {% include image.html src="post-es-phonetic-result.png" data="group" title="Phonetic Result" %}