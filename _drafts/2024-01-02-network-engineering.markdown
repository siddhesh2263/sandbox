---
layout: post
title:  "Network Engineering"
date:   2024-01-02 00:02:29 -0400
categories: jekyll update
---

Revisiting network engineering concepts.

Click [here][video-link] for video source.

<br/>

### Client server architecture
<em>1/3/2025</em>

The client-server architecture introduces concepts like Remote Procedure Call (RPC) and gRPC to enable efficient communication between distributed systems. Many of the principles and advantages of this architecture have been adapted by modern microservices.

In this architecture, the client is typically designed to perform lightweight tasks, such as those performed by edge devices, while the server handles computationally expensive operations. Clients do not need to maintain heavy dependencies, making them more versatile and efficient. However, for effective communication between the client and the server, a standard communication model is essential.

<br/>

### OSI model

The Open Systems Interconnection (OSI) model is foundational to understanding network communication. It serves as a standard framework for networking and ensures that applications do not need to be aware of the underlying network medium.

Applications reside at specific layers of the OSI model. For example, if your application acts as a bridge between two systems, it might interact with MAC or IP addresses and involve tasks like serializing or deserializing JSON data. Without a standard model like OSI, applications would need to be tightly coupled with the network medium, complicating upgrades and maintenance.

<br/>

![image tooltip here]({{ "/assets/network_engineering/006_osi_model.png" | relative_url }})

<br/>

#### Layers of the OSI Model:
1. Layer 7: Application
Protocols like HTTP, FTP, and gRPC operate here.

2. Layer 6: Presentation
Handles encoding and serialization.

3. Layer 5: Session
Manages connection establishment and state (e.g., TLS). Proxies often operate at this layer.

4. Layer 4: Transport
Protocols like TCP and UDP operate here, and these are critical layers for software developers.

5. Layer 3: Network
Focuses on IP routing and addressing.

6. Layer 2: Data Link
Manages frames, MAC addresses, and Ethernet.

7. Layer 1: Physical
Deals with electrical signals and fiber optics.


Switches, routers, proxies, CDNs, reverse proxies, and load balancers facilitate communication between the client and the server. While switches operate at Layer 2 using MAC addresses, routers work at Layer 3 using IP addresses. Proxies and firewalls operate at Layer 4, handling ports and IP addresses transparently. Layer 7 devices like load balancers and CDNs decrypt and process data at the application layer, acting as endpoints for senders while forwarding requests to backend servers.

<br/>

![image tooltip here]({{ "/assets/network_engineering/007_osi_router_switch.png" | relative_url }})

<br/>

![image tooltip here]({{ "/assets/network_engineering/008_osi_proxy_load_balancer.png" | relative_url }})

<br/>

<br/>

### Host to host connection
<em>1/5/2025</em>

Host-to-host communication primarily involves Layer 2 (Data Link) and Layer 3 (Network). Each network device has a unique Media Access Control (MAC) address, which facilitates direct communication.

In small networks, such as a mesh network of four devices (A, B, C, and D), when device A sends a message to device B using B's MAC address, all devices receive the message, but only B processes the data frame. This behavior is typical in public WiFi networks. However, as the number of devices increases, addressing with MAC alone becomes inefficient. Routing packets efficiently requires IP addresses, which consist of two parts: one for identifying the network and the other for identifying the host.


### IP building blocks:

Packets, a key property of Layer 3, encapsulate data with source and destination IP addresses. While these packets may contain application-layer data such as HTTP, JSON, or port information, routers focus only on the IP addresses to forward packets.

#### Network vs host:

Let there be an IP address `a.b.c.d/x`. `x` denotes the network bits, and the remaining `(32 - x)` bits denote the host bits.

Example: IP address is `192.168.254.0/24`. The `24` denotes that the first `24 bits (3 bytes)` of the IP address is the network, while the remaining `8 bits (1 byte)` is for the host. In this example, there can be `2^24` networks, and each network will have 2^8 hosts.

#### Subnet mask:

The term `192.168.254.0/24` is also called a subnet.

A subnet mask determines whether two IP addresses belong to the same subnet. If they do, communication can occur using MAC addresses (host-to-host communication). Otherwise, routing is necessary, and gateways facilitate inter-subnet communication. Each host requires:
1. An IP address.
2. A subnet mask.
3. A gateway IP address for routing outside its subnet.

