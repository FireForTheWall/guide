obfs3 is the precursor to the obfs4 protocol, both developed by the Tor Project to help lift restrictions placed on the Tor Network by countries such as China, Iran, Russia, and more.

Obfs3 is a pluggable transport meant to make traffic look random and not like any other protocol. Although Obfs3 is no longer supported, and its successor, obfs4, does a much better job, it can be used to obfuscate other traffic such as SSH traffic or VPN traffic, and it is still highly effective at bypassing censorship as a standalone obfuscation protocol.

## How it works to bypass censorship

obfs3 is a pluggable transport designed to help Tor users bypass internet censorship by disguising Tor traffic as random data, making it harder for censorship systems to detect and block. It achieves this through two main phases:

- Key Exchange Phase: obfs3 employs a custom version of the Diffie-Hellman key exchange [^1] called UniformDH. Unlike traditional Diffie-Hellman, where public keys have a recognizable structure, UniformDH generates public keys that appear as random strings. This randomness prevents censors using Deep Packet Inspection (DPI) from identifying the handshake as part of a Tor connection.
    
- Data Obfuscation Phase [^2]: After the client and server establish a shared secret key through UniformDH, obfs3 uses this key to encrypt the Tor traffic. This process, often referred to as "superencipherment," transforms the Tor traffic, which typically relies on TLS and has identifiable patterns, into what looks like random data. By blending in with other internet traffic, it becomes harder for censorship firewalls to pinpoint and block it.
    

But the traffic does not have to be Tor traffic; the obfs3 protocol can be used as a standalone pluggable transport to do the same thing for protocols like OpenVPN, SSH, and more. And although it is not supported by Tor anymore, it is still a very good option for obfuscating VPN protocol traffic.

However, obfs3 has limitations. It does not modify packet sizes or timing (unlike obfs4), which means advanced censorship systems can still analyze these metadata patterns to detect Tor usage. In addition, obfs3 is vulnerable to active probing attacks, where censors actively attempt to connect to suspected Tor bridges and confirm their identity by completing the obfs3 handshake.

## Privacy and Security Measures

Because obfs3 does not alter packet sizes or timing, advanced censors can analyze these metadata patterns. However, this does not mean they can open the packets or see the contents inside.

obfs3 is susceptible to active probing, where a server owned by the censorship firewall tests a suspected Tor bridge by attempting to connect to it. If the connection handshake succeeds in a way consistent with obfs3, the firewall can confirm the bridge’s identity and block it.

obfs3 itself does not ensure the integrity or security of the data it transports and depends entirely on the underlying protocol, such as OpenVPN.

obfs3 is no longer supported or updated, meaning any newly discovered vulnerabilities will remain unpatched. This increases the risk of exploitation by attackers, putting users’ security at greater risk over time.

Always use obfs3 alongside a protocol that provides authentication and data integrity, such as OpenVPN and SSH. This ensures that even though obfs3 lacks these features, the underlying connection remains secure against MitM attacks [^3] and data tampering.

The most effective privacy mitigation is to stop using obfs3 and upgrade to obfs4, its successor. obfs4 is designed to resist both flow analysis and active probing attacks.

## Implementations

**obfsproxy** is effectively the only complete and usable standalone obfs3 implementation. Any other references to obfs3 are typically either embedded in legacy versions of Tor software or exist as unmaintained forks or stubs.

|**Implementation**|**Description**|**Windows**|**Linux**|**Android**|**iOS**|**macOS**|
|---|---|---|---|---|---|---|
|**obfsproxy**|Python-based tool implementing obfs3 (deprecated)|✅ Yes|✅ Yes|❌ No|❌ No|✅ Yes|

## How to Use and Set Up

### Using obfs3 as a Standalone Wrapper for SSH

This setup allows you to obfuscate your SSH traffic using obfs3, which helps hide the fact that you’re using SSH from censorship or deep packet inspection systems. obfs3 does not replace SSH’s encryption; it only disguises the traffic patterns so they do not resemble SSH.

### Server Setup

1. **Install `obfsproxy`**    

If you don’t have it installed yet, you can usually get it via pip:

```bash
pip install obfsproxy
```

2. **Run obfsproxy in server mode**

Use this command to start obfsproxy listening for obfs3 connections and forwarding them to the local SSH server:

```bash
obfsproxy obfs3 --dest=127.0.0.1:22 server 0.0.0.0:<obfs3_port>
```

Replace `<obfs3_port>` with the port you want obfs3 to listen on (for example, `9999`). Make sure this port is open in your firewall.

### Client Setup

1. **Install `obfsproxy`** on your local machine.
    
2. **Run obfsproxy in client mode**

This command sets up a local obfs3 proxy that connects to the server’s obfs3 port and forwards traffic to a local port on your machine:

```bash
obfsproxy obfs3 --dest=<server_ip>:<obfs3_port> client 127.0.0.1:<local_port>
```

Replace `<server_ip>` with your server’s IP address, `<obfs3_port>` with the port the server is listening on, and `<local_port>` with a free port on your local machine (for example, `5000`).

3. **Connect your SSH client through the local obfsproxy**

Use your SSH client to connect to the local port:

```bash
ssh -p <local_port> username@127.0.0.1 -D 2080
```

This connection is tunneled through obfs3 and opens a SOCKS5 proxy on `127.0.0.1:2080` which you can set on different applications to bypass censorship and mask the traffic from simple protocol detection.

The same principles and steps can be applied for protocols like OpenVPN as well.

**Important Note:** obfs3 only obfuscates the traffic; use it with protocols that provide strong encryption and proper authentication.

---

[1^]: Further Information: [https://en.wikipedia.org/wiki/Diffie%E2%80%93Hellman_key_exchange](https://en.wikipedia.org/wiki/Diffie%E2%80%93Hellman_key_exchange)
[2^]: obfs3's protocol specs: [https://github.com/isislovecruft/obfsproxy/blob/master/doc/obfs3/obfs3-protocol-spec.txt](https://github.com/isislovecruft/obfsproxy/blob/master/doc/obfs3/obfs3-protocol-spec.txt) 
[3^]: Further Information: [https://en.wikipedia.org/wiki/Man-in-the-middle_attack](https://en.wikipedia.org/wiki/Man-in-the-middle_attack)
