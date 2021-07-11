---
layout: post
title: "Spring Cloud Open Feign"
subtitle: "Make REST Communication easy"
date: 2021-07-11
author: "ChenRiang"
header-style: text
tags:
    - Java
    - Spring
---



In a microservice world, every service is recommended to be small and independent. As a developer, it is fortunate because we only need to look work on a relatively small code base. However, the drawback of microservice is that the developer has to implement the intercommunication between micro service which introduce extra complexity. For a developer that coming from monolith world, you might be asking why should I need to care about the details of REST API, shouldn’t we should spend the time to implement business logic?



Spring address this issue with [Feign](https://github.com/OpenFeign/feign), a declarative web client.



# Overview

To illustrate how can we use Feign, we will try to implement 2 simple microservice service.

{% include image.html src="overview-service-example.png" data="group" title="" %}



We will create an abstartction layer to define the contract of the REST communication. In real world, this two service might maintained by different team, by defnining the contract of REST Service it could de-couple the process dependency.



{% include image.html src="overview-service-with-feign.png" data="group" title="" %}





- **discovery-api (REST API)**

  - REST API contract

    

- **discovery-service (REST SERVICE)** 

  - GET Method - return some value

  - Response example:

    ```json
    [
        {
            "id": "id1",
            "name": "Jason",
            "score": "500"
        },
        {
            "id": "id1",
            "name": "Max",
            "score": "500"
        },
        {
            "id": "id1",
            "name": "Matt",
            "score": "500"
        }
    ]
    ```

    

- **dashboard-service (REST CLIENT)**

  - GET Method - do a REST call to `discovery-service` and return highest score and ranking

  - Response example:

    ```json
    {
        "topScore": "5000",
        "ranking": "Matt,Max,Jason"
    }
    ```







# Implementation 

### discovery-api (REST API)

We will create an simple interface  which will be  implement / extend by  `discovery-serice` and `dashboard-service` later on.

```java
@RequestMapping("/discovery")
public interface DiscoveryService {
    
    @GetMapping
    List<DiscoveredItem> discover();

}
```



A POJO class `DiscoveredItem` is created to represend the response.

```java
@AllArgsConstructor //lombok
@EqualsAndHashCode  //lombok
@Data  				//lombok
public class DiscoveredItem {
    private String id;
    private String name;
    private String score;
}
```



### discovery-service (REST SERVICE)

In this section, we will implement the contract interface that created in `discovery-api`.

```java
@RestController
public class DiscoveryController implements DiscoveryService {
    @Override
    public List<DiscoveredItem> discover() {
        List<DiscoveredItem> discoveredItems = new ArrayList<>();

        // return some random value
        discoveredItems.add(new DiscoveredItem("id1", "Jason", "5000"));
        discoveredItems.add(new DiscoveredItem("id2", "Max", "500"));
        discoveredItems.add(new DiscoveredItem("id3", "Matt", "1000"));

        return discoveredItems;
    }
}
```



### dashboard-service (REST CLIENT)

0. Declare the feign dependency in maven pom.

   ```xml
   <properties>
       <java.version>1.8</java.version>
       <spring.cloud.version>2020.0.2</spring.cloud.version>
   </properties>
   
   <dependencyManagement>
       <dependencies>
           <dependency>
               <groupId>org.springframework.cloud</groupId>
               <artifactId>spring-cloud-dependencies</artifactId>
               <version>${spring.cloud.version}</version>
               <type>pom</type>
               <scope>import</scope>
           </dependency>
       </dependencies>
   </dependencyManagement>
   
   <dependencies>
       <dependency>
           <groupId>org.springframework.cloud</groupId>
           <artifactId>spring-cloud-starter-openfeign</artifactId>
       </dependency>
   </dependencies>
   ```



1. Enable feign in Springboot application

   ```java
   @SpringBootApplication
   @EnableFeignClients
   public class DashboardApplication {
       public static void main(String[] args) {
           SpringApplication.run(DashboardApplication.class, args);
       }
   }
   
   ```

   

2. Extend the `DiscoveryService` created in `discovery-api` and mark it as feign client.

   ```java
   @FeignClient(value = "discoveryServiceClient", url = "${discovery.service.host}")
   public interface DiscoveryServiceClient extends DiscoveryService {
   }
   
   ```

   - `${discovery.service.host}`  - application.yaml configuration 

     

3. Autowired feign client to make a REST service call.

   ```java
   @RestController
   @RequestMapping("/dashboard")
   public class DashboardController {
   
       @Autowired
       private DiscoveryServiceClient discoveryServiceClient;
   
       @GetMapping
       public Map<String, String> getDashboardDetails() {
           Map<String, String> details = new HashMap<>();
   
           List<DiscoveredItem> discoveredItems =
               discoveryServiceClient.discover();
   
           String topScore = discoveredItems.stream()
                   .max(Comparator.comparing(DiscoveredItem::getScore))
                   .get()
                   .getScore();
   
           String ranking = discoveredItems.stream()
                   .sorted(Comparator.comparing(DiscoveredItem::getScore))
                   .map(DiscoveredItem::getName)
                   .collect(Collectors.joining(","));
   
           details.put("topScore", topScore);
           details.put("ranking", ranking);
           return details;
       }
   }
   ```
   
   



# Conclusion

In this article, we implemented the REST communication via a declarative way. With Feign, developer can write lesser code and improve the code readability at the same time.



Check out the source code [here](https://github.com/lcr95/java-example/tree/main/spring-feign-example).
