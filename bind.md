## Walkthrough of Zephyr 1.9.2's _zsock\_bind_ System call

### Source Code Commentary

#### zsock\_bind (subsys/net/lib/sockets/sockets.c:159)

```txt
163: Binds the socket to the address/port contained in the
     addr argument.
```

#### net\_context\_bind (subsys/net/ip/net\_context.c:729)

```txt
zsock_bind
    net_context_bind <-- Here

741-743: Exits early if we are trying to rebind the socket to a
         new address/port.

844: Assigns the first network interface from the linker set to iface.

846: Assigns the address of the static const struct in_addr addr variable
     to ptr.

848: Assigns the address of the net_if_addr structure containing
     addr4->sin_addr.

854: Assigns the in_addr of the net_if_addr structure to ptr.

875: Assigns the network interface's index to context->iface.

877-878: Assigns the address family and IP address to the context argument.

879-889: Checks if the non-zero port number specified in the zsock_bind
         argument is in use and assigns it to the context if it is NOT
         in use. This is essentially reassigning the port number that was
         assigned in zsock_socket.

891-892: Assigns the port number from the network argument, which is the
         port that was assigned in zsock_socket, to addr4->sin_port.

901: Returns 0 in success.
```

#### net\_if\_get\_default (subsys/net/ip/net\_if.c:344)

```txt
zsock_bind
    net_context_bind
        net_if_get_default <-- Here

346-348: Returns NULL if there is no default network interface.

350: Returns the first network interface in the linker set.
```

#### net\_if\_ipv4\_addr\_lookup (subsys/net/ip/net\_if.c:1467)

```txt
zsock_bind
    net_context_bind
        net_if_ipv4_addr_lookup <-- Here

1476-1477: Skips any unicast network interface addresses that are not in
           use or are not AF_INET.

1481-1489: Assigns the network interface that contains a matching IP address
           to the ret argument and returns the address of the unicast entry
           containing the matching IP address.

1493: Returns NULL if no network interface contains a unicast address
      that matches the addr argument.
```

#### net\_context\_get\_type (include/net/net\_context.h:367)

```txt
zsock_bind
    net_context_bind
    net_context_get_type <-- Here

371-373: Returns SOCK_STREAM if NET_CONTEXT_TYPE is set.

375: Returns SOCK_DGRAM if NET_CONTEXT_TYPE is clear.
```

#### net\_context\_recv (subsys/net/ip/net\_context.c:2361)

```txt
zsock_bind
    net_context_bind
    net_context_get_type
    net_context_recv <-- Here

I will do this later.
```
