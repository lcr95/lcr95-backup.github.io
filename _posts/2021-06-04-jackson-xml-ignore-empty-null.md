---
layout:     post
title:      "Jackson Ignore Null and Empty value"
subtitle:   "" 
date:       2021-06-04 12:00:00
author:     "ChenRiang"
header-style: text
catalog: true
tags:
    - Java
    - XML

---



Today, I bump into a problem when fixing a unit test when asserting Jackson XML deserialized string.



**<u>Problem</u>**

Jackson XML deserialize null and empty value.

<u>POJO</u>

```java
@Data  //lombok
@JacksonXmlRootElement(localName = "student")
@JsonPropertyOrder({"id", "_isClassRep", "_isActive", "name", "phone"})
class Student {
    @JacksonXmlProperty(isAttribute = true, localName = "map")
    private String id;
    @JacksonXmlProperty(isAttribute = true, localName = "_isActive")
    private boolean isActive;
    @JacksonXmlProperty(isAttribute = true, localName = "_isClassRep")
    private boolean isClassRep;
    private String phone;
    private String name;
    private String address = "";
    @JacksonXmlElementWrapper(localName = "subjects")
    @JacksonXmlProperty(localName = "subject")
    private List<String> subjects = new ArrayList<>();
}
```



<u>Deserialization</u>

```java
public static void main(String[] args) {
        Student student = new Student();
        student.setId("id");
        student.setActive(true);
        student.setPhone("1234567890");

        XmlMapper xmlMapper = new XmlMapper();
        ByteArrayOutputStream bs = new ByteArrayOutputStream();
        xmlMapper.writerWithDefaultPrettyPrinter().writeValue(bs, student);
        String xmlString = bs.toString();
        System.out.println(xmlString);
}
```



<u>Output</u>

```xml
<student map="id" _isClassRep="false" _isActive="true">
  <name/>
  <phone>1234567890</phone>
  <address></address>
  <subjects/>
</student>
```

   

We don't wan Jackson to include `name` , `address` and `subjects`. Instead our expecting deserialized XML string would be:

```xml
<student map="id" _isClassRep="false" _isActive="true">
  <phone>1234567890</phone>
</student>
```



<br>

**<u>Solution</u>**

We can tell Jackson to ignore null or empty value by using 

- `Include.NON_NULL`  - only include properties with not null value in XML.
- `Include.NON_EMPTY` - only include properties is not null / empty. E.g. `List` / `Map` - method `isEmpty()` return true ; `String` with length 0. 



There is few way we can do.

1. Supply `Include.NON_NULL` or `Include.NON_EMPTY` at **property level.** 

   ```java
   @Data  //lombok
   @JacksonXmlRootElement(localName = "student")
   @JsonPropertyOrder({"id", "_isClassRep", "_isActive", "name", "phone"})
   class Student {
       @JacksonXmlProperty(isAttribute = true, localName = "map")
       private String id;
       @JacksonXmlProperty(isAttribute = true, localName = "_isActive")
       private boolean isActive;
       @JacksonXmlProperty(isAttribute = true, localName = "_isClassRep")
       private boolean isClassRep;
       @JsonInclude(JsonInclude.Include.NON_NULL)    
       private String phone;
       private String name;
       @JsonInclude(JsonInclude.Include.NON_EMPTY)
       private String address = "";
       @JacksonXmlElementWrapper(localName = "subjects")
       @JacksonXmlProperty(localName = "subject")
       @JsonInclude(JsonInclude.Include.NON_EMPTY)
       private List<String> subjects = new ArrayList<>();
   }
   ```


   <br>

2. Supply `Include.NON_NULL` or `Include.NON_EMPTY` at **class level**.

   ```java
   @Data  //lombok
   @JacksonXmlRootElement(localName = "student")
   @JsonPropertyOrder({"id", "_isClassRep", "_isActive", "name", "phone"})
   @JsonInclude(JsonInclude.Include.NON_EMPTY)
   class Student {
       @JacksonXmlProperty(isAttribute = true, localName = "map")
       private String id;
       @JacksonXmlProperty(isAttribute = true, localName = "_isActive")
       private boolean isActive;
       @JacksonXmlProperty(isAttribute = true, localName = "_isClassRep")
       private boolean isClassRep;
       private String phone;
       private String name;
       private String address = "";
       @JacksonXmlElementWrapper(localName = "subjects")
       @JacksonXmlProperty(localName = "subject")
       private List<String> subjects = new ArrayList<>();
   }
   ```

   Note:  `Include.NON_EMPTY` at class level means any property of this class that have empty value will not be included in XML.


   <br>

3. Supply `Include.NON_NULL` or `Include.NON_EMPTY` at **global level** with XmlMapper.

   ```java
   public static void main(String[] args) {
           Student student = new Student();
           student.setId("id");
           student.setActive(true);
           student.setPhone("1234567890");
   
           XmlMapper xmlMapper = new XmlMapper();
           xmlMapper.setSerializationInclusion(Include.NON_EMPTY); 
       
           ByteArrayOutputStream bs = new ByteArrayOutputStream();
           xmlMapper.writerWithDefaultPrettyPrinter().writeValue(bs, student);
           String xmlString = bs.toString();
           System.out.println(xmlString);
   }
   ```

   

