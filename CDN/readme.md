# Communication Data Networks (ECM DE 361): Self-Study Book
## Part I: Architecture and Protocol Layers (Lectures 1-4)
 
**How to use this book.** Every chapter is built as a chain: **unresolved bottleneck → mechanism that fixes it → new limitation created by the fix → next chapter**. At each step, ask the three Professor's-Lens questions:
 
> **K**: What information is *known* at this point?
> **D**: What *decision* must be made with it?
> **L**: What new *limitation* appears after that decision?
 
**Source note.** The lecture decks are outline-level: each gives the problem, the idea, four topics and the "next problem". The explanations, derivations and numericals below come from the textbooks the decks cite (Kurose & Ross, Forouzan, Tanenbaum et al.), organised along the decks' topic lists. Verify numbering and conventions against your adopted edition.
 
---
 
# Chapter 1: From Isolated Computers to Networks
 
## 1.1 The bottleneck: stand-alone machines
 
An isolated computer has three costs:
 
1. **Duplicated data**: the same file lives in many places and drifts out of sync.
2. **Slow collaboration**: sharing means physically moving media.
3. **Expensive resource sharing**: every machine needs its own printer, disk and licence.
**Network definition (precise).** A network is a set of **autonomous** devices (nodes) connected by links that exchange data and share services **under agreed rules (protocols)**. Three words carry the weight: *autonomous* (a terminal wired to a mainframe is not a network of peers), *links* and *agreed rules*.
 
## 1.2 Evolution as a chain of bottlenecks
 
| Stage | Bottleneck it inherited | Fix | New limitation |
|---|---|---|---|
| Terminals to mainframe | Compute is expensive, so share it | Many dumb terminals on one host | Single point of failure; host is the bottleneck |
| LANs | Local sharing of files and printers | Shared medium or switch inside a building | Reach limited to one site |
| Internetworks | Different LANs speak different link technologies | A common layer above them (IP, Chapter 18) | Heterogeneity forces best-effort delivery |
| The Internet | Billions of nodes, many administrators | Hierarchical addressing, decentralised routing | Security, management, scale |
 
## 1.3 Network goals and the tension between them
 
**Connectivity, resource sharing, reliability, scalability.** They conflict:
 
- Reliability wants **redundancy** (extra links, retransmissions), which costs money and capacity.
- Scalability wants **minimal per-node state**, which limits the guarantees the network can give.
## 1.4 Network types by scale
 
| Type | Typical span | Example | Typical technology |
|---|---|---|---|
| **PAN** | about 1-10 m | Bluetooth earbuds | Bluetooth, Zigbee |
| **LAN** | room to campus | Lab Ethernet, Wi-Fi | IEEE 802.3, 802.11 |
| **MAN** | city | Campus-to-campus fibre ring | Metro Ethernet |
| **WAN** | country to globe | ISP backbone | Leased lines, MPLS, optical |
 
## 1.5 The core design decision: how to share links
 
Consider $N$ users sharing a link of rate $R$. Two basic answers:
 
- **Circuit switching**: reserve a fixed share ($R/N$, or a time slot or frequency band) for the whole session. Guaranteed rate, but idle capacity is wasted.
- **Packet switching**: send data in packets that share the link on demand (**statistical multiplexing**). Efficient for bursty traffic, but packets can **queue** and **be dropped**.
### Worked numerical 1.1: statistical multiplexing
 
A link has $R = 1$ Mb/s. Each user sends at 100 kb/s when active and is active a fraction $p = 0.1$ of the time.
 
- **Circuit switching** supports $1\ \text{Mb/s} \,/\, 100\ \text{kb/s} = 10$ users.
- **Packet switching** with $N = 35$ users: congestion occurs only if more than 10 are active at once. With $X \sim \text{Binomial}(35,\,0.1)$:
$$P(X > 10) = \sum_{k=11}^{35} \binom{35}{k}\,0.1^{k}\,0.9^{35-k} \approx 4.2\times 10^{-4}$$
 
So packet switching carries **3.5 times as many users** at a congestion probability of about 0.04%.
 
**Professor's Lens.**
- **K**: average activity is 10%, peak rate is 100 kb/s.
- **D**: reserve for the peak (circuit) or for the statistics (packet)?
- **L**: packet switching trades the *guarantee* for *efficiency*, so it now needs queues, loss handling and congestion control. This is why Chapters 2-4 introduce delay, jitter and layers.
## 1.6 Application grounding: IoT and emergency networks
 
