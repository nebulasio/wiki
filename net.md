# Net Design Doc

## Network Protocol

We build our peer-to-peer network connection based on libp2p and implement the basic peer-to-peer connection, routing table update, node discovery and Simple implementation of message broadcast.

We always use Big-Endian for our network transport protocol.

In go-nebulas version 0.1.0, we just build a simplest implementation of p2p network.we defined several kinds of protocols  for different scenarios.

Perhaps we just need one protocol in the future, we can put the content of protocol to the packet data. so we only need to parse the message body to know the specific protocol.

In go-nebulas version 0.2.0, we will define specific protocol formats.
```
 0               1               2               3              (bytes)
 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                         magic number                          |
+------------------------------------------------+------+-------+
|                   Chain ID                     | msg_l| msg_v |
+------------------------------------------------+------+-------+
|                         message name                          |
+---------------------------------------------------------------+
|                          data_length                          |
+---------------------------------------------------------------+
|                         data_checksum                         |
+---------------------------------------------------------------+
|                        header_checksum                        |
|---------------------------------------------------------------+
|                                                               |
+                             data                              +
|                                                               |
+---------------------------------------------------------------+

magic number: 32 bits(4 CHAR) 
The protocol magic number, A constant numerical or text value used to identify protocol, such as "NEB1".

Chain ID: 24 bits
The Chain ID is used to distinguish the test network and the main network.

msg_l: 4 bits
The msg_l is the length of message name.

msg_v: 4 bits
The msg_v is the specific version of protocol.

message name: (variable length)
The identification or the message name of protocol.

data_length: 32 bits
The total length of packet data

data_checksum: 32 bits
The checksum of the packet data

header_checksum: 32 bits
The checksum of the whole protocol header except header_checksum

data: (variable length)
The packet data.
```

## Message List

* Hello

is used to build connection and shake hands between two peers.

```
version: 0x1

data: struct {
    string node_id  // eg.
    string client_version // x.y.z, eg 0.1.0
}

resp message: Olleh
```

* Olleh

response for Hello.

```
version: 0x1

data: struct {
    string node_version
    string node_id
}
```

* Bye

close a connection.

```
version: 0x1
data: struct {
    string reason
}
```

* GetRoutes

send message for routing table update.

```
version: 0x1
```

* Routes

response for GetRoutes.

```
version: 0x1
data: struct {
    []string peer_ids
}
```


## Network architecture
Our network simply provides the simplest peer-to-peer data propagation without the business. we threw our specific business to the top dispatcher. we use p2p_manager to manage our p2p message and message broadcast.