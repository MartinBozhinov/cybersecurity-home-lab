# Pickle Rick — TryHackMe

Beginner-friendly Rick and Morty themed web machine. Цел: три "Ingredients" файла, скрити зад web enumeration, command injection и privilege escalation.

## Vulnerability

- Web сървър с изложени коментари в HTML/robots.txt, водещи до скрити страници и creds.
- Панел за административни команди, изпълняващ ги без sanitization → OS command injection.
- SUID бинарник, позволяващ privilege escalation до root.

## Attack

1. Enumeration:
   ```bash
   nmap -sV -p- <target-ip>
   curl http://<target-ip>/robots.txt
   ```
   `robots.txt` разкрива низ, който служи като username hint; преглед на HTML source на страницата разкрива парола в коментар.
2. Login в admin панела с намерените credentials → достъп до "Command Panel", изпълняващ shell команди.
3. Command injection през панела за първоначален shell:
   ```
   ls -la; whoami
   ```
4. Enumeration за privesc — търсене на SUID бинарници:
   ```bash
   find / -perm -4000 2>/dev/null
   sudo -l
   ```
5. Използване на разрешен `sudo`/SUID бинарник за ескалация до root и събиране на трите "Ingredients" файла (flags).

## Evidence

*(попълни с реален output от твоята сесия — namите на трите ingredient файла и root shell доказателство)*

## Mitigation

- Никога не оставяй credentials в HTML коментари или публично достъпни файлове.
- Валидирай и sanitize-вай всякакъв входящ команден низ преди подаване към shell (избягвай директно `os.system`/`exec` с user input).
- Ограничавай `sudo`/SUID права до строг минимум, необходим за конкретната функция.