Applications fall into web/cloud, IoT and industrial control. They place very different demands:
 
| Class | Traffic | Priority |
|---|---|---|
| Web/cloud | Bulk, bursty | Throughput |
| IoT sensing | Tiny periodic packets, many nodes | Energy, scale |
| Industrial control | Small, periodic | **Bounded latency, low jitter** |
| Emergency communication | Rare but critical bursts | **Availability** under node failure |
 
For a disaster-communication mesh on ESP32-class nodes, the sensor reports are exactly the "bursty, low-duty-cycle" traffic of Numerical 1.1, which is why packet switching with a shared channel is the natural fit. The emergency requirement, however, is *reliability under failure*, which pushes you toward **redundant paths** (a mesh) rather than a single star.
 
## 1.7 Chapter 1 trade-off summary and bridge
 
The deck's "no single best solution" table, specialised:
 
| Approach | Strength | Limitation |
|---|---|---|
| Circuit switching | Guaranteed rate, no queueing | Wastes idle capacity; call setup |
| Packet switching | Efficient for bursts | Queueing delay, loss, jitter |
| Star topology | Simple, one cable per node | Hub is a single point of failure |
| Mesh topology | Redundant paths | Routing state, more complexity |
 
**Unresolved problem (the bridge).** We said a network needs "agreed rules", but a single wire does not make communication work. What exactly must cooperate for useful communication, and how do we *measure* its quality? That is Chapter 2.
 
---
 
# Chapter 2: How Data Becomes Communication
 
## 2.1 The bottleneck: a wire is not communication
 
Connecting two machines does not guarantee that the receiver can use what arrives. Five components must cooperate:
 
1. **Sender**: source of the message.
2. **Receiver**: destination.
3. **Message**: the information, as text, audio, video or sensor values.
4. **Transmission medium**: copper, fibre or radio.
5. **Protocol**: the agreed rules for format, order, timing and error handling.
Remove the protocol and a perfect wire still delivers garbage: the receiver does not know where the message starts, how it is encoded, or what to do when a bit flips. **This single observation is the seed of the whole course.**
 
Communication quality is judged on four axes: **delivery** (right destination), **accuracy** (unaltered), **timeliness** (within a deadline) and **jitter** (variation in delay).
 
## 2.2 Data versus signal
 
**Data** is the information; a **signal** is the physical representation that travels over the medium. Each can be analog or digital:
 
| | Analog | Digital |
|---|---|---|
| Data | Continuous values (temperature, voice) | Discrete values (bits, characters) |
| Signal | Continuously varying waveform | Finite set of levels |
 
Transmission therefore involves conversions: digital data to digital signal (**line coding**, Chapter 9), digital data to analog signal (**modulation**), and analog data to digital signal (**sampling**). Note the limitation: *every conversion adds a place for error*.
 
## 2.3 Direction of flow
 
| Mode | Directions | Example | Cost |
|---|---|---|---|
| **Simplex** | One way only | Keyboard to computer, broadcast TV | Cheapest, no feedback |
| **Half-duplex** | Both, but **one at a time** | Walkie-talkie, shared-channel Wi-Fi | Needs turn-taking (a MAC problem, Chapter 13) |
| **Full-duplex** | Both **simultaneously** | Telephone, switched Ethernet | Two channels or echo cancellation |
 
**Exam trap.** Half-duplex does *not* mean "half the speed". It means the two directions cannot be used simultaneously, and the extra cost is *coordination*.
 
## 2.4 Link structure
 
- **Point-to-point**: a dedicated link between exactly two devices. No sharing problem.
- **Multipoint (shared)**: more than two devices share one link. **New problem**: who may transmit? (Chapter 13.)
For $n$ devices, a full mesh of point-to-point links needs $\dfrac{n(n-1)}{2}$ links and $n-1$ ports per node. A star needs $n$ links.
 
## 2.5 Performance metrics (the core quantitative toolkit)
 
| Metric | Meaning | Unit |
|---|---|---|
| **Bandwidth** (analog) | Range of frequencies the channel passes | Hz |
| **Bandwidth** (digital) | Maximum bit rate $R$ | b/s |
| **Throughput** | Rate **actually achieved** | b/s |
| **Latency** | Time for a message to go end to end | s |
| **Jitter** | Variation in latency between packets | s |
 
