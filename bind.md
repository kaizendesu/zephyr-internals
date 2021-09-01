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

2382: Calls recv_udp to set up a connection handler for the
      connection.
```

#### recv\_udp (subsys/net/ip/net\_context.c:2298)

```txt
zsock_bind
    net_context_bind
    net_context_get_type
    net_context_recv
        recv_udp <-- Here

2312-2315: Unregisters the context's connection handler if it already
           exists and sets it to NULL.

2334-2339: Copies the net_context structure's address to local_addr->sin_addr
           if the net_context structure has one set.

2341: Assigns the context structure's port to lport.

2347-2355: Registers the connection.
```

#### net\_conn\_unregister (subsys/net/ip/connection.c:348)

```txt
zsock_bind
    net_context_bind
    net_context_get_type
    net_context_recv
        recv_udp
            net_conn_unregister <-- Here

352-354: Returns -EINVAL early if conn is out of bounds of the conns array
         (subsys/net/ip/connection.c:48).

356-358: Returns -ENOENT if the connection is not in use.

360: Removes the connection from the connection cache.

365: Bzeroes the connection.
```

#### cache\_remove (subsys/net/ip/connection.c:325)

```txt
zsock_bind
    net_context_bind
    net_context_get_type
    net_context_recv
        recv_udp
            net_conn_unregister
                cache_remove <-- Here

329-339: Searches the conn_cache for an entry whose index matches
         the connection we want to remove and sets its index to -1.
```

#### net\_conn\_register (subsys/net/ip/connection.c:555)

```txt
zsock_bind
    net_context_bind
    net_context_get_type
    net_context_recv
        recv_udp
            net_conn_unregister
            net_conn_register <-- Here

567: Checks if an identical connection handler already exists and assigns
     its index to i.

569-573: Returns -EALREADY if an identical connection handler was found.

634-636: Copies the local_addr structure to conns[i].local_addr.

638-642: Sets the rank field with either NET_RANK_LOCAL_UNSPEC_ADDR or
         NET_RANK_LOCAL_SPEC_ADDR.

650: Sets the NET_CONN_LOCAL_ADDR_SET flag for the connection.

666-670: Assigns local_port to the connection's port and sets
         NET_RANK_LOCAL_PORT to rank.

672-676: Initializes the rest of the connection structure.

700-704: Assigns the address of the connection to the handle argument
         and returns 0.
```

#### find\_conn\_handler (subsys/net/ip/connection.c:446)

```txt
zsock_bind
    net_context_bind
    net_context_get_type
    net_context_recv
        recv_udp
            net_conn_unregister
            net_conn_register
                find_conn_handler <-- Here


455-457: Skips connections that are not in use.

459-461: Skips connections with a different protocol.

502-504: Skips connections for local addresses that do not have the
         NET_CONN_LOCAL_ADDR_SET flag set.

539-547: Skips connections that do not share the remote port or local
         port.

495-499: Skips the connection if the NET_CONN_REMOTE_ADDR_SET flag is set
         and the remote_addr argument is NULL.

533-537: Skips the connection if the NET_CONN_LOCAL_ADDR_SET flag is set
         and the local_addr argument is NULL.

549: Returns the index of the matching connection.
```

#### cache\_clear (subsys/net/ip/connection.c:276)

```txt
zsock_bind
    net_context_bind
    net_context_get_type
    net_context_recv
        recv_udp
            net_conn_unregister
            net_conn_register
                find_conn_handler
                    cache_clear <-- Here

280-283: Sets the index of every cache entry to -1 and sets the negative
         cache hit value to zero.
```
