---
layout: post
title: "Java 8 Date Time"
subtitle: ""
date: 2021-07-19
author: "ChenRiang"
header-style: text
tags:
    - Java
---



Java's date time API can be quite confusing when you're new to it. Java introduced another new set of date time API to replace the legacy API in version 8. In this article, we will be looking into the new Java 8 date time API. 





# Introduction

The legacy date time API has been there for decades, and one of the biggest feature in Java 8 is the new date time API. You might be asking, since the old API has been established for so long, why Java still create another set of date time API?  



Here's why

- Legacy date time API are mutable and not thread safe.
- Java date time classed is not standardized , e.g. `Date` class in both `java.util` and `java.sql packages`, formatting and parsing classes are placed in `java.text` package.
- The poor API design that lack of day to day operation utilities method.
- Does not cater for time zone use case, developer need write additional logic to handle it.



# Date Time API Classes

In the following example, we will be looking into the creation of different date time objects available in Java 8 date time API.  You will notice that all of these classes have consistent API methods like `now()`, `parse()`, `of()`. 



The `now()` method of these classes has an overloaded method where we can specify ZoneId for getting dates in a specific time zone.

Get all the available list : 

```java
Set<String> availableZoneIds = ZoneId.getAvailableZoneIds();
System.out.println("available zone id = \n" + availableZoneIds);
```

 



### LocalDate

`LocalDate` is a class that represents a date with a default format of `yyyy-MM-dd`. 

```java
private static void localDateExample() {
        LocalDate now = LocalDate.now();
        System.out.println("Current Date = " + now);

        LocalDate secondDayInMarch2020 = LocalDate.parse("2020-03-02");
        System.out.println("second day in march 2020");

        LocalDate thirdDayInMarch2020 = LocalDate.of(2020, 3, 3);
        System.out.println("third day in march 2020 = " + thirdDayInMarch2020);

        LocalDate hundredDayIn2020 = LocalDate.ofYearDay(2020, 100);
        System.out.println("100th day in 2020 = " + hundredDayIn2020);

        LocalDate klDateNow = LocalDate.now(ZoneId.of("Asia/Kuala_Lumpur"));
        System.out.println("Current Date in Kuala Lumpur = " + klDateNow);

        LocalDate tokyoDateNow = LocalDate.now(ZoneId.of("Asia/Tokyo"));
        System.out.println("Current Date in Tokyo = " + tokyoDateNow);

        LocalDate chicagoDateNow = LocalDate.now(ZoneId.of("America/Chicago"));
        System.out.println("Current Date in Chicago = " + chicagoDateNow);
    }
```



Output:

```text
Current Date = 2021-07-19
second day in march 2020
third day in march 2020 = 2020-03-03
100th day in 2020 = 2020-04-09
Current Date in Kuala Lumpur = 2021-07-19
Current Date in Tokyo = 2021-07-19
Current Date in Chicago = 2021-07-19
```



### LocalTime

`LocalTime` is a class that represents a time in human readable format with format `hh:mm:ss.zzz`

```java
private static void localTimeExample() {
    LocalTime now = LocalTime.now();
    System.out.println("Current Time = " + now);

    LocalTime customTime = LocalTime.of(5, 30, 0,0);
    System.out.println("My custom time = " + customTime);

    LocalTime customTime2 = LocalTime.parse("05:45:00.000");
    System.out.println("My custom time2 = " + customTime2);

    LocalTime klTimeNow = LocalTime.now(ZoneId.of("Asia/Kuala_Lumpur"));
    System.out.println("Current Time in Kuala Lumpur = " + klTimeNow);

    LocalTime tokyoTimeNow = LocalTime.now(ZoneId.of("Asia/Tokyo"));
    System.out.println("Current Time in Tokyo = " + tokyoTimeNow);

    LocalTime chicagoTimeNow = LocalTime.now(ZoneId.of("America/Chicago"));
    System.out.println("Current Time in Chicago = " + chicagoTimeNow);

}
```



