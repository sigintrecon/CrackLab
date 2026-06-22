# Rainbow Tables

## Definition

A rainbow table is a precomputed table of password hashes, built in advance to allow extremely fast password recovery through lookup rather than live computation. Instead of guessing a password and hashing it on the spot (as brute force and dictionary attacks do), a rainbow table attack works backward: the hash is already known in advance, and the attacker simply searches a precomputed dataset for a match.

## How It Differs From Brute Force / Dictionary Attacks

| | Brute Force / Dictionary | Rainbow Table |
|---|---|---|
| When computation happens | Live, during the attack (compute hash → compare → repeat) | In advance, before the attack even starts |
| Speed during the attack | Slower — every attempt requires a hash computation | Very fast — just a table lookup |
| Resource cost | CPU/GPU time during the attack | Disk space to store the precomputed table (can run into hundreds of GB) |
| Reusability | Wordlist can be reused, but cracking time is paid every time | Table is built once, then reused instantly across many cracking attempts |

This is fundamentally a **time vs. storage trade-off**: rainbow tables shift the computational cost from "during the attack" to "before the attack," at the price of needing large amounts of disk space to store precomputed hash chains.

## Why Salting Defeats Rainbow Tables

A salt is a random value generated uniquely for each stored password, combined with the password before hashing:

```
hash(password + salt) = stored_hash
```

Without a salt, two users with the same password (`password123`) produce the *identical* hash — meaning one precomputed rainbow table can crack every account using that password, across every breached database that doesn't salt.

With salting, the same password produces a **completely different hash for every user**, because each user has a different random salt. This means an attacker would need a separate, full rainbow table for every possible salt value — which, given a sufficiently random and long salt, is computationally infeasible to precompute.

This is precisely why modern password storage standards (**bcrypt**, **scrypt**, **Argon2**) build salting directly into the algorithm, and why rainbow tables are largely ineffective against any reasonably modern authentication system. They remain relevant primarily against legacy systems and weak/unsalted hash implementations (e.g. older LM/NTLM Windows hashes).

## Practical Tools

### ophcrack
A GUI-based tool, historically well known for cracking Windows LM/NTLM hashes using free, publicly available rainbow tables.
```bash
ophcrack
```
Pre-built tables (e.g. the "Vista free" table set) must be downloaded separately and loaded into the tool.

### rcracki_mt
A command-line rainbow table lookup tool, used against a directory of precomputed tables:
```bash
rcracki_mt -h <hash> /path/to/rainbow_tables/
```

## Real-World Relevance Today

Rainbow tables are largely a *historical* technique in modern penetration testing. Their relevance has declined sharply because:

1. **Salting is now standard practice** across virtually all serious authentication systems.
2. **GPU-accelerated live cracking (Hashcat)** has become fast enough that precomputation often isn't even necessary anymore — a modern GPU can compute billions of hashes per second on the fly, narrowing the speed advantage rainbow tables once offered.
3. **Storage cost** of rainbow tables covering modern password lengths/character sets has become impractical compared to simply running Hashcat directly against a target hash.

Understanding rainbow tables remains important conceptually — they explain *why* salting exists as a defense, and they're still occasionally relevant against legacy or misconfigured systems (e.g. an old Windows domain still issuing unsalted LM hashes for backward compatibility).

## Summary — All Three Cracking Techniques Compared

| Technique | Best Against | Defeated By |
|---|---|---|
| Brute Force | Short, simple passwords | Length (exponential keyspace growth) |
| Dictionary Attack | Predictable, human-chosen passwords | Long random passphrases not in any wordlist |
| Rainbow Table | Unsalted hash databases | Per-user salting |

This closes out the core password cracking methodology covered in this repository: exhaustive search (Brute Force), pattern-based search (Dictionary), and precomputed lookup (Rainbow Table) — each with a distinct trade-off between speed, coverage, and the specific defense that neutralizes it.