For example, placing a database in a different subnet can lead to performance issues if the router becomes congested, causing delays.

<br/>

<br/>

### IP Packet
<em>1/16/2025</em>

#### Anatomy of the IP packet:

![image tooltip here]({{ "/assets/network_engineering/009_ip_packet.png" | relative_url }})

<br/>

An IP packet consists of a header and data. The header is typically 20–60 bytes, while the data section can hold up to 65,535 bytes, limited by the Maximum Transmission Unit (MTU), usually 1,500 bytes. Key components of the IP packet include the following:

#### IHL:
Internet header line defines how long the Options section will be. IHL by default is set to 5.

Some routers block the Options section. One might argue we can use these sections to send more data, but there is no guarentee these will always work.

#### Version:
4 bits to describe (15 numbers, 2^8). These bits are never used.

#### Fragmentation:
This is very hard. We have a concept of MTU, where a limit is set on how much data can be put in an IP packet. If the IP packet doesn’t fit in a single frame, we can do two things: either send a failure message to the user through ICMP or fragment the packet into two frames. Note that these two frames won’t necessarily arrive in order. Also, assembling these packets would be needed at the host, and it can be a security issue. This is why fragmentation is frowned upon.

#### Identification:
This is for indentifying the fragmented packets.

#### Flags:
Used for fragmentation process.

#### Time to live (TTL):
Every IP packet has an 8-bit space that represents a counter. Packets can go through different routers, but in some cases, this can result in getting stuck in an infinite loop. Routers don't save the state of the protocol. So how do we prevent this? An estimation is done to determine how many hops the packet will make until it reaches the destination. The routers then decrement the value of the counter on every hop. Whichever router turns up with `counter = 0` will let the host know through ICMP. This is how traceroute works. Some routers and firewalls disable ICMP.

#### Protocol:
This defines the protocol of contents that's inside the data. Why do we need extra 8 bits for this? It helps at the router stage, which look at this information, instead of parsing the data. This metadata is used to save some performance. It can have up to 255 protocols.

#### Source and destination IP addr:
One of the most important parts. Spoofing involves changing of the source IP address, but it is difficult to do it, since an ISP assigns IP address to the clients.

#### ECN (Explicit congestion notification):
When a router detects congestion, it sets a special bit to signal the issue. Congestion happens when packets start dropping because the router's memory, or buffer, is overwhelmed. Routers use their buffers to process incoming packets, but as the number of packets increases, it takes longer to handle them. This causes queues to grow, and eventually, the buffer can’t keep up, leading to packet drops. At this point, the router may notify the client about the congestion.

From the client’s perspective, if no acknowledgment for a packet is received, it assumes the packet was dropped. Eventually, it may deduce that the router is congested, but this realization is slow. That’s where ECN (Explicit Congestion Notification) comes in—it’s a faster way to notify the client about congestion.

Here’s how it works: when the router’s buffer is on the verge of failure, it sets the ECN bit in the packet. The client sees this bit and immediately knows the router is under strain and packets are likely to drop. This allows the client to take appropriate action. It’s an elegant solution to a tricky problem.

To reduce congestion in general, it’s also important to avoid practices like using bloated JSON payloads or storing and sending duplicate data from databases. These inefficiencies can exacerbate the problem. (The author also mentions the concept of a blackhole TCP connection, which adds another layer of complexity to congestion issues.)

<br/>

### ICMP, Ping, Traceroute

ICMP lives in L3. It stands for internet control message protocol. There is no concept of ports in L3.
ICMP is designed for informational messages, such as host unreachable, port unreachable, fragment needed, packet expiry (infinite loop in routers). Pings and traceroute use ICMP. ICMP doesn't require listeners or ports to be opened. Host just needs to enable ICMP.


### Disabling of ICMP:

Some firewalls block ICMP for security reasons, such as to prevent flooding attacks or backchannel exploits. This is why ping may not work in certain situations. When packets pass through multiple routers, one of the routers might have ICMP disabled, causing communication issues.

Disabling ICMP can lead to serious problems, particularly during connection establishment. For example, if the client needs to receive a "fragmentation needed" message, it won’t be able to. If ICMP is blocked but the TCP connection remains open, the host might unknowingly exceed the MTU limit. However, since ICMP is disabled, the host will never receive the necessary notification to address the issue.

How is TTL determined?

How does ping work? Subnet masks, all those steps are done.

