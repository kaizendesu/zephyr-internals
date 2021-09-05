## Walkthrough of Zephyr 1.9.2's _zsock\_send_ System call

### Source Code Commentary

#### zsock\_send (subsys/net/lib/sockets/sockets.c:214)

```txt
227: Allocates a net_pkt structure to send.

236-243: Caps the length of the packet data to max_len minus the size
         of the TCP header.


```

#### net\_pkt\_get\_debug (subsys/net/ip/net\_pkt.c:495)

```txt
zsock_send
    net_pkt_get_tx
        net_pkt_get_tx_debug
            net_pkt_get_debug <-- Here

526: Calls net_pkt_get_reserve to slab allocate a net_pkt structure.

529-537: Initializes the net_pkt structure's context, interface, and
         address family.

539: Returns the pointer to the packet.
```

#### net\_pkt\_get\_reserve\_debug (subsys/net/ip/net\_pkt.c:305)

```txt
zsock_send
    net_pkt_get_tx
        net_pkt_get_tx_debug
            net_pkt_get_debug
                net_pkt_get_reserve <-- Here

313-317: Slab allocates a net_pkt structure, passing K_NO_WAIT to calls
         made during an interrupt service routine and passing timeout
         (K_FOREVER) otherwise.

323: Bzeroes the net_pkt structure.

327-328: Sets the packet's refcount to 1 and points it to its slab.

337: Returns the pointer to the packet.
```

#### net\_pkt\_append (subsys/net/ip/net\_pkt.c:1169)

```txt
zsock_send
    net_pkt_get_tx
    net_pkt_append <-- Here

1178-1185: Obtains a fragment if the packet's fragment list is empty
           and adds it.

```

#### net\_pkt\_get\_frag\_debug (subsys/net/ip/net\_pkt.c:388)

```txt
zsock_send
    net_pkt_get_tx
    net_pkt_append
        net_pkt_get_frag_debug <-- Here

406: Calls net_pkt_get_reserve_data_debug to obtain the fragment.
```

#### net\_pkt\_get\_reserve\_data\_debug (subsys/net/ip/net\_pkt.c:341)

```txt
zsock_send
    net_pkt_get_tx
    net_pkt_append
        net_pkt_get_frag_debug
            net_pkt_get_reserve_data_debug <-- Here

359-363: Net buf allocates a fragment, passing K_NO_WAIT for calls made in
         an interrupt service routine and passing timeout (K_FOREVER)
         otherwise.

381: Returns the pointer to the fragment.
```

#### net\_pkt\_alloc\_add (subsys/net/ip/net\_pkt.c:128)

```txt
zsock_send
    net_pkt_get_tx
    net_pkt_append
        net_pkt_get_frag_debug
            net_pkt_get_reserve_data_debug
                net_pkt_alloc_add <-- Here

134-136: Skips net_pkt_allocs entries that are already in use.

138: Sets the first unused net_pkt_allocs entry as in use.

139: Assigns the packet that owns the allocation.

141: Assigns the allocation's function.

144: Returns true if we successfully initialized a net_pkt_allocs entry
     to correspond with the recent allocation.
```

#### net\_pkt\_frag\_add\_debug (subsys/net/ip/net\_pkt.c:861)

```txt
zsock_send
    net_pkt_get_tx
    net_pkt_append
        net_pkt_get_frag_debug
        net_pkt_frag_add_debug <-- Here

873-876: Assigns the frag argument to the packet's frag field and returns.
```

#### net\_pkt\_append\_bytes (subsys/net/ip/net\_pkt.c:1137)

```txt
zsock_send
    net_pkt_get_tx
    net_pkt_append
        net_pkt_get_frag_debug
        net_pkt_frag_add_debug
        net_pkt_append_bytes <-- Here

1146: Increments the fragment's len field by count.

1148: Copies count bytes of data from the user buffer into the fragment.

1149: Decrements len by count.

1150-1151: Increments added_len and the user buffer by count.

1153-1155: Returns added_len when finished copying data from the user buffer.

1157-1162: Net buf allocates another fragment to copy more data from the
           user buffer into the packet.
```

#### net\_context\_send (subsys/net/ip/net\_context.c:2149)

```txt
zsock_send
    net_pkt_get_tx
    net_pkt_append
    net_context_send <-- Here

2169-2172: Returns -EDESTADDREQ if the net context's remote address is not
           set.

2181-2182: Sets the address length of an IPv4 address.

2189: Calls and returns sendto.
```

#### sendto (subsys/net/ip/net\_context.c:2050)

```txt
zsock_send
    net_pkt_get_tx
    net_pkt_append
    net_context_send
        sendto <-- Here

2078-2080: Returns -EDESTADDREQ is the dst_addr is NULL.

2124-2125: Calls create_udp_packet.

2146: Calls and returns send_data.
```

#### create\_udp\_packet (subsys/net/ip/net\_context.c:1997)

```txt
zsock_send
    net_context_send
        sendto
            create_udp_packet <-- Here

```

#### net\_ipv4\_create (subsys/net/ip/ipv4.c:62)

```txt
zsock_send
    net_context_send
        sendto
            create_udp_packet
                net_ipv4_create <-- Here

```

#### net\_ipv4\_create\_raw (subsys/net/ip/ipv6.c:610)

```txt
zsock_send
    net_context_send
        sendto
            create_udp_packet
                net_ipv4_create
                    net_ipv4_create_raw <-- Here

```

#### net\_udp\_insert\_raw (subsys/net/ip/udp.c:47)

```txt
zsock_send
    net_context_send
        sendto
            create_udp_packet
            net_udp_insert
                net_udp_insert_raw <-- Here

```

#### net\_ipv4\_finalize\_raw (subsys/net/ip/ipv4.c:85)

```txt
zsock_send
    net_context_send
        sendto
            create_udp_packet
            net_udp_insert
            net_ipv4_finalize
                net_ipv4_finalize_raw <-- Here

```

#### net\_send\_data (subsys/net/ip/net\_core.c:287)

```txt
zsock_send
    net_context_send
        sendto
        send_data
            net_send_data <-- Here

```

#### net\_if\_send\_data (subsys/net/ip/net\_if.c:248)

```txt
zsock_send
    net_context_send
        sendto
        send_data
            net_send_data
                net_if_send_data <-- Here

```

