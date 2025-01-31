# Ruffles 
[![Build Status](https://img.shields.io/appveyor/ci/midlevel/ruffles/master.svg?logo=appveyor)](https://ci.appveyor.com/project/MidLevel/ruffles/branch/master)
[![NuGet](https://img.shields.io/nuget/v/Ruffles.svg?logo=nuget)](https://www.nuget.org/packages/Ruffles)
[![GitHub Release](https://img.shields.io/github/release/midlevel/Ruffles.svg?logo=github)]()
[![Discord](https://img.shields.io/discord/449263083769036810.svg?label=discord&logo=discord&color=informational)](https://discord.gg/FM8SE9E)
[![Licence](https://img.shields.io/github/license/midlevel/ruffles.svg?color=informational)](https://github.com/MidLevel/Ruffles/blob/master/LICENCE)

Ruffles is a fully managed UDP library designed for high performance and low latency.

## Why another reliable UDP library?
There are many RUDP libraries such as ENET, Lidgren, LiteNetLib. While many of them are great, Ruffles aims to fill in one niche that is largely not filled, that being lightweight fully managed libraries.

To compare to the examples above, ENET is amazing and is pretty much what Ruffles want to be, but it's unmanaged. Lidgren, LiteNetLib and many other managed libraries can feel too bloated and contain many features that are unnecessary and they are often much slower.


## Features
Ruffles is still **WIP**, more features might come.

### Connection Challenge
The Ruffle protocol requires a challenge to be completed before a connection can be established. Currently, the challenge is a hashcash like challenge that is supplied by the server, brute force solved by the client and submitted. (Uses Fowler-Noll-Vo hash function instead of SHA1 currently).

### DOS Amplification Prevention
DOS amplification is prevented by requiring unproportional connection message sizes. In addition, because of the connection challenge it's not computationally feasible to attack on Layer 4.

### Slot Filling Prevention
Ruffles has a fixed amount of connection slots that can be used for pending connections, this limits the usability of slot filling attacks on Layer 4. Pending connections have a fixed timeout to solve the computationally expensive HashCash challenge before being knocked out, the slot will be available once again after that. As this only limits slot filling attacks, Ruffles also has a security mechanism where a HashCash has to be solved in the first message. This challenge is generated by the client and the server will verify that the date used is recent and that the IV has not already been used, forcing clients to recompute the HashCash challenge every time they want to initialize a handshake.

With these security mitigations, the only way to bring the server down is to exhaust all CPU resources.

### Connection Management
Ruffles handles all connection management for you. It's a fully connection oriented protocol with heartbeat keepalive packets sent to ensure the connection is alive.

### High Performance
Ruffles is fully garbage free, this is accomplished with a custom memory allocator in GC space. This ensures no memory is leaked to the garbage collector unless for resizing purposes. This makes Ruffles blazing fast. It also avoids memory copies as much as possible.

### Reliability and Sequencing
There are currently a few ways of sending messages in Ruffles. The types are:
#### Reliable
All messages are guaranteed to be delivered, the order is not guaranteed, duplicates are dropped. Uses a fixed sliding window.
#### ReliableSequenced
All messages are guaranteed to be delivered with the order also being guaranteed, duplicates are dropped. Uses a fixed sliding window.
#### ReliableSequencedFragmented
All messages are guaranteed to be delivered with the order also being guaranteed, duplicates are dropped. Uses a fixed sliding window. Allows large messages to be fragmented.
#### Unreliable
Delivery is not guaranteed, nor is the order. Duplicates are dropped.
#### UnreliableOrdered
Delivery is not guaranteed but the order is. Older packets and duplicate packets are dropped.
#### UnreliableRaw
Delivery is not guaranteed nor is the order. Duplicates are not dropped.
#### UnconnectedMessages
Raw UDP packets that does not require a connection.
#### ReliableOrdered
All messages are not guaranteed to be delivered. If you send multiple messages, at least one is guranteed to arrive. If you send a single message, it is guaranteed to arrive. Messages will always been in order. Duplicates are dropped.

### Threading
Ruffles is nativley multi threaded and uses a background worker thread by default to handle network I/O.

### Dependency Free
Ruffles is 100% dependency free, it's thus very portable and should run on most platforms.

### IPv6 Dual Mode
Ruffles supports IPv6 dual socket mode. It does this by using two sockets bound to the same port, thus accomplishing full dual stack functionality that is invisible to the user.

### Small Packet Merging
Small packets will be delayed for sending, this allows them to be merged into one larger packet. This can be disabled and enabled on a per packet basis. The delay and max merge size can also be configured.

### Fragmentation
Packets can be sent as ReliableSequencedFragmented which allows for a single packet to be of a size of up to 2^15*1450 bytes = 47513600 bytes = 47.5 megabyte.

### Ack Merging
Ack packets are merged into bitfields to make them much more compact.

### Connection Statistics
Detailed statistics can retrieved from connections, including the bytes sent, packets sent, round trip times and more.

### Path MTU
Automatically discovers the largest MTU possible for each connection.

## Roadmap
This is stuff I want to and plan to add

* Adaptable window sizes
* Basic bandwidth control, limit the amount of acks that are sent etc
* More Fragmentation Types
* Explicit Nack
* MLAPI.Relay Support
* MLAPI.NAT (Holepuncher) support
* Bloatless Moduled Library (Make all the garbage features like relay support separate modules to keep the core library bloat free and small)

## Maybe Roadmap
Here are the features that are considered but not decided. This is to prevent bloat.
* Multicasting
* Meshing / Peer relaying