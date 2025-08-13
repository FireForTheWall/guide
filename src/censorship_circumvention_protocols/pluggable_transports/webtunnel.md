# WebTunnel

WebTunnel is a pluggable transport protocol developed by the [Tor](/guide/censorship_resistant_networks/tor_network) Project to help bypass censorship in countries like China, Iran, Russia, and others.

WebTunnel works by routing your Tor traffic through standard HTTPS or WebSocket connections, making it look like ordinary web traffic rather than Tor traffic. This allows Tor users to bypass deep packet inspection (DPI) and simple IP blocking without requiring specialized proxy software on the client side.
## How it works to bypass censorship

WebTunnel’s design is centered around making Tor traffic appear as standard HTTPS or WebSocket connections. [^1] This is done using a client-server tunneling architecture with the following steps:

- **Bootstrapping:** The Tor client selects WebTunnel as a bridge. The client then initiates a standard HTTPS/WebSocket connection to a WebTunnel server. The server address can be embedded in the client or obtained via a bridge broker.
    
- **Connection Wrapping:** Tor traffic is encapsulated inside WebSocket frames or HTTPS requests/responses. This makes the traffic indistinguishable from normal web browsing. For example, each Tor cell is wrapped in an HTTP POST body or WebSocket message, then unwrapped by the server and forwarded into the Tor network.
    
- **Traffic Encryption:** While the outer WebSocket or HTTPS layer provides confidentiality and integrity (TLS encryption), Tor handles end-to-end encryption of actual user data. This ensures that even if the WebTunnel server is untrusted, the Tor traffic content remains confidential.
    
- **Server Relay:** The WebTunnel server forwards the unwrapped Tor traffic into the Tor network. To a censor, all that is visible is a normal TLS/WebSocket session to a web server. Any attempt to block these connections would require disrupting widely used web infrastructure, making censorship more costly.

By using ubiquitous web protocols, WebTunnel takes advantage of “collateral freedom”, blocking WebTunnel means blocking normal web traffic. Also TLS protects the transport from eavesdropping and tampering.

WebTunnel servers can be run on ordinary web hosts, including cloud services. This reduces the need for static IP bridges, which are easily blocked.

## Privacy and Security Measures

WebTunnel encrypts traffic over TLS, ensuring confidentiality and integrity between client and server. However, this protects only the tunnel to the WebTunnel server, not the Tor network itself since Tor provides the end-to-end encryption.

WebTunnel disguises Tor traffic as normal HTTPS or WebSocket traffic, hiding Tor fingerprints from DPI. The TLS/WebSocket layer does **not** authenticate the client beyond standard TLS certificates. Malicious servers could see metadata (e.g., timing, volume), but Tor’s end-to-end encryption prevents them from accessing user data.

Users implicitly trust WebTunnel servers to relay traffic correctly. Tor’s layered encryption prevents content leakage, but metadata such as connection timing and size may still be observable by the server.

## Implementations
| Implementation | Description                                              | Windows | Linux | Android | iOS  | macOS |
| -------------- | -------------------------------------------------------- | ------- | ----- | ------- | ---- | ----- |
| webtunnel      | Go-based pluggable transport wrapping Tor over WebSocket | ✅ Yes   | ✅ Yes | ✅ Yes   | ❌ No | ✅ Yes |
## How to Use and Set Up

* **Clients:** Tor Browser and Orbot can use WebTunnel as a bridge. In the Tor Browser, go to **Settings → Tor → Bridges**, enable bridges, and select "webtunnel" as the transport.

* **Servers/Relays:** WebTunnel servers can be run on any host supporting Go. For example, on Debian/Ubuntu:

```bash
sudo apt install golang

git clone https://gitlab.torproject.org/tpo/anti-censorship/pluggable-transports/webtunnel.git

cd webtunnel/server

go build

nohup ./server &
```

WebTunnel itself is designed primarily to tunnel Tor traffic.

---

[^1]: WebTunnel protocol and GitLab repository: https://gitlab.torproject.org/tpo/anti-censorship/pluggable-transports/webtunnel