What is ICMP echo request, ICMP echo reply, ICMP dest. unreachable? Everything is done through the IP packet.

Try to do traceroute google.com.

TTL doesn't get affected if we're sending something in the same network.

ICMP is a very important protocol.

<br/>

<br/>

### ARP
<em>1/18/2025</em>

ARP stands for Address Resolution Protocol. We need a way to route, so that we can eliminate networks to scan. But we still need the MAC addresses to send frames (layer 2) to a system. If we know the IP address, and not the MAC address, we map the IP addresses with the MAC. We use ARP to do this process. An ARP table is a cached IP-MAC mapping.

![image tooltip here]({{ "/assets/network_engineering/001_network_frame_arp.png" | relative_url }})

<br/>

Let's say a machine M1 wants to send a request to machine M2. M1 has M2's IP address, but doesn't have the MAC address. First, M1 will check if M2 is in the same subnet. It then needs M2's MAC address. It checks its ARP table, but it doesn't have this information. So, M1 sends an ARP boradcast to all the machines in the network. It asks who has M2's IP address, and when M2 replies with its MAC address, M1 will update its ARP table to store M2's MAC address.

![image tooltip here]({{ "/assets/network_engineering/002_arp_broadcast_example.png" | relative_url }})

<br/>

Now, what if M2 is not in the same network? M1 will check this using its subnet mask. If M2 is not in the subnet, M1 will need to talk with the gateway. M1 has the gateway's IP address (all systems in the network have the gateway's IP address). If it doesn't have the MAC, it will do an ARP broadcast. This is a dangerous scenario, since any system can say it is the gateway, and share its MAC address. This is called ARP poisoning. There is a concept of NAT that comes into play when gateway is involved.

We need MAC addresses to send frames between machines. ARP is very critical to understand. Virtual Router Redundancy Protocol (VRRP) is one of the powerful concepts that uses ARP, which is used for doing load balancing with a virtual IP address.

<br/>

### Capturing IP, ARP, and ICMP packets using TCPDUMP

Below are some commands that can be used to observe network information:

```
tcpdump -i en0 arp
tcpdump -n -i en0 arp
tcpdump -n -i en0 icmp (this will be empty, so try to run a ping on other terminal to see the ICMP messages)
tcpdump -n -v -i en0 arp
man tcpdump (to know more about the commands)
tcpdump -n -v -i en0 src <IP addr> (this will filter our output for the mentioned IP address.)
```

<br/>

<br/>

### Routing example
<em>1/22/2025</em>

![image tooltip here]({{ "/assets/network_engineering/003_routing_example.png" | relative_url }})

<br/>

We will refer the above image, and see some cases how the IP packets will be moved from one machine to another.

#### A to B
When device A wants to communicate with device B, it first checks whether B is present in the same network. In this case, since B is in the same network, A sends an ARP broadcast to retrieve the MAC address of B. ARP requests are confined to the local network and do not travel to other networks. Once A receives the MAC address of B, it can proceed to send the IP packets.

Typically, when devices power up, they announce their IP and MAC addresses to the switch. The switch then maps these MAC addresses to the respective ports. Being a Layer 2 device, the switch uses this information to facilitate communication within the network.

When A sends a frame to B, the frame is directed to the switch. This frame contains the destination MAC address of B. The switch identifies the correct port connected to B using its internal mapping and forwards the frame through that port. While this is a simplified explanation of how switches work, it provides a high-level overview of the process.

#### D to X
D performs a subnet mask check to confirm if the destination X is not in the same network. Since X is in a different network, D sends the IP packet to the gateway, which is router R. However, for this to happen, D needs the MAC address of the gateway.

To obtain the MAC address, D sends an ARP broadcast message. The switch, having no specific knowledge of the target device for this message, forwards the broadcast to all other machines in the network. Router R responds directly to D with its MAC address. Now, D has the MAC address of R and includes it in the packet.

Although the IP packet sent by D includes the MAC address of R, the router recognizes that the destination IP address belongs to X. At this point, R performs its own ARP request to retrieve the MAC address of X.

From a Layer 2 (Data Link) perspective, the packets flow from D to R. However, from a Layer 3 (Network) perspective, the flow is logically from D to X. This distinction highlights the roles of each layer in the networking process.

#### B to G
When device B determines that G is not in the same subnet, it knows it must communicate through the router, R. To do this, B first requests the MAC address of router R. Once B has the MAC address, it sends the packet to R.

