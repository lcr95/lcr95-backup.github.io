---
layout:     post
title:      "Jackson XML Parsing"
subtitle:   "" 
date:       2021-05-15 12:00:00
author:     "ChenRiang"
header-style: text
catalog: true
tags:
    - Java
    - XML
---

Recently, I have a task to parse XML document and found this awesome library called Jackson. Although it is famous in handling 
JSON parsing, their XML parsing implementation is pretty good and clean too.  


**Maven Dependency**
```xml
<dependency>
    <groupId>com.fasterxml.jackson.dataformat</groupId>
    <artifactId>jackson-dataformat-xml</artifactId>
    <version>2.12.3</version>
</dependency>
```
Refer [here](https://mvnrepository.com/artifact/com.fasterxml.jackson.dataformat/jackson-dataformat-xml) for latest version.

# Jackson XML basic
To use Jackson XML library, we need to initialize a `XmlMapper` object. 

```java
XmlMapper xmlMapper = new XmlMapper();
```


Jackson will require us to write a POJO class map with the incoming XML document that we're going to parse.

**Reading XML**
```java
MyPOJO myPojo = xmlMapper.readValue(xmlString, MyPOJO.class);
```

**Writing XML**
```java
ByteArrayOutputStream bs = new ByteArrayOutputStream();
mapper.writeValue(bs, myPojo);
String xmlString = bs.toString();
```


Use `writerWithDefaultPrettyPrinter()` method to enable pretty print for XML mapper.
```java
mapper.writerWithDefaultPrettyPrinter().writeValue(bs, myPojo);
```



# Mapping root element
To map a root element (e.g. `student`) we can use the annotation `@JacksonXmlRootElement` on the root level object that
use in serialization.

```xml
<student>
    <name>Adam</name>
</student>
``` 

```java
@Data  //lombok
@JacksonXmlRootElement(localName = "student")
public class Student {
    private String name;
}
```

# Mapping XML property
```xml
<student id="id001" _isClassRep="true" _isActive="true" >
    <name>Adam</name>
    <phone>012540701</phone>
</student>
```

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

}
```
- Use `@JsonPropertyOrder` to specify the ordering when deserialize to XML string, else jackson will follow the variable ordering in POJO class.
- Specify `isAttribute`(default=`false`) in `@JacksonXmlProperty` to control if the target property is XML attribute / element.
- Specify `localName` in `@JacksonXmlProperty` to have custom the element/attribute name that different with variable in POJO class. 

# Mapping XML collection
When dealing with array value jackson will add an extra element which contain all the value.

__POJO__:

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
    @JacksonXmlProperty(localName = "subject")
    private List<String> subjects;
}
```

```xml
<student map="id001" _isClassRep="false" _isActive="true">
  <name>Adam</name>
  <phone>015248</phone>
  <subject>
    <subject>english</subject>
    <subject>history</subject>
    <subject>mathematics</subject>
  </subject>
</student>
```

To **rename** the container element we can simply add `@JacksonXmlElementWrapper(localName = "subjects")`:


```java
@JacksonXmlElementWrapper(localName = "subjects")
@JacksonXmlProperty(localName = "subject")
private List<String> subjects;
```


__XML Output__:

```xml
<student map="id001" _isClassRep="false" _isActive="true">
  <name>Adam</name>
  <phone>015248</phone>
  <subjects>
    <subject>english</subject>
    <subject>history</subject>
    <subject>mathematics</subject>
  </subjects>
</student>
```

To **remove** the container element we can simply add `@JacksonXmlElementWrapper(useWrapping = false)`


```java
@JacksonXmlElementWrapper(useWrapping = false)
@JacksonXmlProperty(localName = "subject")
private List<String> subjects;
```


__XML Output__:

```xml
<student map="id001" _isClassRep="false" _isActive="true">
  <name>Adam</name>
  <phone>015248</phone>
  <subject>english</subject>
  <subject>history</subject>
  <subject>mathematics</subject>
</student>
```

# Conclusion
Although XML is not really favor by modern development comparing to JSON. However, it's readable pleasant structure is
definitely a plus point when you have a complex nested data structure. With the help of the library (like Jackson)
that made parsing XML relatively simple, I believe XML will no obsolete in the future.