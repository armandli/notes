## Things to know about TCP
in TCP, bandwidth is a function of latency and time

### TCP 3 way handshake
1. sender pick a random sequence number x and send SYN to receiver with x
2. receiver increment x, choose random sequence number y and send a SYN/ACK packet
3. sender increment x and y, replies ACK packet and the first bytes of application data

the handshake introduce full round-trip time, which depends on the latency of the underlying network. TLS handshake requires up to 2 round trips. until TLS
connection is established, no application data is sent.

### TCP flow control
to prevent the sender from overwhelming the receiver, receiver stores incoming TCP packets waiting to be processed by the application into a receiverd buffer.
receiver coomunicate back to sender about the size of the buffer whenever it ack a packet. the sender use the size to avoid buffer overflow on receiver. the faster
the round trip time, the faster the sender adapts to receiver's capacity.

it also prevents sender from overwhelming the network. the sender measure the congestion empirically, maintaining a congestion window, representing total number of
outstanding packets that can be sent without waiting for ack. this is limited by receiver's window. smaller the window, fewer bytes will be in flight at any given
time.

when new connection is established, the size of the window is set to a system default, for every packet ack-ed, the window increase exponentially. the quicker the
underlying round trip time, the faster the sender adapts to fully utlize the underlying network's bandwidth. when packet is lost, detected through sender timeout,
congestion avoidance mechanism is kicked off, congestion window is reduced.

dividing the size of the congestion window by the round trip time gets the maximum theoretical bandwidth.
```
bandwidth = window_size / RTT
```

therefore in TCP, bandwidth is a function of latency.

### TCP Optimizations
1. bring servers closer to clients, reduce RTT
2. re-use connections to avoid the cold start penalty
