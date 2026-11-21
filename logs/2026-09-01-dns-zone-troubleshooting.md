# DNS Zone Creation Failure on First AD DS Promotion

**Date:** 2026-09-01
**Environment:** Hyper-V, Windows Server 2025 (SERVER01), internal virtual switch
**Status:** ✅ Resolved

## Goal
Promote SERVER01 to a new Active Directory forest (`corp.example`) with an
integrated DNS zone, as the foundation for a home-lab AD environment.

## What happened
After running `Install-ADDSForest` with `-InstallDns`, the promotion reported
success, but:
- `Get-DnsServerZone` showed no forward lookup zone for the domain — only the
  default reverse lookup zones.
- `nslookup` against the domain timed out.
- Manually creating the zone with `Add-DnsServerPrimaryZone` failed with a
  generic `WIN32 123` error, both as an AD-integrated zone and as a
  file-based zone.

## Diagnostic steps
1. Checked `ipconfig /all` — confirmed static IP and that the NIC's DNS
   server pointed at itself.
2. Checked `Get-Service DNS, Netlogon, NTDS` — all running.
3. Ran `dcdiag /test:dns /v` — surfaced `Error 9002 (DNS server failure)`
   and confirmed no host records or SOA record existed for the zone.
4. Checked `Get-DnsServerDirectoryPartition` — the AD application partitions
   (`DomainDnsZones`, `ForestDnsZones`) existed and were healthy.
5. Queried AD directly with `Get-ADObject -Filter 'ObjectClass -eq "dnsZone"'`
   — confirmed no zone object existed for the domain in AD at all, even
   though the partitions were fine.
6. Reviewed the DNS Server event log (`Get-WinEvent -LogName "DNS Server"`)
   — found the DNS service starts before AD DS finishes creating the
   directory partitions on first boot (a benign timing/ordering event), but
   this wasn't the actual blocker.
7. **Root cause found:** tested zone creation with a different domain name
   (`test.local`) — it succeeded instantly. This isolated the problem to the
   specific domain name in use.

## Root cause
`corp.example` uses `.example`, which is a **reserved TLD** under RFC 2606,
set aside by IANA specifically for documentation purposes and explicitly
*not* meant to be deployed as a real, resolvable domain. Windows DNS Server
refuses to create zones under reserved TLDs, which caused every zone
creation attempt to fail — regardless of whether it was AD-integrated or
file-based, and regardless of whether the forest was freshly rebuilt.

## Fix
Rebuilt the domain using a non-reserved name (`corp.local`) instead of
`corp.example`. Zone creation and promotion succeeded immediately with the
same configuration steps that had failed twice before.

## Key takeaway
When picking a lab domain name, avoid IANA-reserved TLDs
(`.example`, `.test`, `.invalid`, `.localhost`). Safe choices for labs:
`.local` (e.g. `corp.local`, `contoso.local`).

## Skills demonstrated
- Systematic troubleshooting: eliminated network, firewall, DNS client, and
  AD replication as causes before isolating the true root cause.
- Reading and interpreting Windows event logs, `dcdiag`, and `repadmin`
  output.
- Understanding of AD-integrated DNS architecture (application directory
  partitions, zone objects, replication scope).
- Awareness of DNS/domain naming standards (RFC 2606).