Router R examines the destination IP address in the packet and recognizes that it is outside the local network. To send the packet to G, R uses its public IP address to route the packet over the external network. While this explanation provides an overview, the process involves additional steps, such as the use of NAT, which we will cover later.

<br/>

How are bridges different from switches, routers?

<br/>

<br/>

### UDP

UDP, or User Datagram Protocol, is a lightweight communication protocol. A common scenario for its use arises when a single host runs multiple applications, and requests are sent to the host's IP address for these applications. To ensure that each request is directed to the correct application within the host, ports are used to uniquely identify and route these requests.

UDP is a stateless protocol, meaning it does not maintain any connection state on the server, similar to how IP operates. This simplicity makes UDP efficient, as it comes with a relatively small header, which reduces overhead during data transmission.

#### Use cases of UDP:
1. Video Streaming: In video streaming, the primary concern is maintaining a consistent stream of data rather than ensuring every video frame is delivered perfectly. While some video frames might be dropped during transmission (not to be confused with Layer 2 frames), the focus is on delivering smooth and uninterrupted playback rather than preserving high quality at all times.

2. VPN (Virtual Private Network)

3. DNS: The Domain Name System (DNS) is a protocol used to resolve human-readable hostnames into IP addresses, enabling users to access websites without needing to remember numerical addresses. However, wherever mappings are involved, there is always a risk of exploitation, such as DNS poisoning or ARP poisoning, where malicious actors manipulate mappings to redirect traffic or intercept communication.

4. WebRTC: Web Real-Time Communication (WebRTC) is a technology that enables real-time audio, video, and data sharing directly between browsers without requiring additional plugins or software. It is commonly used in applications like video conferencing and peer-to-peer file sharing.

<br/>

#### Multiplexing and demultiplexing

When `host 1` runs multiple applications, each application sends requests using different ports. These requests are multiplexed when they arrive at `host 2`, allowing the system to distinguish between them and direct them to the appropriate application.

Source ports are crucial because they enable `host 2` to send responses back to the correct application on the originating machine. By including the source port in the request, `host 2` can identify where the response should be sent.

Every datagram in this process is uniquely identified, ensuring that communication between the applications and the host remains organized and reliable.

![image tooltip here]({{ "/assets/network_engineering/004_UDP_frame.png" | relative_url }})

<br/>

What is TCP meltdown?

<br/>

UDP Datagram Structure:

![image tooltip here]({{ "/assets/network_engineering/005_UDP_datagram_frame.png" | relative_url }})

One might ask, is there a limit to how many connections can be made to a host? Yes. This analogy is drawn from the number of ports at which a UDP datagram can be sent, which has a limit.

<br/>

#### Some other UDP information:
1. We don't use UDP for database connections.
2. One of the security concerns related to UDP is, the use of DNS flooding attack. We flood the server with UDP datagrams, and the server has to process all of it. Since this protocol is stateless, the server doesn't keep track of the datagram, and can eventually lead to a DOS.
3. UDP doesn't support congestion control. However, routers are protected against congestion.

<br/>

<br/>

### Socket, connections, and queues (kernel networking structures)
<em>1/26/2025</em>

A socket is a mechanism that listens for incoming connections, and it is bound to a specific pair of an IP address and a port. Each machine is equipped with **Network Interface Cards (NICs)**, and typically, a MAC and an IP address are associated with each NIC. However, by using multiple NICs, it is possible to create multiple virtual IP addresses.

For development purposes, the **TCP loopback address** `127.0.0.1` is commonly used, which allows communication within the same machine. If an application needs to listen to all incoming IP addresses on a machine, it can be configured to use `0.0.0.0`. This setup enables the application to listen to all interfaces. However, if this configuration is used in production, the application will listen to all machines on the network. For example, if one of these machines is a load balancer connected to a public network, the application could end up listening to all public IP addresses, creating a potential security vulnerability. This type of misconfiguration has been linked to breaches in systems like Elasticsearch and MongoDB (further research on these cases is needed).

While a socket is purely a listening mechanism, a **connection** is a more complex entity that exists between a client and a server. A connection includes details for both the source and destination, representing the communication between the two ends.

<br/>

![image tooltip here]({{ "/assets/network_engineering/010_syn_queues.png" | relative_url }})

<br/>

