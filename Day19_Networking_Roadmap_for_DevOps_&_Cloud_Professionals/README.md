# Day 19: Networking Roadmap for DevOps & Cloud Professionals

> **NexusCorp DevOps Transformation** | Cloud & Infrastructure Series

---

## 📋 Overview

Every piece of cloud infrastructure you will ever build, every container you will ever deploy, every CI/CD pipeline you will ever run depends on a network working correctly underneath it. Networking is not optional knowledge for a DevOps engineer — it is the substrate everything else runs on.

Day 19 is structured differently from previous days. Rather than a single deep-dive topic, it is a **networking roadmap** — a map of 19 interconnected concepts that you will explore through self-directed research and document in your own reference guide. The goal is not passive consumption but active knowledge building.

By the end of this session you will have a personal networking reference that covers every major concept a DevOps or Cloud professional encounters in real infrastructure work.

---

## 🗺️ The 19 Networking Topics

```
┌─────────────────────────────────────────────────────────────────┐
│            Networking Roadmap — DevOps & Cloud                  │
├─────────────────────────────────────────────────────────────────┤
│  FOUNDATIONS          │  INFRASTRUCTURE       │  SECURITY       │
│  1. Networking Intro  │  3. Network Devices   │  8.  Firewalls  │
│  2. IP, Ports, OSI    │  5. DHCP              │  9.  VLANs      │
│                       │  7. NAT               │  10. VPN        │
│  PROTOCOLS            │  11. Load Balancing   │  12. SSL/TLS    │
│  4. DNS               │  15. SDN              │  18. IDS/IPS    │
│  6. TCP/UDP/ICMP      │  16. Network Auto     │                 │
│  17. IPv4 vs IPv6     │                       │  OPERATIONS     │
│                       │  DELIVERY             │  13. Monitoring │
│                       │  19. CDN              │  14. QoS        │
└─────────────────────────────────────────────────────────────────┘
```

---

## 🏢 NexusCorp Scenario

> NexusCorp is expanding from a single data center to a multi-region cloud deployment. The platform team needs engineers who understand not just how to deploy applications, but *why* traffic routes the way it does, *how* DNS resolves service names, *what* happens when a firewall rule blocks a health check, and *where* a packet goes when it leaves a container.
>
> The engineers who understand networking are the ones who can debug the 2 AM incident. Today you start building that foundation.

---

## How to Use This Day

This is a **self-learning and research day**. For each of the 19 topics:

1. Read the concept overview provided here
2. Research deeper using the resources listed
3. Write your own explanation in the reference guide template at the end
4. Connect the dots — understand how each topic relates to the others

The reference guide you build today becomes a living document. Add to it as you encounter these concepts in real infrastructure work.

---

## Part I: Foundations

---

### Topic 1: Introduction to Networking

**What is Networking?**

A computer network is a system of interconnected devices that can communicate and share resources. In DevOps and cloud contexts, networking is what allows your application server to talk to your database, your load balancer to distribute traffic, and your users to reach your service from anywhere in the world.

**Key concepts to research:**
- What is a network protocol?
- What is the difference between LAN (Local Area Network), WAN (Wide Area Network), and the internet?
- What are nodes, hosts, clients, and servers in a network context?

**Understanding Data Packets**

Data transmitted across a network is broken into **packets** — small chunks of data with a header (containing routing information) and a payload (the actual data). Packets from the same transmission may travel different routes and are reassembled at the destination.

**Key concepts to research:**
- What information is in a packet header? (source IP, destination IP, port, protocol, TTL)
- What is packet fragmentation and why does it happen?
- What happens when packets arrive out of order?
- What is MTU (Maximum Transmission Unit)?

**DevOps relevance:** When debugging a microservice that times out, you are often asking: where did the packet go? Tools like `tcpdump` and Wireshark let you capture and inspect packets in transit.

---

### Topic 2: Foundational Concepts — IP Addressing, Ports, OSI & TCP/IP Models

**IP Addressing and Subnetting**

Every device on a network has an IP address — a unique numerical label used for identification and routing.

