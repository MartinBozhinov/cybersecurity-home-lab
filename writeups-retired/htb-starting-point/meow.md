# Meow — HTB Starting Point

- Platform: Hack The Box — Starting Point
- OS: Linux
- Difficulty: Easy

## Reconnaissance

```bash
nmap <TARGET_IP>
```

```
Not shown: 999 closed tcp ports (reset)
PORT   STATE SERVICE
23/tcp open  telnet
```

Only port 23/tcp (telnet) is open — direct, unauthenticated remote access.

## Foothold

```bash
telnet <TARGET_IP>
```

The service accepts a connection with no credentials at all and drops directly into a root session.

```bash
whoami
# root
```

## Flags

| Flag | Location |
|---|---|
| User flag | `/home/<user>/user.txt` |
| Root flag | `/root/root.txt` |

## Attack Narrative

Port 23 (telnet) was the only open port. Rather than assuming SSH would be the entry point, connecting directly with `telnet <IP>` dropped straight into a root shell — no credentials were requested at all.

## Lessons Learned

- Telnet (port 23) is a cleartext protocol with no encryption — treat any exposed instance as high risk.
- Not every open remote-access service requires credentials; test the connection directly before assuming an auth wall exists.

## Mitigation

- Disable telnet entirely; use SSH with key-based authentication instead.
- If telnet must remain for legacy reasons, restrict access via firewall rules to trusted hosts only, and never expose it without authentication.