Let’s review the process of establishing a TCP connection as illustrated above. A TCP connection involves a **three-way handshake**, which is managed by the kernel using what are referred to as the `SYN` and `Accept` queues. In the source code, these are not actual queues but hash sets. However, the term "queues" is used here to simplify the explanation and make the concept of connection management easier to understand.

When a client sends a connection request (a `SYN` request), the socket listens for it and temporarily stores it in the `SYN` queue. The server then sends an acknowledgment (`ACK`) back to the client. Once the server receives the client's `SYN/ACK` response, the request is removed from the `SYN` queue and placed into the `Accept` queue. At this stage, the connection is not fully established between the client and the server.

For the connection to be finalized, the application (client) must call the `accept()` method. This method is typically handled behind the scenes by the framework the developer is using. Once the client accepts the connection, the request is removed from the kernel’s `Accept` queue, which resides inside the receiver's host. This step ensures that the connection between the client and the server is officially established.

Once the connection is established, two additional queues are created: one for sending data to the client and another for receiving data from the client. The specifics of how data parsing is performed at this stage involve deeper technical details, but for now, we’ll focus on the high-level functioning of the process.

<br/>

![image tooltip here]({{ "/assets/network_engineering/011_kernel_connection.png" | relative_url }})

<br/>

Below is a summary of how the connection establishment happens at the kernel level:

![image tooltip here]({{ "/assets/network_engineering/012_kernel_conn_est.png" | relative_url }})

<br/>

#### DOS attack scenario:

Let’s consider what happens if the kernel adds a request to the `SYN` queue and responds with a `SYN/ACK` message to the client. The next step in the process requires the client to reply with an `ACK` message. However, if the client does not send this reply, the `SYN` queue begins to fill up. If enough of these incomplete requests accumulate, the queue can reach its capacity, potentially leading to a **Denial of Service (DoS)** attack. While this issue has largely been mitigated with the implementation of timeouts for incomplete connections, it remains a potential vector for attacks.

Another scenario arises when the client sends a `SYN` request, receives a `SYN/ACK` response, but does not accept the connection. In this case, the connection moves to the `Accept` queue, which can also begin to fill up if many such incomplete connections are received. This situation can strain the server's resources and lead to performance degradation.

Differentiating between legitimate users and malicious actors in such cases is a complex and ongoing area of research. It is also one of the key challenges that services like **Cloudflare** aim to address by providing solutions for mitigating these types of attacks and ensuring reliable connection management.

<br/>

#### Socket sharding:

Socket sharding can be implemented in two ways:

1. **Forking a Socket**: In this method, multiple sockets point to the same physical socket. These sockets share the same `SYN/ACK` queue, allowing them to handle connection requests collaboratively.

2. **Creating Separate Sockets**: This approach involves creating two separate sockets that both point to the same IP address and port pair. These sockets operate as distinct processes. However, this method introduces challenges, such as determining which socket should handle an incoming `SYN` request. Socket sharding in this context can help distribute connection requests more efficiently, making it a useful strategy for scaling up request handling. The Linux kernel supports load balancing in such scenarios to manage connections effectively.

Previously, we discussed the queues used to manage connections, such as the `SYN` and `Accept` queues. We also briefly touched on the queues responsible for sending and receiving data. The process of parsing data within these queues is a complex topic on its own and involves significant technical depth, which we won't dive into here. However, given the complexity of managing the networking stack, it might be necessary to dedicate one or two CPU cores exclusively to handle networking tasks efficiently.

<br/>

<br/>

### TCP

TCP, or Transmission Control Protocol, is a Layer 4 protocol that provides reliable communication over networks. As a Layer 4 protocol, TCP has access to ports. The transmission control limitation of UDP often results in firewalls blocking it. TCP, on the other hand, uses a header segment that is 20 bytes in size but can extend up to 60 bytes when optional fields are included.

TCP is widely used in scenarios requiring reliable communication, such as remote shells, database connections, and any form of bidirectional communication. One key distinction of TCP is its ability to maintain a connection state, making it a Layer 5 concept. A TCP connection represents an agreement between the client and server, identified by four unique properties: the source IP address, source port, destination IP address, and destination port. Before any data transmission occurs, the kernel verifies these values against a lookup table to ensure proper connection management.

Data in TCP is transmitted in the form of segments, which are sequenced and ordered. Each segment is acknowledged by the server, and if a segment is lost, it is retransmitted. This mechanism ensures reliable data delivery, a feature absent in UDP. 