| Concept | What to Research |
|---|---|
| IPv4 address structure | 32-bit, four octets (e.g., `192.168.1.45`) |
| IPv6 address structure | 128-bit, eight groups of hex (e.g., `2001:db8::1`) |
| Public vs private IP ranges | RFC 1918: `10.x.x.x`, `172.16-31.x.x`, `192.168.x.x` |
| CIDR notation | `192.168.1.0/24` — what does `/24` mean? |
| Subnetting | How to divide a network into smaller subnets |
| Subnet mask | `255.255.255.0` — what does this define? |

**Key calculation to understand:**
```
Network: 192.168.1.0/24
Subnet mask: 255.255.255.0
Usable hosts: 192.168.1.1 – 192.168.1.254  (254 hosts)
Network address: 192.168.1.0
Broadcast address: 192.168.1.255
```

**Ports and Protocols**

Ports allow a single IP address to host multiple services simultaneously. Think of an IP address as a building address — the port is the apartment number.

| Port Range | Type | Examples |
|---|---|---|
| 0–1023 | Well-known / system ports | 22 (SSH), 80 (HTTP), 443 (HTTPS), 53 (DNS), 3306 (MySQL) |
| 1024–49151 | Registered ports | 8080 (HTTP alt), 5432 (PostgreSQL), 6379 (Redis) |
| 49152–65535 | Dynamic / ephemeral ports | Used temporarily by clients for outbound connections |

**Key concepts to research:**
- Why does HTTP use port 80 and HTTPS use port 443?
- What is an ephemeral port and when is it assigned?
- How do firewalls use port numbers to control traffic?

**The OSI Model**

The OSI (Open Systems Interconnection) model is a conceptual framework that divides network communication into 7 layers. It is the reference model used to describe where a protocol or problem sits in the network stack.

| Layer | Name | What Happens Here | Examples |
|---|---|---|---|
| 7 | Application | User-facing protocols | HTTP, HTTPS, FTP, DNS, SMTP |
| 6 | Presentation | Encryption, compression, encoding | SSL/TLS, JPEG, ASCII |
| 5 | Session | Managing sessions between applications | NetBIOS, RPC |
| 4 | Transport | End-to-end data transfer, error recovery | TCP, UDP |
| 3 | Network | Logical addressing and routing | IP, ICMP, OSPF |
| 2 | Data Link | Physical addressing, frame delivery | Ethernet, MAC addresses, ARP |
| 1 | Physical | Raw bit transmission | Cables, radio waves, fiber |

**The TCP/IP Model**

The TCP/IP model is the practical implementation used in real networks — fewer layers, maps directly to how the internet works:

| TCP/IP Layer | Equivalent OSI Layers | Protocols |
|---|---|---|
| Application | 5, 6, 7 | HTTP, DNS, SMTP, SSH |
| Transport | 4 | TCP, UDP |
| Internet | 3 | IP, ICMP, ARP |
| Network Access | 1, 2 | Ethernet, Wi-Fi |

**DevOps relevance:** When a service "can't connect," engineers think in layers — is it a physical issue (Layer 1–2), a routing issue (Layer 3), a port/firewall issue (Layer 4), or an application configuration issue (Layer 7)?

---

## Part II: Infrastructure

---

### Topic 3: Network Devices and Infrastructure

**Routers, Switches, and Hubs**

| Device | Layer | Function |
|---|---|---|
| **Hub** | Layer 1 | Broadcasts all traffic to all ports — dumb, rarely used today |
| **Switch** | Layer 2 | Forwards frames based on MAC addresses to the correct port only |
| **Router** | Layer 3 | Routes packets between different networks using IP addresses |
| **Gateway** | Layer 3+ | Entry/exit point between networks — often the router's role |

**Key concepts to research:**
- How does a switch learn which MAC address is on which port (MAC address table)?
- What is a default gateway and why does every host need one?
- How does a router decide which path to send a packet? (routing tables, static vs dynamic routing)

**Ethernet vs Wi-Fi**

