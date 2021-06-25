---
layout:     post
title:      "Install plugin on Elasticsearch container"
subtitle:   ""
date:       2020-05-20 12:00:00
author:     "ChenRiang"
catalog: true
header-style: text
tags:
    - Elasticsearch 
    - Docker 
---

**Elasticsearch version : 7.7**

In this article, we will be using docker-compose to create a container with plugin pre-installed.

# Core Plugin
Plugins that maintained by Elasticsearch team.
List of plugin: [doc](https://www.elastic.co/guide/en/elasticsearch/plugins/7.7/index.html).

1. Create a ``docker-compose.yml`` file : 

    ```yaml
    version: '2'
    services:
      es_v7:
        image: elasticsearch:7.7.0
        environment:
          - node.name=es01
          - cluster.name=es-docker-cluster
          - discovery.type=single-node
        ports:
          - "9200:9200"
        volumes:
          - .:/mnt
        entrypoint: /mnt/docker-entrypoint-es.sh
    
    ```
    
    - Set ``discovery.type=single-node``  if you intented to run single node mode. 
    - Port is exposed to ``9200``.
    - The location of the script is mount to ``/mnt`` directory in the container.
    <br><br>


2. Create a script file ``docker-entrypoint-es.sh`` :

    ```sh
    #!/bin/bash
    
    bin/elasticsearch-plugin install analysis-phonetic
    exec /usr/local/bin/docker-entrypoint.sh elasticsearch
    
    ```
    <br><br>
3. Run ``docker-compose up``.


# Custom Plugin
Custom plugins that developed by community. 

1. Create ``docker-compose.yml`` file  
(Refer [Core Plugin - Step 1](#core-plugin))
    <br><br>

2. Create a directory (e.g: analysis-plugin) same location as ``docker-compose.yml``. <br>
E.g.
    {% include image.html src="post-es-custom-plugin.png" data="group" title="Create plugin directory" %}

3. Move the plugin jars and properties into the created directory. <br>
   E.g.<br>
   In my case, I'm trying to import [analysis-pinyin](https://github.com/medcl/elasticsearch-analysis-pinyin) which has artifacts of 2 jars + 1 properties file.

    {% include image.html src="post-es-custom-plugin-2.png" data="group" title="Custom Plugin's artifact" %}


4. Create a script file ``docker-entrypoint-es.sh`` :

    ```sh
    #!/bin/bash

    cp -R /apps/<custom-plugin> /usr/share/elasticsearch/plugins
    exec /usr/local/bin/docker-entrypoint.sh elasticsearch

    ```
    <br><br>

 5. Run ``docker-compose up``.


# Validate
Validate is the plugin successfully installed by using Elasticsearch [cat plugin api](https://www.elastic.co/guide/en/elasticsearch/reference/current/cat-plugins.html).

Postman: 
{% include image.html src="post-es-plugin-list.png" data="group" title="Validate using POSTMAN" %}