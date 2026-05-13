# Hashing Basics

| Field | Details |
|---|---|
| **Module** | Cyber Security 101 — Cryptography |
| **Difficulty** | Easy |
| **Platform** | TryHackMe |
| **Room Link** | https://tryhackme.com/room/hashingbasics |
| **Date Completed** | February 2026 |
| **Author** | Adwait Joshi |

---

## What This Room Covers

This is the third room in the CS101 Cryptography section. The previous two rooms covered encryption — a reversible process that requires a key to undo. This room covers hashing — a one-way process with no key and no reversal. Hashing transforms input data of any size into a fixed-length output called a hash or digest. The same input always produces the same hash; a different input — even by a single bit — produces a completely different hash. These properties make hashing the correct tool for two specific tasks: storing passwords securely without storing the password itself, and verifying that a file has not been modified. The room covers hash functions, insecure password storage, secure password storage with salting, recognising common hash formats, cracking hashes with hashcat, file integrity checking, HMAC, and the distinction between hashing, encoding, and encryption.

---

## Key Concepts

### Hash Functions

A hash function takes an input of any length and produces a fixed-length output — the hash, digest, or checksum. Hash functions are deterministic: the same input always produces the same output. They are one-way: given a hash, it is computationally infeasible to recover the original input. They are also avalanche-sensitive: changing even one bit of the input produces a completely different hash — not a slightly different one.

The avalanche effect is demonstrated in the room with two files containing adjacent ASCII characters: `T` (0x54 in hex, binary `01010100`) and `U` (0x55 in hex, binary `01010101`) — differing by exactly one bit. Their MD5 hashes are entirely different:

    md5sum file1.txt    # b9ece18c950afbfa6b0fdbfa4ff731d3
    md5sum file2.txt    # 4c614360da93c0a041b22e537de151eb

Common hash algorithms and their output sizes:

| Algorithm | Output Size (bytes) | Output Size (bits) | Status |
|---|---|---|---|
| MD5 | 16 | 128 | Broken — collision attacks trivial; do not use for security |
| SHA-1 | 20 | 160 | Deprecated — collision demonstrated in 2017 |
| SHA-256 | 32 | 256 | Current standard — widely used |
| SHA-512 | 64 | 512 | Higher security margin — used where extra strength is warranted |

Hash output is typically encoded in hexadecimal — each raw byte becomes two hex characters. MD5's 16 bytes become 32 hex characters. SHA-256's 32 bytes become 64 hex characters. Base64 encoding is an alternative that produces shorter strings but is less commonly used in Linux command-line tools.

The number of possible hash values for an N-bit hash output is 2ᴺ. For an 8-bit hash output, the number of possible hash values is `256` (2⁸).

The SHA-256 hash of `passport.jpg` in `~/Hashing-Basics/Task-2` on the lab VM is obtained by running:

    sha256sum ~/Hashing-Basics/Task-2/passport.jpg

The result is `77148c6f605a8df855f2b764bcc3be749d7db814f5f79134d2aa539a64b61f02`.

### Insecure Password Storage

The temptation to store passwords in plaintext is the worst possible approach — a single database breach exposes every user's password immediately. Encrypting passwords is also incorrect — encryption is reversible with the key, so anyone who compromises the key (or the server) recovers all passwords. The correct approach is hashing.

When a user registers, the server hashes their password and stores the hash. When they log in, the server hashes the submitted password and compares the result to the stored hash. The original password is never stored. The question of whether to encrypt passwords in password-verification systems is answered firmly: `Nay`.

The problem with naive hashing — storing `hash(password)` with no modification — is rainbow tables. A rainbow table is a precomputed dataset mapping hash values back to their original inputs. Because the hash function is deterministic, any dictionary word hashed with MD5 always produces the same result. A rainbow table for common passwords against MD5 is a lookup structure — given a hash, finding the password is an O(1) operation. The hash `4c5923b6a6fac7b7355f53bfe2b8f8c1` looked up in the rainbow table provided in the room returns `inS3CyourP4$$`.

