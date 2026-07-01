# Fawn — HTB Starting Point

- Platform: Hack The Box — Starting Point
- OS: Unix/Linux
- Difficulty: Very Easy

## Reconnaissance

```bash
nmap -sV <TARGET_IP>
```

Only port 21/tcp (FTP) is open.

```bash
nmap -sV -p21 <TARGET_IP>
```

- FTP version: vsftpd 2.3.4
- OS type: Unix

## Foothold — Anonymous FTP Login

```bash
ftp <TARGET_IP>
```

- Username: `anonymous`
- Password: `anonymous`
- Response code on success: `230` (Login successful)

### File enumeration

```bash
ls
# or
dir
```

Found `flag.txt`.

```bash
get flag.txt
```

## Attack Narrative

Nmap showed only port 21 open, running vsftpd 2.3.4 on Unix. Anonymous login (`anonymous:anonymous`) was tried first and succeeded immediately with a `230` response code. Listing the directory revealed `flag.txt`, which was downloaded with `get`.

## Lessons Learned

- FTP transmits data in cleartext — no encryption.
- Anonymous login is a common misconfiguration; always test `anonymous:anonymous` first.
- `nmap -sV` is usually enough for version detection when `-O` doesn't return reliable OS info.
- SFTP (over SSH) is the secure alternative to plain FTP.
- vsftpd 2.3.4 is separately known for a backdoor vulnerability (CVE-2011-2523) — a different issue from anonymous login misconfiguration (see the Metasploitable2 write-up in this repo).

## Mitigation

- Disable anonymous FTP access unless explicitly required, and never pair it with write access.
- Prefer SFTP/FTPS over plain FTP.
- Keep FTP daemon versions patched and verified against official sources.
