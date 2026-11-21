# Device and protocol support matrix (research note)

> Status: researched, pending on-device validation on the remote GNS3 server.
> Access date for all sources: **2026-11-21**.
> Owner-confirmed image inventory (2026-11-21): CSR1000v **IOS XE Amsterdam 17.3.8a**,
> ASAv **ASA 9.24(x)**, FTDv **10.0.0**, IOSv **15.x** (exact build still open), plus an
> Ubuntu endpoint (GNS3 Docker/QEMU node). Reference topology is limited to **5 nodes**
> by server load; the firewall (FTDv) is mandatory in the reference project.

## Protocol support per image

| Image | Version | Console (GNS3 telnet) | SSH/CLI | NETCONF | REST |
|---|---|---|---|---|---|
| CSR1000v | IOS XE 17.3.8a | yes | yes | **yes** — `netconf-yang`, TCP 830 | **yes** — RESTCONF, `restconf`, TCP 443 |
| ASAv | ASA 9.24(x) | yes | yes | no | **yes** — ASA REST API, `rest-api agent`, base `/api/` |
| FTDv | 10.0.0 | yes | yes (FTD CLI over SSH) | no | **yes** — FDM REST API, base `/api/fdm/v1/` (alias `/api/fdm/latest/`), HTTPS, token auth |
| IOSv | 15.x | yes | yes | no | no |
| Ubuntu endpoint | Docker/QEMU node | — | yes | — | runs pyATS / test scripts |

Key point: **NETCONF is mandatory in the course, and CSR1000v 17.3.8a is the only
candidate image that supports it.** IOSv is SSH/console only; ASA and FTD are REST-only
on the management plane. Device-specific labs must be designed around this matrix.

## Sources

| Claim | Source | URL |
|---|---|---|
| NETCONF/RESTCONF are IOS XE / IOS-XR / NX-OS features; CSR1000v NETCONF since IOS XE 16.3, RESTCONF since 16.6 | Cisco DevNet, "Standard Device Interfaces & Models" platform matrix | https://developer.cisco.com/site/standard-network-devices/ |
| IOS XE Amsterdam 17.3.x has dedicated NETCONF and RESTCONF configuration chapters | Cisco, "Programmability Configuration Guide, Cisco IOS XE Amsterdam 17.3.x" | https://www.cisco.com/c/en/us/td/docs/ios-xml/ios/prog/configuration/173/b_173_programmability_cg.html |
| Classic IOS 15 (IOSv) is not in the NETCONF/RESTCONF matrix | same DevNet matrix (IOS XE/XR/NX-OS rows only) | https://developer.cisco.com/site/standard-network-devices/ |
| ASA REST API exists since 9.3, ASAv ships the REST agent built in (`rest-api agent`); no NETCONF on ASA | Cisco, "ASA REST API Quick Start Guide"; "About the ASA REST API" | https://www.cisco.com/c/en/us/td/docs/security/asa/api/quick-start/qsg-asa-api.html |
| ASA 9.24(x) is a current train | Cisco, "Release Notes for the Cisco Secure Firewall ASA, 9.24(x)" | https://www.cisco.com/c/en/us/td/docs/security/asa/asa924/release/notes/asarn924.html |
| FTD 10.0 exists (upgrades from 7.3/7.4/7.6/7.7 supported); FDM REST API base path `/api/fdm/v1/`, `latest` alias documented; no NETCONF on FTD | Cisco, "Secure Firewall Threat Defense REST API Guide"; "Release Notes … Version 10" | https://www.cisco.com/c/en/us/td/docs/security/firepower/ftd-api/guide/ftd-rest-api/ftd-rest-api-intro.html ; https://www.cisco.com/c/en/us/td/docs/security/secure-firewall/release-notes/threat-defense/100/device-manager-release-notes-10-0.html |
| FTDv runs in 90-day evaluation without Smart License registration | Cisco, FDM Configuration Guide, "Licensing the System" | https://www.cisco.com/c/en/us/td/docs/security/firepower/70/fdm/fptd-fdm-config-guide-700/fptd-fdm-license.html |
| GNS3 server 3.0.x is GPL-3.0 (no license fee); REST API v3 (`/v3/...`); console via telnet host:port; Docker nodes supported; `gns3fy` targets API v2 (v3 unverified — teach `requests`). Course standardizes on 3.0.x + version shim; v2/v3 diff documented in `gns3-api-v2-vs-v3.md` | GNS3/gns3-server LICENSE (GPL-3.0), releases; api.gns3.net; apiv3.gns3.net; docs.gns3.com; gns3fy docs | https://github.com/GNS3/gns3-server/blob/master/LICENSE ; https://api.gns3.net ; https://apiv3.gns3.net ; https://davidban77.github.io/gns3fy/ |
| Cisco images (IOSv, CSR1000v, ASAv, FTDv) require a Cisco/VIRL entitlement; GNS3 does not distribute them | GNS3 docs, "Install an appliance from the Marketplace" | https://docs.gns3.com/docs/using-gns3/beginners/install-from-marketplace |
| pyATS install and usage (`pip install pyats[full]`, aetest, testbed YAML) | Cisco DevNet pyATS docs; PyPI | https://developer.cisco.com/docs/pyats/ ; https://pypi.org/project/pyats/ |
| ncclient NETCONF client | ncclient docs / PyPI | https://pypi.org/project/ncclient/ |
| Python Essentials 1/2 (OpenEDG/NetAcad) do not cover threading/asyncio/multiprocessing | OpenEDG Python Institute, "PCAP: Programming Essentials in Python (2.0) Overview" | https://zsl.gliwice.pl/wp-content/uploads/2021/03/Python-Essentials-2-0-Overview.pdf |

## Enablement commands (for lab design; synthetic credentials only)

- CSR1000v 17.3.8a: `netconf-yang` (NETCONF, TCP 830), `restconf` (RESTCONF, TCP 443),
  SSH via `username ... privilege 15` + `ip ssh version 2`, `aaa new-model`.
- ASAv 9.24: `http server enable`, `rest-api agent`, REST API at `https://<asa>/api/`.
- FTDv 10.0: FDM REST API at `https://<ftd>/api/fdm/latest/`, OAuth2 token
  (`/api/fdm/v1/fdm/token` — verify exact path on device), HTTPS management on the
  outside/diagnostic interface.
- Console bootstrap for all: GNS3 console = telnet to `<server>:<console_port>`.

## Risks and open validation items

- [ ] On-device check: CSR1000v 17.3.8a NETCONF `<hello>` capabilities via ncclient.
- [ ] On-device check: FTD 10.0 accepts `/api/fdm/latest/` and token endpoint path.
- [ ] CSR1000v without a license boots in limited mode with a throughput cap
      (e.g. 100 Kbps limited mode / 2.5 Mbps after eval) — irrelevant for ping/DNS
      lab traffic, but do not plan bulk transfers through it.
      Source: Cisco CSR 1000v Configuration Guide, "Installing Cisco CSR 1000v Licenses".
- [ ] Exact IOSv 15.x build number still to be confirmed (SSH-only either way).
- [ ] Server RAM/CPU/disk sizing: FTDv typically needs ~8 GB RAM and 4 vCPUs;
      validate the 5-node reference topology against actual server resources.
- [ ] WireGuard VPN provisioning + GNS3 account procedure document is still to be written.
