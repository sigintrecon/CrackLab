# Practical Walkthrough — TryHackMe Hydra Room

## Objective

Use Hydra to brute-force credentials for a known user (`molly`) against two separate services on a deployed TryHackMe machine: a web login form and SSH. The room provides one known username and requires recovering the corresponding password for each service.

## Pre-Requisites

- Connected to the TryHackMe VPN
- Target machine deployed, IP address noted from the TryHackMe room interface
- `rockyou.txt` available and extracted:
```bash
gunzip /usr/share/wordlists/rockyou.txt.gz
```

---

## Flag 1 — Web Login Form

### Reconnaissance

Before attacking the form, its exact structure must be confirmed:
1. Open the login page in a browser
2. Open Developer Tools → Network tab
3. Submit a deliberately wrong login attempt
4. Inspect the request to identify the exact field names used for username/password
5. Inspect the response to identify the exact text returned on a failed login (the failure string)

### Command

```bash
hydra -l molly -P /usr/share/wordlists/rockyou.txt <MACHINE_IP> http-post-form "/:username=^USER^&password=^PASS^:F=incorrect" -V
```

| Component | Value Used | Purpose |
|---|---|---|
| `-l molly` | Known username provided by the room | Fixes the username, only the password is brute-forced |
| `-P` | rockyou.txt | Dictionary attack — assumes a realistic, human-chosen password |
| Path | `/` | Login form is on the homepage |
| Failure string | `F=incorrect` | Identifies failed attempts in the server's response |
| `-V` | — | Verbose mode, confirms field names/failure string are correct in real time |

### Result

On match, Hydra prints the valid username/password combination directly. Logging into the web application with the recovered credentials reveals **Flag 1**.

---

## Flag 2 — SSH

### Command

```bash
hydra -l molly -P /usr/share/wordlists/rockyou.txt <MACHINE_IP> -t 4 ssh
```

SSH brute forcing requires no form reconnaissance — Hydra communicates with the SSH service directly using the standard protocol.

### Accessing the Target

```bash
ssh molly@<MACHINE_IP>
```

Once authenticated, **Flag 2** is located in the home directory or desktop:
```bash
ls -la
cat flag2.txt
```

(`-la` is used deliberately, since flag files are sometimes hidden with a leading dot.)

---

## Issues Encountered & Resolved

| Issue | Cause | Fix |
|---|---|---|
| Hydra syntax parsed incorrectly | Extra whitespace inserted after a colon in the `http-post-form` string (`: F=incorrect` instead of `:F=incorrect`) | Removed whitespace — Hydra parses the string literally |
| Invalid target | Used `192.168.0.0`, a network address rather than a host address | Replaced with the actual machine IP assigned by TryHackMe |
| Wrong username used | Initially substituted a personal username instead of the room-specified `molly` | Re-read room instructions; used the exact username specified in the challenge |

## Key Takeaways

- Web form brute forcing requires manual reconnaissance of the form's field names and failure condition — this cannot be guessed reliably without inspecting the actual HTTP request/response.
- SSH brute forcing is comparatively simpler since the protocol is standardized and requires no field-name discovery.
- Small syntax errors (stray whitespace, malformed IP addresses) are the most common cause of Hydra failures — not the underlying technique itself.
