# Obfs4


obfs4 is a Tor Project pluggable transport designed to make encrypted traffic look completely random, helping users evade censorship. It is the successor to obfs3 and like obfs3, it disguises Tor traffic, but it also serves as a generic tunnel for other protocols (e.g. SSH or OpenVPN). [^1]

Obfs4 can be used by anyone with basic command-line skills on common platforms (Windows, Linux, macOS, Android), requiring only a pluggable-transport proxy. It **does not** itself provide anonymity or encryption beyond obfuscation - it relies on the underlying protocol (Tor, SSH, VPN, etc.) for security.

## How it works to bypass censorship

obfs4 disguises network traffic as random, unstructured data. It is specifically designed to resist both passive inspection and active probing [^2], making it harder for censors to detect and block. By hiding handshake patterns, encrypting all frames, and optionally obfuscating traffic flow, obfs4 effectively masks protocols like Tor, SSH, or VPNs so they blend into background internet noise. Its design builds on obfs3 but adds critical improvements, especially in preventing active scanning and resisting metadata analysis.

- **Authenticated Handshake (Key Exchange):** obfs4 uses a strong, one-way handshake (the “ntor” key exchange) with all public keys hidden by the Elligator2 [^3] method. In practice, both client and server generate Curve25519 public keys, but those keys are mapped to random-looking strings via Elligator2 so they appear as pure noise. This means a censor’s Deep Packet Inspection (DPI) cannot recognize the key exchange as Tor or any known protocol. For example, Elligator’s mapping allows you to turn random public keys into actual random bytes, which hides metadata. As a result, the obfs4 handshake looks like a harmless stream of random data, not a recognizable cryptographic handshake.
    
- **Encrypted Data Framing:** After the handshake, obfs4 derives a shared secret which is expanded into cipher keys. All subsequent traffic is encrypted and authenticated using NaCl’s secretbox (XSalsa20 stream cipher with Poly1305 MAC). Each data frame is individually encrypted and tagged, so the payload appears random. For instance, the obfs4 spec describes splitting data into frames and encrypting them with `crypto_secretbox_xsalsa20poly1305`. Because of this strong encryption, an observer cannot decipher contents or identify specific protocol markers. The protocol also XOR-obfuscates each frame’s length using a SipHash-derived mask, further hiding packet sizes.
    
- **Traffic Polymorphism:** obfs4 inherits and extends the ScrambleSuit idea of protocol polymorphism. It can insert random padding and shape traffic bursts to mask timing and volume patterns. By default it obfuscates burst sizes and (optionally, at some performance cost) can randomize packet lengths. In practical terms, obfs4 can vary how much data is sent in each chunk and even add delays, so that Tor or VPN usage fingerprints (fixed-size cells, byte patterns) are hidden. This ScrambleSuit-like flow obfuscation makes it much harder for statistical traffic analysis to detect Tor or VPN behavior.
    
- **Versatile Wrapping of Traffic:** Although designed for Tor bridges, obfs4 is a generic TCP wrapper. The same obfs4proxy software can tunnel any TCP-based protocol. This means you can run obfs4proxy to wrap SSH or OpenVPN connections: your application speaks to a local obfs4 proxy instead of directly to the Internet. The proxy then performs the obfs4 handshake with the remote end and relays encrypted data, making the entire connection look like random noise.
    

## Privacy and Security Measures

    
Obfs4 only obfuscates traffic; it does not provide full encryption, authentication, or integrity protection for the underlying data. Its job is to disguise your connection so it looks like random noise, not to secure the content itself. For that reason, you must always use obfs4 together with a protocol that provides strong security features, such as Tor, SSH, or a VPN (e.g., OpenVPN or WireGuard). This ensures that even if someone detects or interferes with your obfs4 tunnel, the data inside remains encrypted and protected against tampering.

To maintain your privacy and security when using obfs4:

- **Pair it with a secure protocol** like Tor, SSH, or OpenVPN for end-to-end encryption and authentication.
    
- **Keep obfs4proxy up to date** to patch vulnerabilities that could be exploited to detect or block your connection.
    