The same word "bandwidth" has two meanings. The link between them (Nyquist/Shannon, Chapter 6) is the reason capacity is ultimately a *physical* limit.
 
**Latency components** (developed fully in Chapter 4):
 
$$T_{\text{latency}} = T_{\text{proc}} + T_{\text{queue}} + T_{\text{trans}} + T_{\text{prop}},\quad T_{\text{trans}} = \frac{L}{R},\quad T_{\text{prop}} = \frac{d}{v}$$
 
where $L$ is the packet size (bits), $R$ the link rate, $d$ the distance and $v$ the propagation speed (about $2\times10^{8}$ m/s in copper or fibre).
 
**Bandwidth-delay product (BDP)**: the number of bits "in flight" on a link,
 
$$\text{BDP} = R \times T_{\text{prop}}\quad(\text{one way}),\qquad \text{BDP}_{\text{RTT}} = R \times \text{RTT}$$
 
### Worked numerical 2.1: why a fast link can still give low throughput
 
A point-to-point link: $R = 10$ Mb/s, $d = 2000$ km, $v = 2\times10^8$ m/s. Packets of $L = 1500$ B $= 12{,}000$ bits, sent **stop-and-wait** (send one, wait for its acknowledgement).
 
1. $T_{\text{prop}} = \dfrac{2\times10^{6}}{2\times10^{8}} = 10\ \text{ms}$, so $\text{RTT} \approx 20$ ms (ignoring ACK transmission time).
2. $T_{\text{trans}} = \dfrac{12{,}000}{10^{7}} = 1.2\ \text{ms}$.
3. Link utilisation:
$$U = \frac{T_{\text{trans}}}{T_{\text{trans}} + \text{RTT}} = \frac{1.2}{1.2+20} \approx 0.0566 \;(5.66\%)$$
 
4. Throughput $= U \times R \approx 0.566$ Mb/s.
5. BDP (one way) $= 10^{7} \times 0.01 = 10^{5}$ bits $\approx 12.5$ kB, and $\text{BDP}_{\text{RTT}} = 2\times10^{5}$ bits. The sender must keep roughly **17 packets** outstanding ($2\times10^{5}/12{,}000 \approx 16.7$) to fill the pipe.
**Professor's Lens.**
- **K**: $R$, $d$, $L$.
- **D**: how many packets to keep in flight?
- **L**: more outstanding data needs **sequence numbers and buffers**, the origin of sliding-window protocols.
### Multi-hop wireless intuition (mesh nodes)
 
If every node shares **one half-duplex radio channel** and all nodes in a chain are within interference range, only one hop can transmit at a time. Under this idealised model, an $n$-hop path delivers roughly
 
$$\text{Throughput} \approx \frac{R}{n}\qquad (n \le 3),$$
 
and about $R/3$ for longer chains with spatial reuse. The qualitative lesson: **every extra hop costs capacity** in a half-duplex wireless mesh. This is a modelling assumption, not a measured ESP-MESH figure.
 
## 2.6 Bridge
 
We now know *what* must cooperate and *how to measure* it. But "protocol" is one word hiding enormous complexity: framing, addressing, routing, reliability, encoding, voltages. One monolithic program cannot be designed, tested and evolved. **Chapter 3 introduces layering as the answer.**
 
---
 
# Chapter 3: Why Networks Need Layers
 
## 3.1 The bottleneck: complexity
 
A single monolithic networking program would be impossible to design, test and evolve. Change the cable from copper to fibre and the web browser would have to change. **Layering** decomposes communication into **services, interfaces and peer protocols**.
 
**Three terms to separate cleanly (frequent exam target):**
 
| Term | Direction | Meaning |
|---|---|---|
| **Service** | Vertical (layer $N$ to layer $N+1$) | What layer $N$ offers upward |
| **Interface** | Vertical | How the upper layer requests the service |
| **Protocol** | **Horizontal** (peer to peer) | Rules between the same layer on two machines |
 
The result is **modularity**: a technology can change inside one layer without redesigning the network.
 
## 3.2 The two reference models
 
| OSI layer | Function | PDU | TCP/IP layer |
|---|---|---|---|
| 7 Application | User-facing services | Message / data | Application |
| 6 Presentation | Encoding, encryption, compression | Message | (merged into Application) |
| 5 Session | Dialogue control | Message | (merged into Application) |
| 4 Transport | Process-to-process delivery | **Segment** (TCP) / **datagram** (UDP) | Transport |
| 3 Network | Host-to-host delivery across networks | **Packet** | Internet |
| 2 Data link | Hop-to-hop delivery in one network | **Frame** | Link |
| 1 Physical | Bits as signals | **Bit / signal** | Physical (sometimes grouped with Link) |
 
