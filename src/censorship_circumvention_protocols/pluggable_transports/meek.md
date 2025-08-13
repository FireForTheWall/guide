# Meek

Meek is a pluggable transport developed by the Tor Project to help users bypass internet censorship in restrictive environments such as China, Iran, and Russia. It builds on earlier obfuscation techniques like obfs3 and obfs4 but introduces domain fronting [^1], making Tor traffic appear as connections to popular web services like Microsoft Azure or Amazon CloudFront. While meek is still supported and integrated into Tor, its effectiveness has diminished in some regions due to major providers like Google and Amazon disabling domain fronting in 2018. However, meek-azure continue to function reliably on Microsoft Azure.

Meek is designed to make Tor traffic indistinguishable from ordinary HTTPS requests to high-profile domains, Using content delivery networks (CDNs) to route data without revealing the true destination. Although primarily built for Tor, meek can be used as a standalone transport to obfuscate other protocols like SSH or VPN traffic, providing an additional layer of camouflage against deep packet inspection (DPI) and blocking.

## How it works to bypass censorship

Meek is a blocking-resistant pluggable transport that tunnels Tor traffic through HTTPS using domain fronting, a technique that hides the true destination of the connection by using different domain names at various protocol layers. This makes the traffic appear as if it's directed to a permitted web service (e.g., a Microsoft or Amazon domain) rather than a Tor bridge. [^2]

### Domain Fronting Phase

In domain fronting, the client specifies an "allowed" front domain (e.g., ajax.aspnetcdn.com for Azure) in the visible parts of the connection, such as the DNS query and the TLS Server Name Indication (SNI) extension. However, the actual forbidden destination (the Tor bridge's domain) is placed in the HTTP Host header, which is encrypted under HTTPS and only readable after the TLS handshake by the intermediate CDN. The censor sees a connection to a popular domain. The CDN decrypts the request, reads the Host header, and forwards it to the hidden Tor bridge.

### Data Tunneling Phase

Once the connection is established, meek encodes the data stream (e.g., Tor cells) as a sequence of HTTPS requests and responses. The client sends data in HTTP POST request bodies, and the server responds with data in response bodies. To handle bidirectional communication, the client periodically sends polling requests to fetch any pending server-to-client data.

However, meek has limitations. It does not alter packet timings or sizes significantly, leaving it potentially vulnerable to advanced traffic analysis. Also, its reliance on domain fronting means it's only effective where CDNs still support it, and it can be slower due to the overhead of HTTPS polling and intermediary routing.
## Privacy and Security Measures

Because meek relies on HTTPS encryption, censors cannot easily inspect packet contents, but they may detect patterns through metadata analysis, such as consistent polling intervals or chunk sizes that differ from typical web traffic. However, this does not allow decryption of the underlying data, which remains protected by the transported protocol (e.g., Tor's encryption).

Meek is susceptible to TLS fingerprinting, where censors identify non-browser-like TLS handshakes. To mitigate this, implementations often use browser extensions (e.g., for Chrome or Firefox) to camouflage the TLS signature as a standard web browser.

Active probing attacks are resisted because probing the front domain would require interacting with a major CDN, but if the censor controls a trusted certificate authority, they could perform man-in-the-middle (MitM) [^3] attacks to inspect Host headers - though certificate pinning in modern clients helps defend against this.

Meek itself does not provide authentication or integrity for the transported data; it depends on the underlying protocol, such as SSH or OpenVPN, to handle encryption and MitM resistance.

As meek is still maintained, newly discovered vulnerabilities are addressed, but its dependency on third-party CDNs introduces risks if providers like Microsoft disable domain fronting.

Always use meek with protocols that offer strong authentication and integrity, like SSH or OpenVPN with TLS. This ensures security even if meek's transport layer is compromised.

The most effective privacy mitigation is combining meek with other transports if needed, or migrating to newer alternatives like WebTunnel (which also supports domain fronting) for better resistance to traffic analysis and probing.

## Implementations

Meek's primary implementation is in Go which makes it portable across platforms. It is integrated into Tor Browser and available as standalone tools via the official repository.  

| Implementation | Description                                                                  | Windows | Linux | Android                 | iOS                       | macOS |
| -------------- | ---------------------------------------------------------------------------- | ------- | ----- | ----------------------- | ------------------------- | ----- |
| meek           | Go-based pluggable transport for domain fronting (maintained by Tor Project) | ✅ Yes   | ✅ Yes | ✅ Yes (via Tor Browser) | ✅ Yes (via Onion Browser) | ✅ Yes |
| meek-lite      | Lighter variant for resource-constrained environments (e.g., in Whonix)      | ✅ Yes   | ✅ Yes | ✅ Yes                   | ❌ No                      | ✅ Yes |

## How to Use and Set Up

### Using meek as a Standalone Wrapper for SSH

This setup obfuscates SSH traffic using meek's domain fronting, disguising it as HTTPS to a major web service. Meek does not encrypt the data itself, it only hides the protocol patterns and security relies on SSH's encryption and authentication.

---

### Server Setup

1. **Install meek-server**

Clone the repository and build (requires Go):

```bash

git clone https://gitlab.torproject.org/tpo/anti-censorship/pluggable-transports/meek.git

cd meek/meek-server

go build

```

2. **Run meek-server**

Start it to listen for fronted connections and forward to local SSH:

```bash

./meek-server --port=443 --disable-tls

```

You also need a reflector on a CDN (e.g., Azure). Deploy a PHP or App Engine reflector as per the repo docs, pointing to your server's domain.

---

### Client Setup

1. **Install meek-client** on your local machine (build similarly from repo).

2. **Run meek-client in client mode**

This sets up a local SOCKS proxy that fronts through a CDN to the server:

```bash

./meek-client --url=https://your-reflector-url/ --front=ajax.aspnetcdn.com --log=meek.log

```

Replace with your reflector URL (e.g., for meek-azure) and front domain.

3. **Connect your SSH client through the meek proxy**

Use SSH with the SOCKS proxy (meek-client listens on 1080 by default):

```bash

ssh -o ProxyCommand="nc -x 127.0.0.1:1080 %h %p" username@server_ip -D 2080

```

This tunnels SSH through meek, opening a SOCKS5 proxy on `127.0.0.1:2080` for applications to route traffic and bypass censorship.

The same principles apply to protocols like OpenVPN: configure OpenVPN to use the meek-provided SOCKS proxy.

---

**Important Note:** Meek only obfuscates via domain fronting; pair it with encrypted protocols and verify CDN availability, as some (e.g., Google) no longer support it.

---

**References:**

[^1]: Further Information: [https://en.wikipedia.org/wiki/Domain\_fronting](https://en.wikipedia.org/wiki/Domain_fronting)

[^2]: meek's protocol details: [https://www.icir.org/vern/papers/meek-PETS-2015.pdf](https://www.icir.org/vern/papers/meek-PETS-2015.pdf)

[^3]: Further Information: [https://en.wikipedia.org/wiki/Man-in-the-middle\_attack](https://en.wikipedia.org/wiki/Man-in-the-middle_attack)