**rockyou.txt** is the most commonly used password wordlist in security work. It was leaked from the rockyou.com breach in 2009 and contains approximately 14 million real-world passwords. The 20th password in the list is `qwerty`, found with:

    head -n 20 /usr/share/wordlists/rockyou.txt

### Secure Password Storage — Salting

The defence against rainbow tables is a salt — a random value generated uniquely for each user and appended to their password before hashing. The stored entry becomes `hash(password + salt)` along with the salt itself (stored in plaintext alongside the hash). Because each user has a different salt, two users with identical passwords produce different hashes — rainbow table lookups fail entirely, since the precomputed table was built without knowledge of each user's salt.

Modern password hashing functions are specifically designed for this purpose: bcrypt, scrypt, and Argon2. Unlike general-purpose hash functions (MD5, SHA-256) which are designed to be fast, password hashing functions are deliberately slow — they include a configurable work factor or cost parameter that increases computation time. A hash that takes 100 milliseconds to compute is imperceptible to a legitimate user logging in but makes brute-force cracking 10 million times more expensive than a function that takes 10 nanoseconds.

### Recognising Password Hashes

Different hash algorithms produce recognisably different output formats. The room covers how to identify hashes from their structure:

| Format | Identifier | Example |
|---|---|---|
| MD5 (Unix) | `$1$` | `$1$THM$...` |
| bcrypt | `$2a$` or `$2b$` | `$2a$06$...` |
| SHA-512 (Unix) | `$6$` | `$6$GQXVvW4EuM$...` |
| scrypt | `$7$` | `$7$...` |
| Cisco-IOS (scrypt) | `$9$` | `$9$...` |

The format prefix in Unix password hashes is standardised — `$6$` means SHA-512 crypt, `$9$` means scrypt. Identifying the algorithm from the hash format is required before selecting the correct cracking mode. For Windows: NTHash (NTLM) is the format used to store user and service passwords on modern Windows systems, held in the SAM database. The hashcat mode number for Cisco-ASA MD5 is `2410`, found by consulting the hashcat example hashes reference. The hashing algorithm used in Cisco-IOS when the hash starts with `$9$` is `scrypt`.

### Password Cracking with Hashcat

Hashcat is the GPU-accelerated password cracking tool. It accepts a hash file, a hash mode number identifying the algorithm, and a wordlist, and attempts to crack the hash by comparing candidates against the stored hash.

Basic syntax:

    hashcat -m MODE hash_file.txt wordlist.txt

Common hash modes:

| Mode | Algorithm |
|---|---|
| `0` | MD5 |
| `100` | SHA-1 |
| `1000` | NTLM |
| `1800` | sha512crypt — Unix `$6$` |
| `3200` | bcrypt — `$2a$` or `$2b$` |
| `2410` | Cisco-ASA MD5 |
| `1750` | HMAC-SHA512 (key = $pass) |

Practical cracking results from the room's Task 6 hash files:

- `hash1.txt` (`$2a$06$7yoU3Ng8dHTXphAg913cyO6Bjs3K5lBnwq5FJyA6d01pMSrddr1ZG`) — bcrypt — cracked to `85208520`
- `hash2.txt` (SHA-256: `9eb7ee7f551d2f0ac684981bd1f1e2fa4a37590199636753efe614d4db30e8e1`) — cracked to `halloween`
- `hash3.txt` (`$6$GQXVvW4EuM$...`) — sha512crypt — cracked to `spaceman`
- `hash4.txt` (`b6b0d451bbf6fed658659a9e7e5598fe`) — MD5 — cracked to `funforyou`

### Hashing for File Integrity

Because the same input always produces the same hash, hashing is an ideal verification mechanism. Software vendors publish SHA-256 checksums alongside their downloads. After downloading, running `sha256sum` on the file and comparing the result to the published checksum confirms the file is identical to what the vendor distributed — any modification during transit or at the download mirror will produce a different hash.

