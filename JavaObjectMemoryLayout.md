### Java Object Memory Layout
#### rules
* objects are 8 byte aligned in memory, thus any object addresses are divisible by 8
* all fields are type aligned, long/double are 8 aligned, integer/float are 4, short/char are 2
* fields are packed in the order of their size, except for references which are last
* classes fields are never mixed, so if B extends A, object of class B will be laid out in memory with A's fields first, then B
* sub class fields start at 4 byte alignment
* if first fiel of a class is long/double and the last starting point is not 8 aligned, then smaller field may be swapped to fill the 4 bytes gap

java does not respect user's ordering of class fields because
* unaligned access are bad, so JVM saves user from bad layout
* naive layout of fields could waste memory, JVM finds a way to improve overall size of the object
* JVM implementation requires type to have consistent layout, thus having sub-class rules

java reserves the right to change object layout. in fact, object layout was changed between Java 6 and Java 7

problem with Java memory object layout:
* false false sharing protection

you can test for false sharing using UnsafeAccess

you can obtain java object layout using openJDK jol tool

way to manipulate java object layout: use base class to group hot fields that should be accessed together

[Reference](psy-lob-saw.blogpost.com)