Output:

```text
Current Time = 15:50:09.394
My custom time = 05:30
My custom time2 = 05:45
Current Time in Kuala Lumpur = 15:50:09.394
Current Time in Tokyo = 16:50:09.394
Current Time in Chicago = 02:50:09.394
```





### LocalDateTime

`LocalTime` is a class that represents a combination of date and time with format `yyyy-MM-dd-HH-mm-ss.zzz`.

```java
private static void localDateTimeExample() {
    LocalDateTime now = LocalDateTime.now();
    System.out.println("Current Date Time = " + now);

    LocalDateTime customDateTime = LocalDateTime.of(2020, 3,19,5, 30);
    System.out.println("My custom Date Time = " + customDateTime);

    LocalDateTime customDateTime2 = LocalDateTime.parse("2020-03-19T06:30:00");
    System.out.println("My custom Date Time = " + customDateTime2);

    LocalDateTime klDateTimeNow = LocalDateTime.now(ZoneId.of("Asia/Kuala_Lumpur"));
    System.out.println("Current Date Time in Kuala Lumpur = " + klDateTimeNow);

    LocalDateTime tokyoDateTimeNow = LocalDateTime.now(ZoneId.of("Asia/Tokyo"));
    System.out.println("Current Date Time in Tokyo = " + tokyoDateTimeNow);

    LocalDateTime chicagoDateTimeNow = LocalDateTime.now(ZoneId.of("America/Chicago"));
    System.out.println("Current Date Time in Chicago = " + chicagoDateTimeNow);
}
```



Output:

```text
Current Date Time = 2021-07-19T16:29:28.399
My custom Date Time = 2020-03-19T05:30
My custom Date Time = 2020-03-19T06:30
Current Date Time in Kuala Lumpur = 2021-07-19T16:29:28.400
Current Date Time in Tokyo = 2021-07-19T17:29:28.400
Current Date Time in Chicago = 2021-07-19T03:29:28.400
```



### OffsetDateTime

`OffsetDateTime` class represents a date time with an offset from UTC/Greenwich in the ISO-8601 calendar system.



```java
private static void offsetDateTimeExample() {
    OffsetDateTime now = OffsetDateTime.now();
    System.out.println("Current Offset Date Time = " + now);

    OffsetDateTime customDateTime = OffsetDateTime.of(2020, 3,19,5, 30,0,0, ZoneOffset.of("+08:00"));
    System.out.println("My custom Offset Date Time = " + customDateTime);

    OffsetDateTime customDateTime2 = OffsetDateTime.parse("2020-03-19T06:30:00+08:00");
    System.out.println("My custom Offset Date Time = " + customDateTime2);

    OffsetDateTime klDateTimeNow = OffsetDateTime.now(ZoneId.of("Asia/Kuala_Lumpur"));
    System.out.println("Current Offset Date Time in Kuala Lumpur = " + klDateTimeNow);

    OffsetDateTime tokyoDateTimeNow = OffsetDateTime.now(ZoneId.of("Asia/Tokyo"));
    System.out.println("Current Offset Date Time in Tokyo = " + tokyoDateTimeNow);

    OffsetDateTime chicagoDateTimeNow = OffsetDateTime.now(ZoneId.of("America/Chicago"));
    System.out.println("Current Offset Date Time in Chicago = " + chicagoDateTimeNow);
}
```



Output:

```text
Current Offset Date Time = 2021-07-19T16:56:00.466+08:00
My custom Offset Date Time = 2020-03-19T05:30+08:00
My custom Offset Date Time = 2020-03-19T06:30+08:00
Current Offset Date Time in Kuala Lumpur = 2021-07-19T16:56:00.482+08:00
Current Offset Date Time in Tokyo = 2021-07-19T17:56:00.482+09:00
Current Offset Date Time in Chicago = 2021-07-19T03:56:00.484-05:00
```





### Instant