| | Ethernet | Wi-Fi |
|---|---|---|
| Medium | Physical cable (copper or fiber) | Radio waves |
| Speed | Up to 100 Gbps (fiber) | Up to several Gbps (Wi-Fi 6) |
| Latency | Very low | Higher, variable |
| Reliability | High — no interference | Susceptible to interference |
| Security | Harder to intercept | Easier to intercept without encryption |
| DevOps use | Servers, data centers | Developer workstations, IoT |

**Network Topologies**

| Topology | Structure | Pros | Cons |
|---|---|---|---|
| Star | All devices connect to a central switch | Easy to manage, failure isolates to one device | Central switch is single point of failure |
| Bus | All devices on a single cable | Simple | One break affects all |
| Ring | Devices in a loop | Predictable performance | One break affects all |
| Mesh | Every device connected to every other | Highly redundant | Complex, expensive |
| Hybrid | Combination of above | Flexible | Complex |

**DevOps relevance:** Cloud VPC (Virtual Private Cloud) design mirrors physical network topology concepts — you design subnets, route tables, and gateways the same way a network engineer designs a physical network.

---

### Topic 5: Dynamic Host Configuration Protocol (DHCP)

**Role of DHCP in IP Assignment**

DHCP automatically assigns IP addresses to devices on a network, eliminating the need to configure each device manually. Without DHCP, every server, container, and workstation would need a manually configured static IP.

**The DHCP Lease Process (DORA)**

```
Client                              DHCP Server
  │                                     │
  │──── DISCOVER (broadcast) ──────────►│  "I need an IP address"
  │                                     │
  │◄─── OFFER ──────────────────────────│  "Here's 192.168.1.50, valid for 24h"
  │                                     │
  │──── REQUEST (broadcast) ───────────►│  "I'll take 192.168.1.50"
  │                                     │
  │◄─── ACKNOWLEDGE ────────────────────│  "It's yours for 24 hours"
```

**Key concepts to research:**
- What happens when a DHCP lease expires?
- What is a DHCP reservation and when would you use one?
- How does DHCP work in containerized environments (Docker, Kubernetes)?
- What is a DHCP scope?

**DevOps relevance:** In cloud environments, DHCP is handled by the cloud provider's VPC infrastructure. When you launch an EC2 instance, it receives its private IP via DHCP from AWS's managed DHCP service.

---

### Topic 7: Network Address Translation (NAT)

**Purpose of NAT**

NAT allows multiple devices on a private network to share a single public IP address when accessing the internet. It translates private IP addresses to a public IP (and back) at the network boundary.

```
Private Network          Router/NAT          Internet
192.168.1.10  ──────────►                ──────────► 203.0.113.5:54321
192.168.1.11  ──────────►  Public IP:    ──────────► 203.0.113.5:54322
192.168.1.12  ──────────►  203.0.113.5   ──────────► 203.0.113.5:54323
```

**Types of NAT:**
- **SNAT (Source NAT)** — translates the source IP (outbound traffic from private to public)
- **DNAT (Destination NAT)** — translates the destination IP (inbound traffic, e.g., port forwarding)
- **PAT (Port Address Translation)** — many-to-one, using port numbers to distinguish sessions (most common form)

**Key concepts to research:**
- Why did NAT become necessary? (IPv4 address exhaustion)
- What is port forwarding and how does it use DNAT?
- How does NAT work in AWS (NAT Gateway, NAT Instance)?
- What problems does NAT cause for peer-to-peer applications?

---

### Topic 11: Load Balancing

**Basics of Load Balancing**

A load balancer distributes incoming network traffic across multiple backend servers, ensuring no single server bears the full load. It is a critical component of any scalable, highly available architecture.

```
                    ┌─── Server 1 (192.168.1.10)
Users ──► Load ─────┼─── Server 2 (192.168.1.11)
         Balancer   └─── Server 3 (192.168.1.12)
```

**Load balancing algorithms:**

