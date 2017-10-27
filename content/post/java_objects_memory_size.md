+++
date = "2017-10-27T16:40:07+03:00"
draft = false
title = "Java objects memory size"

+++

All numbers here are given for 64-bit JVMs. 

## Primitive types

 Type   | Size, bytes 
 -------|-------------
 byte   | 1           
 char   | 2           
 short  | 2           
 int    | 4
 long   | 8
 float  | 4
 double | 8

boolean is 1 bit.

## Size of objects

Generally, size of any Java object in memory is a sum of following: 

* object header
* memory for primitive fields
* memory for reference fields
* padding, usually multiple by 8 byte :w

### Object header
 
On 64-bit JVM object header has size 16 bytes and following form:

{{< gist iryndin 563ae369764d52224d6e3881226a21e3 "ObjectHeader64.txt" >}}

### Memory for primitive types

Memory for primitive types is allocated according to their respective sizes (see first section of this doc)

### Memory for reference fields

Each reference costs 8 bytes

### Example 

Consider following class:

```
class Person {
  String name;
  int age;
  boolean male;
}
```  

Let's calculate memory necessary for it: 

16 bytes (object header) + 8 bytes (reference to String variable _name_) + 4 bytes (int) + 1 byte (boolean) + 3 bytes (for padding) = 32 bytes


## Size of arrays

Now let's talk about arrays. In Java each array is a object, and hence each array has a 16-bytes object header. Then array has integer _length_
field, and this adds another 4 bytes. 3rd pard is array itself, I mean array cells, and memory required for it is calculated as number of these cells multiplied
by size of each cell. If it is array of primitive types, this means that each cell size will be in accordance with Section 1 of this doc, if it is 
array of objects this means that each cell size will be 8 bytes.

But referene to an array goes as a reference to an object and takes 8 bytes.

### Example 1. Arrays of primitives

Let's consider following array: **int[100]**. 16 bytes (object header) + 4 bytes (length) + 400 bytes (100 ints) + 4 (padding) = 424 bytes.

### Example 2. Arrays of booleans.

Let's consider following array: **boolean[10]**. 16 bytes (object header) + 4 bytes (length) + 16 bytes (100 ints) + 4 (padding) = 40 bytes. 

You see that it is quite not effective to use array of booleans, and for boolean only 1 bit is informational, and another 7 are wasted. 
Hence, if you need to use array of booleans please consider usage of [BitSet](https://docs.oracle.com/javase/8/docs/api/java/util/BitSet.html) for this.


### Example 3. Array of objects. 

Let's consider following array: **String[10]**. 16 bytes (object header) + 4 bytes (length) + 80 bytes (10 references to String objects) + 4 (padding) = 104 bytes.

## Links

Useful links:

* [Memory usage in Java](https://www.javamex.com/tutorials/memory/)
* [StackOverflow - Calculate size of Object in Java](https://stackoverflow.com/questions/9368764/calculate-size-of-object-in-java)
* [GitHub - Java Memory Measurer](https://github.com/DimitrisAndreou/memory-measurer)
* [Javaworld - Java Tip 130: Do you know your data size?](https://www.javaworld.com/article/2077496/testing-debugging/java-tip-130--do-you-know-your-data-size-.html)
* [Javaworld - Sizeof for Java](https://www.javaworld.com/article/2077408/core-java/sizeof-for-java.html)
* [Habr - Size of java objects](https://habrahabr.ru/post/134102/)
* [OpenJDK - CompressedOops](https://wiki.openjdk.java.net/display/HotSpot/CompressedOops)