`Instant` is a class represents a date time with machine readable time format, Unix timestamp.

```java
private static void instantExample() {
    Instant now = Instant.now();
    System.out.println("Current timestamp = " + now);

    Instant currentInstant = Instant.ofEpochMilli(System.currentTimeMillis());.
        System.out.println(" = " + currentInstant);

}
```



```text
Current timestamp = 2021-07-19T09:28:22.691Z
Current timestamp2 2021-07-19T09:28:22.758Z
```



# Date Time Utilities API

The utilities API is the thing I most appreciate after migrating to Java 8 date time API. It provides lots of utility functions that cover most of the day to day operation scenario. 



### Date

For class that cover date such as `LocalDate` and `LocalDateTime` does provide some useful utility method that ease our job. 

Utility methods to :

- plus / minus day, month, year
- compare two dates object
- check if date is leap year

```java
public static void dateUtilityMethods() {

    LocalDateTime now1 = LocalDateTime.now();
    System.out.println("now = " + now1);

    LocalDateTime twoDaysLater = now1.plusDays(2);
    System.out.println("two days later = " + twoDaysLater);

    LocalDate now = LocalDate.now();
    System.out.println("now = " + now);

    LocalDate tomorrow = now.plusDays(1);
    System.out.println("Tomorrow = " + tomorrow);

    LocalDate twoWeekAfter = now.plusWeeks(2);
    System.out.println("2 week after today = " + tomorrow);

    LocalDate nextMonth = now.plus(1, ChronoUnit.MONTHS);
    System.out.println("1 month after today = " + tomorrow);

    DayOfWeek dayOfWeek = now.getDayOfWeek();
    System.out.println("Day of week = " + dayOfWeek.name());

    int dayOfYear = now.getDayOfYear();
    System.out.println("Day of year = " + dayOfYear);

    boolean isLeapYear = now.isLeapYear();
    System.out.println("Is Leap Year = " + isLeapYear);

    boolean notBefore = now.isBefore(tomorrow);
    System.out.println(now + "is before " + tomorrow + " = " + notBefore);

    boolean isAfter = nextMonth.isAfter(now);
    System.out.println(nextMonth + "is after " + now + " = " + isAfter);

}
```

  

Output:

```text
now = 2021-07-19T19:15:32.087
Truncate until days = 2021-07-19T00:00
now = 2021-07-19
Tomorrow = 2021-07-20
2 week after today = 2021-07-20
1 month after today = 2021-07-20
Day of week = MONDAY
Day of year = 200
Is Leap Year = false
2021-07-19is before 2021-07-20 = true
2021-08-19is after 2021-07-19 = true
```



