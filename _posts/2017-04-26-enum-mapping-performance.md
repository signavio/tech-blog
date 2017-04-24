---
title: "ORM enum mapping performance"
description: "Comparing the performance of string and ordinal mapping of enums"
tags: enum, string, ordinal, mapping, orm, jpa, sql, performance, benchmark, hibernate, spring-boot, sniffy
author: Nils Faupel
layout: article
---

# The performance impact of enum mapping with JPA

The [Java Persistence API (JPA)](http://www.oracle.com/technetwork/java/javaee/tech/persistence-jsp-140049.html) provides a model for object-relational mapping to persist Plain Old Java Objects (POJO) in a relational database.
This Java EE standard has different implementations, for example the reference implementation [EclipseLink](https://www.eclipse.org/eclipselink/#jpa), but also more widely used is [Hibernate](http://hibernate.org/orm/) e.g. as the default implementation for [Spring-Boot](https://projects.spring.io/spring-boot/).
The JPA standard provides two options to persist an enum as string or ordinal. This blog post will compare both options against each other with a focus on their read performance.

# Mapping of enums

An enum typed attribute of a class must be annotated with `@Enumerated(EnumType.STRING)` to store the `EnumValue.name()` string representation of the enum in the database
or use `@Enumerated(EnumType.ORDINAL)` to store the `EnumValue.ordinal()` ordinal integer representation of the enum.
`@Enumerated` without a specific value will default to `@Enumerated(EnumType.ORDINAL)`.
E.g. we have an enum with the values `VALUE_A, VALUE_B, VALUE_C` and an object holding the value `VALUE_B` in an attribute.
The `VALUE_B` of the object could be mapped as string `VALUE_B` or ordinal `1` (indexed based enumerating starts by 0) to the database.
Both values will be mapped back to the `VALUE_B` of the enum when loaded from the database.


# The test setup

I created a simple example, that you can find on [GitHub](https://github.com/nfaupel/performance-enum), to measure the read performance for both options of persisting an enum.
The setup consists of one enum with ten different values and one class, that stores always the same enum in four attributes to persist them as string and ordinal both with and without and index on the column in the database.

```java
@Entity @Table(indexes = {
        @Index(name = "performanceEnumStringIndexed", columnList = "performanceEnumStringIndexed"),
        @Index(name = "performanceEnumOrdinalIndexed", columnList = "performanceEnumOrdinalIndexed")})
public class PerformanceEntity {
    @Id @GeneratedValue
    private Long id;
    @Enumerated(EnumType.STRING)
    private PerformanceEnum performanceEnumString;
    @Enumerated(EnumType.STRING)
    private PerformanceEnum performanceEnumStringIndexed;
    @Enumerated(EnumType.ORDINAL)
    private PerformanceEnum performanceEnumOrdinal;
    @Enumerated(EnumType.ORDINAL)
    private PerformanceEnum performanceEnumOrdinalIndexed;
    //getters and setters
```

You can repeat the measurement on your own computer if you follow the instructions in the readme of the repository.
The queries select all rows in the database matching one given enum value.
Because all entries are created always with the next enum value, each query selects 10% of the total existing entries.
All queries have the same result set but filter on the different columns.
I will look at the average execution time of the SQL query out of 100 iterations.
Before taking the actual measurement, the Just In Time (JIT) compiler of the Java Virtual Machine (JVM) is also warmed up with 100 iterations.

The runtime of the queries is measured by [Sniffy](http://sniffy.io/), that I introduced in my last [blog post](https://tech.signavio.com/2017/sniffy-database-profiler).
This is a very basic setup and does not take every detail into account you can consider when creating a performance measurement like deactivating [Turbo Boost](https://en.wikipedia.org/wiki/Intel_Turbo_Boost) or eliminating the impact of garbage collection.

# The results

Let the numbers speak for themselves.

|Query execution time in ms|100,000 entries|300,000 entries|
|--------------------------|--------------:|--------------:|
|String  without index     |           53.7|          174.8|
|Ordinal without index     |           51.4|          160.7|
|String  with index        |           27.1|           90.0|
|Ordinal with index        |           23.5|           75.2|

Without an index on the column the ordinal persistence is about 5-8% faster to select depending on the size of the table.
With an index added both strategies return their results fast, as expected.
But the ordinal representation benefits even more from the usage of an index as it is now about 13-16% faster to select compared to the string representation.

# The explanation

The observed effect can be explained based on the used implementation in the database.
The ordinal representation is mapped by default to an `int(11)` column using four bytes to store one value.
The string representation is mapped by default to a `varchar(255)` column using up to three bytes per character to store the value depending on the character set (UTF-8 in our setup).
You can read more about the size needed to store a value in the different data types in the [MySQL Reference Manual](https://dev.mysql.com/doc/refman/5.7/en/data-types.html).

The column length definition could be shortened for both mapping strategies.
E.g. `@Column(length = 32)` allowing 32 characters for the string mapping or `@Column(columnDefinition = "TINYINT(1)")` allowing one byte worth 127 values for the ordinal mapping.
But this has a minimal effect on the execution runtime, because this describes only the maximum allowed size to store one value and has a minor impact on the actual used size that is already optimized based on the actual value.

For selecting the rows the values in the column need to be compared.
You can see that the string representation needs far more bytes to store the same enum representation than the integer representation needs, because the whole string is stored instead of just one number.
This means more operations to load and compare the value from the column in the CPU and this results in more time needed to execute the string based query.

# Pros and Cons to consider

As we have seen, the ordinal mapping of enums results in a better read performance (and also not measured better write performance as less bytes need to be stored) on the SQL database.
But the ordinal representation has also a big drawback.
Whenever you add new values to your enum, these values must be appended at the end, if you do not want to migrate existing values.
Otherwise you change the mapping of already persisted values when they are loaded from the database, because on the ordinal position in the enum you may added a new value and shifted all existing values one position downwards.
In this way the enum cannot reflect a logical order of all values for the developer after adding additional values after the first release, without the overhead of a migration of existing values.

When using the string mapping you can add values and rearrange their order at any time, without the need for a migration, because the full string representation of the enum is written to the database.
This makes it also easier to understand the values in your database if you ever access your table without the mapping of your enum by hand.
On the other hand you can easily rename an ordinal mapped enum value as long as you do not change the order, because the database stores only the position and not the name.
If you rename a value of a string mapped enum, you need to migrate your existing values in the database, otherwise they cannot be mapped back to an enum because their representation was removed.

# Conclusion

We have seen that the ordinal mapping of enums to the database result in a 5-16% better read performance compared to the mapping as a string.
You have to consider for your own application if this performance gain is worth the drawback that the enum can no longer reflect a logical order after new values have been added to the enum.
Also the values in the table are not understandable without the knowledge from the mapping of the enum.
Now you make your decision based on numbers instead of guessing and remember that the default mapping of `@Enumerated` is ordinal if you do not make it explicitly `@Enumerated(EnumType.STRING)`.
Also indexing the column used in the `where` clause has a far bigger impact on the read performance than changing the column type.
