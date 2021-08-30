## Walkthrough of Zephyr 1.9.2's _zsock_socket_ System call

### Source Code Commentary

#### zsock\_socket (subsys/net/lib/sockets/sockets.c:76)

```txt
80: Obtains an net_context structure from the static context array located
     in subsys/net/ip/net_context.c on line 80.

82: Initializes the context's recv_q lists, which includes the data_q, wait_q.
    and poll_events list.

85: Returns a pointer to the context structure.
```

#### net\_context\_get (subsys/net/ip/net\_context.c:390)

```txt
zsock_socket
    net_context_get <-- Here

371-472: Checks for any unsupported protocol families and returns the
         appropriate error if found.

476-479: Searches for the first unused network context.

497: Initializes the context flags to 0.

500: OR's NET_CONTEXT_FAMILY into context->flags for ipv6.

501: OR's NET_CONTEXT_TYPE into context->flags for SOCK_STREAM.

502: OR's NET_CONTEXT_PROTO into context->flags for IPPROTO_TCP.

507-508: Bzeroes the remote and local structures within contexts.

511-520: Obtains a free port for the ipv6 sockaddr_in6 structure.

524-533: Obtains a free port for the ipv4 sockaddr_in structure.

540: Assigns the address of the contexts array entry to our the context
     double pointer argument.

542: Assigns ret to zero.

567: Returns ret.
```

#### find\_available\_port (subsys/net/ip/net\_context.c:368)

```txt
zsock_socket
    net_context_get
        find_available_port <-- Here

375: Assigns a random port number to local_port, OR'ing 0x8000 to ensure
     that the port is at least 32,768.

384: Returns the free port number we just found in network-byte order.

387: Returns the port number that is already in use.
```

#### net\_context\_get\_ip\_proto (include/net/net\_context.h:411)

```txt
zsock_socket
    net_context_get
        find_available_port
            net_context_get_ip_proto <-- Here

415-416: Returns IPPROTO_TCP if NET_CONTEXT_PROTO is set.

419: Returns IPPROTO_UDP if NEXT_CONTEXT_PROTO is cleared.
```

#### check\_used\_port (subsys/net/ip/net\_context.c:328)

```txt
zsock_socket
    net_context_get
        find_available_port
            net_context_get_ip_proto
            check_used_port <-- Here

336-338: Skips any unused unused net_context structures.

340-344: Checks if local_port is already in use by searching for any
         net_context structure with the same IP protocol as the ip_proto
         argument whose sin_port == local_port.

346-362: Checks if local_addr is already in use by searching for any
         net_context structure with the same address family whose
         sin6_addr/sin_addr == local_addr.

365: Returns zero is the address/port combination is not in use.
```

#### k\_queue\_init (kernel/queue.c:50)

```txt
zsock_socket
    net_context_get
    k_fifo_init
        k_queue_init <-- Here

52: Initializes the singly linked data_q list.

53: Initializes the doubly linked wait_q list.

55: Initializes the doubly linked poll_events list.
```

#### SYS\_TRACING\_OBJ\_INIT (include/debug/object\_tracing\_common.h:34)

```txt
zsock_socket
    net_context_get
    k_fifo_init
        k_queue_init
            SYS_TRACING_OBJ_INIT <-- Here


39-40: Inserts the queue object at the head of the _trace_list_k_queue list.
```

