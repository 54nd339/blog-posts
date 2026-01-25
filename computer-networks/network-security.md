---
title: Layer by Layer - Unpacking Security Across the Network Stack
description: Explore network security at different layers - IPsec for the Network Layer, SSL/TLS for the Transport Layer, and HTTPS, SSH, PGP, S/MIME for the Application Layer.
date: 2023-06-09
draft: false
slug: /pensieve/computer-networks/network-security
tags:
  - Computer Networks
  - CS Basics
---

Hey cyber-sentinels and digital explorers! We've journeyed through the core concepts of networking, authentication, and cryptography. Now, let's put it all together and see how security is implemented not just as a single shield, but as a series of reinforcing defenses at different layers of the network stack. ️

Just like a medieval castle had outer walls, inner baileys, and finally the keep, our digital communications rely on layered security. If one layer is breached, others stand ready. Today, we'll focus on how security is woven into the Network, Transport, and Application layers.

---
## Wireless Network Security Protocols and Threats
Securing wireless networks is critical due to the broadcast nature of the medium.
**Common Wireless Threats:**
* **Eavesdropping:** Anyone within range can potentially sniff unencrypted traffic.
* **Unauthorized Access:** Attackers gaining access to the network if security is weak.
* **Rogue Access Points:** Malicious APs set up to trick users into connecting.
* **Evil Twin Attacks:** An AP impersonating a legitimate one to intercept traffic.
* **Denial of Service:** Jamming signals or flooding the AP.

**Evolution of Security Protocols:**
* **WEP (Wired Equivalent Privacy):** The original 802.11 security protocol.
    * **Security:** Severely flawed and easily broken due to weaknesses in its use of the RC4 stream cipher, short IVs, and poor key management. **Considered obsolete and insecure.**
* **WPA (Wi-Fi Protected Access):** An interim solution to address WEP's flaws.
    * Introduced **TKIP (Temporal Key Integrity Protocol)**, which still used RC4 but with per-packet key mixing, a message integrity check (MIC called Michael), and dynamic key generation.
    * Stronger than WEP but still had underlying vulnerabilities from RC4.
* **WPA2 (Wi-Fi Protected Access II):** The long-term, more secure replacement.
    * Mandates use of **AES (Advanced Encryption Standard)** with **CCMP (Counter Mode Cipher Block Chaining Message Authentication Code Protocol)**, providing strong encryption and integrity.
    * Much more secure than WEP and WPA. This has been the standard for many years.
* **WPA3 (Wi-Fi Protected Access III):** The latest standard, offering further security enhancements.
    * **Stronger Encryption:** Even in Personal mode, it enforces use of Simultaneous Authentication of Equals (SAE), making it more resistant to offline dictionary attacks against passphrases.
    * **Individualized Data Encryption:** In open networks (like coffee shops), WPA3 enhances privacy by encrypting traffic between each device and the AP individually.
    * **Simplified IoT Onboarding:** Easier ways to connect headless IoT devices securely.
    * Mandates Protected Management Frames (PMF) to protect against deauthentication/disassociation attacks.

For secure Wi-Fi, using WPA3 is recommended if supported, otherwise WPA2 with a strong, unique passphrase is the minimum.

---
## Security at the Network Layer: Fortifying IP with IPsec

The Network Layer, primarily driven by the Internet Protocol (IP), is responsible for routing packets from source to destination. Securing this layer is crucial for protecting data as it traverses potentially untrusted networks. The primary suite of protocols for this is **IPsec (Internet Protocol Security)**.

### IP Security (IPsec) Overview
IPsec is a framework of open standards for ensuring private, secure communications over IP networks, through the use of cryptographic security services. It can provide:
* **Confidentiality:** Encrypting IP datagrams.
* **Integrity:** Ensuring data hasn't been tampered with in transit.
* **Authentication:** Verifying the origin of IP datagrams.
* **Anti-replay protection:** Preventing attackers from capturing and re-injecting legitimate packets.

IPsec can operate in two modes:
* **Transport Mode:** Only the payload (the data part) of the IP packet is encrypted and/or authenticated. The original IP header is kept, so routing is unaffected. Typically used for host-to-host security.
* **Tunnel Mode:** The entire original IP packet (header and payload) is encrypted and/or authenticated and then encapsulated in a new IP packet with a new IP header. Commonly used for creating Virtual Private Networks (VPNs) between gateways (e.g., router-to-router) or from a host to a gateway.