**Mnemonic for the unit of delivery:** physical delivers *bits*, link delivers *frames over one hop*, network delivers *packets host to host*, transport delivers *segments process to process*.
 
**OSI versus TCP/IP.** OSI is a conceptual model (the theory); TCP/IP is the implemented stack. Exam questions often ask which layers TCP/IP merges (Presentation and Session into Application) and what the Internet layer does (IP).
 
## 3.3 Encapsulation and decapsulation
 
On the way down the stack the sender **encapsulates**: each layer wraps the unit from above with its own **header** (the link layer adds a **trailer** as well). The receiver **decapsulates** on the way up.
 
$$\text{Frame} = H_{2} \,\|\, \underbrace{H_{3} \,\|\, \underbrace{H_{4} \,\|\, \text{Data}}_{\text{segment}}}_{\text{packet}} \,\|\, T_{2}$$
 
**Overhead and efficiency.** With payload $P$ bits and total header/trailer $H$ bits,
 
$$\eta = \frac{P}{P + H}$$
 
### Worked numerical 3.1
 
A TCP segment carries $P = 1460$ B. Overheads: TCP 20 B, IPv4 20 B, Ethernet header 14 B, Ethernet FCS 4 B (so $H = 58$ B).
 
$$\eta = \frac{1460}{1460+58} = \frac{1460}{1518} \approx 96.2\%$$
 
Counting preamble + SFD (8 B) and inter-frame gap (12 B) as well, the wire carries $1538$ B:
 
$$\eta_{\text{wire}} = \frac{1460}{1538} \approx 94.9\%$$
 
For a tiny IoT payload of **1 B**: $\eta = \dfrac{1}{1+58} \approx 1.7\%$.
 
**Professor's Lens.**
- **K**: payload size, header sizes.
- **D**: aggregate sensor readings into larger packets, or send each immediately?
- **L**: aggregation raises efficiency but adds **latency and loss exposure**. This is exactly the trade-off an IoT firmware designer faces.
## 3.4 What layering costs
 
| Benefit | Cost |
|---|---|
| Modularity, interoperability | **Header overhead** on every packet |
| Replaceable technologies | Duplicated functions (error checking at several layers) |
| Divide-and-conquer design | **Hidden information** between layers, so cross-layer optimisation (for example radio signal strength guiding routing) is hard |
 
## 3.5 Layered view of an ESP-MESH style emergency node
 
| Layer | Role in the node |
|---|---|
| Application | Sensor/alert message (for example "water level high") |
| Transport / network | UDP/TCP and IP toward a gateway; mesh forwarding between nodes |
| Link | Wi-Fi MAC: frames, MAC addresses, channel access |
| Physical | 2.4 GHz radio |
 
ESP-MESH builds a self-organising, self-healing network on top of Wi-Fi, where nodes relay packets toward a root node. Treat it as a layer that **provides multi-hop forwarding using the link layer beneath it**. Each hop is a separate link-layer delivery, which is the key idea of Chapter 4.
 
## 3.6 Bridge
 
Knowing the layer names is not the same as understanding a transmission. *What actually happens, field by field and decision by decision, when a browser opens a web page?* **Chapter 4.**
 
---
 
# Chapter 4: A Packet's End-to-End Journey
 
## 4.1 The bottleneck: names without mechanism
 
Students recite layer names yet cannot explain a page load. The cure is to trace **one message** from application data to segments, packets, frames and signals, and back.
 
## 4.2 Addressing at each layer
 
| Layer | Identifier | Scope | Changes hop to hop? |
|---|---|---|---|
| Application | URL / hostname | Global name | No |
| Transport | **Port number** | Process on a host | No |
| Network | **IP address** | Host interface, global logical | **No** (unless NAT) |
| Link | **MAC address** | One local network | **Yes, rewritten at every router** |
 
**Rule to memorise:** across routers, the **IP addresses stay the same** while the **MAC addresses change on every hop**. (TTL decrements and the header checksum is recomputed at each router.)
 
## 4.3 Switch versus router: two different decisions
 
