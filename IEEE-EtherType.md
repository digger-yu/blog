# IEEE 802 Numbers - EtherType Registry

本文档整理自 [IANA IEEE 802 Numbers Registry](https://www.iana.org/assignments/ieee-802-numbers/ieee-802-numbers.xhtml)

## 网络层协议

| EtherType (Hex) | Description |
|------------------|-------------|
| 0x0800 | IPv4 |
| 0x0806 | Address Resolution Protocol (ARP) |
| 0x86DD | IPv6 |

## 局域网管理与安全

| EtherType (Hex) | Description |
|------------------|-------------|
| 0x8100 | Customer VLAN Tag Type (802.1Q) |
| 0x8847 | MPLS Unicast |
| 0x8848 | MPLS Multicast |
| 0x888E | EAP over LAN (802.1X) |
| 0x88CC | Link Layer Discovery Protocol (LLDP) |
| 0x8809 | Link Aggregation Control Protocol (LACP) |
| 0x010B | Spanning Tree Protocol (STP) |

## 存储与高性能计算

| EtherType (Hex) | Description |
|------------------|-------------|
| 0x8906 | Fibre Channel over Ethernet (FCoE) |
| 0x8914 | RDMA over Converged Ethernet (RoCE) |
| 0x8915 | RDMA over Converged Ethernet (RoCEv2) |

## 实验与保留

| EtherType (Hex) | Description |
|------------------|-------------|
| 0x88B5 | IEEE Std 802.3 Local Experimental Ethertype |
| 0x88B6 | IEEE Std 802.3 Local Experimental Ethertype |

## 厂商私有协议

| EtherType (Hex) | Description |
|------------------|-------------|
| 0x2004 | Cisco Shared Spanning Tree Protocol |
| 0x22EA | Cisco Discovery Protocol (CDP) |
| 0x8783 | Cisco Systems Inc. |
| 0x8870 | IEEE Std 802.1CS Logical Link Control Sublayer Exp. |
| 0x9100 | IEEE Std 802.1Q - Customer VLAN Tag (Double Tagging) |

## 其他常用协议

| EtherType (Hex) | Description |
|------------------|-------------|
| 0x0805 | Frame Relay ARP |
| 0x0807 | Symbolics Private |
| 0x081C | Texas Instruments |
| 0x0842 | Wake on LAN |
| 0x22F0 | Audio Video Bridging (AVB) |
| 0x22F3 | IEEE 1722 Audio Video Bridging Transport Protocol |
| 0x6002 | DEC MOP Remote Console Protocol |
| 0x6003 | DECnet Phase IV |
| 0x6004 | DEC LAT |
| 0x8035 | Reverse Address Resolution Protocol (RARP) |
| 0x809B | AppleTalk (Ethertalk) |
| 0x80F3 | AppleTalk Address Resolution Protocol (AARP) |
| 0x8137 | Novell IPX (alt) |
| 0x8204 | SNA over Ethernet |
| 0x872D | OpenFlow |
| 0x8808 | Link Pause |
| 0x8863 | PPP over Ethernet (PPPoE) Discovery Stage |
| 0x8864 | PPP over Ethernet (PPPoE) Session Stage |
| 0x88A2 | ATA over Ethernet |
| 0x88A8 | Service VLAN tag identifier (802.1ad) |
| 0x88E1 | HomePlug 1.0 MME |
| 0x88E3 | Media Redundancy Protocol (MRP) |
| 0x88E5 | MAC Security (802.1AE) |
| 0x88F7 | Precision Time Protocol (PTP) over Ethernet (1588) |
| 0x8902 | IEEE Std 802.1ag Connectivity Fault Management (CFM) |
| 0x890D | TIPC |
| 0x892F | NSH (Network Service Header) |
| 0x893A | In-band Network Telemetry (INT) |
| 0x9000 | Loopback |
