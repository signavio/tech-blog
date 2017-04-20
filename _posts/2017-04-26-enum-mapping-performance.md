---
title: "ORM Enum mapping performance"
description: "Comparing the performance of string and ordinal mapping of enums"
tags: enum, string, ordinal, mapping, orm, jpa, sql, performance, benchmark, hibernate, spring-boot, sniffy
author: Nils Faupel
layout: article
---

# The performance impact of enum mapping with JPA

The Java Persistence API (JPA) provides a model for object-relational mapping to persist Plain Old Java Objects (POJO) in a relational database.
This Java EE standard has different implementations for example the reference implementation EclipseLink, but more widely used is Hibernate e.g. as the default implementation for Spring-Boot.
JPA standard provides us two options to persist an enum as String or Ordinal. This blog post will compare both options against each other with a focus on their read performance.

# Mapping of enums

An enum attribute of a class must be annotated with @Enumerated(EnumType.STRING) to store the .name() representation of the enum in the database
or use @Enumerated(EnumType.ORDINAL) to store the .ordinal() representation.
@Enumerated will default to @Enumerated(EnumType.ORDINAL).


# The test setup

I created a simple example, that you can access on GitHub, to measure the read performance for both options of persisting an enum.
The setup consists of one class, that stores the same enum in four attributes to persist them as String and Ordinal with and without and index on the column in the database.
// Entity code snippet here.
You can repeat the measurement on your own computer if follow the instructions in the readme of the repository.
I will look at the average execution time of the SQL query out of 100 iterations.
Before taking the actual measurement, the Just In Time (JIT) compiler of the Java Virtual Machine (JVM) is also heated up with 100 iterations.
The runtime of the queries is measured by sniffy, that I introduced in my last blog post.
This is a very basic setup and does not take every detail into account you can consider when creating a performance measurement like deactivating Turbo Boost or eliminating the impact of garbage collection.

# The measurements

Let the numbers speak for them self.
//insert table of execution time here

Without an index on the column the ordinal persistence is about 10% faster.
With an index added both strategies return their results fast, as expected.
But the ordinal representation benefits even more from the usage of an index as it is now about 15% faster than the String representation.

# The explanation

The observed effect can be explained based on the used implementation.
The ordinal representation is mapped by default to an integer(11) column using four bytes to store one value.
The String representation is mapped by default to a varchar(255) column using up to three bytes per character to store the value depending on the character set (UTF-8 in our setup).
So for selecting the rows the values in the column need to be compared.
And you can see that the string representation needs far more bytes to a store for the same enum representation, because the whole String is stored instead of just one number.
This means more operations to load and compare the value on the CPU and this results in more time needed to execute the string based query.

# Pros and Cons to consider

As we have seen, the ordinal mapping of enums results in a better read performance (and also not measured write performance) on the SQL database.
But the ordinal representation has also a big drawback.
Whenever you add new values to your enum, these values must be appended at the end.
Otherwise you change the mapping of already persisted values when they are loaded from the database,
because on the ordinal position in the enum you may added a new value and shifted all existing values one position downwards.
In this way the enum cannot reflect a logical order of all values for the developer over time.
When using the String mapping you can add values and rearrange their order at any time, because the full String representation of the enum is written to the database.
This makes it also easier to understand the values in your database if you ever access your table without the mapping of your enum by hand.

# Conclusion

We have seen that the ordinal mapping of enums to the database result in a 10-15% better read performance compared to the mapping as a string.
You have to consider for your own application if this performance gain is worth the drawback that the enum can no longer reflect a logical order after new values have been added to the enum.
Also the values in the table are not understandable without the knowledge from the mapping of the enum.
Now you make your decision based on numbers instead of guessing and remember that the default mapping of @Enumerated is ordinal if you do not make it explicitly @Enumerated(EnumType.STRING).

