---
title: "Trouble building maps with the Java Stream API"
description: "Pitfalls when rewriting for-loops with Java stream API and collect to Map"
tags: java, java8, stream, for, loop, map, hashmap, collect, exception, null
author: Nils Faupel
layout: article
image:
  feature: /2018/maps.jpg
  alt: A pile of maps - unrelated to the Java Map
---

Java 8 is around for a while now and has introduced the [JAVA 8 stream API](https://docs.oracle.com/javase/8/docs/api/java/util/stream/package-summary.html) making it easy to apply functions on every item of a stream.
Also the well known collection implementations can be handled as a stream resulting in less code written to apply the same functionality compared to a for-loop approach.
We will have a look on how to rewrite a simple for-loop to use the Java 8 stream API instead and explain two problems with it which can easily occur.

# Adding items from a `List` to a `Map`

Let's assume you have a `List` of `Person` objects with ID and name and you want to create a `Map` from ID to the name.
This is a common use case if you frequently want to access the name by the user's ID without iterating over the whole list over and over again.

```java
public class Person {
    private int id;
    private String name;
    //getters and setters omitted for simplicity
}
```

You could write the following code with a for-loop.

```java
List<Person> people; // value provided from an external class
Map<Integer, String> idToPerson = new HashMap<>();
for (Person person : people) {
    idToPerson.put(person.getId(), person.getName());
}
```

# Using Java 8 stream API

But let's rewrite this to use a Java 8 stream API to collect the map in a single line.

```java
Map<Integer, String> idToName = persons.stream()
        .collect(Collectors.toMap(Person::getId, Person::getName));
```

If you think this code does exactly the same, you are already wrong and should continue reading.

## Exceptional behavior on duplicated keys

Let's try out what happens if we have a duplicated `Person` in our input list.

```java
Person a = new Person(1, "alpha");
Person b = new Person(2, "beta");
List<Person> people = Arrays.asList(a, a, b);
Map<Integer, String> idToName = people.stream()
        .collect(Collectors.toMap(Person::getId, Person::getName));
```

We get an `IllegalStateException` saying we have a duplicate key (which is true).

```java
Exception in thread "main" java.lang.IllegalStateException: Duplicate key alpha
    at java.util.stream.Collectors.lambda$throwingMerger$113(Collectors.java:133)
    at java.util.stream.Collectors$$Lambda$3/764977973.apply(Unknown Source)
    at java.util.HashMap.merge(HashMap.java:1245)
    at java.util.stream.Collectors.lambda$toMap$171(Collectors.java:1320)
    at java.util.stream.Collectors$$Lambda$5/1595428806.accept(Unknown Source)
    at java.util.stream.ReduceOps$3ReducingSink.accept(ReduceOps.java:169)
    at java.util.Spliterators$ArraySpliterator.forEachRemaining(Spliterators.java:948)
    at java.util.stream.AbstractPipeline.copyInto(AbstractPipeline.java:512)
    at java.util.stream.AbstractPipeline.wrapAndCopyInto(AbstractPipeline.java:502)
    at java.util.stream.ReduceOps$ReduceOp.evaluateSequential(ReduceOps.java:708)
    at java.util.stream.AbstractPipeline.evaluate(AbstractPipeline.java:234)
    at java.util.stream.ReferencePipeline.collect(ReferencePipeline.java:499)
    ...
```

But the message does not contain the duplicated key which is `1`.
It describes the value for the duplicated key that is already in the `Map`, which is `"alpha"`.
If you don't know that the message contains the value and not the key, this can get really confusing when you are searching for that key to resolve the problem.
This behavior is tracked as a [bug in the OpenJDK](https://bugs.openjdk.java.net/browse/JDK-8173464) and is fixed in JDK9.

So where does this Exception come from?
If you look into the documentation [Collectors#toMap()](https://docs.oracle.com/javase/8/docs/api/java/util/stream/Collectors.html#toMap-java.util.function.Function-java.util.function.Function) you will find the following hint:
> If the mapped keys contains duplicates (according to Object.equals(Object)), an IllegalStateException is thrown when the collection operation is performed.
> If the mapped keys may have duplicates, use toMap(Function, Function, BinaryOperator) instead.

So the right way to implement the for-loop's behavior overwriting an existing value in the `Map` would be:

```java
Map<Integer, Person> idToName = persons.stream()
        .collect(Collectors.toMap(Person::getId, person -> person, (existingValue, newValue) -> newValue));
```

This will replace the existing value in the `Map` with the new value added if the key already exists.
So this code should do the same as the for-loop, right? Well, not quite.

## Always be careful with `null`

Let's see what happens when one of our Person's name is `null`.

```java
Person a = new Person(1, null);
Person b = new Person(2, "beta");
List<Person> people = Arrays.asList(a, b);
Map<Integer, String> idToName = people.stream()
        .collect(Collectors.toMap(Person::getId, Person::getName, (existingValue, newValue) -> newValue));
```

We get a `NullPointerException` and this time there is no hint about that behavior in the documentation.

```java
Exception in thread "main" java.lang.NullPointerException
    at java.util.HashMap.merge(HashMap.java:1216)
    at java.util.stream.Collectors.lambda$toMap$171(Collectors.java:1320)
    at java.util.stream.Collectors$$Lambda$5/1595428806.accept(Unknown Source)
    at java.util.stream.ReduceOps$3ReducingSink.accept(ReduceOps.java:169)
    at java.util.Spliterators$ArraySpliterator.forEachRemaining(Spliterators.java:948)
    at java.util.stream.AbstractPipeline.copyInto(AbstractPipeline.java:512)
    at java.util.stream.AbstractPipeline.wrapAndCopyInto(AbstractPipeline.java:502)
    at java.util.stream.ReduceOps$ReduceOp.evaluateSequential(ReduceOps.java:708)
    at java.util.stream.AbstractPipeline.evaluate(AbstractPipeline.java:234)
    at java.util.stream.ReferencePipeline.collect(ReferencePipeline.java:499)
    ...
```

The merge method of the `HashMap` used by the `Collectors#toMap()` does not allow values to be `null` although the `HashMap` allows `null` values in general to be stored in the map.

Whether you can rewrite every for-loop to a stream collect-based code style depends on your use case.
You also need to be really sure that you will never have `null` values.
Having that in mind and that the documentation does not always describe the actual behavior of the code in every edge case you should be sure what libraries you use and how they handle edge cases.
And, as always, it is a good idea to test your code with data that is as close to your production data as possible including edge cases.

This behavior is tracked as a [bug in the OpenJDK](https://bugs.openjdk.java.net/browse/JDK-8148463).
This is still not fixed in JDK17.
Alternative implementations to the for-loop are also discussed on [Stack Overflow](https://stackoverflow.com/questions/24630963/java-8-nullpointerexception-in-collectors-tomap/1515052).

<a style="background-color:#555;color:white;text-decoration:none;padding:4px 6px;font-family:-apple-system, sans-serif;font-size:12px;font-weight:bold;line-height:1.2;display:inline-block;border-radius:3px" href="https://unsplash.com/@andrewtneel?utm_medium=referral&amp;utm_campaign=photographer-credit&amp;utm_content=creditBadge" target="_blank" rel="noopener noreferrer" title="Download free do whatever you want high-resolution photos from Andrew Neel"><span style="display:inline-block;padding:2px 3px"><svg xmlns="http://www.w3.org/2000/svg" style="height:12px;width:auto;position:relative;vertical-align:middle;top:-1px;fill:white" viewBox="0 0 32 32"><title>unsplash-logo</title><path d="M20.8 18.1c0 2.7-2.2 4.8-4.8 4.8s-4.8-2.1-4.8-4.8c0-2.7 2.2-4.8 4.8-4.8 2.7.1 4.8 2.2 4.8 4.8zm11.2-7.4v14.9c0 2.3-1.9 4.3-4.3 4.3h-23.4c-2.4 0-4.3-1.9-4.3-4.3v-15c0-2.3 1.9-4.3 4.3-4.3h3.7l.8-2.3c.4-1.1 1.7-2 2.9-2h8.6c1.2 0 2.5.9 2.9 2l.8 2.4h3.7c2.4 0 4.3 1.9 4.3 4.3zm-8.6 7.5c0-4.1-3.3-7.5-7.5-7.5-4.1 0-7.5 3.4-7.5 7.5s3.3 7.5 7.5 7.5c4.2-.1 7.5-3.4 7.5-7.5z"></path></svg></span><span style="display:inline-block;padding:2px 3px">Andrew Neel</span></a>
