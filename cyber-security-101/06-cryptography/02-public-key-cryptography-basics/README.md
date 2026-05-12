# Public Key Cryptography Basics

| Field | Details |
|---|---|
| **Module** | Cyber Security 101 — Cryptography |
| **Difficulty** | Easy |
| **Platform** | TryHackMe |
| **Room Link** | https://tryhackme.com/room/publickeycrypto |
| **Date Completed** | February 2026 |
| **Author** | Adwait Joshi |

---

## What This Room Covers

This is the second room in the CS101 Cryptography section. Cryptography Basics introduced symmetric encryption, XOR, and modulo — the tools for keeping data confidential. This room addresses the harder problem: how do two parties establish a shared secret over a channel neither trusts, verify each other's identities, and guarantee the authenticity of messages — all without ever having met before? The answer is public key cryptography. The room covers RSA as the foundational asymmetric algorithm, Diffie-Hellman key exchange, SSH key authentication, digital signatures and certificates, and OpenPGP/GPG for encrypted messaging. All of these are not just theoretical constructs — they are the active mechanisms behind HTTPS, SSH, TLS, email signing, and software verification.

---

## Key Concepts

### Why Asymmetric Cryptography Exists

Symmetric encryption has a fundamental deployment problem: both parties must possess the same secret key before they can communicate privately. Transmitting that key over an untrusted network exposes it. Meeting in person to exchange keys does not scale to the internet. Asymmetric cryptography solves this through mathematical key pairs — a public key that anyone can have, and a private key that never leaves its owner.

A common analogy: the public key is an open padlock that anyone can use to lock a message. Only the owner of the corresponding private key can unlock it. The padlock can be distributed freely — knowing what it looks like does not help anyone open it.

In practice, asymmetric encryption is computationally expensive relative to symmetric encryption. The solution is hybrid encryption: use asymmetric cryptography once to securely exchange a symmetric session key, then conduct the actual data exchange using symmetric encryption. This is exactly what TLS does — asymmetric operations during the handshake, AES for the session. Asymmetric cryptography also provides authentication and non-repudiation through digital signatures — capabilities symmetric encryption cannot provide.

### RSA

RSA (Rivest-Shamir-Adleman) is the most widely known asymmetric encryption algorithm. Its security rests on the computational difficulty of factoring large numbers: multiplying two large prime numbers together is fast, but reversing the operation — finding the two factors of a very large composite number — is computationally infeasible for numbers above approximately 600 digits.

The key variables in RSA:

| Variable | Role |
|---|---|
| p, q | Two large prime numbers — kept secret |
| n | n = p × q — part of both public and private keys |
| ϕ(n) | Euler's totient: ϕ(n) = (p−1) × (q−1) |
| e | Public exponent — chosen to be coprime with ϕ(n); forms the public key with n |
| d | Private exponent — calculated as the modular inverse of e mod ϕ(n); forms the private key with n |
| m | Plaintext message (as a number) |
| c | Ciphertext: c = mᵉ mod n |

The room's numerical examples:

Given p = 4391 and q = 6659:

    n = p × q = 4391 × 6659 = 29,239,669
    ϕ(n) = (p−1) × (q−1) = 4390 × 6658 = 29,228,620

RSA appears frequently in CTF challenges where specific values of p, q, e, d, or c are given and you must calculate the remaining variables or recover the plaintext. The mathematical process is standard — the formulas above are the complete reference.

### Diffie-Hellman Key Exchange

Diffie-Hellman (DH) solves a specific problem: how can two parties agree on a shared secret over a public channel, without transmitting the secret itself? The intuition uses the analogy of mixing paint — two parties each have a private colour, they share a common colour publicly, mix the common colour with their private colour, exchange the results, and then mix the received result with their own private colour. Both arrive at the same final mixture without ever transmitting their private colour.

The mathematical implementation uses modular exponentiation. Public values are a large prime p and a generator g. Each party generates a private key and computes a public key to share:

    Alice's private key: a
    Alice's public key: A = gᵃ mod p

    Bob's private key: b
    Bob's public key: B = gᵇ mod p

Each party computes the shared secret using their own private key and the other party's public key:

    Key calculated by Alice: Bᵃ mod p
    Key calculated by Bob: Aᵇ mod p

