---
title: "Java 8 stream collect to Map pitfalls"
description: "Pitfalls when rewriting for-loops with java streams and collect to Map"
tags: java, java8, stream, for, loop, map, hashmap, collect, exception, null
author: Nils Faupel
layout: article
---

# Adding items from a list to a map

Let's assume you have a `List` of `Person`s with ID and name and you want to create a `Map` from ID to the name.
This is a common use case if you frequently want to access the name by the user's ID without iterating over the whole list over and over again.

```java
public Class Person {
    private int id;
    private String name;
    //getters and setters omitted for simplicity
}
```

You could write the following code with a for-loop.

```java
List<Person> persons; // value provided from an external class
Map<Integer, String> idToPerson = new HashMap<>();
for (Person person : persons) {
    idToPerson.put(person.getId(), person.getName());
}
```

# Using Java 8 stream

But let's rewrite this to use a Java 8 stream to collect the map in a single line.

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
List<Person> persons = Arrays.asList(a, a, b);
Map<Integer, String> idToName = persons.stream()
        .collect(Collectors.toMap(Person::getId, Person::getName));
```

We get an `IllegalStateException` saying we have a duplicate key (which is true), but the message does not contain the duplicated key which is `1`.
It describes the value for the duplicated key that is already in the map, which is `"alpha"`.
If you don't know that the message contains the value and not the key, this can get really confusing when you are searching for that key to resolve the problem.

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

So where does this Exception come from?
If you look into the documentation [Collectors#toMap()] (https://docs.oracle.com/javase/8/docs/api/java/util/stream/Collectors.html#toMap-java.util.function.Function-java.util.function.Function-) you will find the following hint:
> If the mapped keys contains duplicates (according to Object.equals(Object)), an IllegalStateException is thrown when the collection operation is performed.
> If the mapped keys may have duplicates, use toMap(Function, Function, BinaryOperator) instead.

So the right way to implement the for-loop's behavior overwriting an existing value in the `Map` would be:

```java
Map<Integer, Person> idToName = persons.stream()
        .collect(Collectors.toMap(Person::getId, person -> person, (existingValue, newValue) -> newValue));
```

This will replace the existing value in the `Map` with the new value added if the key already exists.

## Always be careful with `null`

So this should do the same as the for-loop, right? Well, not quite.
Let's see what happens when one of our Person's name is `null`.

```java
Person a = new Person(1, null);
Person b = new Person(2, "beta");
List<Person> persons = Arrays.asList(a, b);
Map<Integer, String> idToName = persons.stream()
        .collect(Collectors.toMap(Person::getId, Person::getName, (existingValue, newValue) -> newValue));
```

We get a `NullPointerException` and this time there is no hint for that behavior in the documentation.

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

So it depends on your use case if you can rewrite every for-loop to a stream collect based code style and if you are really sure that you will never have `null` values.
Having that in mind and that the documentation does not always describe the actual behavior of the code in every edge case you should be sure what used libraries are doing and how they handle edge cases.
And as always it is a good idea to test your code with data that is as close to your productive data as possible including edge cases.

This behavior is tracked as a [bug in the OpenJDK](https://bugs.openjdk.java.net/browse/JDK-8148463)
and alternative implementations to the for-loop are also discussed on [Stack Overflow](https://stackoverflow.com/questions/24630963/java-8-nullpointerexception-in-collectors-tomap/1515052).