| | **Switch** (layer 2) | **Router** (layer 3) |
|---|---|---|
| **K** (known) | Destination **MAC** in the frame | Destination **IP** in the packet |
| **D** (decision) | Which **port** to forward the frame out of (forwarding table learned by observing source MACs, Chapter 15) | Which **next hop** (routing table, longest-prefix match) |
| Header rewritten? | No | Yes: new link-layer header; TTL decremented |
| **L** (new limitation) | Stays inside one broadcast domain | Needs routing state and per-packet processing |
 
## 4.4 Tracing a page load
 
1. **DNS**: the host knows a hostname but needs an IP address. A UDP query goes to the resolver.
2. **Same subnet or not?** The host compares the destination IP against its own subnet. If remote, the frame goes to the **default gateway's MAC**, which requires **ARP** (Chapter 17).
3. **TCP handshake**: SYN, SYN-ACK, ACK (one RTT before any data moves).
4. **HTTP request** is encapsulated: HTTP data, TCP segment, IP packet, Ethernet frame, line-coded signal.
5. **Each hop** decapsulates the frame, makes a forwarding decision, and re-encapsulates into a new frame for the next link.
6. **Server** reverses the path; the response is segmented, so many packets flow in a pipeline.
Notice that each step supplies information the previous one lacked: DNS supplies the IP, ARP supplies the MAC, TCP supplies reliability. **Each missing field forces a protocol into existence.**
 
## 4.5 The four delay components
 
$$d_{\text{nodal}} = d_{\text{proc}} + d_{\text{queue}} + d_{\text{trans}} + d_{\text{prop}}$$
 
| Component | Formula | Depends on |
|---|---|---|
| **Processing** | microseconds, typically small | Header check, lookup |
| **Queueing** | Varies per packet (a source of **jitter**) | Traffic intensity |
| **Transmission** | $d_{\text{trans}} = L/R$ | Packet size, link rate |
| **Propagation** | $d_{\text{prop}} = d/v$ | Distance, medium |
 
**Traffic intensity** $I = \dfrac{La}{R}$ ($a$ = average packet arrival rate in packets/s). As $I \to 1$, queueing delay grows without bound; if $I > 1$ the queue grows indefinitely and packets are dropped.
 
**Transmission versus propagation, the classic confusion.** Transmission delay is the time to *push all bits onto the wire* (depends on $L$ and $R$). Propagation delay is the time for *one bit to travel the wire* (depends on $d$ and $v$). Doubling $R$ halves $d_{\text{trans}}$ but leaves $d_{\text{prop}}$ untouched.
 
## 4.6 Store-and-forward and pipelining
 
Routers and switches receive an **entire** packet before forwarding it. For a message of $M$ bits, split into $P$ packets of $L = M/P$ bits, sent over $N$ links of equal rate $R$ (propagation and queueing ignored):
 
$$T_{\text{no split}} = N\,\frac{M}{R},\qquad T_{\text{split}} = (N + P - 1)\,\frac{L}{R}$$
 
### Worked numerical 4.1
 
$M = 8\times10^{6}$ bits over $N = 3$ links, each $R = 2$ Mb/s.
 
- No segmentation: $T = 3\times\dfrac{8\times10^{6}}{2\times10^{6}} = 12\ \text{s}$.
- With $P = 1000$ packets of $L = 8000$ bits: $T = (3+1000-1)\times\dfrac{8000}{2\times10^{6}} = 1002\times 0.004 = 4.008\ \text{s}$.
- With only $P = 10$ packets ($L = 8\times10^{5}$ bits): $T = 12\times 0.4 = 4.8$ s.
Segmenting cuts the delay roughly threefold ($N$-fold in the limit), because the links work **in parallel** once the pipeline fills. **But** smaller packets mean more headers (Section 3.3), so there is an optimum.
 
**Professor's Lens.**
- **K**: $M$, $N$, $R$.
- **D**: how many packets?
- **L**: header overhead $\eta$ falls as packets shrink, and per-packet processing and queueing rise.
## 4.7 Mesh application
 
In a multi-hop mesh, each hop is its own **link-layer delivery**, with its own transmission delay and (on a shared half-duplex radio) its own channel contention. Per-hop store-and-forward means a 4-hop route has four times the transmission delay of a single hop, plus queueing at each relay. For emergency traffic, the useful design question is not "what is the link rate?" but **"what is the worst-case end-to-end delay over the longest route?"**
 
## 4.8 Bridge
 
