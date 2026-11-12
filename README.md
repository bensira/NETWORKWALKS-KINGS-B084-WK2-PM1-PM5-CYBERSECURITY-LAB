# 🔍 Footprinting, OSINT & Network Scanning — Cybersecurity Internship (Week 2)

**Author:** Emeka Nnorom  
**Program:** Cybersecurity Internship, Networkwalks (`B084A-Networkwalks`)  
**Date:** 10 November 2026  
**Modules Covered:** W2-PM1 → W2-PM5  

---

## ⚠️ Disclaimer
All activity documented here was performed with explicit authorized permission against `networkwalks.com`, or against infrastructure I personally own (my own local LAN / VMs). This write-up is for educational and portfolio purposes only. Do not run these techniques against systems you don't have permission to test — unauthorized access is illegal in most jurisdictions.
---

## Overview
This project covers three core hands-on reconnaissance and network discovery modules completed during Week 2 of my cybersecurity internship:

| # | Module | Tool(s) Used | Target / Scope |
|---|---|---|---|
| 1 | **Footprinting with multiple tools** | `whois`, `whatweb`, `nslookup`, `curl`, `wafw00f`, `dnsrecon` | `networkwalks.com` (Authorized) |
| 2 | **Footprinting with theHarvester** | `theHarvester` (Baidu & All Sources) | `microsoft.com` (Practice OSINT) |
| 3 | **Network scanning** | Zenmap (Nmap GUI), `ipconfig` | Local LAN `10.0.0.10/24` (Owned VM) |

**Target(s):**
* `networkwalks.com` — authorized external reconnaissance target
* `microsoft.com` — passive OSINT syntax and tooling practice target
* Local LAN `10.0.0.10/24` — personal VirtualBox lab environment

---

## Tools Used
* Kali Linux & Windows 10 (VirtualBox)
* `WHOIS`, `WhatWeb`, `nslookup`, `curl`, `wafw00f`, `dnsrecon`
* `theHarvester` (OSINT email & subdomain gathering)
* Zenmap (Nmap GUI for network host discovery and topology mapping)

---

## 1️⃣ Footprinting — `networkwalks.com

### WHOIS
* **Registrar:** GoDaddy.com, LLC
* **Created:** `2019-11-06` · **Expires:** `2027-11-06` · **Updated:** `2025-11-12`
* **Name servers:** `NS6135.HOSTGATOR.COM`, `NS6136.HOSTGATOR.COM`
* **Registrant Identity:** Protected via Domains By Proxy, LLC
* **DNSSEC:** Unsigned

### WhatWeb
* **HTTP â HTTPS Redirect** confirmed.
* **Server & IP:** Apache running on `192.232.216.135`.
* **CMS & Frameworks:** WordPress 7.1.4, WordPress Download Manager 3.3.76, Bootstrap 7.1.4, jQuery 3.7.1.
* **Metadata:** Google Tag Manager present; contact email exposed (`info@networkwalks.com`); Page title: *"Networkwalks Academy"*.

### Nslookup
* Resolves to `192.232.216.135` (via local resolver `192.168.100.1`) â consistent with WhatWeb findings.

### curl (-I)
```bash
curl -I https://networkwalks.com
```
* `HTTP/2 200` response with `server: Apache` and `x-nginx-cache: WordPress`.
* `link: <https://networkwalks.com/wp-json/>` — exposes WordPress REST API endpoints.
* `set-cookie: __wpdm_client=...; secure; HttpOnly` — tied directly to WordPress Download Manager.
* `permissions-policy` references Google reCAPTCHA, Cloudflare Turnstile, and hCaptcha..

### Wafw00f
* Confirmed target is protected behind **ModSecurity (SpiderLabs)** WAF.

### DNSRecon
* Performed general enumeration against `networkwalks.com`, confirming A record resolution and testing DNSSEC/SOA/SRV query responses.

---

