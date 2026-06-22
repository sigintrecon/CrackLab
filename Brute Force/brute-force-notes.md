# Brute Force Attacks

## Definition

A brute force attack is the most straightforward password cracking technique. It involves systematically trying every possible combination of characters until the correct password is found. It does not rely on any prior knowledge about the password, relying instead on exhaustive computation — making it a "dumb" but ultimately effective method, given enough time and computing power.

## How It Works

1. The attacker defines a character set (e.g., lowercase letters, digits, special characters) and a length range.
2. Every possible combination within that space is generated and tried in sequence.
3. The attack continues until the correct password is found, or the entire keyspace is exhausted.

There is no intelligence involved — no assumptions about likely passwords are made. This is the key difference from a dictionary attack (see `02-Dictionary-Attacks/`).

## Characteristics

| Factor | Impact |
|---|---|
| Password length/complexity | Directly increases time required — exponentially, not linearly |
| Computing power | More threads/GPU power reduces time, but cannot overcome very long passwords |
| Detectability | Highly detectable — generates a large volume of failed login attempts in a short window |
| Target type | Effective against short/simple passwords; impractical against long, high-entropy ones |

## Why Length Matters More Than Complexity

A keyspace grows exponentially with length. A 4-character numeric PIN has only 10,000 possible combinations (`10^4`) — trivial to brute force in seconds. An 8-character password using mixed case, digits, and symbols has a keyspace in the trillions. A 16+ character random password is computationally infeasible to brute force with current technology, regardless of GPU power available to an attacker.

This is why modern security guidance has shifted from "complex but short" passwords toward "long" passphrases — length defeats brute force far more effectively than special characters alone.

## Detection & Defenses

Brute force attacks are noisy by nature and can be mitigated through:

- **Account lockout policies** — locking an account after N failed attempts
- **Rate limiting / throttling** — delaying responses after repeated failures
- **Multi-Factor Authentication (MFA)** — renders a cracked password alone insufficient
- **Login attempt monitoring** — services like Instagram and Facebook flag unusual login patterns (new device, new location, high attempt frequency) and challenge with CAPTCHA or temporary lockouts
- **Strong password policies** — minimum length requirements significantly raise the computational cost of a successful attack

## Practical Tool Reference

See `hydra-walkthrough.md` in this folder for live, tested examples of brute forcing SSH and web login forms using Hydra.

For offline brute forcing of password hashes (e.g., when the password itself is unknown but a hash has been recovered), John the Ripper's incremental mode is used:

```bash
john --incremental hash_file.txt
```

This differs fundamentally from Hydra's approach: John operates offline against a static hash with no network interaction, while Hydra attacks a live service and is subject to detection, lockouts, and network latency.