The SHA-256 hash of `libgcrypt-1.11.0.tar.bz2` found in `~/Hashing-Basics/Task-7` is:

    sha256sum ~/Hashing-Basics/Task-7/libgcrypt-1.11.0.tar.bz2
    # 09120c9867ce7f2081d6aaa1775386b98c2f2f246135761aae47d81f58685b9c

HMAC (Hash-Based Message Authentication Code) extends file and message integrity checking by incorporating a secret key into the hash computation. The formula is `HMAC(K, M) = H((K ⊕ opad) || H((K ⊕ ipad) || M))` — a two-pass hash that combines the key with the message. The output verifies both integrity (the message has not been altered) and authenticity (it was produced by someone who knows the key). The hashcat mode number for HMAC-SHA512 where the key is the password is `1750`.

### Hashing vs Encoding vs Encryption

The room closes by distinguishing three concepts that are frequently confused:

| Operation | Reversible? | Key Required? | Purpose |
|---|---|---|---|
| Hashing | No | No | Integrity verification, password storage |
| Encoding | Yes | No | Data representation (Base64, hex) — no security |
| Encryption | Yes | Yes | Confidentiality |

Encoding is not a security measure — Base64 is a representation format, not protection. Decoding `RU5jb2RlREVjb2RlCg==` with `base64 --decode` returns `ENcodeDEcode`. Anyone who can see the encoded string can trivially decode it.

---

## Walkthrough Notes

The room runs through eight tasks. A VM is deployed for the practical hashing and cracking tasks.

**Task 1 (Introduction):** Notes two primary uses of hashing in security — password storage and file integrity. Introduces the lab VM. No answer required.

**Task 2 (Hash Functions):** Covers hash function properties, output sizes, and encoding. Questions: SHA-256 hash of `passport.jpg` — `77148c6f605a8df855f2b764bcc3be749d7db814f5f79134d2aa539a64b61f02` (run `sha256sum ~/Hashing-Basics/Task-2/passport.jpg`); output size in bytes of MD5 — `16`; number of possible hash values for an 8-bit output — `256`.

**Task 3 (Insecure Password Storage):** Covers plaintext and naive hash storage risks. The question asks for the 20th password in `rockyou.txt` — `qwerty`, found with `head -n 20 /usr/share/wordlists/rockyou.txt`.

**Task 4 (Using Hashing for Secure Password Storage):** Covers rainbow tables and salting. Questions: manually check hash `4c5923b6a6fac7b7355f53bfe2b8f8c1` against the provided rainbow table — `inS3CyourP4$$`; crack hash `5b31f93c09ad1d065c0491b764d04933` using an online tool — `tryhackme`; should you encrypt passwords in password-verification systems — `Nay`.

**Task 5 (Recognising Password Hashes):** Covers hash format identifiers. Questions: hashcat mode for Cisco-ASA MD5 — `2410` (found via hashcat example hashes reference); hashing algorithm for Cisco-IOS hashes starting with `$9$` — `scrypt`.

**Task 6 (Password Cracking with Hashcat):** Practical cracking of four hash files in `~/Hashing-Basics/Task-6/`. Hashcat is run against each with the appropriate mode and the rockyou wordlist. Results: `hash1.txt` → `85208520`; `hash2.txt` → `halloween`; `hash3.txt` → `spaceman`; `hash4.txt` → `funforyou`.

**Task 7 (Hashing for Integrity Checking):** Covers file verification with SHA-256 checksums and HMAC. Questions: SHA-256 hash of `libgcrypt-1.11.0.tar.bz2` — `09120c9867ce7f2081d6aaa1775386b98c2f2f246135761aae47d81f58685b9c` (run `sha256sum ~/Hashing-Basics/Task-7/libgcrypt-1.11.0.tar.bz2`); hashcat mode number for HMAC-SHA512 (key = $pass) — `1750`.

**Task 8 (Encoding, Encryption, and Hashing):** Covers the distinction between the three. The practical question requires decoding `RU5jb2RlREVjb2RlCg==` using `base64 --decode decode-this.txt` — the result is `ENcodeDEcode`.

