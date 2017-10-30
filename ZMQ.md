## Zero MQ
efficient, embeddable library that solves most of the problems an application need to become nicely elastic across a network with minimal cost.
* handles IO asynchronously in the background, communicate with application threads using lock-free data structures, concurrent ZMQ application has no locks, semaphores or wait states
* components can come and go dynamically and ZeroMQ will automatically reconnect. can create SOA where services can join and leave at any time
* can deal with over-full queue, can automatically block sender and throw away message
* talk using arbitrary transports: TCP, multicast in-process, inter-process.
* handles slow/blocked readers safely using different strategies that depend on messaging pattern
* allow routing of messages using variety of patterns such as request-reply, pub-sub, push-pull. allowing the creation of topology in network
* can create proxies to queue, forward, or capture messages with a single call. proxies can reduce interconnection complexity of a network
* delivers whole messages exactly as they were sent
* does not impose any format on message
* handle network errors intelligently, retrying automatically in case where it makes sense
* reuce carbon footprint
* uniform API that allows you to setup communication across "nodes" communicating on the same process, different process, different machine
* ZMQ 3.2 added new API methods: `zmq_disconnect`, `zmq_unbind`, `zmq_monitor`, `zmq_ctx_set`

tradeoffs:
* crash recovery is not free

#### ZMQ message format
messages in ZMQ are not null terminated, instead the size of the string is written before the string in the message. ZMQ transports raw bytes

### Messaging Mode: Server Client
```
zmq_socket(context, ZMQ_REQ); //client
zmq_socket(context, ZMQ_REP); //server
```

use the member fucntions `send` and `recv` to transport messages.

### Messaging Mode: Publish Subscribe
```
zmq_socket(context, ZMQ_PUB); //publisher
zmq_socket(context, ZMQ_SUB); //subscriber
```

publisher `bind` to a URI and `send` messages, while subscriber connect to the URI, subscribe to specific messages (filter using `zmq_setsockopt`) and `recv` message. if subscriber does not subscribe to anything, it will not receive anything (must set a pattern to look for in messages). subscriber can be subscribed to multiple publishers, and it uses round robin to prevent one publisher to choke other publishers. 

* publisher can handle multiple subscribers, if publisher has no connected subscriber, message will be dropped
* subscriber can subscribe to multiple publishers, data is interleaved between each publisher
* PUB-SUB is **asynchronous**, and likely the subscriber will miss the first message
* hacky solution is for publisher to sleep, but that's not the right solution
* it is an error to send on a subscriber socket and recv on a publisher socket
* subscriber will not receive **anything** if not not using subscribe pattern to filter for a pattern
* it is never clear when subscriber will actually start receive messages, first message is almost always missed
* this is true even publisher is created after subscriber, the first message from publisher will be missed, this is because before subscriber is connected to the publisher, the publisher may already send the message out
* message can get queued up in the publisher. need to **protect publisher** from high water mark
* ZMQ V3 filtering happens on the publisher side when using protocol tcp or ipc, but using epgm protocol, the filtering happens on the subscriber side. in ZMQ V2, all filtering happens on subscriber side

### Messaging Mode: Parallel Pipeline

```
zmq_socket(context, ZMQ_PUSH);
zmq_socket(context, ZMQ_PULL);
```

a model where a ventillator produces tasks that can be done in parallel, a set of workers tha does the task and a collector collects result from workers

* have to synchronize the start of the batch with all workers being up and running. there is no easy solution to avoid it. if ventillator push while not all workers are up and pulling, some worker will get all the work while other workers get none.
* push will push evenly to all connected workers, it is load balanced
* pull will collect results from all workers evenly

## Architecture of ZMQ
* ZMQ always have a context. It's a container for all sockets in a single process, and acts as transport for inproc sockets, which is the fastest way to connect threads within a process. Each context makes ZMQ behave as independent process. if each thread will behave as independent ZMQ entity, they should not have access to each other's context.
* if any socket is open, `zmq_ctx_destory` will hang forever, and `zmq_ctx_destroy` will wait forever if there are pending connects or sends unless you set **LINGER** to zero before closing
* 3 objects in the system: context, socket, message
* `zmq_send` and `zmq_recv` whenver you can, avoid **`zmq_msg_t`**
* if no use `zmq_msg_recv`, always release the received message as soon as you're done with it by calling **`zmq_msg_close`**
* avoid creating a lot of sockets, use simple design because sockets are not closed until context is destroyed.
* when exit the program, close the sockets and then call **`zmq_ctx_destroy`**, which destroys the context
* do not use the same socket from multiple threads
* need to shut down each socket that has ongoing requests, proper way is to set a low LINGER value.