| Algorithm | How It Works | Best For |
|---|---|---|
| Round Robin | Requests distributed sequentially | Equal-capacity servers |
| Least Connections | Route to server with fewest active connections | Varying request duration |
| IP Hash | Same client always goes to same server | Session-based applications |
| Weighted Round Robin | Servers with higher capacity get more requests | Mixed-capacity servers |
| Health-check based | Only route to servers that pass health checks | High availability |

**Layer 4 vs Layer 7 Load Balancing:**

| | L4 (Transport) | L7 (Application) |
|---|---|---|
| Operates on | TCP/UDP | HTTP/HTTPS |
| Routing decision | Based on IP + port | Based on URL, headers, cookies |
| Speed | Faster (no content inspection) | Slower but smarter |
| Example (AWS) | NLB (Network Load Balancer) | ALB (Application Load Balancer) |

**DevOps relevance:** Every production application uses a load balancer. Understanding how they work is essential for designing health checks, configuring sticky sessions, and debugging traffic distribution issues.

---

### Topic 15: Software-Defined Networking (SDN)

**Introduction to SDN**

Traditional networking has the **control plane** (routing decisions) and **data plane** (packet forwarding) tightly coupled in each physical device. SDN separates them — the control plane is centralized in software (the SDN controller), while physical devices just forward packets as instructed.

```
Traditional:                     SDN:
Each router decides               Central SDN Controller
its own routing.                  tells all devices what to do.
```

**Key concepts to research:**
- What is the difference between control plane and data plane?
- What is OpenFlow and why is it significant?
- How does SDN enable network automation?
- How does Kubernetes networking use SDN concepts (CNI plugins)?

---

### Topic 16: Network Automation

**The Need for Automation**

Manual network configuration does not scale. A modern cloud environment may have hundreds of virtual networks, thousands of routing rules, and millions of firewall entries. Managing these manually is error-prone and slow.

**Key approaches:**

| Approach | Tools | What It Automates |
|---|---|---|
| Infrastructure as Code (IaC) | Terraform, CloudFormation | Provisioning VPCs, subnets, security groups |
| Configuration Management | Ansible, Salt | Configuring network device settings |
| API-driven networking | AWS API, GCP API | Creating/modifying network resources programmatically |
| GitOps for networking | ArgoCD, Flux | Applying network config from Git repositories |

**Key concepts to research:**
- How does Terraform manage AWS VPC resources?
- What is the difference between imperative and declarative network automation?
- How does Ansible configure network devices?

---

## Part III: Protocols

---

### Topic 4: Domain Name System (DNS)

**How DNS Works**

DNS translates human-readable domain names (`nexuscorp.com`) into IP addresses (`203.0.113.45`) that computers use to route traffic. It is the internet's phonebook.

**DNS resolution process:**

```
Browser asks: "What is the IP for nexuscorp.com?"

1. Check local cache → not found
2. Ask Recursive Resolver (usually your ISP or 8.8.8.8)
3. Recursive Resolver asks Root Name Server → returns TLD server address
4. Recursive Resolver asks .com TLD Server → returns nexuscorp.com NS records
5. Recursive Resolver asks nexuscorp.com Authoritative Name Server
6. Returns: 203.0.113.45
7. Browser connects to 203.0.113.45
```

**DNS Record Types:**

| Record | Purpose | Example |
|---|---|---|
| `A` | Maps hostname to IPv4 address | `nexuscorp.com → 203.0.113.45` |
| `AAAA` | Maps hostname to IPv6 address | `nexuscorp.com → 2001:db8::1` |
| `CNAME` | Alias — maps name to another name | `www → nexuscorp.com` |
| `MX` | Mail server for the domain | `nexuscorp.com → mail.nexuscorp.com` |
| `TXT` | Text records (SPF, DKIM, domain verification) | Various |
| `NS` | Name servers authoritative for the domain | `nexuscorp.com → ns1.cloudflare.com` |
| `PTR` | Reverse DNS — IP to hostname | `203.0.113.45 → nexuscorp.com` |
| `SOA` | Start of Authority — zone metadata | TTL, admin contact |

**DevOps relevance:** DNS configuration is critical for every deployment. Service discovery in Kubernetes uses DNS internally. Misconfigured DNS TTLs can cause stale cache issues during deployments.