- **Protect your bridge information** (especially the `cert` parameter) to prevent censors from probing and blocking your bridge.
    
- **Ensure all traffic flows through the obfs4 tunnel** by configuring applications and firewall rules to block any direct connections.
    
- **Use traffic shaping options** if needed in high-risk environments to make traffic analysis harder.
    
- **Replace bridges** if they become blocked or compromised.

## Implementations

The official obfs4 implementation is **obfs4proxy**, a Go-based pluggable-transport proxy by the Tor Project.

| Implementation | Description                                  | Windows | Linux | Android | iOS  | macOS |
| -------------- | -------------------------------------------- | :-----: | :---: | :-----: | :--: | :---: |
| **obfs4proxy** | Official Tor pluggable transport proxy (Go)  |  ✅ Yes  | ✅ Yes |  ✅ Yes  | ❌ No | ✅ Yes |

## How to Use and Set Up

Obfs4proxy can be used as a standalone wrapper for services like SSH or OpenVPN. The general approach is to run one proxy on the server side forwarding to the local service, and another on the client side forwarding your local traffic to the remote obfs4 endpoint. 

Below is an SSH example:

### Server Setup

- First install obfs4proxy on the server (e.g. `sudo apt install obfs4proxy` on Debian/Ubuntu). Create a dedicated state directory (e.g. `/var/lib/tor/pt_state/obfs4/`). 

- Then start obfs4proxy in server mode with environment variables specifying the transport.

- For example, to listen on port 2222 and forward to a local SSH daemon (port 22), you might run something like:

```bash 
TOR_PT_MANAGED_TRANSPORT_VER="1"  
TOR_PT_STATE_LOCATION="/var/lib/tor/pt_state/obfs4"  
TOR_PT_SERVER_TRANSPORTS="obfs4"  
TOR_PT_SERVER_BINDADDR="obfs4-0.0.0.0:2222"  
TOR_PT_ORPORT="127.0.0.1:22"  
obfs4proxy -enableLogging -logLevel INFO
```


This tells obfs4proxy to bind (0.0.0.0:2222) for obfs4 connections and forward them to SSH on (127.0.0.1:22). 

The proxy will output a generated “cert=” token (its bridge fingerprint) which you will need to use on the client. Make sure firewall allows incoming traffic on the chosen port.

###  Client Setup
- On your local machine, also install obfs4proxy. Then run it in client mode, pointing to the server’s IP and obfs4 port. For example:

```bash
TOR_PT_MANAGED_TRANSPORT_VER="1" \
TOR_PT_STATE_LOCATION="/var/lib/tor/pt_state/obfs4" \
TOR_PT_CLIENT_TRANSPORTS="obfs4" \
obfs4proxy -enableLogging -logLevel INFO
````

- The above will launch a client obfs4proxy that connects to the server’s obfs4 endpoint. It will open a local SOCKS5 port (printed as “CMETHOD obfs4 socks5 127.0.0.1:xxxxx” in the output) listening for application traffic. 

- Finally, configure your SSH client (or other application) to use this proxy. For instance, you could use `ssh -o 'ProxyCommand ncat --proxy 127.0.0.1:<socks-port> --proxy-type socks5' user@destination` (using `ncat` from the nmap package) to route SSH through the obfs4 proxy.
 
- In simpler cases, you could point SSH directly at `127.0.0.1:<socks-port>` if it supports it. The client’s traffic will then be wrapped by obfs4 and sent to the server’s obfs4proxy.

The steps above can similarly wrap OpenVPN, HTTPS, or any TCP traffic: just run obfs4proxy on both ends and route the protocol through the local proxy port. 

---

[^1]: Official obfs4 protocol specification: https://gitlab.torproject.org/tpo/anti-censorship/pluggable-transports/obfs4/-/blob/main/doc/obfs4-spec.txt

[^2]: Elligator technical details: https://elligator.cr.yp.to/

[^3]: Overview of active probing: https://blog.torproject.org/learning-more-about-gfws-active-probing-system/
