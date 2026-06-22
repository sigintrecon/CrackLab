# Practical Walkthrough — Cracking a Password-Protected Archive with John the Ripper

## Objective

Create a password-protected archive on Windows, transfer it to Kali Linux, and recover the password using John the Ripper — while documenting the real troubleshooting process, since archive cracking rarely works on the first attempt without understanding the underlying file format.

## Step 1 — Creating the Archive (Windows, WinRAR)

1. Right-click the target file → **Add to archive...**
2. In the dialog, select an **Archive format**
3. Click **Set password...**, enter a password, confirm
4. Click **OK**

**Lesson learned:** WinRAR defaults to the **RAR** format even when the output file is manually renamed with a `.zip` extension. A `.zip` extension does not guarantee a ZIP-format file — the archive format must be explicitly selected as **ZIP** in the dialog, not just assumed from the file name.

## Step 2 — Installing John the Ripper (Kali)

```bash
sudo apt update
sudo apt install john -y
```

Verify the installation:
```bash
john --list=formats | head
```

Note: this version of John (Jumbo) does not support a `--version` flag — listing supported formats is a more reliable way to confirm the tool is functional.

## Step 3 — First Attempt (Failed)

```bash
zip2john sqli.zip > zip_hash.txt
```

Output:
```
Did not find End Of Central Directory.
```

### Diagnosis

This error means `zip2john` could not parse the file as a valid ZIP archive. Checking the actual file type revealed the issue:

```bash
file sqli.zip
```
```
sqli.zip: RAR archive data, v5
```

The file had a `.zip` extension but was, in fact, a RAR v5 archive — confirming the lesson from Step 1. `zip2john` only understands the ZIP format; it cannot parse RAR archives regardless of file extension.

## Step 4 — Correct Tool for the Format

For a RAR archive, the equivalent extraction tool is `rar2john`, not `zip2john`:

```bash
rar2john sqli.zip > rar_hash.txt
john rar_hash.txt
john --show rar_hash.txt
```

**Note:** RAR v5 uses AES-256 encryption, which is significantly stronger than the legacy ZipCrypto encryption used in traditional ZIP archives. Cracking time is correspondingly higher for RAR archives, especially against complex passwords.

## Step 5 — Password Strength and Practical Cracking Limits

An initial test used a deliberately strong, long password (35+ characters, mixed case, digits, symbols). This password was **not** crackable via dictionary attack (not present in any wordlist) and was **not realistically brute-forceable** given the size of the keyspace at that length.

This is not a tool failure — it is the expected, correct behavior. A sufficiently long, random password is specifically resistant to all three cracking techniques covered in this repository (see `03-Rainbow-Tables/rainbow-tables-notes.md` for the underlying theory).

For demonstration purposes, the password was changed to a common, dictionary-predictable value, after which cracking succeeded quickly using `rockyou.txt`:

```bash
john --wordlist=/usr/share/wordlists/rockyou.txt zip_hash.txt
john --show zip_hash.txt
```

## Recommended Workflow Going Forward

For consistent, reproducible cracking practice, create test archives directly in Kali using the ZIP format and legacy encryption, avoiding cross-platform format ambiguity entirely:

```bash
zip -e test.zip document.docx
```

This guarantees compatibility with `zip2john` and avoids the RAR/ZIP format confusion documented above.

## Key Takeaways

- Always verify a file's actual format with `file <filename>` before assuming based on its extension.
- `zip2john` and `rar2john` are format-specific — using the wrong one produces a parsing error, not a cracking failure.
- A successful crack depends entirely on password predictability (dictionary match) or feasible keyspace size (brute force) — not on the tool itself. Strong passwords are *supposed* to resist these tools; that is the entire point of password strength requirements.