---

### Topic 6: TCP vs UDP and ICMP

**TCP (Transmission Control Protocol)**

TCP provides **reliable, ordered, error-checked** delivery of data. Before sending data, TCP establishes a connection via the **three-way handshake**.

```
Three-Way Handshake:
Client ──── SYN ─────────────────────► Server    "Can we connect?"
Client ◄─── SYN-ACK ─────────────────  Server    "Yes, I'm ready"
Client ──── ACK ─────────────────────► Server    "Great, let's go"
[Connection established — data transfer begins]
```

**UDP (User Datagram Protocol)**

UDP is **connectionless** — it sends packets without establishing a connection or guaranteeing delivery. Faster than TCP because there is no handshake and no retransmission.

| | TCP | UDP |
|---|---|---|
| Connection | Connection-oriented (handshake) | Connectionless |
| Reliability | Guaranteed delivery, retransmits lost packets | Best-effort, no retransmission |
| Order | Packets delivered in order | May arrive out of order |
| Speed | Slower (overhead of reliability) | Faster |
| Use cases | HTTP/HTTPS, SSH, FTP, database connections | DNS, video streaming, VoIP, gaming |
| Header size | 20 bytes | 8 bytes |

**ICMP (Internet Control Message Protocol)**

ICMP is used for network diagnostics and error reporting — not for data transfer. The `ping` command uses ICMP echo requests and replies to test reachability and measure latency.

```bash
ping -c 4 google.com          # ICMP echo request/reply
traceroute google.com         # ICMP TTL-exceeded messages reveal each hop
```

**Key concepts to research:**
- Why would you choose UDP over TCP for a video streaming application?
- What is TCP flow control and congestion control?
- Why do some firewalls block ICMP and what impact does this have?

---

### Topic 17: IPv4 vs IPv6

| Feature | IPv4 | IPv6 |
|---|---|---|
| Address length | 32 bits | 128 bits |
| Address format | Dotted decimal: `192.168.1.1` | Hex groups: `2001:db8::1` |
| Total addresses | ~4.3 billion | ~340 undecillion |
| Header size | 20 bytes (variable) | 40 bytes (fixed) |
| NAT required? | Yes (address exhaustion) | No (enough addresses for every device) |
| Built-in security | No | IPsec built in |
| Adoption | ~99% of current internet | Growing rapidly |

**Key concepts to research:**
- Why are we running out of IPv4 addresses?
- What is IPv6 address autoconfiguration (SLAAC)?
- What is dual-stack networking?
- How does AWS handle IPv6 in VPCs?

---

## Part IV: Security

---

### Topic 8: Firewalls and Network Security

**Understanding Firewalls**

A firewall monitors and controls incoming and outgoing network traffic based on predefined security rules. It acts as a barrier between trusted internal networks and untrusted external networks.

**Types of firewalls:**

| Type | How It Works | Example |
|---|---|---|
| Packet filter | Inspects packet headers — source/dest IP and port | iptables, AWS Security Groups |
| Stateful | Tracks connection state — knows if a packet belongs to an established session | Most modern firewalls |
| Application (L7) | Inspects payload content — HTTP headers, URLs | AWS WAF, Nginx |
| Next-Generation (NGFW) | Combines stateful + application + IDS/IPS | Palo Alto, Fortinet |

**AWS equivalents:**

| AWS Feature | Firewall Role |
|---|---|
| Security Groups | Stateful instance-level firewall (allow rules only) |
| Network ACLs | Stateless subnet-level firewall (allow and deny rules) |
| WAF | Application layer protection (Layer 7) |

**Network Security Best Practices:**
- Principle of least privilege — allow only the traffic that is explicitly needed
- Default deny — block everything, then open specific ports
- Separate public and private subnets — databases should never be directly internet-accessible
- Regularly audit and review firewall rules — remove unused allow rules
- Enable logging on all firewall rules for audit and incident response

---

### Topic 9: Virtual LAN (VLAN)