Similar to UDP, TCP can handle multiplexing and demultiplexing of requests when multiple applications are running on the same machine. By leveraging port information, TCP efficiently manages communication for multiple processes, ensuring data is routed to the correct application.

<br/>

#### Connection Establishment

Suppose App1, running on the machine with IP address `10.0.0.1`, wants to send data to AppX on `10.0.0.2`. To initiate communication, App1 sends a `SYN` packet to AppX to synchronize the sequence numbers of its segments. In response, AppX sends back a `SYN/ACK` packet to synchronize its own sequence numbers with App1. Finally, App1 completes the process by sending an `ACK` packet to AppX. This sequence of events is referred to as the **three-way handshake**, which establishes a reliable connection between the two applications.

Once the connection is established, the connection information is stored as a file descriptor. This file descriptor is used by the kernel to manage and verify subsequent transmissions of data between the two applications. If a machine attempts to send data to the server without having an established connection, the data will be rejected and dropped by the server. This ensures that only valid, authenticated connections are allowed to communicate.

<em>How many TCP connections can be established?</em>

If a connection is destroyed, the file descriptor needs to be deleted from the hash table which the kernel maintains. This can cause problems. This is a CPU intensive task, and will reach a limit eventually.

![image tooltip here]({{ "/assets/network_engineering/013_tcp_connection_est.png" | relative_url }})

<br/>

#### Sending Data

App1 now sends data to AppX. App1 encapsulates the data in a segment, which AppX acknowledges, since they have a connection.

![image tooltip here]({{ "/assets/network_engineering/014_tcp_send_data.png" | relative_url }})

Flow control is determined by the server and the routers the IP packet passed through, while congestion control is handled by the packet itself.

<em> Can App1 send new segments before the ACK of the old segment arrives? Yes.</em> We can acknowlegde mulitple segemtns by a single ACK message. The below image gives an overview:

![image tooltip here]({{ "/assets/network_engineering/015_multiple_ack.png" | relative_url }})

<br/>

#### Lost Data

If some segments are lost, the client sees that it didn't receive the ACK for those. In this case, it sends the lost sequence again. This is the feature of retransimssion the TCP protocol provides.

![image tooltip here]({{ "/assets/network_engineering/016_lost_data_tcp.png" | relative_url }})

<br/>

#### Closing a Connection

The below image shows how the connection can be closed. When the FIN is sent, the server checks if the connection has its file descriptor. If present, the server sends a ACK, and closes the connection.
However note that the file descriptor on the client end is not deleted yet. This is called the time wait state for App1. It remains for a finite amount of time before it is removed.

![image tooltip here]({{ "/assets/network_engineering/017_close_connection.png" | relative_url }})

<br/>

<br/>

### TCP Segment

A TCP segment is encapsulated within an IP packet as its data payload. The segment header, which occupies 20 bytes, contains critical information about sequence numbers, acknowledgments, flow control, and other properties required for reliable communication.

![image tooltip here]({{ "/assets/network_engineering/018_tcp_segment.png" | relative_url }})

*Note: Although the count of available sequence numbers is finite, it is significantly large. Handling this limitation is an interesting topic, with real-world examples like the Postgres case study offering valuable insights.*

The default maximum window size for TCP is 65KB, but it can be scaled up to 1GB to accommodate higher throughput. Within the TCP segment, there are nine 1-bit flags, each serving a specific purpose:
- **SYN**: Used to initiate a connection.
- **FIN**: Signals the end of a connection.
- **RST**: Resets a connection.
- **PSH**: Pushes data to the receiving application immediately.
- **ACK**: Acknowledges received data.
- **URG**: Marks urgent data for priority processing.
- **ECE**: Part of congestion control, set by the server when nearing congestion.

The **maximum segment size (MSS)** determines how much data can be included in a single TCP segment. This size is directly influenced by the **MTU (Maximum Transmission Unit)** of the network. For most internet connections, the default MTU is 1500 bytes. This includes:
- 20 bytes for the IP header.
- 20 bytes for the TCP segment header.
- Leaving 1460 bytes available for actual data.

In some cases, such as with certain cloud providers, **jumbo frames** are used, where the MTU can reach up to 9000 bytes or more. These larger MTUs allow for higher efficiency by fitting more data into a single frame, reducing the overhead caused by headers.

<br/>

<br/>


[video-link]: https://www.udemy.com/course/fundamentals-of-networking-for-effective-backend-design