### IP Security Policy (SP)
Before IPsec can protect traffic, a system needs to know *what* traffic to protect and *how* to protect it. This is defined by the **Security Policy Database (SPD)**.
* The SPD contains rules that specify, for any given IP packet, whether it should be discarded, allowed to pass through without IPsec protection, or protected by IPsec.
* If IPsec protection is required, the SPD points to a corresponding **Security Association (SA)**. An SA is a one-way (simplex) agreement between two communicating parties that defines the specific IPsec protocols (AH or ESP), modes (transport or tunnel), cryptographic algorithms, and keys to be used. For two-way communication, two SAs are needed (one for each direction).
* The SPD uses **selectors** (like IP addresses, port numbers, protocol type) to match traffic to policies.

### Authentication Header (AH)
The IPsec Authentication Header (AH) provides connectionless integrity, data origin authentication, and optional anti-replay protection for IP packets.
* **What it does:** AH calculates a cryptographic checksum (an Integrity Check Value, or ICV, typically a keyed hash like HMAC) over most of the IP header fields that are immutable or predictable in transit, plus the entire payload.
* **What it *doesn't* do:** AH **does not provide confidentiality** (no encryption).
* **Placement:** In transport mode, AH is inserted after the original IP header and before the transport layer protocol (e.g., TCP, UDP). In tunnel mode, it authenticates the entire original inner IP packet and the outer IP header.

### Encapsulating Security Payload (ESP)
The IPsec Encapsulating Security Payload (ESP) provides confidentiality (encryption) and can also provide integrity, data origin authentication, and anti-replay protection.
* **Confidentiality:** ESP encrypts the IP payload using symmetric encryption algorithms (e.g., AES).
* **Integrity/Authentication (Optional but Recommended):** ESP can also include an ICV (like AH) to provide integrity and authentication for the encrypted payload and parts of the ESP header.
* **Placement:**
    * In **transport mode**, ESP encrypts the transport-layer segment. The ESP header and trailer are added around the payload. The original IP header remains unencrypted (though it can be authenticated if ESP with authentication is used).
    * In **tunnel mode**, ESP encrypts the entire original IP packet. This encrypted packet becomes the payload for a new, outer IP packet with its own header.
* **ESP vs. AH:** ESP is more commonly used because it can provide encryption. If both AH and ESP are used on the same packet, ESP is typically applied first, then AH authenticates the ESP-protected packet. However, it's more common to use ESP with its own authentication option.

### Internet Key Exchange (IKE)
IPsec needs Security Associations (SAs), and SAs need shared secret keys and agreed-upon cryptographic algorithms. Manually configuring these is impractical for large networks. **Internet Key Exchange (IKE)** is a protocol used to dynamically negotiate SAs and establish shared cryptographic keys for IPsec.
* IKE is a hybrid protocol, typically using Diffie-Hellman key exchange to establish a shared secret, authenticated using pre-shared keys, digital signatures (with certificates), or public key encryption.
* **Phases:**
    * **IKE Phase 1:** Establishes a secure, authenticated channel between two IPsec peers. This results in an IKE SA (also called an ISAKMP SA). The main purpose is to protect subsequent IKE communications.
    * **IKE Phase 2:** Uses the secure channel established in Phase 1 to negotiate IPsec SAs (for AH and/or ESP) that will be used to protect actual application traffic.
* IKE handles SA lifetime management and re-keying. IKEv2 is the current version, simplifying and improving upon IKEv1.

---
## Security at the Transport Layer: Securing End-to-End Communication (SSL/TLS)

While IPsec secures data at the IP packet level, security at the transport layer aims to secure end-to-end communication channels between specific applications on different hosts. The most well-known protocols here are **Secure Sockets Layer (SSL)** and its successor, **Transport Layer Security (TLS)**. TLS is the standard today; SSL is considered insecure and deprecated.

### SSL/TLS Architecture
SSL/TLS operates as a layer between the application layer and the transport layer (typically TCP). It provides a secure channel by:
* **Authenticating** the server (and optionally the client) using public key cryptography and digital certificates.
* **Negotiating** cryptographic algorithms and a shared session key.
* **Encrypting** the application data transmitted over the channel using symmetric encryption with the negotiated session key.
* Ensuring **message integrity** using MACs (Message Authentication Codes).

The SSL/TLS architecture consists of two main protocol layers:
1.  **SSL/TLS Record Protocol:** This protocol sits directly on top of a reliable transport protocol (like TCP). It takes application data, fragments it into manageable blocks, optionally compresses it, applies a MAC, encrypts it using symmetric cryptography, and then prepends an SSL/TLS Record header. It handles the encapsulation and de-capsulation of data.
2.  **SSL/TLS Handshake Protocols (layered on top of the Record Protocol):**
    * **Handshake Protocol:** The most complex part. It allows the server and client to authenticate each other, negotiate encryption and MAC algorithms, and establish shared secret keys (session keys). This involves a series of messages exchanged between client and server.
    * **Change Cipher Spec Protocol:** A single message that signals the transition to the newly negotiated cryptographic parameters.
    * **Alert Protocol:** Used to convey SSL/TLS-related alerts (warnings or fatal errors) to the peer entity.

