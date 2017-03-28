---
title: "Profile SQL statements with Sniffy"
description: "List your database execution time and count in your browser"
tags: java, sniffy, profiler, database, performance, sql
author: Nils Faupel
layout: article
---

## Profile SQL statements with Sniffy

Too many or too slow database queries can slow down the response time of your application, which in turn impacts the user experience.
Slow response times can cause your users to quit using your application.
Therefore you should always test the response time of your application in development in addition to functional tests.

But development systems have a smaller data set compared to the production system, making the response time fast in development but slow in production.
So how can a developer identify a potential slow part of the application if flawed behavior depends on a larger database?

# Identify SQL anti-pattern

[Sniffy](http://sniffy.io/) helps you to identify anti-patterns like the [N+1 query](https://secure.phabricator.com/book/phabcontrib/article/n_plus_one/) by listing all executed database queries of a web application directly in your browser.
If you want to play around and evaluate the features of Sniffy, you can visit a [live demo](http://demo.sniffy.io/owners?lastName=).

You should see a widget in the lower right corner showing you the loading time of the page and the number of executed SQL queries for the current site.

![sniffy-widget](../2017/sniffy-widget.png)

Clicking on it opens a detailed pop-up showing you the statements of the executed queries, the execution count per statement and the total execution time.
You can even open the stacktrace to find the place of the execution in your code and apply changes to optimize the query.

![sniffy-executed-queries](../2017/sniffy-executed-queries.png)

The listing of the executed queries combined with the execution count helps to identify anti-patterns like N+1 queries where you have a high execution count but a low number of returned rows per statement.
The anti pattern of loading several rows by ID in a loop can be replaced with one query with an IN statement in the where cause taking multiple parameters.
Even if the SQL execution would take the same time for both statements you do not add the latency between application server and database for each row, and the total execution time does not grow with each returned row.
If the latency between server and database is 1ms that would be 100ms for 100 rows and 1s for 1000 rows in total compared to 1ms for the batch loading if the execution time of the SQL statement is close to 0.
The second query in the picture is executed 13 times and the where clause is restricted by one ID. It should be replaced with an IN statement.


# Use Sniffy in your own application

To enable Sniffy in your own Java application follow the [setup guide](http://sniffy.io/docs/latest/#_datasource).
If your application is deployed on a Tomcat, it is as easy as this:

1. Download the `sniffy.jar` from the [release page on github](https://github.com/sniffy/sniffy/releases/latest) and save it in your `<TOMCAT-HOME>/lib` folder
2. Change your database URL to add the prefix `sniffy:` e.g. `jdbc:mysql://localhost:3306/platform` becomes `sniffy:jdbc:mysql://localhost:3306/platform`
3. Change your database DriverClass to be `io.sniffy.sql.SniffyDriver` e.g. replace `com.mysql.jdbc.Driver`
4. Add the filter definition and mapping to your `web.xml`
```XML
<filter>
    <filter-name>sniffer</filter-name>
    <filter-class>io.sniffy.servlet.SniffyFilter</filter-class>
    <async-supported>true</async-supported>
    <init-param>
        <param-name>enabled</param-name>
        <param-value>true</param-value>
    </init-param>
    <init-param>
        <param-name>inject-html</param-name>
        <param-value>true</param-value>
    </init-param>
</filter>
<filter-mapping>
    <filter-name>sniffer</filter-name>
    <url-pattern>/*</url-pattern>
</filter-mapping>
```

If you use Spring-Boot you simply add the `@EnableSniffy` annotation to your `@SpringBootApplication` annotated class and add the following dependency.

Maven
```XML
<dependency>
    <groupId>io.sniffy</groupId>
    <artifactId>sniffy-web</artifactId>
    <version>3.1.2</version>
</dependency>
````
Gradle
```
dependencies {
    compile 'io.sniffy:sniffy-web:3.1.2'
}
```

# Summary and outlook

Sniffy shows the total response time of RESTful requests and database request within the requests of one page directly in your browser.
In addition to the presented database feature that lists the execution count and time of SQL statements, Sniffy can block requests to other servers (e.g. database, 3rd party service) to test the fault tolerance of your application.
You can also integrate all the features of Sniffy in common [unit test frameworks](http://sniffy.io/docs/latest/#_unit_and_component_tests) to assert a maximum of executed statements.
Sniffy is under active development on [github](https://github.com/sniffy/sniffy) and distributed under [The MIT License](https://opensource.org/licenses/MIT), which makes it easy to use even for commercial projects.
In summary Sniffy is a nice tool to profile the response time of your application and is a free alternative to the commercial [XRebel](https://zeroturnaround.com/software/xrebel/) with a comparable feature set.