In Java 8 package also include a utility class, [`Period`](https://docs.oracle.com/javase/8/docs/api/java/time/Period.html) that: 

- serve as a date unit for add / minus operation

- compare two dates object

  

```java
private static void periodUtilityClass() {
    LocalDate now = LocalDate.now();
    System.out.println("now = " + now);

    Period fiveDay = Period.ofDays(5);
    LocalDate fiveDaysLater = now.plus(fiveDay);
    System.out.println("5 days later = " + fiveDaysLater);

    Period twoMonth = Period.ofMonths(2);
    LocalDate twoMonthBefore = now.minus(twoMonth);
    System.out.println("2 months before = " + twoMonthBefore);

    Period oneYear = Period.ofYears(1);
    LocalDate oneYearAfter = now.plus(oneYear);
    System.out.println("1 year after = " + oneYearAfter);

    int daysBetween = Period.between(now, fiveDaysLater).getDays();
    System.out.println("Days difference between " + now + " and " + fiveDaysLater + " = " + daysBetween);

    int daysDiff1 = Period.between(now, oneYearAfter).getDays();
    System.out.println("Days difference between " + now + " and " + oneYearAfter + " = " + daysDiff1);

    long daysDiff2 = ChronoUnit.DAYS.between(now, oneYearAfter);
    System.out.println("Days difference between " + now + " and " + oneYearAfter + " = " + daysDiff2);

}
```

** Note: `betwee()`only calculates specific date unit that we request. e.g. if the date difference is exactly 1 year, the returned result will be 0.  (see `daysDiff1` and `dayDiff2` in the example)

Output:

```text
now = 2021-07-19
5 days later = 2021-07-24
2 months before = 2021-05-19
1 year after = 2022-07-19
Days difference between 2021-07-19 and 2021-07-24 = 5
Days difference between 2021-07-19 and 2022-07-19 = 0
Days difference between 2021-07-19 and 2022-07-19 = 365
```



### Time

For class that cover time such as `LocalTime` and `LocalDateTime` also provide some useful utilities method that ease our job. 

Utilities methods to :

- plus / minus hour, minutes, second, nanosecond 
- compare two times object
- min / max value. This will be useful when constructing database queries to find records within a given span of time.

```java
private static void timeUtilityMethods() {
    LocalDateTime now1 = LocalDateTime.now();
    System.out.println("now = " + now1);

    LocalDateTime oneHourBefore = now1.plusHours(1);
    System.out.println("one hour before" + oneHourBefore);

    LocalDateTime twoDaysLater = now1.plusDays(2);
    System.out.println("two days later = " + twoDaysLater);

    LocalDateTime truncatedToHours = now1.truncatedTo(ChronoUnit.HOURS);
    System.out.println("Truncate until hours = " +  truncatedToHours);

    LocalTime now = LocalTime.now();
    System.out.println("now = " + now);

    LocalTime twoHoursLater = now.plusHours(2);
    System.out.println("2 hours later = " + twoHoursLater);

    LocalTime thirtyBefore = now.minusMinutes(30);
    System.out.println("30 minutes before = " + thirtyBefore);

    LocalTime fourHoursBefore = now.minus(4, ChronoUnit.HOURS);
    System.out.println("4 hours before = " + fourHoursBefore);

    boolean isBefore = now.isBefore(twoHoursLater);
    System.out.println(now + " is before " + twoHoursLater + " = " + isBefore);

    boolean isAfter = twoHoursLater.isAfter(now);
    System.out.println(twoHoursLater + " is after " + now + " = " + isAfter);

    LocalTime maxTime = LocalTime.MAX;
    System.out.println("Max Time = " + maxTime);

    LocalTime minTime = LocalTime.MIN;
    System.out.println("Min Time = " + minTime);

}
```



Output:

```text
now = 2021-07-19T19:25:25.453
one hour before2021-07-19T20:25:25.453
two days later = 2021-07-21T19:25:25.453
Truncate until hours = 2021-07-19T19:00
now = 19:25:25.453
2 hours later = 21:25:25.453
30 minutes before = 18:55:25.453
4 hours before = 15:25:25.453
19:25:25.453 is before 21:25:25.453 = true
21:25:25.453 is after 19:25:25.453 = true
Max Time = 23:59:59.999999999
Min Time = 00:00
```



In Java 8 package also include a utility class, [`Duration`](https://docs.oracle.com/javase/8/docs/api/java/time/Duration.html) that: 

- serve as a time unit 
- compare two time object



```java
private static void durationUtilityClass() {
    LocalTime now = LocalTime.now();
    System.out.println("now = " + now);

    Duration twoHours = Duration.ofHours(2);
    LocalTime twoHoursLater = now.plus(twoHours);
    System.out.println("2 hours later = " + twoHoursLater);

    Duration fortyMinutes = Duration.ofMinutes(40);
    LocalTime fortyMinutesBefore = now.minus(fortyMinutes);
    System.out.println("40 minutes before = " + fortyMinutesBefore);

    Duration thirtySeconds = Duration.ofSeconds(30);
    LocalTime thirtySecondsLater = now.plus(thirtySeconds);
    System.out.println("30 seconds later = " + thirtySecondsLater);

    long hoursDiff = Duration.between(now, twoHoursLater).toHours();
    System.out.println("Hours difference between " + now + " and " + twoHoursLater + " = " + hoursDiff);

    long secondsDiff1 = Duration.between(now, fortyMinutesBefore).getSeconds();
    System.out.println("Second difference between " + now + " and " + fortyMinutesBefore + " = " + secondsDiff1);

}
```



Output:

```text
now = 19:31:34.322
2 hours later = 21:31:34.322
40 minutes before = 18:51:34.322
30 seconds later = 19:32:04.322
Hours difference between 19:31:34.322 and 21:31:34.322 = 2
Second difference between 19:31:34.322 and 18:51:34.322 = -2400
```





# Parsing and Formatting

Java 8 provide a formatter class, `DateTimeFormatter` that can be used to format all the Java 8 date time API.

### Parsing

```java
private static void parseExample(){

    LocalDate localDate = LocalDate.parse("2020/01/19", DateTimeFormatter.ofPattern("yyyy/MM/dd"));
    System.out.println(localDate);

    LocalTime localTime = LocalTime.parse("08-55", DateTimeFormatter.ofPattern("HH-mm"));
    System.out.println(localTime);

    LocalDateTime localDateTime = LocalDateTime.parse("27::05::2020 21::39::48",
                                                      DateTimeFormatter.ofPattern("d::MM::uuuu HH::mm::ss"));
    System.out.println(localDateTime);
}
```



Output:

```text
2020-05-27T21:39:48
2020-01-19
08:55
```

### Formatting

```java
private static void formatExample() {
    System.out.println("Local Date");
    LocalDate localDate = LocalDate.now();
    System.out.println(localDate.format(DateTimeFormatter.ofPattern("dd/MM/yyyy")));
    System.out.println(localDate.format(DateTimeFormatter.ofPattern("dd-MMM-yyyy")));
    System.out.println(localDate.format(DateTimeFormatter.ISO_DATE));

    System.out.println("\nLocal Time");
    LocalTime localTime = LocalTime.now();
    System.out.println(localTime.format(DateTimeFormatter.ofPattern("hh:mm:ss")));
    System.out.println(localTime.format(DateTimeFormatter.ofPattern("hh:mm")));
    System.out.println(localTime.format(DateTimeFormatter.ISO_TIME));


    System.out.println("\nLocal Date Time");
    LocalDateTime localDateTime = LocalDateTime.now();
    System.out.println(localDateTime.format(DateTimeFormatter.ofPattern("d::MMM::uuuu HH::mm::ss")));
    System.out.println(localDateTime.format(DateTimeFormatter.ISO_DATE_TIME));
}
```



Output:

```text
Local Date
19/07/2021
19-Jul-2021
2021-07-19

Local Time
09:09:12
09:09
21:09:12.845

Local Date Time
19::Jul::2021 21::09::12
2021-07-19T21:09:12.847
```





# Legacy API Support

Adopting Java 8 date time API is pretty straightforward if you are creating your application from scratch. Often time, migrate existing application to use new API is a nightmare. Fortunately, Java 8 release included some method in the legacy API to assist developer perform the migration. 



In `java.util.Date` there is a method `toInstant()` that will convert it to `Instant` object.

```java
private static void legacySupport() {
    Instant instant = new Date().toInstant();
    LocalDateTime now = LocalDateTime.ofInstant(instant, ZoneId.systemDefault());
    System.out.println("Date from legacy API = " + now);

    Date now2 = Date.from(Instant.now());
    System.out.println("Date from new API = " + now2);
}
```



Output:

```text
Date from legacy API = 2021-07-19T21:21:29.524
Date from new API = Mon Jul 19 21:21:29 SGT 2021
```





# Conclusion

The new Java 8 API is definitely a thumbs up feature that ease the development.  I will definitely recommend you to start using the new API, if you haven't done so. 



Check out the source code [here](https://github.com/lcr95/java-example/tree/main/java8-date-example).