### SSL/TLS Protocols in Action (Simplified Handshake)
A typical TLS handshake (simplified) involves:
1.  **ClientHello:** Client sends its TLS version, a list of supported cipher suites (combinations of key exchange algorithm, symmetric encryption algorithm, and MAC algorithm), and a client random number.
2.  **ServerHello:** Server chooses a cipher suite from the client's list, sends its TLS version, its digital certificate (containing its public key), and a server random number. It might also request the client's certificate if client authentication is required.
3.  **(Optional) Certificate & ClientKeyExchange:** If server requested client authentication, client sends its certificate and a message to prove possession of its private key. Client generates a pre-master secret, encrypts it with the server's public key (from the server's certificate), and sends it in a `ClientKeyExchange` message.
4.  **(ServerKeyExchange - if needed):** For certain key exchange methods (like Diffie-Hellman), the server sends this message with necessary parameters.
5.  **Client & Server derive master secret:** Both client and server use the pre-master secret and the random numbers to independently compute the same master secret.
6.  **Client & Server derive session keys:** From the master secret, both sides derive the symmetric keys for encryption and MACing for this session.
7.  **Client ChangeCipherSpec & Finished:** Client sends a message indicating it will start using the new keys, followed by an encrypted `Finished` message (containing a hash of previous handshake messages) to verify the handshake.
8.  **Server ChangeCipherSpec & Finished:** Server does the same.
9.  **Secure Communication:** The handshake is complete, and application data can now be exchanged, encrypted and MACed with the session keys.

TLS is the foundation for HTTPS and many other secure protocols.

---
## Security at the Application Layer: Protecting User Data and Interactions

Even with network and transport layer security, applications themselves need to be secure and often implement their own security measures or use secure protocols.

### HTTPS (HTTP Secure)
* HTTPS is simply HTTP layered on top of SSL/TLS.
* It encrypts the HTTP communication (including URLs, headers, cookies, and page content) between the client (browser) and the web server.
* Provides confidentiality, integrity, and server authentication (via the server's SSL/TLS certificate).
* Indicated by `https://` in the URL and often a padlock icon in browsers. Essential for e-commerce, online banking, and protecting user privacy.

### SSH (Secure Shell)
* SSH is a cryptographic network protocol for operating network services securely over an unsecured network.
* Common applications include remote login (replacing insecure Telnet) and secure file transfer (SFTP, SCP).
* **Key Features:**
    * Provides a secure (encrypted and authenticated) channel between client and server.
    * Authenticates the server to the client (typically using the server's host key).
    * Authenticates the client to the server (using methods like passwords, public key authentication, or Kerberos).
    * Can tunnel other application protocols.
* SSH uses its own set of protocols for key exchange, server authentication, user authentication, and secure data transmission.

### Secure E-mail
Standard email (SMTP) transmits messages in plaintext, making it vulnerable to eavesdropping and tampering. Secure email solutions add end-to-end encryption and digital signatures:

* **PGP (Pretty Good Privacy):**
    * A widely used data encryption and decryption computer program that provides cryptographic privacy and authentication.
    * Uses a combination of symmetric-key encryption for message content (for speed) and public-key encryption to encrypt the symmetric session key (forming a digital envelope).
    * Supports digital signatures for message integrity and sender authentication.
    * Often relies on a decentralized **"web of trust"** model for public key verification, where users sign each other's keys to vouch for their authenticity, rather than a formal hierarchical PKI.

* **S/MIME (Secure/Multipurpose Internet Mail Extensions):**
    * An IETF standard for providing cryptographic security services for electronic messaging applications: authentication, message integrity and non-repudiation of origin (using digital signatures) and privacy and data security (using encryption).
    * Relies on a hierarchical **Public Key Infrastructure (PKI)** using X.509 digital certificates issued by Certificate Authorities (CAs) for key management and trust.
    * Integrated into many modern email clients (e.g., Outlook, Apple Mail).

Both PGP and S/MIME aim to ensure that only the intended recipient can read the email (confidentiality) and that the email is genuinely from the sender and unaltered (authentication and integrity).

---
## Conclusion: A Multi-Layered Fortress

Securing our digital interactions isn't about a single silver bullet. It's about building a robust, multi-layered defense strategy that addresses security at each layer of the network stack.
* **IPsec** at the Network Layer can protect all IP traffic, transparently to applications, often used for VPNs.
* **SSL/TLS** at the Transport Layer secures end-to-end application channels, ensuring data exchanged between specific client and server applications is protected.
* **Application-specific security protocols** like HTTPS, SSH, and secure email mechanisms like PGP/S/MIME provide tailored protection for the data and interactions unique to those applications.

By understanding and implementing security measures at these different layers, we can create a much more resilient and trustworthy digital environment.
