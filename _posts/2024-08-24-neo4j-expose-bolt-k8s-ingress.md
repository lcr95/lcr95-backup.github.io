---
layout: post
title: Exposing Neo4j Bolt on AKS Using Kubernetes Ingress
subtitle: A Step-by-Step Guide to Configuring Ingress for Neo4j Access
date: 2024-08-24
author: ChenRiang
header-style: text
tags:
  - Neo4j
  - Azure
---

Recently, I embarked on a project to deploy Neo4j on Azure Kubernetes Service (AKS), aiming to leverage its powerful graph database capabilities within a cloud-native environment. Everything was running smoothly until I encountered a challenge: external application(which sit inside another cluster) needed to access Neo4j via the Bolt protocol, which operates on port 7687.

In this post, I will walk you through the process of setting up an Ingress for Neo4j in a Kubernetes environment, specifically focusing on exposing the 7687 Bolt port. This blog assumes that Neo4j is deployed using the official Neo4j Helm chart and that Nginx is installed using the Nginx Ingress Controller Helm chart.

# Solution

The manual steps are outlined in the Neo4j documentation, specifically under [Access you data via Cypher Shell](https://neo4j.com/docs/operations-manual/current/kubernetes/accessing-neo4j-ingress/#_access_your_data_via_cypher_shell) . However, since we used the Nginx Ingress Controller Helm chart to deploy the ingress controller, it automated all the steps defined in the documentation by supplying it with helm configuration parameters.

1. Identify the Neo4j service that exposes the Bolt service and its namespace. In my case, the service that exposes the Bolt port (7687) is `neo4j-cluster-lb-neo4j`, which resides in the `neo4j` namespace.


2. Apply the following helm configuration value on Nginx Ingress Controller.
```yaml
nginx:  
  enabled: true  
  tcp:
      7687: "neo4j/neo4j-cluster-lb-neo4j:7687"
```

# Validation 

1. Then, we can validate the port 7687 is exposed by using telnet.
   {% include image.html src="telnet-bolt-port.png" data="group" title="" %}

2. After validating that the port is exposed through the ingress, we can proceed to verify that the application can actually access Neo4j. Below is a sample application to test the connection.
```java
public static void main(String[] args) {  
        // Update the URI to include the correct scheme if needed  
        String uri = "neo4j+ssc://<ingress url>/";  
        String user = "<user>";  
        String password = "<password>";  
  
        // Create the driver instance  
        try (Driver driver = GraphDatabase.driver(uri, AuthTokens.basic(user, password));Session session = driver.session()) {  
            driver.verifyConnectivity();  
            session.executeRead(tx -> {  
                List<Record> records = tx.run("show databases").list();  
                for (Record record : records) {  
                    System.out.println("name :" + record.get("name"));  
                    System.out.println("address :" + record.get("address") );  
                    System.out.println("requestedStatus :" + record.get("requestedStatus"));  
                    System.out.println("-----------");  
                }  
                return null;  
            });  
  
        } catch (Exception e) {  
            e.printStackTrace();  
            System.out.println("Error connecting to Neo4j: " + e.getMessage());  
        }  
    }
```
Below is a sample response from the program if everything is configured correctly:
```text
name :"system"
address :"server-2.neo4j.svc.cluster.local:7687"
requestedStatus :"online"
-----------
name :"system"
address :"server-1.neo4j.svc.cluster.local:7687"
requestedStatus :"online"
-----------
name :"system"
address :"server-4.neo4j.svc.cluster.local:7687"
requestedStatus :"online"
-----------
```

