# DPI (Deep Packet Inspection)
 
**DPI (Deep Packet Inspection)** is a network traffic analysis technique that goes beyond simple packet headers. Instead of just reading metadata like source/destination IP addresses or port numbers, DPI inspects the _payload_ (the actual data inside the packet).

DPI can:

- Identify the application or protocol in use (e.g., HTTPS, SSH, OpenVPN, WireGuard, etc.).
    
- Detect specific content or keywords inside communications.
    
- Monitor, filter, or block traffic based on patterns even in fully encrypted traffic.
    

DPI requires significant computational resources and sophisticated algorithms, meaning it is typically employed by large ISPs or government entities. Making it a tool of state or corporate control, often deployed with little transparency or accountability.

## How Does It Enable Censorship?

DPI can detect protocols like VPNs, Tor, or encrypted messaging apps and selectively block or throttle them to prevent circumvention.

DPI can interfere with connections by injecting TCP reset packets or slowing down traffic to degrade service quality for certain content.

Rather than outright blocking, DPI can prioritize or degrade certain content, enabling subtle censorship that’s harder to detect.

Unlike simple IP or DNS blocking, DPI examines content inside the packet, allowing censorship of specific messages or content even within allowed domains. And because DPI inspects packet contents, simple workarounds like changing IP addresses or domains don’t work well. It can also detect encrypted VPN traffic by patterns or protocol signatures.

DPI inherently enables mass surveillance, since inspecting payloads means potentially capturing private communications.

Increasing use of HTTPS and other encryption protocols limits DPI's effectiveness. But even encrypted traffic has characteristic patterns (packet sizes, timing, sequence of packet lengths, handshake patterns). Also, DPI systems identify what protocol or application is being used (e.g., VPN type, Tor, HTTPS) by these signatures. For example, OpenVPN and WireGuard have distinctive packet sizes and timing patterns, making them easy for DPI to throttle or outright prevent handshakes and connection establishment using these protocols.

Also, TLS handshakes send some unencrypted information (like the list of cipher suites, extensions). DPI can use these to guess which protocols or websites might be involved.

On top of that, DPIs can perform SNI (Server Name Indication) inspection. In the TLS handshake, the SNI field reveals the hostname the client wants to connect to; this part is usually unencrypted. DPI systems can easily block access based on this hostname.

**Note:** TLS 1.3 with Encrypted SNI (ESNI), now called Encrypted Client Hello (ECH), aims to prevent this. But DPI systems can prevent ECH connections if they're desperate.

## How to Circumvent It?

DPI systems rely on **analyzing packets, headers, payloads, or traffic metadata** to:

- Detect certain keywords, signatures, or protocols.
    
- Identify applications or user behaviors.
    
- Enforce blocking, throttling, or modification.
    

Circumvention means preventing DPI from accurately detecting or classifying your traffic or content.

### Expensive Protocols

Expensive protocols are those protocols whose blocking entirely breaks almost all modern web services.

These are protocols like TLS, HTTPS, QUIC, WebSocket, SSH, etc. Firewalls usually avoid full blocking and instead use targeted DPI or traffic analysis to identify specific forbidden content inside these protocols.

Using these protocols can and will help bypass firewalls unless they're willing to break everything for the sake of preventing the flow of information.

### Protocol Obfuscation

Use of obfuscation protocols like obfs3, obfs4, etc., can help make the underlying traffic look like random noise and prevent attacks through timings and active probing (in the case of obfs4).

### Camouflage Protocols

Camouflage protocols are protocols that make your traffic look like another protocol or look like legitimate traffic. For example, Snowflake makes your traffic look like a normal WebRTC meeting call; Trojan protocol can make your traffic look like normal HTTPS website traffic; or VLESS protocol can make your traffic look like normal WebSocket/QUIC/gRPC connections.
