# Scripts

- `recon/` — nmap/recon automation (`auto-recon.sh`): phases, comments, echo output for every step.
- `ad/` — Active Directory enumeration helpers (BloodHound/SharpHound wrappers), added in Phase 2.

Style: bash scripts with clear phases (`# --- Phase: ... ---`), comments for every command, echo status between steps. No hardcoded credentials or real IPs — everything through arguments/variables.
