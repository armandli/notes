### JVM Write Barrier - Card Marking
in theory, writing a reference field is the same as writing the same sized primitive, but in practice some accounting takes place to support GC, this accounting overhead is the write barrier.

in JVM, a **ordinary object pointer (OOP)** is the way JVM views java object references. they are pointer representations instead of actual pointers. Since objects are managed memory, OOP reads/writes may require a memory barrier of the memory management kind (instead of JMM ordering barrier kind).

a barrier is a block on read from or writing to certain memory locations by certain threads or processes. barriers can be implemented in either software or hardware. software barriers involve additional instructions around load and store operation, which would typically be added by a cooperative compiler. hardware barriers don't require compiler support and may be implemented in common operating systems using memory protection.

write barriers are used for incremental or concurrent garbage collection. they are also used to maintain remembered sets for generational collectors.

#### Card Marking
during GC, an object is considered alive if
* it is referenced by a live object
* if a static reference to it exists (part of root set)
* if a stack reference to it exists (part of root set)

in generational GC, tracing starts from the root set and does not cover older generation objects, which reduces the size of object graph, but this creates a problem: what if an object in the older generation references a younger object which is not traceable through any other chain of references to root ? we use card marking

the heap is divided into a set of cards, each of which is usually smaller than a memory page. the JVM maintains a card map, with one bit corresponding to each card in the heap. each time a pointer field in an object in the heap is modified, the corresponding bit in the card map for that card is set.

card marking could cause false sharing. to avoid false sharing in card marking, use option `UseCondCardMark` which checks if card if already marked, if it is, stop marking it. to prevent having writes invalidate cache line.

card marking also means G1GC will use more memory, so mixing up generations is painful.
