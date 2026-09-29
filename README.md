 Network Traffic Analysis and Protocol Identification using Wireshark

This project captures live Wi-Fi traffic with *Wireshark* and analyses it to identify protocols (TCP, DNS, ICMP, ARP, UDP) and to study traffic behaviour using Wireshark's statistics tools.
Tools used:Wireshark, Windows, Wi-Fi interface
Capture size:about 5.2 million packets

 1. TCP Packet Analysis
Filter: tcp
Observation: Frame 79 is a TLSv1.2 Application Data packet from 172.20.76.254 to 4.213.25.240 (port 49435 to 443). The IPv4 header shows TTL 128 and Don't Fragment set. The list also shows normal ACKs, a RST, a FIN/ACK, a TCP Retransmission and a Dup ACK. The status bar shows 281,289 of 1,959,756 packets displayed (14.4%).*Conclusion:Most traffic is TCP on port 443 (HTTPS/TLS), and Wireshark highlights problem packets in different colours.

2. DNS Packet Analysis
Filter:dns
Observation: This is a DNS response with flags 0x8183 (Standard query response, *No such name*). Questions: 1, Answer RRs: 0, Authority RRs: 1. The response came 17.2 ms after the request (request in packet 637827).
Conclusion:The DNS server replied NXDOMAIN, so the requested domain name does not exist.

 3. ICMP Analysis
Filter: icmp
Observation: 8 ICMP packets, all *Destination unreachable (Host unreachable)*, Type 3, Code 1. They were sent by the gateway 172.20.64.1 to 172.20.76.254. The embedded original packet was a TCP packet going to 172.25.0.149.
Conclusion:The network could not reach host 172.25.0.149, so the router reported the error back to the sender.

 4. ARP Analysis
Filter: arp
Observation:Many broadcast requests of the form "Who has 172.20.x.x? Tell 172.20.x.x". The selected packet has Opcode 1 (request), sender 172.20.72.73 (MAC 96:0a:e2:1b:f3:8d), target IP 172.20.67.16, and target MAC 00:00:00:00:00:00 because it is unknown.
Conclusion:ARP maps IP addresses to MAC addresses inside the local network, using broadcast requests.

5. UDP Analysis
Filter:ip (the selected packet is UDP)
Observation: Most visible packets are mDNS (Multicast DNS) sent to the multicast address 224.0.0.251, with UDP source and destination port 5353. The selected packet is from 172.20.68.74, with Protocol: UDP (17) and TTL 255. An IGMPv2 membership report is also visible.
Conclusion: UDP is connectionless and is used here for service discovery (Spotify, Google Cast, ADB) on the local network.

 6. Follow TCP Stream / Entire Conversation
Filter: tcp.stream eq 702 (Analyze > Follow > TCP Stream)
Observation: The conversation between 172.20.76.254:57174 and 16.15.245.155:443 is about 47 MB, with 2,495 client packets and 12 server packets in the stream view. The payload is unreadable.
Conclusion:The data is TLS encrypted, so the content cannot be read. This confirms the traffic is secure HTTPS.

 7. TCP Conversations
Path: Statistics > Conversations > TCP
Observation: 172.20.76.254:57174 to 16.15.245.155:443, Stream ID 702, 7,628 packets, 48 MB. A to B: 2,570 packets (48 MB). B to A: 5,058 packets (319 kB).
Conclusion: This is a large upload from the client to the server, and the server mostly sends ACKs back.

 8. Protocol Hierarchy
Path: Statistics > Protocol Hierarchy
Observation:Frame, Ethernet, IPv4, TCP (100%), then Transport Layer Security (22.8% of packets) and a little Data.
Conclusion:The traffic in this view is entirely IPv4 + TCP, and TLS is the application layer on top.

 9. IPv4 Endpoints
Path: Statistics > Endpoints > IPv4
Observation: Two endpoints: 172.20.76.254 (local machine) and 16.15.245.155 (remote server), each with 7,628 packets and 48 MB. The local machine sent 2,570 packets (48 MB) and received 5,058 packets (319 kB).
Conclusion: These are the two hosts in the main conversation, and the local machine is the one uploading.

10. Packet Lengths
Path: Statistics > Packet Lengths
Observation:4,864,592 packets, average size 540.53 bytes, minimum 42, maximum 64,566. The biggest group is 40-79 bytes (46.23%), then 320-639 bytes (22.53%) and 160-319 bytes (12.04%).
Conclusion: Almost half the traffic is small control packets (ACKs, DNS, ARP), and a few very large packets carry bulk data.

11. I/O Graph
Path: Statistics > I/O Graphs
Observation:Three graphs at 1-second intervals: All Packets, TCP Errors (tcp.analysis.flags, red) and Filtered packets (tcp.stream eq 702). The capture runs about 7,200 s. Normal traffic is around 1 kpkts/s, with a major spike of about 5 kpkts/s near 2,350 s.
Conclusion: Traffic is fairly steady with a few bursts, and TCP errors appear throughout the capture.

 12. TCP SYN Analysis
Filter: tcp.flags.syn == 1 && tcp.flags.ack
Observation: 6,094 SYN-ACK packets (0.3%), all from port 7680 on 172.20.76.254 to other hosts on the network. Each has MSS=1460, WS=256, SACK_PERM and Win=65535.
Conclusion: These are the second step of the TCP 3-way handshake, where the server accepts a connection. Port 7680 is used for Windows update sharing (Delivery Optimization).
SYN, then SYN-ACK, then ACK, followed by the TLS Client Hello (SNI: an Amazon S3 host) and Server Hello.

13. TCP RST Analysis
Filter: tcp.flags.reset == 1
Observation:717 RST packets (0.0%), shown in red. Most are [RST, ACK] with Win=0 from 172.20.76.254 to port 443 (HTTPS) and port 53 (DNS over TCP). The flag details confirm RST + ACK are set (0x014).
Conclusion:Connections were abruptly closed instead of the normal FIN handshake, which is common when apps or browsers drop connections.

14. Multiple Protocol Traffic
Filter:tcp || udp || dns || icmp || arp
Observation: A mix of TCP ACKs, TCP Dup ACKs, TLSv1.3 Application Data and segment reassembly, with several conversations running at the same time.
Conclusion: Combining filters shows several protocols together, which helps in seeing the overall network activity.

Conclusion
Using Wireshark, I identified and analysed TCP, DNS, ICMP, ARP and UDP traffic, used statistics tools (Conversations, Endpoints, Protocol Hierarchy, Packet Lengths, I/O Graph), and found handshake, reset and error behaviour in the capture.
