# Scripts

- `recon/` — nmap/recon automation (`auto-recon.sh`): фази, коментари, echo output за всяка стъпка.
- `ad/` — Active Directory enumeration helpers (BloodHound/SharpHound wrappers), добавя се в Phase 2.

Стил: bash скриптове с ясни фази (`# --- Phase: ... ---`), коментари за всяка команда, echo статус между стъпките. Никакви hardcoded credentials или реални IP-та — всичко през аргументи/променливи.