Both calculations produce the same result — the shared secret — without either party transmitting their private key.

The room's numerical examples with p = 29, g = 5:

    a = 12: A = 5¹² mod 29 = 7
    b = 17: B = 5¹⁷ mod 29 = 24
    Key by Bob: Bᵃ mod p = 24¹² mod 29 = 24
    Key by Alice: Aᵇ mod p = 7¹⁷ mod 29 = 24

Both sides arrive at the shared key `24`. Diffie-Hellman is commonly used for key agreement while RSA provides authentication — the two protocols are complementary and are combined in TLS and other protocols.

### SSH Key Authentication

SSH supports public key authentication as an alternative to passwords. The mechanism: the user generates an RSA (or ed25519, ECDSA, etc.) key pair, places the public key in `~/.ssh/authorized_keys` on the remote server, and authenticates using the private key which never leaves the local machine.

Generating an SSH key pair:

    ssh-keygen -t rsa -b 4096

The `-t` flag specifies the key type. Supported types from `man ssh-keygen`: `dsa`, `ecdsa`, `ecdsa-sk`, `ed25519`, `ed25519-sk`, `rsa`. The private key file stored in `~/Public-Crypto-Basics/Task-5` on the lab VM uses the RSA algorithm — confirmed by reading the key file header which contains `-----BEGIN RSA PRIVATE KEY-----`.

The SSH host key verification prompt seen on first connection — "The authenticity of host cannot be established. Are you sure you want to continue?" — is the client asking whether it should trust the server's public key fingerprint. Accepting it stores the fingerprint in `~/.ssh/known_hosts` for future verification.

### Digital Signatures and Certificates

A digital signature allows a sender to cryptographically prove that a message came from them and has not been altered. The sender encrypts a hash of the message with their private key — this encrypted hash is the signature. The recipient decrypts it with the sender's public key and compares the result to their own hash of the message. If they match, the message is authentic and unaltered. If they differ, either the message was modified or the signature is invalid.

Certificates extend this to identity verification at scale. A certificate contains a public key and an assertion of who owns it, signed by a Certificate Authority (CA). The browser or client trusts the certificate because it trusts the CA that signed it. What a remote web server uses to prove its identity to the client is a certificate.

The chain of trust: a root CA is embedded in the operating system and browser as inherently trusted. Intermediate CAs are trusted because the root CA signed their certificate. The server's certificate is trusted because an intermediate CA signed it. This chain allows trust to propagate from a small number of root CAs to the billions of TLS-protected websites on the internet. TLS certificates for domains can be obtained for free from Let's Encrypt.

### OpenPGP and GPG

PGP (Pretty Good Privacy) is a standard for encrypted messaging and file signing, used primarily for email encryption and software package verification. GPG (GNU Privacy Guard) is the open-source implementation of the OpenPGP standard and is the tool used in the room's practical.

GPG uses the same public/private key pair concept. To decrypt a message encrypted for you, you must first import the sender's key and then decrypt using your private key:

    gpg --import tryhackme.key
    gpg --decrypt message.gpg

The room's Task 7 requires importing a provided key and decrypting a GPG-encrypted message to obtain the flag.

---

## Walkthrough Notes

The room runs through eight tasks. A VM is deployed and accessible via split-screen view for the practical tasks.

**Task 1 (Introduction):** Frames the three problems asymmetric cryptography solves — authentication, authenticity, and integrity — using the analogy of an in-person meeting where you can verify identity visually. Notes that the next three tasks cover RSA, Diffie-Hellman, and SSH. No answer required.

**Task 2 (Common Use of Asymmetric Encryption):** Covers hybrid encryption — why asymmetric is used for key exchange and symmetric for data transfer. No standalone question beyond context understanding.

**Task 3 (RSA):** Covers RSA key generation math. Questions: given p = 4391 and q = 6659, what is n — the answer is `29239669` (4391 × 6659); what is ϕ(n) — the answer is `29228620` (4390 × 6658).

**Task 4 (Diffie-Hellman Key Exchange):** Covers DH key exchange using the paint-mixing analogy and the modular exponentiation mechanism. Questions using p = 29, g = 5: A (Alice's public key with a = 12) = `7`; B (Bob's public key with b = 17) = `24`; key calculated by Bob (Bᵃ mod p) = `24`; key calculated by Alice (Aᵇ mod p) = `24`.

