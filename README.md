# CrackLab — Password Cracking Techniques & Practical Lab Notes

> A focused, hands-on documentation of password cracking methodology — Brute Force, Dictionary Attacks, and Rainbow Tables — backed by real lab exercises and tested commands.

[![Status](https://img.shields.io/badge/status-active-brightgreen)]()
[![Focus](https://img.shields.io/badge/focus-Password%20Cracking-blue)]()

---

## About This Repository

This repository is a dedicated deep-dive into password cracking — one of the most fundamental skills in offensive security. Rather than just listing commands, every technique documented here was tested against real targets in an isolated lab: live services (SSH, web login forms), offline hash files, and encrypted archives.

The goal is methodology, not memorization: understanding *when* each technique applies, *why* it works, and *what defenses* neutralize it.

---

## Lab Environment

| Component | Details |
|---|---|
| Attacker Machine | Kali Linux |
| Targets | Windows 7, Metasploitable2, local test files |
| Platforms | TryHackMe (Jr Penetration Tester path) |

All exercises were performed against authorized lab machines and self-created test files only.

---

## Repository Structure

```
CrackLab/
├── README.md
├── 01-Brute Force/
│   ├── brute-force-notes.md
│   └── hydra-walkthrough.md
├── 02-Dictionary Attacks/
│   └── dictionary-attack-notes.md
├── 03-Rainbow Tables/
│   └── rainbow-tables-notes.md
├── Practical Walkthroughs/
    ├── zip-cracking-with-john.md
    └── thm-hydra-room.md

```

---

## Topics Covered

| Technique | Description | Status |
|---|---|---|
| Brute Force | Systematically trying every possible combination | ✅ |
| Dictionary Attack | Testing passwords from curated wordlists (e.g. rockyou.txt) | ✅ |
| Rainbow Tables | Precomputed hash lookups vs. live computation, and why salting defeats them | ✅ |
| Online Cracking | Live service attacks (SSH, HTTP POST forms) with Hydra | ✅ |
| Offline Cracking | Hash file attacks with John the Ripper / Hashcat | ✅ |
| Archive Cracking | Password-protected ZIP/RAR recovery (zip2john, rar2john) | ✅ |

---

## Tools Used

`Hydra` `John the Ripper` `Hashcat` `Medusa` `zip2john` `rar2john` `rcracki_mt`

---

## Sample Walkthrough — Cracking a Password-Protected Archive

```bash
# Step 1: Extract a crackable hash from the archive
zip2john protected.zip > zip_hash.txt

# Step 2: Run John the Ripper against it
john --wordlist=/usr/share/wordlists/rockyou.txt zip_hash.txt

# Step 3: Reveal the cracked password
john --show zip_hash.txt
```

Full walkthrough with troubleshooting (RAR vs ZIP format issues, encryption type mismatches) available in `Practical-Walkthroughs/zip-cracking-with-john.md`.

---

## Key Takeaways

- **Brute force** guarantees a result but is computationally expensive — impractical against long, high-entropy passwords.
- **Dictionary attacks** are effective specifically because most real-world passwords are predictable, not random.
- **Rainbow tables** trade disk space for speed, but are rendered ineffective by per-user salting — which is why modern systems (bcrypt, scrypt, Argon2) remain resistant even to GPU-accelerated cracking.
- **Defense always matters as much as offense**: account lockout policies, MFA, and salted hashing are covered alongside every attack technique.

---

## Ethics Statement

All cracking techniques in this repository were performed against self-owned test files, intentionally vulnerable lab machines (Metasploitable2), or authorized environments (TryHackMe-deployed rooms). This repository is intended strictly for educational purposes and does not support unauthorized access to any system.

---

## About Me

I'm an aspiring penetration tester documenting a structured path toward formal cybersecurity education and a career in red teaming. This repository is part of a broader, ongoing learning series.

**Connect:** [[GitHub Profile](https://github.com/sigintrecon)]

---

*Actively maintained — new techniques and walkthroughs added as labs are completed.*
