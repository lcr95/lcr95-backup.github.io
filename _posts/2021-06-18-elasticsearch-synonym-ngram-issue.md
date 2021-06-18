---
layout:     post
title:      "Elasticsearch Synonym + Ngram = Problem"
subtitle:   "" 
date:       2021-06-18 12:00:00
author:     "ChenRiang"
header-style: text
catalog: true
tags:
    - Elasticsearch
---



I believe [Synonym Token Filter](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-synonym-tokenfilter.html) and [NGram Token Filter](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-ngram-tokenfilter.html) are the two frequent filters that most people normally used in Elasticsearch(ES). In this article, I'm going to talk about some of the annoying stuff I met when I combine both of them in my analyzer.



# Background

Some quick intro about these two filters:

**Synonym Token Filter** - As the name stated, it is a filter that allows user to handle synonyms easily during text analysis.

e.g.

In Malaysia, Mcdonald/McD is fontly known as "mekdi" (I know this is weird, but it's real -> [mekdi](https://www.mcdonalds.com.my/Mekdi)). To create this synonym link we can specify it in ES as : `mcdonalds, mcd => mekdi`.

<br>



**NGram Token Filter** - This is a filter that tokenize a string to specified "N" size token. 

e.g. 

With a Tri-gram filter (N=3), the word `aaabbb` will become `["aaa", "bbb"]`.





# The Problem

Let's talk about the problem when we try to use these two filter in an analyzer. 



**Problem 1** : ES does not allow us to apply NGram before Synonym. You will get following error if you do so. 

```json
{
    "error": {
        "root_cause": [
            {
                "type": "illegal_argument_exception",
                "reason": "Token filter [trigrams_filter] cannot be used to parse synonyms"
            }
        ],
        "type": "illegal_argument_exception",
        "reason": "Token filter [trigrams_filter] cannot be used to parse synonyms"
    },
    "status": 400
}
```



**Problem 2** : You will get some irrelevant match result.

 Say we has an index `restaurants` which apply synonym and follow by a trigram(N=3) filter. The synonym will convert  `mcdonalds` and `mcd` to `mekdi`.

```json
PUT /restaurants

{
  "settings": {
    "analysis" : {
      "analyzer":{
        "name_analyzer":{
          "type":"custom",
          "tokenizer":"standard",
          "filter": ["synonym","trigrams_filter"]
        }
      },
      "filter": {
        "trigrams_filter": {
          "type": "ngram",
          "min_gram": 3,
          "max_gram": 3
        },
        "synonym": {
          "type": "synonym",
          "lenient": true,
          "synonyms": ["mcdonalds, mcd => mekdi"]
        }

      }
    }
  },
  "mappings": {
    "properties": {
      "name": { "type": "text", "analyzer": "name_analyzer" }
    }
  }
}
```



And in ES we have following data insert into ES.

```json
POST /_bulk

{ "create" : { "_index" : "restaurants", "_id": "1", "_type" : "_doc" }
{  "name": "mcdonalds"}
{ "create" : { "_index" : "restaurants", "_id": "2", "_type" : "_doc" }
{  "name": "mek sue cafe"}

```



The problem start when we query for the name `mcdonalds`.

```json
GET /_search

{
  "query": {
    "match": {
      "name": {
        "query": "mcdonalds",
        "operator": "and"
      }
    }
  }
}
```



<u>Result:</u>

```json
{
    "took": 2,
    "timed_out": false,
    "_shards": {
        "total": 1,
        "successful": 1,
        "skipped": 0,
        "failed": 0
    },
    "hits": {
        "total": {
            "value": 2,
            "relation": "eq"
        },
        "max_score": 0.3382834,
        "hits": [
            {
                "_index": "restaurants",
                "_type": "_doc",
                "_id": "1",
                "_score": 0.3382834,
                "_source": {
                    "name": "mcdonalds"
                }
            },
            {
                "_index": "restaurants",
                "_type": "_doc",
                "_id": "2",
                "_score": 0.19363807,
                "_source": {
                    "name": "mek sue cafe"
                }
            }
        ]
    }
}
```

You will notice that we get an irrelevant result, `mek sue cafe` appear in our result. 



**Why?**

Let's examinate how the analyzer process the word `mcdonalds`.

```json
GET /_analyze

{
  "analyzer" : "name_analyzer",
  "text" : "mcdonalds"
}
```



<u>Result:</u>

```json
{
    "tokens": [
        {
            "token": "mek",
            "start_offset": 0,
            "end_offset": 9,
            "type": "SYNONYM",
            "position": 0
        },
        {
            "token": "ekd",
            "start_offset": 0,
            "end_offset": 9,
            "type": "SYNONYM",
            "position": 0
        },
        {
            "token": "kdi",
            "start_offset": 0,
            "end_offset": 9,
            "type": "SYNONYM",
            "position": 0
        }
    ]
}
```

From the result, we understand that the word `mcdonalds` is converted into trigram of `["mek", "ekd", "kdi"]`. 

And, the token `mek` caused the match of`mek sue cafe`.



# Conclusion

It is totally fine to combining synonym and ngram filter, but we should be extra cautious when doing this as we might get some undesired search results.
