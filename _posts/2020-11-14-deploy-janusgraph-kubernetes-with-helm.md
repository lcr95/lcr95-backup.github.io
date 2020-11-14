---
layout:     post
title:      "Deploy Janusgraph in Kubernetes with Helm"
subtitle:   ""
date:       2020-11-14 12:00:00
author:     "ChenRiang"
header-style: text
catalog: true
tags: 
    - Janusgraph
    - Kubernetes
    - Helm
---

Janusgraph is an open source graph database that is highly scalable and optimized for storing and querying large graph. 
In this article, we will be looking into how to Helm charts for Janusgraph which:
 - Use Cassandra as storage
 - Use ElasticSearch for indexing

Note: We will be running in Window docker desktop environment.

Source code: [GitHub](https://github.com/lcr95/janusgraph-helm)

# Whats is Helm
Helm is a package manager for k8s. It is equivalent of apt or yum on linux. Helm deploys chart(~package application). 

# Helm chart folder structure
Below is the folder structure for our Janusgraph Helm's chart:
```yaml
-> janusgraph-helm
   -> templates
      -> janusgraph.yaml
   -> Chart.yaml
   -> requirements.yaml
   -> values.yaml
```

## Chart.yaml
It is a YAML file which contains the information of the chart. We will describe the chart as below:  

```yaml
apiVersion: v1
name: my-janusgraph
version: 0.1.0
appVersion: 1.0
description: A simple customized janusgraph chart that use cassandara and elastic search
```

## requirements.yaml
It is a YAML file which lists out the dependencies for the chart. We will be using Cassandra and Elasticsearch form Bitnami.
```yaml
dependencies:
- name: cassandra
  version: 5.6.1
  repository: "https://charts.bitnami.com/bitnami"
  alias: cassandra
- name: elasticsearch
  version: 12.6.1
  repository: "https://charts.bitnami.com/bitnami"
  alias: elasticsearch
```

## templates
Template directory is the place we place our k8s manifest file to describe how we want the cluster to be deployed.
By default, Helm use [Go templating](https://golang.org/pkg/text/template/) engine.

We will be creating a Deployment and Service for Janusgraph. 

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: janusgraph
  name: janusgraph
spec:
  replicas: 1
  selector:
    matchLabels:
      app: janusgraph
  template:
    metadata:
      labels:
        app: janusgraph
    spec:
      initContainers:
      - name: check-elasticsearch
        image: busybox
        command: ['sh', '-c', 'until [ "$(wget -q -O - janusgraph-helm-elasticsearch-master:9200/_cat/nodes | wc -l)" = "{{ add .Values.elasticsearch.master.replicas .Values.elasticsearch.coordinating.replicas .Values.elasticsearch.data.replicas }}" ];
                  do echo "Waiting for Elasticsearch cluster..."; sleep 10; done;
                  echo "Elasticsearch cluster is ready, launching JanusGraph Server..."']
      containers:
      - image: janusgraph/janusgraph:0.5.2
        name: janusgraph
        env:
          - name: JANUS_PROPS_TEMPLATE
            value: cql-es
          - name: janusgraph.storage.hostname
            value: janusgraph-helm-cassandra
          - name: janusgraph.storage.username
            value: cassandra
          - name: janusgraph.storage.password
            value: password
          - name: janusgraph.index.search.hostname
            value: janusgraph-helm-elasticsearch-master
```

Notes:
- Janusgraph version 0.5.2 will be used.
- It will only start initializing when elasticsearch is ready.
- We have also specified cassandra and elasticsearch hostname in the `containers.env`.

```yaml
apiVersion: v1
kind: Service
metadata:
  labels:
    app: janusgraph
  name: janusgraph
spec:
  ports:
  - port: 8182
    name: janusgraph
    protocol: TCP
    targetPort: 8182
  selector:
    app: janusgraph
  type: ClusterIP
```

## values.yaml
The configuration value for the chart.

```yaml
global:
  storageClass: hostpath
cassandra:
  dbUser:
    user: cassandra
    password: password
elasticsearch:
  master:
    replicas: 1
  coordinating:
    replicas: 1
  data:
    replicas: 1
```

# Provision Janusgraph Cluster
## Deploy Janusgraph Cluster
Before deploying Janusgraph we will need to make sure Helm pulls the required dependencies.
```bash
helm dependency update
``` 

After the dependency fulfilled, we can run the following command to deploy Janusgraph cluster to k8s environment. 
```
helm install janusgraph-helm .
```

## Validate Environment
Run `kubectl get pods` to check whether environment is up.
Example:
```
❯ kubectl get pods
NAME                                                              READY   STATUS    RESTARTS   AGE
janusgraph-9bf995c79-lhmtg                                        1/1     Running   2          6h31m
janusgraph-helm-cassandra-0                                       1/1     Running   2          6h31m
janusgraph-helm-elasticsearch-coordinating-only-549d8b75c-g6mqz   1/1     Running   2          6h31m
janusgraph-helm-elasticsearch-data-0                              1/1     Running   2          6h31m
janusgraph-helm-elasticsearch-master-0                            1/1     Running   2          6h31m
```

