# home-lab-it
 
A Hyper-V home lab for practising IT support and Windows administration. It is built around an Active Directory domain (`corp.local`) and documented as a build log: what I set up, what broke, and how I fixed it.
 
## Lab environment
 
| Machine | Role | Details |
| --- | --- | --- |
| SERVER01 | Domain controller (AD DS, DNS) | Windows Server 2025, static IP 192.168.50.10 |
| CLIENT01 | Domain client | Windows 11 Pro VM |
 
- **Virtualisation:** Hyper-V, with all machines on an internal virtual switch (`LAB-SWITCH`)
- **DHCP:** scope 192.168.50.100 – 192.168.50.200
- **Domain:** `corp.local` (a lab name; a production domain would be a subdomain of a registered domain)
## Active Directory structure
 
- **OUs:** `Corp Users` (sub-OUs: IT, HR, Finance), `Corp Computers` (Workstations), `Corp Groups`
- **Security groups:** IT-Users, HR-Users, Finance-Users
- **First user:** alex.morgan, placed in the IT OU
## Build log
 
Each entry describes the goal, the diagnostic steps and the fix.
 
| Date | Entry | Topic |
| --- | --- | --- |
| 2026-09-01 | [DNS zone creation failure on first AD DS promotion](logs/2026-09-01-dns-zone-troubleshooting.md) | Troubleshooting with `dcdiag`, event logs and PowerShell; first domain name failed, rebuilt as `corp.local` |
 
## Next steps
 
- Second client VM (USER02)
- Everyday L1/L2 ticket simulations: account management, Group Policy, file and printer sharing
- Later: more system administration and networking topics
## Skills practised
 
Active Directory, DNS, DHCP, Hyper-V, PowerShell, systematic troubleshooting, technical documentation
 