**Why VLANs Are Used**

A VLAN logically segments a physical network into multiple isolated networks without requiring separate physical infrastructure. Devices on different VLANs cannot communicate directly — they must go through a router.

```
Physical Switch
├── VLAN 10 (Engineering) — ports 1–8
├── VLAN 20 (Finance) — ports 9–16
└── VLAN 30 (Guest Wi-Fi) — ports 17–24
```

**Key concepts to research:**
- What is VLAN tagging (IEEE 802.1Q)?
- What is a trunk port and how does it carry multiple VLANs?
- What is inter-VLAN routing?
- How do VLANs map to AWS VPC subnets?

---

### Topic 10: Virtual Private Networks (VPN)

**Understanding VPNs**

A VPN creates an encrypted tunnel over a public network (typically the internet), allowing devices to communicate as if they were on the same private network.

```
Remote Developer ──[encrypted tunnel]──► VPN Gateway ──► NexusCorp Private Network
(Internet)                                               (Internal resources)
```

**VPN use cases in DevOps:**
- Remote access to private cloud resources (AWS VPC, GCP VPC)
- Site-to-site VPN between on-premises data center and cloud VPC
- Connecting multiple cloud regions securely

**Common VPN protocols:**

| Protocol | Notes |
|---|---|
| OpenVPN | Open source, widely used, strong security |
| WireGuard | Modern, fast, simple configuration |
| IPsec | Standard protocol, used in site-to-site VPNs |
| AWS VPN | Managed site-to-site or client VPN service |

---

### Topic 12: SSL/TLS

**Understanding SSL/TLS**

SSL (Secure Sockets Layer) and its successor TLS (Transport Layer Security) encrypt data in transit between a client and server. Whenever you see `https://`, TLS is active.

**TLS Handshake (simplified):**

```
Client                              Server
  │                                    │
  │──── ClientHello (TLS version) ────►│
  │◄─── ServerHello + Certificate ─────│
  │     (verify certificate is         │
  │      signed by trusted CA)         │
  │──── Key Exchange ──────────────────►│
  │◄─── Finished ───────────────────────│
  │                                    │
  │  [Encrypted communication begins]  │
```

**Key concepts to research:**
- What is a TLS certificate and what information does it contain?
- What is a Certificate Authority (CA) and why do we trust them?
- What is the difference between symmetric and asymmetric encryption in TLS?
- What is SNI (Server Name Indication) and why is it important?
- How does Let's Encrypt work?

---

### Topic 18: Network Security Best Practices — IDS/IPS

**Intrusion Detection Systems (IDS) and Intrusion Prevention Systems (IPS)**

| | IDS | IPS |
|---|---|---|
| **Function** | Monitors and alerts on suspicious traffic | Monitors AND actively blocks suspicious traffic |
| **Placement** | Passive — out of band | Inline — in the traffic path |
| **Response** | Alert only | Alert + block/drop |
| **Risk** | False negatives — misses attacks | False positives — blocks legitimate traffic |

**Key concepts to research:**
- What is signature-based vs anomaly-based detection?
- How does AWS GuardDuty function as a cloud-native IDS?
- What is a honeypot and how is it used in network security?

---

## Part V: Operations

---

### Topic 13: Network Monitoring and Troubleshooting

**Basics of Network Monitoring**

Network monitoring tracks the health, performance, and availability of network infrastructure in real time. In DevOps, this means knowing when latency spikes, packet loss increases, or a service becomes unreachable — before users report it.

**Common monitoring metrics:**
- **Latency** — round-trip time between two points
- **Packet loss** — percentage of packets that don't reach the destination
- **Bandwidth utilization** — how much of available capacity is being used
- **Jitter** — variation in packet arrival times (critical for VoIP/video)
- **Error rates** — malformed or dropped packets on an interface

**Essential troubleshooting tools:**