## Socket Patterns
to create a connection, the "server" uses `zmq_bind` to bind to a URI and the client uses `zmq_connect` to connect to the server. ZMQ connections have the following characteristics:
1. go across arbitrary transport (inproc, ipc, tcp, pgm, epgm)
2. one socket can have many outgoing and incoming connections
3. no `zmq_accept`, accepting connection is automatic
4. network connection itself happen in the background, and ZMQ will automatically reconnect if network connection is broken
5. application oce cannot work with connections directly, they are encapsulated under socket

there is no rule that server has to start before client. when `zmq_connect` or `zmq_bind` is used, connection is created and nodes can write messages to the socket. 

binding of a socket to multiple URI is possible, even to different transport protocol. this means the socket will use multiple transport protocols. e.g.

```
zmq_bind(socket, "tcp://*:5555");
zmq_bind(socket, "tcp://*:9999");
zmq_bind(socket, "inproc://somename");
```

binding to the same endpoint is possible multiple times, usually used to allow process recover from crash.

difference between TCP and ZMQ Socket:
1. ZMQ socket carry message, with fixed length, not a stream of bytes
2. ZMQ do IO in the background. Message arrive in local input queues and are sent from local output queues, no matter what the application is doing
3. ZMQ sockets have 1toN behavior built-in.

sending and receiving messages go through a async queue and sent asynchronously. does not block except some exceptional case.

ZMQ provides both unicast transports: inproc, ipc, tcp, and multicast transports: epgm, pgm. 

**IPC** is disconnected, does not work on windows, and need users running on different nodes to have the same user access, may not be usable under different user ID.

**INPROC** is a connected signalling transport. it's faster but **the server must issue a bind before any client issue a connect**

ZMQ is not a **neutral carrier**, it imposes a framing on the transport protocol it uses. this framing is not compatible with existing protocols. Hoever there is an option **ZMQ_ROUTER_RAW** that allows to write data without framing. This can be used to work with other network protocol, such as HTTP.

By default, one context owns one IO thread, the general rule of thumb is to allow one IO thread per gigabyte of data in or out per second. to raise the number of IO threads, use `zmq_ctx_set` before creating any sockets. multi-threaded communciation can allow IO thread to be 0, but not a significant optimization.

#### Messaging Patterns
ZMQ patterns are implemented by pairs of sockets with matching types.

* **request-reply** - connect a set of clients and set of services. this is a remote procedure call and task distribution pattern
* **pub-sub** - connect set of publishers to set of subscribers. this is a data distribution pattern
* **pipeline** - connect nodes in a fan-out/fan-in pattern that can have multiple steps and loops. this is a parallel task distribution and collection pattern.
* **exclusive pair** - connect 2 sockets exclusively. this pattern is for connecting 2 threads in a process, not to be confused with normal pairs of sockets.

valid combinations:
* PUB SUB
* REQ REP
* REQ ROUTER (REQ here inserts an extra null frame)
* DEALER REP (REP assumes a null frame)
* DEALER ROUTER
* DEALER DEALER
* ROUTER ROUTER
* PUSH PULL
* PAIR PAIR
* XPUB XSUB (raw version of PUB SUB)

it is possible to bridge different socket pairs via code. i.e. read from one and write to another

#### Send and Recv Message
2 APIs
* `zmq_send` and `zmq_recv`, `zmq_recv` truncates message to whatever buffer size you provide
* `zmq_msg_t` architecture with richer but more difficult API:
  * `zmq_msg_init` `zmq_msg_init_size` `zmq_msg_init_data` for initialize a message
  * `zmq_msg_send` `zmq_msg_recv` for send and receive a message
  * `zmq_msg_close` for release a message
  * `zmq_msg_data` `zmq_msg_size` `zmq_msg_more` for access message content
  * `zmq_msg_get` `zmq_msg_set` for work with message properties
  * `zmq_msg_copy` and `zmq_msg_move` for message manipulation

in memory, ZMQ messages are `zmq_msg_t` structures with basic rules:
1. create and pass around `zmq_msg_t` objects, not blocks of data
2. use `zmq_msg_init` to create a `zmq_msg_t` object to be used to receive mesage using `zmq_msg_recv`
3. use `zmq_msg_init_size` to create a message for send, which allocate block of data for some size. fill the data with memcpy, then pass it to `zmq_msg_send`
4. release message call `zmq_msg_close`
5. access the content using `zmq_msg_data`, to know the size of the data use `zmq_msg_size`
6. **do not use** `zmq_msg_move` `zmq_msg_copy` or `zmq_msg_init_data` unless you read the man pages and know precisely why you need these
7. ZMQ clear the message when `zmq_msg_send` is called, which sets the data size to 0. you cannot send the same message twice. you cannot access the message after sending
8. these rules don't apply to `zmq_send` and `zmq_recv`, which takes or puts bytes of data instead of message structure

