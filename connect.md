## Walkthrough of Zephyr 1.9.2's _zsock\_connect_ System call

### Source Code Commentary

#### zsock\_connect (subsys/net/lib/sockets/sockets.c:176)

```txt
180: Associates the remote address with the socket's net_context
     structure.

182: Reregisters the socket's connection handler to one with that
     contains a local address and a remote address.
```

#### net\_context\_connect (subsys/net/ip/net\_context.c:1368)

```txt
zsock_connect
    net_context_connect <-- Here

1392: bind_default returns 0 for contexts that have already been
      bound to a local address.

1481-1482: Copies the remote address from the addr argument to the
           context's remote field.

1484-1485: Assigns the addr argument's port and address family to
           the context.

1487-1491: Sets the NET_CONTEXT_REMOTE_ADDR_SET for specified remote
           addresses and clears it for unspecified remote addresses.
```
