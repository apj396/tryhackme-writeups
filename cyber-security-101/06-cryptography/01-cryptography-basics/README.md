# Cryptography Basics

| Field | Details |
|---|---|
| **Module** | Cyber Security 101 — Cryptography |
| **Difficulty** | Easy |
| **Platform** | TryHackMe |
| **Room Link** | https://tryhackme.com/room/cryptographybasics |
| **Date Completed** | February 2026 |
| **Author** | Adwait Joshi |

---

## What This Room Covers

This is the first room in the CS101 Cryptography section. Cryptography is the mathematical foundation underneath every security protocol covered in this path — TLS, SSH, HTTPS, certificate authorities, password storage, and file integrity all depend on it. This room introduces the core vocabulary and concepts: what encryption is, the distinction between symmetric and asymmetric encryption, the role of keys and ciphers, and the two mathematical operations that most cryptographic algorithms are built on — XOR and modulo. The room uses the Caesar Cipher as a concrete, accessible entry point before moving to modern ciphers, establishing both the intuition and the vocabulary that the next three rooms build on.

---

## Key Concepts

### The Language of Cryptography

A precise shared vocabulary is the prerequisite for discussing cryptography. The room establishes the following definitions:

| Term | Definition |
|---|---|
| Plaintext | The original, readable data before encryption |
| Ciphertext | The result of encrypting plaintext — unreadable without the key |
| Cipher | The algorithm used to perform encryption and decryption |
| Key | A string of bits used by the cipher to encrypt or decrypt data |
| Encryption | The process of converting plaintext into ciphertext using a cipher and a key |
| Decryption | The reverse process — converting ciphertext back into plaintext using a cipher and a key |

A critical design principle underlies all of this: the cipher is public knowledge — its algorithm can be published and studied. Only the key must remain secret. This is Kerckhoffs's principle, and it means the security of any cryptographic system should rest entirely on the secrecy of the key, not the secrecy of the algorithm. A system that requires the algorithm to stay secret is a weaker system, because algorithms are much harder to change than keys once the system is deployed.

Cryptography serves three primary goals in security: confidentiality (only authorised parties can read the data), integrity (data has not been altered in transit or storage), and authenticity (the data genuinely comes from who it claims to come from). Symmetric encryption primarily provides confidentiality. Asymmetric encryption addresses all three, and is the focus of the next room.

### Historical Context — The Caesar Cipher

Cryptography dates to ancient Egypt around 1900 BCE. One of the most historically significant ciphers is the Caesar Cipher, used by Julius Caesar in the first century BCE. The mechanism is simple: each letter in the plaintext is shifted by a fixed number of positions in the alphabet. A shift of 3 moves A to D, B to E, C to F, and so on. To decrypt, the shift is reversed by the same amount.

The room's Caesar Cipher question asks for the original plaintext of `XRPCTCRGNEI`. Using an online tool such as Cryptii or dcode.fr with a backward shift of 11 (or equivalently a forward key of 15), the result is `ICANENCRYPT`.

The Caesar Cipher illustrates both the concept of encryption and its fragility when the keyspace is small. With only 25 possible shifts (excluding no-shift), brute force is trivial — try all 25. This weakness motivates the design of modern ciphers with astronomically larger keyspaces.

### Symmetric Encryption

Symmetric encryption uses the same key for both encryption and decryption. Because both parties must possess an identical copy of the key, it is also called private key cryptography or secret key cryptography. The primary challenge is key distribution — getting the shared key to the intended recipient securely, particularly over an untrusted channel, without exposing it to interception.

Key symmetric algorithms:

**DES (Data Encryption Standard)** was adopted as a standard in 1977 and uses a 56-bit key. By 1999, its 56-bit keyspace had been shown to be insufficient — a distributed computing effort cracked a DES-encrypted message in under 24 hours. DES should not be trusted for any modern application.

**3DES (Triple DES)** applied DES three times in sequence to increase effective key strength to 112 bits. It was a temporary measure while a successor was developed and was deprecated in 2019.

**AES (Advanced Encryption Standard)** was adopted as the encryption standard in 2001. It supports key sizes of 128, 192, and 256 bits and is the current industry standard for symmetric encryption. AES is widely used in TLS, file encryption, disk encryption, and VPNs.

### Asymmetric Encryption

Asymmetric encryption uses a mathematically linked key pair — a public key and a private key. Data encrypted with the public key can only be decrypted with the corresponding private key, and vice versa. The public key can be shared freely; the private key must never leave the owner's possession.

This design solves the key distribution problem that symmetric encryption faces. Two parties can establish a shared encrypted channel without ever transmitting a secret over the network — the public key is transmitted openly, the private key never moves. The details of specific asymmetric algorithms — RSA, Diffie-Hellman, elliptic curve systems — are covered in the next room.

### The Mathematics — XOR and Modulo

Modern cryptographic algorithms are built on mathematical operations that are computationally efficient in one direction and extremely difficult to reverse. The room introduces two fundamental operations:

**XOR (Exclusive OR)** is a binary operation that compares two bits and returns 1 if they differ and 0 if they are the same. The truth table: `0 ⊕ 0 = 0`, `0 ⊕ 1 = 1`, `1 ⊕ 0 = 1`, `1 ⊕ 1 = 0`. XOR has properties that make it directly useful in symmetric encryption: it is its own inverse (`A ⊕ A = 0`), it is commutative (`A ⊕ B = B ⊕ A`), and applying it with a key produces ciphertext that can be recovered simply by applying the same key again. Given plaintext P and key K, ciphertext C = P ⊕ K, and plaintext can be recovered as C ⊕ K = P. This makes XOR a valid building block for symmetric encryption — though in practice, the key must be as long as the plaintext (a one-time pad) to be theoretically unbreakable, which is operationally impractical.

**Modulo** (written as `%` or `mod`) returns the remainder of a division. `X % Y` is the remainder when X is divided by Y: `25 % 7 = 4`. The modulo operation bounds a value within a fixed range — a property that makes it essential in public-key cryptography. Large prime number arithmetic using modulo underlies RSA and Diffie-Hellman. A key property is that modulo is easy to compute but difficult to reverse — knowing that `X % N = R` with a large N, finding X from R is computationally infeasible without additional information.

---

## Walkthrough Notes

The room runs through seven tasks, all conceptual — no lab VM is deployed.

**Task 1 (Introduction):** Sets the context for the three-room cryptography series. Notes prerequisites — no prior cryptography knowledge required, but familiarity with basic binary and hex is helpful. No answer required.

**Task 2 (Importance of Cryptography):** Covers the three goals of cryptography — confidentiality, integrity, authenticity. Questions: what do you call the encrypted plaintext — the answer is `ciphertext`; what do you call the process that returns the plaintext — the answer is `decryption`.

**Task 3 (Plaintext to Ciphertext):** Introduces the Caesar Cipher as a historical example and defines key, cipher, plaintext, and ciphertext. The question asks for the original plaintext of `XRPCTCRGNEI` encrypted with Caesar Cipher — the answer is `ICANENCRYPT`. This is found by trying backward shifts using an online Caesar Cipher tool; a shift of 11 backward reveals the plaintext.

**Task 4 (Symmetric and Asymmetric Encryption):** Covers DES, 3DES, AES, and the concept of asymmetric key pairs. Questions: should you trust DES — `Nay`; when was AES adopted as an encryption standard — `2001`.

**Task 5 (Basic Math for Cryptography):** Covers XOR and modulo. The truth table questions are answered by applying the XOR operation bit by bit. The modulo questions are answered by performing the division and reading the remainder.

**Task 6 (Summary):** Recaps symmetric and asymmetric encryption, XOR, and modulo. Points to Public Key Cryptography Basics. No answer required.

---

## Tools Referenced

External tools useful for this room:

    https://cryptii.com/pipes/caesar-cipher    # Caesar Cipher encoder/decoder
    https://www.dcode.fr/caesar-cipher         # Alternative Caesar Cipher tool

---

## Real-World Mapping

| Concept | Real-World Application |
|---|---|
| Plaintext vs ciphertext distinction | Incident analysis — determining whether data captured in transit or at rest was encrypted or in plaintext is a fundamental triage question; plaintext credentials in a network capture represent an immediate credential exposure finding |
| Cipher and key separation (Kerckhoffs's principle) | Security architecture review — any system where security depends on keeping the algorithm secret rather than the key is poorly designed and fragile; this principle underpins why AES is safe to use openly |
| DES deprecation | Compliance and vulnerability assessment — identifying legacy systems still using DES or 3DES in transit is a finding in any cryptographic posture review; PCI DSS and other frameworks explicitly prohibit DES |
| AES as current standard | Encryption posture verification — confirming that file encryption, disk encryption, and VPN tunnels use AES-128 or AES-256 is a standard configuration review item |
| XOR as symmetric building block | Malware analysis — XOR encoding is the most common obfuscation technique in malware; recognising repeating XOR patterns in binary analysis identifies the key and decodes payloads |
| Modulo in public key cryptography | Conceptual foundation for RSA and DH — understanding that large prime modulo arithmetic is computationally one-directional is the basis for understanding why RSA key length matters and why small keys are breakable |

---

## Takeaways

1. **The Caesar Cipher teaches the two requirements any cipher must satisfy — and immediately illustrates how the Caesar Cipher fails both.** A cipher must make ciphertext appear random to an observer, and it must require the key to reverse. The Caesar Cipher produces ciphertext that is clearly alphabetic, and its keyspace of 25 is small enough to brute force by hand. Modern ciphers — AES with 256-bit keys — produce output that is statistically indistinguishable from random noise and have keyspaces so large that brute force is computationally impossible. The gap between Caesar and AES is the entire history of cryptography.

2. **The separation of cipher (public) and key (secret) is not a convenience — it is a security requirement.** If the algorithm itself must remain secret to provide security, then the system fails the moment the algorithm is reverse-engineered or leaked — which happens inevitably over time. If only the key must remain secret, then rotating keys restores security. This asymmetry is why open cryptographic standards like AES are more trustworthy than proprietary or secret algorithms: they have been publicly analysed, attacked, and validated by the entire cryptography research community.

3. **XOR as a building block for obfuscation appears everywhere in malware — and recognising it is an entry-level analysis skill.** Malware authors XOR-encode payloads, strings, and configuration data to avoid signature detection. Because XOR is its own inverse and trivially reversible when the key is known, a single-byte XOR key can often be brute-forced across 255 possible values in seconds. Understanding how XOR works at the bit level makes this analysis immediate rather than opaque.

---