**Task 5 (SSH):** Covers SSH key pair authentication and the `ssh-keygen` key types. The question asks what algorithm the private key in `~/Public-Crypto-Basics/Task-5` uses — navigating to the directory and reading the key file reveals `-----BEGIN RSA PRIVATE KEY-----`, so the answer is `RSA`.

**Task 6 (Digital Signatures and Certificates):** Covers how signatures prove authenticity and how certificates chain trust through CAs. The question asks what a remote web server uses to prove its identity to the client — the answer is `certificate`.

**Task 7 (PGP and GPG):** Practical task requiring importing `tryhackme.key` and decrypting the provided `.gpg` file. Running `gpg --import tryhackme.key` followed by `gpg --decrypt message.gpg` reveals the decrypted message and the flag.

**Task 8 (Conclusion):** Summarises the room — RSA, Diffie-Hellman, SSH, digital signatures, certificates, and GPG. Points to Hashing Basics. No answer required.

---

## Commands Used

    ssh-keygen -t rsa -b 4096
    cat ~/Public-Crypto-Basics/Task-5/id_rsa
    gpg --import tryhackme.key
    gpg --decrypt message.gpg

---

## Real-World Mapping

| Concept | Real-World Application |
|---|---|
| RSA key factoring difficulty | Key length requirements — RSA-1024 is considered breakable with sufficient resources; RSA-2048 is the current minimum recommendation; RSA-4096 is used for long-lived certificates where forward secrecy is important |
| Diffie-Hellman key exchange | TLS perfect forward secrecy — ECDHE (Elliptic Curve Diffie-Hellman Ephemeral) in TLS generates a fresh session key for every connection; even if the server's private key is later compromised, past sessions cannot be decrypted |
| SSH key authentication | Phishing resistance — SSH keys cannot be phished; a password can be typed into a fake login page, but a private key never leaves the machine; organisations that enforce key-only SSH authentication eliminate password-based SSH brute force entirely |
| Certificate chain of trust | Certificate validation in investigations — when investigating a suspicious HTTPS site, checking the certificate chain reveals who issued it, when it expires, and whether it is self-signed; newly issued certificates from uncommon CAs are a phishing indicator |
| Digital signatures | Software integrity verification — package managers (apt, yum) verify GPG signatures on downloaded packages before installation; a failed signature check indicates tampering with the package in transit or at the source |
| GPG for message encryption | Secure communications and evidence handling — GPG encryption is used by security researchers for encrypted disclosure, by journalists for source protection, and for transmitting sensitive investigation findings between teams |

---

## Takeaways

1. **The combination of Diffie-Hellman for key agreement and RSA for authentication is not an either-or design decision — it is a deliberate architecture.** DH provides a shared secret that neither party transmitted; RSA provides proof of identity that prevents a man-in-the-middle from substituting their own DH parameters. Using only DH would allow an attacker to intercept and substitute. Using only RSA for key exchange does not provide forward secrecy — if the private key is later compromised, all previously recorded sessions can be decrypted. Together, they provide both security properties simultaneously.

2. **The certificate chain of trust is the mechanism that makes HTTPS trustworthy at scale — and it is also the mechanism that makes compromised CAs catastrophic.** When a CA is compromised or issues fraudulent certificates, every site vouched for by that CA becomes suspect. Historical CA compromises — DigiNotar in 2011, Comodo in the same year — resulted in fraudulent certificates for major domains including Google and intelligence agencies. Browser vendors responded by hard-coding CT (Certificate Transparency) log requirements and revocation checking. Understanding the chain of trust explains both why HTTPS works and why CA compromise is a systemic risk.

3. **SSH key authentication is strictly superior to password authentication in every dimension that matters for security.** Keys cannot be guessed, cannot be phished, cannot be reused across sites, and cannot be captured by a keylogger. The private key never leaves the originating machine. Failed attempts to use a key against a server that has not authorised it leave no credentials behind for an attacker to capture. Organisations that still permit SSH password authentication on internet-facing systems are carrying a preventable risk.

---
