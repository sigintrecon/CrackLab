# Dictionary Attacks

## Definition

A dictionary attack is a password cracking technique that tests passwords from a pre-built list of likely candidates — known as a wordlist — rather than generating every possible combination from scratch. It relies on the observation that most real-world passwords are not random; they are words, names, dates, or predictable patterns that humans choose because they are easy to remember.

## Why Dictionary Attacks Work

Brute force assumes nothing about the password and pays the full computational cost of an unstructured search. Dictionary attacks instead exploit a simple fact: human-generated passwords cluster heavily around a relatively small set of patterns — common words, keyboard sequences, names, dates, and predictable substitutions (`a` → `@`, `o` → `0`, appending `123` or `!`).

This is why a 35-character random string (uppercase, lowercase, digits, symbols) is effectively immune to a dictionary attack — it doesn't appear in any wordlist, no matter how large — while a password like `Summer2024!` is cracked almost instantly, despite "looking" complex.

## Wordlists

| Wordlist | Description |
|---|---|
| `rockyou.txt` | ~14 million real passwords leaked from the 2009 RockYou breach; the de facto standard wordlist for cracking practice |
| Custom wordlists | Built using target-specific information (names, dates, company names) via tools like `cupp` or `cewl` |
| Rule-mutated wordlists | A base wordlist combined with transformation rules (e.g. appending digits, leetspeak substitutions) to cover predictable variations |

On Kali Linux, `rockyou.txt` ships compressed by default:
```bash
gunzip /usr/share/wordlists/rockyou.txt.gz
```

## Practical Usage

### Offline — John the Ripper
```bash
john --wordlist=/usr/share/wordlists/rockyou.txt hash_file.txt
john --show hash_file.txt
```

### Offline — Hashcat (GPU-accelerated, significantly faster for large wordlists)
```bash
hashcat -m 1000 -a 0 ntlm_hashes.txt rockyou.txt
```
(`-m 1000` specifies NTLM hash mode; `-a 0` specifies straight dictionary mode)

### Rule-Based Dictionary Attacks (Hashcat)
Real passwords are rarely an exact dictionary match — they're often a dictionary word with minor mutations. Rules apply common transformations automatically:
```bash
hashcat -m 1000 -a 0 ntlm_hashes.txt rockyou.txt -r rules/best64.rule
```
This single command effectively tests thousands of mutated variants of every word in the base wordlist (capitalization, appended digits, symbol substitution) without manually generating them.

### Online — Hydra
Dictionary attacks apply equally to live services, using the same `-P` wordlist flag covered in `01-Brute-Force/hydra-walkthrough.md`. The technique is identical; the distinction between "brute force" and "dictionary" with Hydra lies entirely in whether `-x` (generated combinations) or `-P` (curated wordlist) is used.

## Generating a Custom Wordlist

When attacking a specific, known individual (a realistic scenario in social-engineering-aware penetration tests), a targeted wordlist often outperforms a generic one:

```bash
cupp -i
```
This interactively prompts for known details about the target (name, birthdate, pet names, etc.) and generates a custom wordlist of likely password candidates based on common human patterns.

## Defenses

- **Banned password lists** — many platforms now reject passwords found in known breach compilations (a direct countermeasure to rockyou.txt-style attacks)
- **Passphrase encouragement** — longer, multi-word passphrases resist dictionary attacks better than single complex words
- **Rate limiting & account lockout** — same as brute force defenses; dictionary attacks still require many attempts against a live service
- **Salted hashing** — does not prevent dictionary attacks directly, but ensures cracked hashes can't be reused across a breached database via precomputed lookups

## Key Distinction From Brute Force

| | Brute Force | Dictionary Attack |
|---|---|---|
| Assumption | None — exhaustive search | Passwords follow predictable human patterns |
| Speed | Slow, scales exponentially with length | Fast, bounded by wordlist size |
| Coverage | Guaranteed eventually | Limited to what's in the wordlist (+ rules) |
| Effectiveness today | Poor against long/random passwords | Very effective against real-world human-chosen passwords |