to send a mesage twice, first create another `zmq_msg_t` using `zmq_msg_init`, use `zmq_msg_copy` to create a reference of the copied message, then send out the copied message. the message will only be destroyed once all references are cleared.

ZMQ support **multipart messages**, which let you send and receive a list of frames as a single on-the-wire message.

**ZMTP** is the protocol ZMQ uses to read and write frames on a TCP connection.

ZMQ was designed for one frame per message, but was later changed. now a message can be one or more frames, which part is a `zmq_msg_t` object, you send and receive each frame separately in the low level API, higher level API provide wrappers to send the entire multi-part message.

* it is possible to send zero length message
* ZMQ quarantees to deliver all frames of a message, or none of them
* ZMQ does not send the message right away, any message, multi or single frame, must fit in memory
* using multi-part message will not reduce memory consumption

to read from multiple sockets, use **`zmq_poll`**

#### Multipart Message
when working with multi-part messages, each part is a `zmq_msg` item. sending a multi-part message uses multiple calls to `zmq_msg_send` e.g.

```
zmq_msg_send(&message, socket, ZMQ_SNDMORE);
zmq_msg_send(&message, socket, ZMQ_SNDMORE);
```

to receive, e.g.

```
while (1){
  zmq_msg_t message;
  zmq_msg_init(&message);
  zmq_msg_recv(&message, socket, 0);
  ...//process the message
  zmq_msg_close(&message);
  if (!zmq_msg_more(&message))
    break;
}
```

* when send a multi-part message, the first part are only actually sent on the wire when you send the final part.
* if `zmq_poll` is used, when the first part of the message arrives, the rest of the message also arrived
* all parts of the message will be received, or none at all
* each part of a message is a separate `zmq_msg_t`
* all parts of the message will arrive no matter the check for more property is done or not
* on sending, ZMQ queues message frames in memory until the last is received, then send them all
* there is no way to cancel a partially sent message, except by closing the socket

#### Dynamic Discovery Problem (XPUB and XSUB)
simple answer is to have a message broker, a static point where all other nodes knows and connect to. ZMQ does not have a built-in message broker, or a pub-sub proxy. the proxy opens a **XPUB** and a **XSUB** socket, then all publishers and subscribers connect through the proxy instead of each other.

XPUB and XSUB are like PUB and SUB except socket except they do subscription forwarding from subscribers to publishers, exposing subscriptions as special message.

#### Shared Queue (DEALER and ROUTER socket)
the scaled up version of REQ-REP pattern where we have multiple **stateless** services serving the same data. Only states that can exist are either inside the message itself or in some database.

we have a broker that polls on both REQ and REP, shuttles message between the sockets, does not queue anything (already queued on the REQ and REP socket ends).

REQ-REP is also synchronous pattern, when client sent REQ, it must wait for REP, sending another REQ will cause error. When having multiple services, we could not have synchronous pattern. the broker between services and clients will poll REQ and REP and distribute them, it will **not be synchronous**, or it is **non-blocking** whereas REQ-REP is **blocking**.

2 sockets: **DEALER** and **ROUTER** that does non-blocking REQ and REP. REQ will talk to ROUTER, while DEALER talks to REP, in between ROUTER and DEALER, there is code pushing an pulling from one end to the other. therefore the REQ-ROUTER acts as the front end queue connecting the proxy to clients, and the DEALER-REP acts as the backend queue connecting proxy to workers (servers).

#### ZMQ Proxy Function
ZMQ has builtin proxy that does the frontend and backend shared queue using a function **`zmq_proxy(frontend, backend, capture)`**, `zmq_proxy` accepts ROUTER/DEALER, XSUB/XPUB, PULL/PUSH

### Handling Errors and ETERM
philosophy of ZMQ error handling is it should be as vulnerable as possible to internal errors, and as robust as possible against external attack and error.

when ZMQ detects external fault, it returns error code. in some cases it drops the message silently if there is no strategy of recovery from the error.

error handling should be done whenever zmq functions are called.

2 main exception condition that should be handled as non-fatal:
1. when receiving a message with ZMQ_DONTWAIT option and there is no waiting data, ZMQ will return -1 with errno EAGAIN.
2. when one thread calls `zmq_ctx_destroy` and other threads are doing blocking work. the `zmq_ctx_destroy` destroys the blocking call and return -1 with errno set to ETERM.