| Tool | Purpose | Example |
|---|---|---|
| `ping` | Test reachability, measure latency | `ping -c 4 8.8.8.8` |
| `traceroute` / `tracepath` | Show each hop between source and destination | `traceroute google.com` |
| `nslookup` / `dig` | Query DNS records | `dig A nexuscorp.com` |
| `netstat` / `ss` | Show active connections and listening ports | `ss -tulnp` |
| `curl` / `wget` | Test HTTP connectivity and response | `curl -I https://nexuscorp.com` |
| `tcpdump` | Capture and inspect live packets | `tcpdump -i eth0 port 80` |
| `nmap` | Scan ports on a host | `nmap -sV 192.168.1.1` |
| `iperf3` | Measure network bandwidth between two hosts | `iperf3 -s` / `iperf3 -c host` |
| `wireshark` | GUI packet capture and analysis | GUI tool |
| `mtr` | Combines ping + traceroute with live updates | `mtr google.com` |

**Systematic troubleshooting approach:**
```
1. Can I reach my own gateway?        → ip route; ping <gateway>
2. Can I reach an external IP?        → ping 8.8.8.8
3. Does DNS resolve?                  → dig google.com
4. Is the service port open?          → nc -zv host port
5. Is the service responding?         → curl -I https://host
6. What does the packet path look like? → traceroute host
```

---

### Topic 14: Quality of Service (QoS)

**Understanding QoS**

QoS is a set of techniques to manage network traffic and ensure critical applications receive the bandwidth and low latency they need, even when the network is congested.

**Key mechanisms:**
- **Traffic classification** — identify and label traffic by type (VoIP, video, bulk data)
- **Prioritization** — give high-priority traffic preferential treatment
- **Traffic shaping** — rate-limit lower-priority traffic
- **Queuing** — manage how packets are buffered and forwarded

**Key concepts to research:**
- How does QoS work in cloud environments (AWS Traffic Mirroring, bandwidth limits)?
- What is DSCP (Differentiated Services Code Point)?
- How do Kubernetes resource limits relate to QoS concepts?

---

## Part VI: Delivery

---

### Topic 19: Content Delivery Networks (CDN)

**Understanding CDNs**

A CDN is a globally distributed network of servers (Points of Presence / PoPs) that cache and serve content from locations geographically close to the end user — reducing latency and offloading traffic from the origin server.

```
Without CDN:                     With CDN:
User in India ──────────────►    User in India ──► CDN PoP in Mumbai
Origin server in US              (Content served locally, ~10ms vs ~200ms)
(~200ms latency)
```

**What CDNs cache:**
- Static assets (images, CSS, JavaScript, fonts)
- Video content
- Software downloads
- API responses (with appropriate cache headers)

**Major CDN providers:** Cloudflare, AWS CloudFront, Fastly, Akamai, Azure CDN

**Key concepts to research:**
- What HTTP headers control CDN caching? (`Cache-Control`, `ETag`, `Expires`)
- What is cache invalidation and why is it hard?
- How does a CDN handle HTTPS certificates at the edge?
- What is edge computing and how does it extend the CDN model?
- How does AWS CloudFront integrate with S3 and EC2?

---

## 🔧 Your Challenge: Build Your Reference Guide

For each of the 19 topics, research, explore, and document your findings. The guide you build is tailored to your own understanding — not copied from documentation, but written in your own words.

### Reference Guide Template

Use this structure for each topic:

```markdown
## Topic N: [Name]

### What It Is (1–2 sentences in your own words)

### How It Works (key mechanism — diagram or bullet points)

### Why It Matters for DevOps/Cloud
(One concrete scenario where this knowledge is used)

### Key Terms
- Term 1: definition
- Term 2: definition

### Commands / Tools
(If applicable — specific commands you tested)

### Questions I Still Have
(Honest — write what you don't yet understand)

### Resources That Helped Me
(Links, videos, articles)
```

---

## 📖 Glossary

