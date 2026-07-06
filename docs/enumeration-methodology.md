# Enumeration Methodology

A repeatable workflow to run against every open port, so I don't skip steps under
pressure. Written in my own words; all addresses are placeholders.

> The lesson that produced this doc: on a service-enumeration lab I lost ~2 hours
> not because I didn't know the techniques, but because I had no workflow — I
> skipped banners, didn't check non-standard ports, and kept hitting the same wall.

## Step 0 — Port map

```bash
ip a                                    # my own network
nmap -sn <NETWORK>/24                    # host discovery
nmap -sS -p- --min-rate 5000 <TARGET>    # ALL ports, not just top 1000
nmap -sS -sV -sC -p <OPEN_PORTS> <TARGET> # versions + default scripts
```

Record the **actual port** next to every service. A non-standard port (FTP on
5554, SSH on 2222) breaks tools silently — you then pass `-s` / `RPORT` / `-p`.

## Step 1 — Same order for every service

```
1. BANNER       → what does the service announce (version, warning, hint)?
2. VERSION      → -sV; look for known CVEs (searchsploit)
3. ANON/NULL    → anonymous login / null session (FTP anon, SMB -N)
4. USER ENUM    → pull usernames (smb_enumusers, SMTP VRFY, ...)
5. BRUTE        → found users + wordlist (hydra / *_login)
6. LOGIN        → get in; grab flags/files/credentials
7. CHAINING     → any creds found → try them on ALL other services
```

## Step 2 — Map port → technique

| Port | Service | First moves |
|---|---|---|
| 21 | FTP | anonymous login, `hydra ftp://<TARGET> -s <port>` |
| 22 | SSH | read the login banner first, then brute if needed |
| 25 | SMTP | `VRFY`/`EXPN` user enumeration |
| 80/443 | HTTP | dir/vhost fuzzing, tech fingerprint, source review |
| 139/445 | SMB | `smbclient -N //<TARGET>/share`, `enum4linux -a` |
| 3306 | MySQL | default/weak creds, `mysql -h <TARGET> -u root` |

## Rules that keep me unstuck

- **Banner before brute force.** Flags/hints often sit in login banners (SSH!).
- **Check the port.** Tool "not working"? 90% of the time it's a non-standard port → `-s` (hydra) / `RPORT` (msf) / `-p` (ssh/nmap).
- **Time-box 30 min per vector**, then move on and come back.
- **Credential chaining.** Every password/hash found → try it on every other service.
- **Wordlists live in `/usr/share/wordlists/`** — check before writing a script from scratch.

## smbclient syntax gotcha

On Linux use forward slashes: `smbclient -N //<TARGET>/share`. The backslash form
`\\\\<TARGET>\\share` only works with escaped backslashes; unescaped, the shell
eats them and the connection fails. Default to `//`.
