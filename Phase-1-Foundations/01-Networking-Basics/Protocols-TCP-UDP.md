Protocols TCP UDP
======
## TCP (Transmission Control Protocol)
TCP is the **reliable, polite delivery guy** of the internet.

When data travels across the internet, it's chopped into **packets** (small chunks). TCP ensures:
- Every packet arrives in **order**.
- Missing packets are **re-sent**.
- The sender doesn't overwhelm the receiver.

### Analogy
Imagine you send a 5-page letter via registered mail:
- TCP makes sure the receiver gets **all 5 pages, in the rigth order**, even if some got lost along the way.
- You get a **confirmation** when it arrives.
- If a page is missing, the postal service **re-sends it**.

### TCP Three-Way Handshake (Connection Setup)
![TCP Three-Way Handshake](./images/three-way-handshake.excalidraw.png)
#### Steps:
1. **SYN** (Synchronize): Client initiates connection.
2. **SYN-ACK** (Synchronize-Acknowledge): Server ackonwledges and agrees.
3. **ACK** (Acknowledge): Client confirms, connection established.

### Real-World Examples
- **Web browsing (HTTP/HTTPS)**: Every website you visit uses TCP.
- **Email (SMTP)**: Sending emails requires reliable delivery.
- **File Transfer (FTP/SFTP)**: Downloading files needs all packets.
- **SSH**: Remote server access can't afford lost commands.

### Trade-offs
|Feature|Benefit|Cost|
|:------|:------|:------|
|Reliable delivery|No data loss|Slower (retransmissions)|
|Ordered packets|Data arrives in sequence|More overhead|
|Connection-oriented|Guaranteed delivery|3-way handshake latency|
|Flow control|Prevents overwhelm|Reduced throughput|

#### notes:
- **More overhead**: TCP uses extra resources (like memory, CPU, and control data) to manage packet order, retransmissions, and acknowledgements. Making it slower but more reliable.

- **Prevents overwhelm**: TCP uses flow control and congestion control. **Flow control** prevents the sender from overloading the receiver, while **congestion control** slows data when the network shows signs of congesion.

- **Reduced throughput**: data is still delivered correctly, but more slowly. Less data per second reaches the destination.

### When NOT to use TCP:
- Real-time gaming (lag from retransmission).
- Live video streaming (old frames not useful).
- DNS queries (single packet, no need for connection).