## 2️⃣ Footprinting with `theHarvester` (Practice Task — `microsoft.com`)
This was a deliberate syntax and tooling practice run on `microsoft.com`, separate from the `networkwalks.com` target.

* **Task 1 (Baidu Search):** `theHarvester -d microsoft.com -l 1000 -b baidu` -> Returned zero results due to strict search engine automated filtering and regional indexing.
* **Task 2 (All Sources):** `theHarvester -d microsoft.com -l 50 -b all` -> Successfully initialized OSINT modules, gracefully handling missing API keys for commercial services (BeVigil, BuiltWith, Censys, etc.).

**Takeaway:** `theHarvester`'s effectiveness relies heavily on configured API keys for proprietary data sources, but remains a powerful passive OSINT tool for collecting public subdomains and emails without touching target perimeters.

---

## 3️⃣ Network Scanning — Zenmap (Local LAN)

### Host & IP Configuration (`ipconfig` on Windows 10 VM):
* **IPv4 Address:** `10.0.0.10`
* **Subnet Mask:** `255.255.255.0`
* **Default Gateway:** `10.0.0.1`

### Zenmap Ping Scan (`nmap -sn 10.0.0.10/24`):
* Scanned 256 IP addresses in **0.50 seconds** and identified 4 live hosts:
  1. `10.0.0.1`  MAC `52:55:0A:00:00:01` (Default Gateway)
  2. `10.0.0.2`  MAC `08:00:27:8A:35:D2` (Oracle VirtualBox virtual NIC)
  3. `10.0.0.3`  MAC `52:55:0A:00:00:03` (Active local node)
  4. `10.0.0.10` Scanning host (Windows 10 VM)
* **Topology View:** Inspected the star topology map in Zenmap, verifying that all discovered live hosts connect radially through localhost.

---
## 🚨 Risk Summary
Finding | Risk | Potential Impact / Observation |
|---|---|---|
| WordPress/plugin versions exposed (`WhatWeb`, `curl`) | 🟠 Medium | Exact versions (WP 7.1.4, WPDM 3.3.76) can be cross-referenced against public CVE databases. |
| Server IP identifiable | 🟢 Low | Maps hosting provider infrastructure (`192.232.216.135`). |
| HTTP headers leak plugin/API metadata | 🟠 Medium | Exposes internal cookies (`__wpdm_client`) and REST API paths (`/wp-json/`). |
| WAF vendor identifiable (`ModSecurity`) | 🟢 Low | Reveals perimeter defense architecture. |
| Contact email discoverable via OSINT | 🟢 Low | Exposes `info@networkwalks.com` for potential phishing or social engineering. |
| 4 live hosts visible on local LAN | 🟠 Medium | Demonstrates rapid internal subnet discovery and DHCP allocation mapping. |

*Note: These are observations from reconnaissance, not confirmed vulnerabilities. No exploitation or credential testing was performed.*
---

## ✅ Recommendations
* Suppress version-revealing headers (e.g., specific plugin banners and redirection tags).
* Keep WordPress core, themes, and plugins patched and current.
* Keep WAF rules tuned and monitored.
* Replace plaintext contact emails with a secure contact form to prevent harvesting.
* Periodically re-scan the LAN and verify all MAC vendors.
* Configure API keys for OSINT tooling for complete coverage during reconnaissance.
* Only test systems with explicit, documented authorization.

---

## 🧾 Conclusion
This week reinforced that a lot can be learned about a target before any exploitation through DNS records, HTTP headers, WHOIS data, and OSINT tools. It also highlighted the operational reality that OSINT tools are only as good as their configuration (API keys), and that good reporting means being transparent about what worked, what didn't, and why.

All activity was performed within an authorized scope: either against `networkwalks.com` with explicit permission, or against infrastructure I own.

*Full evidence screenshots available in `/evidences` â see the companion Word report (`W2-PM-FINAL_Emeka_Nnorom_B084A-v2.docx`) for the complete write-up with all 11 figures.*
