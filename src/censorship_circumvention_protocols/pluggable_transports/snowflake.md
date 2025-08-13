# Snowflake

Snowflake is a pluggable transport protocol developed by the [Tor](/guide/censorship_resistant_networks/tor_network) Project to help bypass censorship imposed on Tor in heavily restricted countries like China, Iran, Russia, and more.

Snowflake works by routing your traffic through volunteer-operated WebRTC proxies to make it look like you are on a video call instead of using Tor.

## How it works to bypass censorship

Snowflake hides Tor traffic inside encrypted WebRTC connections, making it appear like ordinary real-time web communication such as video or voice calls. It achieves this by using volunteer-run Snowflake proxies and a central broker, combined with domain fronting and NAT traversal techniques. [^1]

The process follows these steps:

- **Bootstrapping:**  
    The Tor client (such as Tor Browser or Orbot) is configured to use Snowflake as a bridge. It first contacts the Snowflake broker using **domain-fronted HTTPS** - the request appears to target a large, unblockable CDN domain (e.g., `google.com`), while the actual broker hostname is hidden inside the encrypted TLS headers.
    
- **Proxy Discovery:**  
    The broker assigns the client an available volunteer Snowflake proxy and sends the proxy’s WebRTC “offer” back to the client. Because the broker communication is domain-fronted, a censor only sees an HTTPS request to a major domain, not the broker’s actual address.
    
- **WebRTC Connection Setup:**  
    The client and proxy exchange ICE candidates (through the broker) to find a direct communication path, even if both are behind NAT or firewalls. This uses public **STUN** servers, and **TURN** servers if direct UDP fails. Once a path is found, a **WebRTC DataChannel** is established.
    
- **Traffic Relay:**  
    The volunteer proxy forwards the encrypted traffic into the Tor network. To outside observers, this appears as generic DTLS-encrypted WebRTC traffic, indistinguishable from online conferencing or real-time multiplayer games.
    
- **Ephemeral Proxies:**  
    Because Snowflake proxies are short-lived and run inside volunteers’ browsers or standalone apps, their IP addresses change constantly. This makes IP-based blocking nearly impossible without over-blocking huge swaths of the Internet.

Snowflake’s design uses both the ubiquity of major CDN domains (for broker communication) and the difficulty of distinguishing WebRTC flows from normal interactive traffic, making it extremely resistant to censorship.
## Privacy and Security Measures


Snowflake tunnels traffic over WebRTC DataChannels, which use DTLS and SRTP for encryption. This ensures that the traffic between the client and the proxy remains confidential and integrity-protected. However, this encryption only covers the connection to the proxy; Tor itself provides end-to-end encryption within the network.

Snowflake disguises Tor traffic as normal WebRTC and HTTPS requests. This makes the traffic look like a generic video call and hides Tor fingerprints from DPI, but the obfuscation layer does **not** authenticate the client or proxy beyond the WebRTC handshake. As a result, malicious proxies could potentially observe traffic metadata if Tor’s end-to-end encryption is not used.

Because Snowflake proxies are operated by volunteers, users implicitly trust that proxies will forward traffic correctly. Tor’s layered encryption protects the actual user data, preventing proxies from seeing decrypted Tor traffic, though they can still infer metadata such as connection timing and volume.
## Implementations

Snowflake is available on all major operating systems. Both volunteers and censored users can run Snowflake in various forms:

|Implementation|Description|Windows|Linux|Android|iOS|macOS|
|---|---|---|---|---|---|---|
|Snowflake (Browser)|Volunteer-run WebRTC proxy as a browser extension or embedded in a webpage|✅ Yes|✅ Yes|❌ No|❌ No|✅ Yes|
|Snowflake (Standalone)|Go-based standalone Snowflake proxy/client (maintained by Tor Project) for persistent operation|✅ Yes|✅ Yes|✅ Yes (via Orbot)|✅ Yes (via Onion Browser)|✅ Yes|
|Snowflake (Tor Bridge)|Integrated into Tor Browser and Orbot for end users in censored regions|✅ Yes|✅ Yes|✅ Yes|✅ Yes|✅ Yes|
## How to Use and Set Up

Snowflake clients (the Tor user side) can run anywhere Tor Browser or Orbot runs. Volunteers have two main options:

* **Browser Extension (Proxy):** Desktop users (Windows/Linux/macOS) can install the Snowflake add-on in Chrome, Firefox, or Edge. Once enabled, keeping the browser tab open turns it into an ephemeral proxy that serves blocked users.

* **Standalone Snowflake Proxy:** For higher bandwidth, you can run the Snowflake proxy application outside the browser. This Go program is provided by the Tor Project. Official guides show how to deploy it on servers (e.g., via Docker or building from source). On Debian/Ubuntu, for example:

```bash
sudo apt install golang

git clone https://gitlab.torproject.org/tpo/anti-censorship/pluggable-transports/snowflake.git

cd snowflake/proxy

go build

nohup ./proxy &
```

(This sets up a Snowflake proxy listening on all UDP ports. The Tor Project’s documentation and scripts handle any additional configuration.)

* **Orbot:** On Android (and iOS), Orbot integrates Snowflake as a bridge. Users can enable Snowflake in Orbot’s settings and route apps through Tor. Orbot essentially provides a VPN interface that can include Snowflake as a transport.

Clients and proxies are cross-compatible: any Snowflake client can connect to any Snowflake proxy regardless of OS, since the handshake uses WebRTC and standard HTTPS. The only requirement is that the system supports WebRTC/STUN.

**Using Snowflake with Tor (end users):**

The easiest method is via the Tor Browser or Orbot UI. For Tor Browser (desktop/mobile), go to **Settings → Tor → Bridges**, enable bridges, and select “snowflake” as the transport. Connect normally to Tor – if the network is blocked, Tor will automatically use Snowflake. No bridge address is needed, as the Snowflake broker’s URL is built into the client.

For Tor daemon or non-GUI setups, add the following to your `torrc`:

```

UseBridges 1

ClientTransportPlugin snowflake exec /path/to/snowflake-client \

-url https://snowflake-broker.torproject.net.global.prod.fastly.net/ \

-front cdn.sstatic.net \

-ice stun:stun.voip.blackberry.com:3478,stun:stun.antisip.com:3478,...

Bridge snowflake 192.0.2.3:1

```

(Replace `/path/to/snowflake-client` with the actual binary path.) The placeholder `192.0.2.3:1` is required syntactically but is not a real proxy address. After restarting Tor, it will bootstrap via Snowflake.

**Other Protocols:**

Snowflake is designed as a **Tor-only** transport. It provides a SOCKS interface into Tor, not a generic VPN for arbitrary services. You cannot directly use Snowflake as an SSH or OpenVPN proxy. If needed, you would first connect to Tor via Snowflake and then route other applications through Tor (e.g., using `ssh -o ProxyCommand` or Orbot’s VPN mode). In practice, Snowflake is intended solely to reach Tor.

---

[^1]: Snowflake protocol specification and overview: [https://gitlab.torproject.org/tpo/anti-censorship/pluggable-transports/snowflake](https://gitlab.torproject.org/tpo/anti-censorship/pluggable-transports/snowflake)