We have been quietly assuming that a "link" simply moves bits. It does not: copper, fibre and radio carry **waveforms**, not bits. How can information cross a physical medium at all? **Chapter 5** (Part II) begins at the bottom of the stack.
 
---
 
# Part I Question Bank (Predictive)
 
> **Reading the patterns honestly.** I cannot see your university's past papers, so "predictive" here means: (a) the lecture decks' own topic lists, (b) the standard assessment forms for these topics (definitions-with-reasoning, one multi-step delay numerical, one trade-off justification), and (c) the repeated emphasis in the decks on "address / field / decision / limitation". Treat the probability tags as my judgement, not data.
>
> **GATE note.** As far as I know, the GATE ECE syllabus does not list computer networks (CSMA/CD, ARP, VLANs and so on); those are in the CS/IT paper. What *does* carry over to GATE ECE is the **Shannon/Nyquist and information-theory material (Part II)** and the **coding/CRC material**. Check the current official GATE brochure to confirm. Part I and Part III questions here build analytical stamina and target the university exam.
 
## Section A: Conceptual with reasoning (2-3 marks each)
 
**Q1. [High probability]** Distinguish a **service**, an **interface** and a **protocol**. Explain why replacing Ethernet with Wi-Fi does not require changing the web browser.
 
*Answer outline.* Service and interface are vertical, protocol is horizontal (peer to peer). The browser depends only on the transport service, which depends on the network service, which hides the link technology. The IP layer's service model is unchanged when the link layer changes.
 
**Q2. [High]** State the five components of a data communication system. Which one's absence makes a perfect physical channel useless, and why?
 
*Answer outline.* Protocol: without agreed format, timing, addressing and error handling, the receiver cannot interpret what arrives.
 
**Q3. [Medium-high]** Half-duplex versus full-duplex: give one example each and explain what extra mechanism half-duplex needs.
 
*Answer outline.* Turn-taking / channel access (a MAC protocol), because two senders on the same channel at the same time collide.
 
**Q4. [High]** A packet crosses three routers. Which of {source IP, destination IP, source MAC, destination MAC, TTL, port numbers} change at each router? Justify.
 
*Answer.* Source MAC, destination MAC and TTL change at every router (and the IPv4 header checksum is recomputed because TTL changes). Source/destination IP and ports are unchanged (ignoring NAT). Reason: MAC addresses have local scope (one link), IP has end-to-end scope.
 
## Section B: Numericals
 
**Q5. [Very high, multi-step]** A source sends a file of $12\times10^{6}$ bits through a path of **4 links** (3 routers). Every link has $R = 10$ Mb/s and propagation delay $5$ ms. Ignore processing and queueing. Compute the end-to-end delay when
(a) the file is sent as one packet; (b) it is split into 1200 packets of 10,000 bits each (ignore headers).
 
*Solution.*
(a) $T = N\dfrac{M}{R} + N\,d_{\text{prop}} = 4\times 1.2 + 4\times0.005 = 4.8 + 0.02 = 4.82\ \text{s}$.
(b) $L/R = 10^{4}/10^{7} = 1\ \text{ms}$. $T = (N+P-1)\dfrac{L}{R} + N\,d_{\text{prop}} = (4+1200-1)\times0.001 + 0.02 = 1.203 + 0.02 = 1.223\ \text{s}$.
*Insight:* about 3.9 times faster, bounded above by $N = 4$.
 
**Q6. [Very high]** A link has $R = 10$ Mb/s and length $2000$ km ($v = 2\times10^{8}$ m/s). A stop-and-wait sender uses 1500 B packets. Find (a) $T_{\text{prop}}$, (b) the utilisation, (c) the BDP in packets, and (d) the throughput.
 
*Solution.* (a) 10 ms. (b) $U = \dfrac{1.2}{1.2+20} \approx 5.66\%$. (c) $\text{BDP}_{\text{RTT}} = 2\times10^{5}$ bits $\approx 16.7$ packets, so about 17 packets must be outstanding. (d) $\approx 0.566$ Mb/s.
 
**Q7. [High]** 35 users share a 1 Mb/s link; each is active 10% of the time and needs 100 kb/s when active. (a) How many users can circuit switching support? (b) Define the condition under which packet switching suffers congestion and state its probability expression. (c) Why is the answer an approximation of real traffic?
 