---

## Commands Used

    md5sum file.txt
    sha1sum file.txt
    sha256sum file.txt
    sha512sum file.txt
    sha256sum ~/Hashing-Basics/Task-2/passport.jpg
    sha256sum ~/Hashing-Basics/Task-7/libgcrypt-1.11.0.tar.bz2
    head -n 20 /usr/share/wordlists/rockyou.txt
    hashcat -m 3200 ~/Hashing-Basics/Task-6/hash1.txt /usr/share/wordlists/rockyou.txt
    hashcat -m 1400 ~/Hashing-Basics/Task-6/hash2.txt /usr/share/wordlists/rockyou.txt
    hashcat -m 1800 ~/Hashing-Basics/Task-6/hash3.txt /usr/share/wordlists/rockyou.txt
    hashcat -m 0 ~/Hashing-Basics/Task-6/hash4.txt /usr/share/wordlists/rockyou.txt
    base64 --decode decode-this.txt

---

## Real-World Mapping

| Concept | Real-World Application |
|---|---|
| SHA-256 for file integrity | Malware analysis and software verification — comparing the SHA-256 hash of a suspicious file against VirusTotal or a known-clean reference immediately establishes whether it has been tampered with or matches a known malicious sample |
| Rainbow table attacks | Password breach assessment — when a password database without salting is breached, rainbow table lookups against MD5 or SHA-1 hashes recover most common passwords within seconds; this is why unsalted hashes in breached databases are treated as plaintext-equivalent |
| bcrypt / scrypt / Argon2 | Password storage review — auditing an application's authentication code to confirm it uses a slow, salted hashing function rather than MD5 or SHA-1 is a standard secure code review item |
| NTLM hash format recognition | Credential material identification — recognising an NTLM hash in a captured file or memory dump allows immediate identification of Windows credential material without needing to crack it (Pass-the-Hash attacks use the hash directly) |
| Hashcat with rockyou.txt | Credential audit — organisations run hashcat against their own password hashes to identify weak passwords before attackers do; this is a standard component of an internal Active Directory security assessment |
| HMAC for message authentication | API security — HMAC is used to authenticate API requests; the client signs the request with a shared secret, the server verifies the HMAC before processing; a failed HMAC verification indicates tampering or replay |
| Base64 encoding vs encryption | Analyst awareness — malware and attackers frequently use Base64 to encode payloads, commands, or data, relying on analysts confusing encoding with encryption; recognising Base64 strings and decoding them immediately is a basic triage skill |

---

## Takeaways

1. **Hashing is not encryption, and treating it as such is a security failure.** Encryption is reversible — a hashed password database protected with encryption is only as secure as the encryption key. If the key is on the same server, a compromise yields both the encrypted data and the key. Hashing is not reversible — a hashed password database exposes only hashes, and recovering passwords from hashes requires brute force against each hash individually. The distinction has concrete consequences: storing passwords as `AES(password)` is a critical vulnerability; storing them as `bcrypt(password + salt)` is the correct design.

2. **The speed of a hash function is a security parameter for password storage — faster is worse.** MD5 can compute billions of hashes per second on consumer GPU hardware. bcrypt with a cost factor of 12 computes fewer than 1,000 per second on the same hardware. For file integrity checking or HMAC, speed is desirable. For password storage, that same speed advantage goes entirely to the attacker. Choosing a fast hash function for password storage is a design error with measurable consequences — it is the difference between a breach that exposes a few cracked passwords and one that exposes millions.

3. **Recognising hash formats by their structure is a practical skill that accelerates incident response.** When a compromised database contains `$2a$06$...` entries, the incident responder immediately knows these are bcrypt hashes and informs the organisation that cracking them in bulk is computationally infeasible with current hardware — buying time for a mandatory password reset. When the database contains 32-character hex strings (unsalted MD5), the urgency calculus changes entirely — most of those passwords will be cracked within hours using a GPU and rockyou.txt. The hash format determines the timeline, and the timeline determines the response.

---