### Multithreading
rules in using ZMQ for multi-threading:
1. isolate data within its thread and never share data across multiple threads. only exception is ZMQ context, which is thread-safe
2. don't use mutex, critical section, semaphores, these are anti-pattern in ZMQ
3. create ZMQ context at the start and pass it to all threads if inproc sockets are used
4. use **attached threads** to create structure within application, and connect these to their parent threads using PAIR socket over inproc. the pattern is: bind parent socket, then create child thread which connect to the socket
5. use **detached threads** to simulate independent task, with their own context. connect to these using tcp. later these threads can be moved to independent processes without much code change
6. all interactions between threads happen as ZMQ message
7. don't share ZMQ sockets between threads. ZMQ sockets are not thread safe.

**Do not use or close sockets except in the thread that created them** this applies when a thread is used as proxy. It's important that the front end and back end sockets for the proxy are created within the proxy. If it is created in other threads, initially it will work, but later it will fail randomly.

ZMQ uses native OS threads instead of virtual green threads. 

### Signaling Between Threads (PAIR)
to create exclusive communication between 2 threads in a process, use PAIR-PAIR pattern, which can be used to create tightly bounded thraeds with fast communcations, also structurally dependent.
* unlike PUSH-PULL, PAIR-PAIR is exclusive, there can only be 2 entities involved. PAIR will refuse additional connection.
* unlike ROUTER-DEALER, ROUTER wraps the message in an envelope, making a zero sized message into multi-part message, reading the data from ROUTER will give the wrong message. DEALER also distributes the message, which is more than what you need
* unlike PUB-SUB, the SUB have to configure an empty subscription in order to work like PAIR-PAIR
* pair sockets do not try to reconnect when remote node goes away and come back, which is not idealy when dealing with process/network communcation

PAIR-PAIR is the best method in coordinating between 2 exclusive threads.

### Node Coordination
PAIR-PAIR is great for inter-thread communication, but when nodes are running in different processes, we have to use a different pattern.

example a synchronized publish-subscribe. where publisher also opens a REQ-RES socket for all subscribers to signal if they are active. the publisher knows how many subscribers in total, and once all subscribers registered with the publisher, then publisher starts publishing. however the example also need to sleep between subscriber SUB and subscriber REQ-REP with publisher. this is because SUB request is asynchronous and may not complete before doing REQ-REP signal with the publisher, therefore can lose data still unless we know for sure SUB is complete before REQ-REP.

a more robust model is publisher sends hello message through PUB, and subscriber takes the same message and do REQ-REP with publisher one it receives the hello. publisher will only send real data once all hello are received by subscribers.

#### Zero Copy
ZMQ message API can send and receive messages directly from and to application buffer without copying data. it is called **zero-copy** for improving performance. This is best used on large blocks of memory that's frequently sent/received. small blocks or infrequent messaging does not receive significant benefit. Using zero-copy also makes code messier.

to use zero-copy, use `zmq_msg_init_data` to create a message that refers to a block of data already allocated, then pass it to `zmq_msg_send`. when creating the message, it will also take a function to free it when finished sending the message. with `zmq_msg_init_data` you don't have to call `zmq_msg_close`.

there is no way to do zero-copy on receive. ZMQ cannot write data directly into application buffers.

it works well with multi-part message.

### PUB-SUB Message Envelope
we can split the key into a separate message frame we call **envelope**. this is done by application manually. it is a cleaner solution for complicated systems.

subscriptions do prefix match, which means without envelope, it can match into the data instead of just the key. using a separate envelope frame prevents that as prefix match does not go beyond frame boundary.

it is also standard to add a separate frame for the publisher address in case subscriber wants to use other socket to talk to the publisher

### High Water Mark
in ZMQ V2, the HWM is set to be infinite, in ZMQ V3, it is set to 1000 by default.

when queue reaches HWM, it will either block or drop data depending on the type of the socket. PUB and ROUTER will drop data, while other types will block. in inproc sockets, the sender and receiver share the same buffer, therefore the real HWM is the sum of the HWM set for both side.

HWM is not exact, it depends on the raw buffer size, the real number of message could be much lower.

### Missing Message Problem Solver
There are multiple ways to miss message:
Socket Type | Solution
======================
SUB         | 1. check if subscription is set 2. make sure SUB is connected before PUB sends messages 3. slow joiner problem. SUB connect is async
----------------------
REQ and REP | check for return code. make sure to send and recv in a loop
----------------------
PUSH        | PULL can grab as many messages while others are still busy connecting, use a load balancing pattern ROUTER/DEALER sockets
----------------------
inproc      | 1. usable only if sharing the same context. 2. must bind before connect
----------------------
ROUTER      | 1. check reply sockets are valid, ZMQ drop message if it can't route 2. make sure identities are set before connect

make sure a socket is used only by one thread. 