*Solution.* (a) 10. (b) More than 10 users simultaneously active, $P = \sum_{k=11}^{35}\binom{35}{k}0.1^{k}0.9^{35-k} \approx 4.2\times10^{-4}$. (c) Real users are not independent and activity is not memoryless (traffic is correlated and bursty at several timescales).
 
**Q8. [Medium-high]** A TCP segment carries 1460 B. Headers: TCP 20, IP 20, Ethernet 14 + 4. Compute efficiency (a) for the frame and (b) including the 8 B preamble and 12 B inter-frame gap. (c) Repeat (a) for a 10 B IoT payload (ignore Ethernet padding) and comment.
 
*Solution.* (a) $1460/1518 \approx 96.2\%$. (b) $1460/1538 \approx 94.9\%$. (c) $10/68 \approx 14.7\%$. Per-packet overhead dominates small payloads, so batching reduces overhead at the cost of latency.
 
## Section C: Multi-step analytical questions
 
**Q9. [High]** *Trace-the-journey.* Host A (192.168.1.10/24) browses to a server on a different network. The gateway is 192.168.1.1. Describe the sequence of (i) DNS, (ii) subnet check, (iii) ARP, (iv) TCP handshake, (v) first data frame. For the first data frame, state the source and destination **MAC** and **IP** at A's NIC and at the far side of the first router.
 
*Key points.* At A's NIC: src MAC = A, dst MAC = **gateway's MAC** (not the server's), src IP = A, dst IP = server. After the first router: src MAC = router's outgoing interface, dst MAC = next hop (or server if directly attached), IPs unchanged, TTL reduced by 1.
 
**Q10. [Medium-high]** *Remove one mechanism.* Predict the failure if (a) ARP is disabled, (b) segmentation is removed, (c) the transport layer drops port numbers. What next-layer mechanism does each failure motivate?
 
**Q11. [High]** *Where is the bottleneck?* Three links in series have rates 100 Mb/s, 10 Mb/s and 100 Mb/s, equal propagation delays. A long file is sent. (a) What is the steady-state throughput? (b) Which delay component grows at the router before the slow link, and why? (c) What happens if the arrival rate exceeds the slow link rate for a long time?
 
*Answer.* (a) 10 Mb/s (the minimum link rate). (b) Queueing delay, as traffic intensity $I = La/R$ approaches or exceeds 1. (c) The buffer fills and packets are dropped, which motivates congestion control.
 
## Section D: Strict protocol trade-off evaluations (4-5 marks)
 
**Q12. [High]** Compare **circuit switching** and **packet switching** for (i) a voice call, (ii) bursty sensor reports from 500 IoT nodes, and (iii) an emergency-alert broadcast that must arrive within a bounded time. Recommend a design for each and state the new limitation your choice introduces.
 
*Grading skeleton:* each recommendation needs a *reason tied to traffic shape*, and a *named cost* (setup/waste for circuit; queueing, jitter and loss for packet).
 
**Q13. [Medium-high]** A designer must choose between **sending each sensor reading immediately (payload 20 B)** and **batching 10 readings per packet (payload 200 B)**. Headers total 58 B. (a) Compute $\eta$ in each case. (b) Express the added latency of batching if readings arrive every 100 ms. (c) For an emergency system, argue which you would use for routine telemetry and which for alarms.
 
*Solution.* (a) $\eta_1 = 20/78 \approx 25.6\%$; $\eta_{10} = 200/258 \approx 77.5\%$. (b) The first reading waits up to $9\times100 = 900$ ms. (c) Batch routine telemetry, send alarms immediately: efficiency matters for routine data, bounded latency matters for alarms.
 
**Q14. [Medium]** Why does a mesh of $n$ half-duplex wireless hops on one shared channel deliver roughly $R/n$ (for $n\le3$) rather than $R$? How would using two radios on different channels change this, and what new limitation appears?
 
*Answer outline.* Only one transmission at a time in the interference range, so hops take turns. Two radios on different channels allow simultaneous receive/forward, raising throughput, but add cost, power consumption and channel-assignment complexity.
 
---
 
## Self-check before moving on
 
1. Can you state, without notes, which fields change at a router and which do not?
2. Can you write $(N+P-1)L/R$ and explain every term?
3. Can you explain why *each* of DNS, ARP, TCP and IP exists by naming the missing information it supplies?
If yes, you are ready for **Part II: Physical Transmission and Media Limits (Lectures 5-9)**, where we leave the abstraction and ask how a bit becomes a voltage, and why Shannon's formula says there is a ceiling.
 
