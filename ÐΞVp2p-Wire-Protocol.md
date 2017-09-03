---
name: DEV P2P Wire Protocol
category: 
---

Peer-to-peer communications between nodes running Ethereum/Whisper/&c. clients are designed to be governed by a simple wire-protocol making use of existing ÐΞV technologies and standards such as [RLP](https://github.com/ethereum/wiki/wiki/RLP) wherever practical.

运行Ethereum / Whisper /＆c的节点之间的对等通信。客户端被设计为通过简单的有线协议来管理，使用现有的ÐΞV技术和标准，如[RLP]（https://github.com/ethereum/wiki/wiki/RLP），无论何处实用。


This document is intended to specify this protocol comprehensively.

本文件旨在全面指定此协议。

### Low-Level

ÐΞVp2p nodes may connect to each other over TCP only. Peers are free to advertise and accept connections on any port(s) they wish, however, a default port on which the connection may be listened and made will be 30303.


ÐΞVp2p节点可以通过TCP连接到彼此。对等人可以自由地在任何希望的端口上发布和接受连接，但是可以在其上连接的默认端口为30303。

Though TCP provides a connection-oriented medium, ÐΞVp2p nodes communicate in terms of packets. These packets are formed as a 4-byte synchronisation token (0x22400891), a 4-byte "payload size", to be interpreted as a big-endian integer and finally an N-byte RLP-serialised data structure, where N is the aforementioned "payload size". To be clear, the payload size specifies the number of bytes in the packet ''following'' the first 8.

虽然TCP提供了面向连接的介质，但是ÐΞVp2p节点就是通过数据包进行通信。这些分组形成为4字节的同步令牌（0x22400891），一个4字节的“有效载荷大小”，被解释为大端整数，最后是N字节RLP序列化数据结构，其中N是上述“有效载荷大小”。要清楚，有效载荷大小指定前8个数据包“后跟”的字节数。

### Payload Contents

There are a number of different types of payload that may be encoded within the RLP. This ''type'' is always determined by the first entry of the RLP, interpreted as an integer.

有许多不同类型的有效载荷可以在RLP内进行编码。这个“类型”总是由RLP的第一个条目决定，解释为一个整数。

ÐΞVp2p is designed to support arbitrary sub-protocols (aka _capabilities_) over the basic wire protocol. Each sub-protocol is given as much of the message-ID space as it needs (all such protocols must statically specify how many message IDs they require). On connection and reception of the `Hello` message, both peers have equivalent information about what subprotocols they share (including versions) and are able to form consensus over the composition of message ID space.


ÐΞVp2p旨在通过基本的线路协议支持任意的子协议（aka _capabilities_）。每个子协议都需要尽可能多的消息ID空间（所有这些协议必须静态指定需要多少个消息ID）。在连接和接收“Hello”消息时，两个对等体都具有关于它们共享哪些子协议（包括版本）的等效信息，并且能够对消息ID空间的组合形成共识。


Message IDs are assumed to be compact from ID 0x10 onwards (0x00-0x10 is reserved for ÐΞVp2p messages) and given to each shared (equal-version, equal name) sub-protocol in alphabetic order. Sub-protocols that are not shared are ignored. If multiple versions are shared of the same (equal name) sub-protocol, the numerically highest wins, others are ignored.

假设ID为0x10以上的消息ID（0x00-0x10为ÐΞVp2p消息保留）并按照字母顺序给予每个共享（等同名称，相同名称）子协议。未共享的子协议将被忽略。如果多个版本由相同（相同的名称）子协议共享，则数字最高的胜利，其他版本被忽略。

### P2P

**Hello**
[`0x00`: `P`, `p2pVersion`: `P`, `clientId`: `B`, [[`cap1`: `B_3`, `capVersion1`: `P`], [`cap2`: `B_3`, `capVersion2`: `P`], `...`], `listenPort`: `P`, `nodeId`: `B_64`] First packet sent over the connection, and sent once by both sides. No other messages may be sent until a Hello is received.
* `p2pVersion` Specifies the implemented version of the P2P protocol. Now must be 1.
* `clientId` Specifies the client software identity, as a human-readable string (e.g. "Ethereum(++)/1.0.0").
* `cap` Specifies a peer capability name as a length-3 ASCII string. Current supported capabilities are `eth`, `shh`.
* `capVersion` Specifies a peer capability version as a positive integer. Current supported versions are 34 for `eth`, and 1 for `shh`.
* `listenPort` specifies the port that the client is listening on (on the interface that the present connection traverses). If 0 it indicates the client is not listening.
* `nodeId` is the Unique Identity of the node and specifies a 512-bit hash that identifies this node.


[`0x00`: `P`, `p2pVersion`: `P`, `clientId`: `B`, [[`cap1`: `B_3`, `capVersion1`: `P`], [`cap2`: `B_3`, `capVersion2`: `P`], `...`], `listenPort`: `P`, `nodeId`: `B_64`] 在接收到Hello之前，不会发送其他消息。
*`p2pVersion`指定实现的P2P协议版本。现在一定是1。
*`clientId`指定客户端软件身份，作为人类可读的字符串（例如“Ethereum（++）/ 1.0.0”）。
*`cap`指定一个对等能力名称作为长度为3的ASCII字符串。当前支持的功能是“eth”，“shh”。
*`capVersion`将对等能力版本指定为正整数。当前支持的版本是“eth”是34，“shh”是1。
*`listenPort`指定客户端正在侦听的端口（在当前连接遍历的接口上）。如果为0则表示客户端没有收听。
*`nodeId`是节点的唯一标识，并指定标识此节点的512位散列。


**Disconnect**
[`0x01`: `P`, `reason`: `P`] Inform the peer that a disconnection is imminent; if received, a peer should disconnect immediately. When sending, well-behaved hosts give their peers a fighting chance (read: wait 2 seconds) to disconnect to before disconnecting themselves.
* `reason` is an optional integer specifying one of a number of reasons for disconnect:
  * `0x00` Disconnect requested;
  * `0x01` TCP sub-system error;
  * `0x02` Breach of protocol, e.g. a malformed message, bad RLP, incorrect magic number &c.;
  * `0x03` Useless peer;
  * `0x04` Too many peers;
  * `0x05` Already connected;
  * `0x06` Incompatible P2P protocol version;
  * `0x07` Null node identity received - this is automatically invalid;
  * `0x08` Client quitting;
  * `0x09` Unexpected identity (i.e. a different identity to a previous connection/what a trusted peer told us).
  * `0x0a` Identity is the same as this node (i.e. connected to itself);
  * `0x0b` Timeout on receiving a message (i.e. nothing received since sending last ping);
  * `0x10` Some other reason specific to a subprotocol.
  
  
  
[`0x01`: `P`, `reason`: `P`] 通知对端断断续续的时候，如果收到，对等体应立即断开连接。发送时，表现良好的主机给他们的同伴一个打破机会（读取：等待2秒）断开连接，然后再断开连接。
*`reason`是一个可选整数，指定断开连接的多种原因之一：
  *`0x00`断开连接请求;
  *`0x01` TCP子系统错误;
  *`0x02`违反协议，例如一个错误的信息，坏的RLP，不正确的魔法数字＆c。
  *`0x03`无用的对等体
  * 0x04太多的对等体
  * 0x05已连接;
  *`0x06`不兼容的P2P协议版本;
  *`0x07'接收到空节点身份 - 这是自动无效的;
  *`0x08`客户退出
  *`0x09`意外的身份（即与以前的连接不同的身份/可靠的对等体告诉我们）。
  *`0x0a`身份与此节点相同（即连接到自身）;
  *`0x0b`接收到消息时超时（即发送最后一次ping后没有收到）;
  *`0x10`一个特定于子协议的其他原因。

**Ping**
[`0x02`: `P`] Requests an immediate reply of `Pong` from the peer.

请求立即从管道回复“乒乓”。

**Pong**
[`0x03`: `P`] Reply to peer's `Ping` packet.

**NotImplemented (was GetPeers)**
[`0x04`: `...`]

**NotImplemented (was Peers)**
[`0x05`: `...`]

### Node identity and reputation

In a later version of this protocol, node ID will become the public key. Nodes will have to demonstrate ownership over their ID by interpreting a packet encrypted with their node ID (or perhaps signing a random nonce with their private key).


在该协议的较后版本中，节点ID将成为公钥。节点必须通过解释使用其节点ID加密的分组（或者可能用其私钥签名随机随机数）来证明其ID的所有权。

A proof-of-work may be associated with the node ID through the big-endian magnitude of the public key. Nodes with a great proof-of-work (public key of lower magnitude) may be given preference since it is less likely that the node will alter its ID later or masquerade under multiple IDs.


工作证明可以通过公钥的大端子大小与节点ID相关联。可以给出具有很好的工作证明（较低幅度的公钥）的节点，因为节点不太可能稍后改变其ID，或者在多个ID下伪装。


Nodes are free to store ratings for given IDs (how useful the node has been in the past) and give preference accordingly. Nodes may also track node IDs (and their provenance) in order to help determine potential man-in-the-middle attacks.


节点可以自由地存储给定ID的评级（节点过去有用），并相应地优先。节点还可以跟踪节点ID（及其来源），以帮助确定潜在的中间人攻击。

Clients are free to mark down new nodes and use the node ID as a means of determining a node's reputation. In a future version of this wire protocol, n

客户端可以自由地标记新节点，并使用节点ID作为确定节点信誉的方法。在该线路协议的未来版本中，n


### Example Packets

`0x22400891000000088400000043414243`

A Hello packet specifying the client id is "ABC".

Peer 1: `0x22400891000000028102`

Peer 2: `0x22400891000000028103`

A Ping and the returned Pong.

### Session Management

Upon connecting, all clients (i.e. both sides of the connection) must send a `Hello` message. Upon receiving the `Hello` message and verifying compatibility of the network and versions, a session is active and any other P2P messages may be sent.


连接时，所有客户端（即连接的两侧）必须发送一个“Hello”消息。收到“Hello”消息并验证网络和版本的兼容性后，会话是活动的，并且可能会发送任何其他P2P消息。

At any time, a Disconnect message may be sent.