| Term | Definition |
|---|---|
| **Packet** | A unit of data transmitted over a network, containing a header (routing info) and payload (data) |
| **IP Address** | A numerical label assigned to each device on a network for identification and routing |
| **Subnet** | A logical subdivision of an IP network |
| **CIDR** | Classless Inter-Domain Routing — notation for IP ranges (e.g., `192.168.1.0/24`) |
| **Port** | A numerical endpoint on a host that identifies a specific service or process |
| **Protocol** | A set of rules governing how data is formatted and transmitted between devices |
| **OSI Model** | A 7-layer conceptual framework for understanding network communication |
| **TCP/IP Model** | The 4-layer practical model used by the internet |
| **Router** | A Layer 3 device that forwards packets between different networks |
| **Switch** | A Layer 2 device that forwards frames within a network based on MAC addresses |
| **DHCP** | Dynamic Host Configuration Protocol — automatically assigns IP addresses to devices |
| **DNS** | Domain Name System — translates domain names to IP addresses |
| **NAT** | Network Address Translation — maps private IPs to a public IP for internet access |
| **Firewall** | A system that monitors and controls network traffic based on security rules |
| **VPN** | Virtual Private Network — encrypted tunnel over a public network |
| **VLAN** | Virtual LAN — logical network segment on a shared physical switch |
| **Load Balancer** | Distributes incoming traffic across multiple backend servers |
| **SSL/TLS** | Protocols for encrypting data in transit between client and server |
| **IDS/IPS** | Intrusion Detection / Prevention System — monitors and optionally blocks malicious traffic |
| **SDN** | Software-Defined Networking — separates control plane from data plane |
| **CDN** | Content Delivery Network — geographically distributed servers that cache content near users |
| **QoS** | Quality of Service — traffic management to prioritize critical network flows |
| **TCP** | Transmission Control Protocol — reliable, ordered, connection-oriented data transfer |
| **UDP** | User Datagram Protocol — fast, connectionless, best-effort data transfer |
| **ICMP** | Internet Control Message Protocol — used for diagnostics (`ping`, `traceroute`) |
| **MTU** | Maximum Transmission Unit — maximum packet size a network can transmit |
| **TTL** | Time to Live — limits how many hops a packet can traverse |
| **MAC Address** | Hardware address of a network interface — unique per device |
| **ARP** | Address Resolution Protocol — maps IP addresses to MAC addresses |
| **BGP** | Border Gateway Protocol — routing protocol used between autonomous systems on the internet |

---

## 📚 Resources

- [What Happens When — GitHub](https://github.com/alex/what-happens-when) — an insightful deep dive into what happens when you type a URL in your browser — covers DNS, TCP, TLS, HTTP, and more
- [Top 100 Networking Interview Q&A — GitHub](https://github.com/bregman-arie/devops-exercises#network) — practical networking questions for DevOps interviews
- [TWS Computer Networking Masterclass — YouTube](https://www.youtube.com/results?search_query=TWS+Computer+Networking+Masterclass) — comprehensive video course on networking fundamentals
- [Cloudflare Learning Center](https://www.cloudflare.com/learning/) — excellent free articles on DNS, TLS, CDN, DDoS, and more — written for practitioners
- [Julia Evans — Networking Zines](https://jvns.ca/) — visual, beginner-friendly explanations of networking concepts
- [Cisco Networking Basics](https://www.netacad.com/courses/networking/networking-basics) — free foundational networking course
- [AWS Networking Fundamentals](https://aws.amazon.com/training/learn-about/networking/) — cloud-specific networking from AWS

---

## 🔭 Day 20 Preview: Cloud Computing Fundamentals

With networking fundamentals established, you are ready for the infrastructure layer where DevOps engineering actually happens at scale.

Coming up:
- What is cloud computing? IaaS, PaaS, SaaS defined
- AWS core services — EC2, S3, IAM, VPC
- Cloud networking — how VPCs map to physical network concepts
- Security groups, NACLs, and route tables in practice
- How everything from Day 19 applies the moment you launch your first cloud instance

The networking concepts you've mapped today are not abstract — you'll see every single one of them appear in the AWS console within the next few sessions.

---

> 💬 *"A DevOps engineer who doesn't understand networking is like a driver who doesn't understand roads. You can follow GPS instructions for a while — until something goes wrong and you have no idea where you are."*
>
> — DevOps learning by Yukta