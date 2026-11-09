# Cisco IOS: NAT & PAT Configuration Guide

![Cisco](https://img.shields.io/badge/Cisco-1BA0D7?style=for-the-badge&logo=cisco&logoColor=white)
![Networking](https://img.shields.io/badge/Networking-NAT%20%26%20PAT-blue?style=for-the-badge&logo=diagramsdotnet&logoColor=white)
![IPv4](https://img.shields.io/badge/Protocol-IPv4-green?style=for-the-badge&logo=cloudflare&logoColor=white)

A comprehensive guide to **Network Address Translation (NAT)** and **Port Address Translation (PAT)** on Cisco IOS routers, covering core concepts, addressing terminology, configuration examples, and verification commands.

---

## 1. Introduction

**NAT (Network Address Translation)** translates private IP addresses into public IP addresses and vice versa. This allows devices on a private local network (LAN) to communicate over the global Internet.

### Key Advantages

* **IPv4 Address Conservation:** Mitigates the exhaustion of public IPv4 addresses.
* **Security:** Hides the internal private network addressing scheme from external networks.

### RFC 1918 Private IP Address Ranges

| Class | Private IP Range | CIDR Notation |
| :--- | :--- | :--- |
| **Class A** | `10.0.0.0` – `10.255.255.255` | `10.0.0.0/8` |
| **Class B** | `172.16.0.0` – `172.31.255.255` | `172.16.0.0/12` |
| **Class C** | `192.168.0.0` – `192.168.255.255` | `192.168.0.0/16` |

---

## 2. NAT Terminology

NAT defines two main address families—**Local** (private/internal) and **Global** (public/external)—which combine into four specific address types:

| Term | Description |
| :--- | :--- |
| **Inside Local** | The private IP address assigned to a host inside the LAN. |
| **Inside Global** | The public IP address (assigned by the ISP) that represents the internal host to the Internet. |
| **Outside Global** | The public IP address assigned to a host on the external network (Internet). |
| **Outside Local** | The IP address of an external host as it appears to users on the internal network. |

---

## 3. Types of NAT & Configuration

### A. Static NAT

Maps one **Inside Local** address to one **Inside Global** address (1-to-1 mapping).

* **Use Case:** Making internal servers (e.g., web or mail servers) accessible from the Internet.

#### Configuration

```cisco
! Map internal IP 192.168.1.2 to public IP 202.67.3.1
Router(config)# ip nat inside source static 192.168.1.2 202.67.3.1

! Define the inside (LAN) interface
Router(config)# interface FastEthernet0/0
Router(config-if)# ip nat inside
Router(config-if)# exit

! Define the outside (WAN) interface
Router(config)# interface Serial2/0
Router(config-if)# ip nat outside
Router(config-if)# exit
```

### B. Dynamic NAT

Automatically maps internal private IP addresses to the first available public IP address from a predefined **pool** of public addresses.

#### Configuration

```cisco
! 1. Create an ACL to permit the internal network to be translated
Router(config)# access-list 1 permit 192.168.1.0 0.0.0.255

! 2. Define the pool of public IP addresses
Router(config)# ip nat pool TMSIR 202.67.3.11 202.67.3.14 netmask 255.255.255.0

! 3. Link the ACL to the NAT pool
Router(config)# ip nat inside source list 1 pool TMSIR

! 4. Assign NAT roles to interfaces
Router(config)# interface FastEthernet0/0
Router(config-if)# ip nat inside
Router(config-if)# exit

Router(config)# interface Serial2/0
Router(config-if)# ip nat outside
Router(config-if)# exit
```

### C. PAT (Port Address Translation / NAT Overload)

Maps multiple **Inside Local** IP addresses to a **single** public **Inside Global** IP address by differentiating sessions using 16-bit source port numbers.

* **Capacity:** Up to 65,536 theoretical translations per public IP (practically around 4,000).
* **Port Allocation:** PAT attempts to preserve the original source port. If unavailable, it assigns a port from the following ranges: `0–511`, `512–1023`, or `1024–65535`.

#### Configuration (Using the Outgoing Interface)

```cisco
! 1. Create an ACL for the internal network
Router(config)# access-list 1 permit 192.168.1.0 0.0.0.255

! 2. Enable PAT (overload) on the outside interface
Router(config)# ip nat inside source list 1 interface Serial2/0 overload

! 3. Assign NAT roles to interfaces
Router(config)# interface FastEthernet0/0
Router(config-if)# ip nat inside
Router(config-if)# exit

Router(config)# interface Serial2/0
Router(config-if)# ip nat outside
Router(config-if)# exit
```

*(Note: You can also overload a NAT pool using `ip nat inside source list <acl-num> pool <pool-name> overload`.)*

---

## 4. Verification & Troubleshooting

Run the following commands in **Privileged EXEC mode** (`Router#`) to monitor, verify, and clear NAT/PAT translations:

| Command | Description |
| :--- | :--- |
| `show ip nat translations` | Displays active NAT/PAT translations and their durations. |
| `show ip nat statistics` | Displays NAT statistics, hit/miss counters, and interface roles. |
| `clear ip nat translation *` | Clears all dynamic NAT/PAT translation entries from the table. |
| `debug ip nat [detailed]` | Displays real-time NAT translation events for troubleshooting. |
