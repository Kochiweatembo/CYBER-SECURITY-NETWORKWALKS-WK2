# PENETRATION TESTING

**Footprinting & Network Scanning Phases**

W2-PM-FINAL | CYBERSECURITY | NETWORKWALKS

| Field | Detail |
|---|---|
| **Pentester Name (Cybersecurity Professional)** | Kochiwe Anthony Tembo |
| **Program/Batch** | Networkwalks Cybersecurity Internship 2026, Batch B084
| **Date** | 14 November 2026 (tools run on 13-14 November 2026) |
| **Modules completed** | W2-PM1 (Multiple Kali Tools)<br>W2-PM3 (Maltego)<br>W2-PM4 (theHarvester)<br>W2-PM5 (Zenmap Scanning) |
| **Client/Target** | 1. Networkwalks (`networkwalks.com`), permission secured<br>2. My own network, my phone's hotspot (Zenmap scan)<br>3. `microsoft.com`, passive OSINT only, target set by module W2-PM4 |
| **Permission secured from client?** | Yes
| **Phases covered** | **Phase 1:** Reconnaissance & Footprinting<br>**Phase 2:** Scanning & Network Discovery<br>**Phase 3-5:** In Progress |

> The full report with all evidence is also available as a Word file: [W2-PM-FINAL_Pentest_Report_Kochiwe-Tembo.docx](./W2-PM-FINAL_Pentest_Report_Kochiwe-Tembo.docx)

---

## 1. Liability Disclaimer

I have performed these activities only on the systems & devices where I had secured written permission or the devices/systems that I own myself, or, for passive open-source research, on public information sources. All these materials are for education and research purpose only. Do not use anything from here to break the law. The instructor, the authors and Networkwalks are not responsible for what you do with this knowledge. Every action you take is your own responsibility. Misuse can lead to criminal charges, heavy fines, loss of your job and a permanent record. In most countries unauthorised access is a crime even when nothing is damaged.

## 2. Introduction

This report covers four Week 2 project modules of the Networkwalks Cybersecurity internship: footprinting `networkwalks.com` with six Kali Linux tools (W2-PM1), finding email addresses with Maltego (W2-PM3), collecting public information with theHarvester (W2-PM4), and discovering hosts on a network with Zenmap (W2-PM5). Together they show how an attacker moves from gathering public information to mapping live hosts.

Footprinting commands were run in Kali Linux 2026.3 inside VirtualBox. Maltego and Zenmap were run on a Windows PC. For every step I record the exact command, what I observed, a screenshot as evidence, and why the finding matters from an attacker's point of view.

**Scope.** Networkwalks gave permission for me to test its own academy website (`networkwalks.com`) as part of the internship, and the Zenmap scan was run on my own network (my phone's mobile hotspot). The W2-PM4 module specifies `microsoft.com` as its practice target, so I used it only with theHarvester's public search sources (Baidu and the "all" source list), without the DNS lookup or brute-force options, and I did not scan or connect to Microsoft systems myself. Nothing was exploited, and no passwords or accounts were tested.

## 3. Tools Used

| Tool | Purpose |
|---|---|
| Kali Linux & Windows | Operating systems used (Kali Linux 2026.3 in VirtualBox; Windows PC for Zenmap) |
| WHOIS | Find domain registration details (registrar, dates, name servers) |
| WhatWeb | Fingerprint web technologies (server, CMS, plugins, IP) |
| nslookup | Resolve the domain name to its IP address using DNS |
| curl -I | Read the HTTP response headers of the website |
| wafw00f | Detect whether a Web Application Firewall protects the site |
| dnsrecon | Enumerate DNS records (SOA, NS, A, SRV) and DNS server software |
| Maltego | Graph-based OSINT: find email addresses linked to the domain |
| theHarvester | Passive OSINT: collect emails, hosts and IPs from public sources |
| Zenmap (Nmap GUI) | Ping-scan to discover live hosts and draw the network topology |
| Windows CMD | Local IP address, subnet and MAC identification (`ipconfig`) |

## 4. Activities Performed

### 4.1 Footprinting with Kali Tools (W2-PM1)

I ran six tools against `networkwalks.com`: **WHOIS, WhatWeb, nslookup, curl, wafw00f and dnsrecon**. Each collected a different kind of information, and several results agree with each other (the same IP address and the same email address appear in more than one tool).

#### WHOIS: domain registration

**Command:** `whois networkwalks.com` (Figures 1 and 2)

- Registrar: GoDaddy.com, LLC (IANA ID 146). Created 2019-11-06, expires 2027-11-06, last updated 2025-11-12.
- Owner details are hidden by a privacy service (Domains By Proxy, LLC, Tempe, Arizona, US); contact is only through a GoDaddy web form.
- Status flags: `clientTransferProhibited`, `clientUpdateProhibited`, `clientRenewProhibited`, `clientDeleteProhibited` (registrar locks).
- Name servers: `NS6135.HOSTGATOR.COM` and `NS6136.HOSTGATOR.COM`. The registrar's own record also lists `NS29.DOMAINCONTROL.COM` and `NS30.DOMAINCONTROL.COM`.
- DNSSEC: unsigned.

**Why it matters:** it identifies the registrar and DNS hosting provider and the domain's age. The privacy service and transfer locks are good controls; the extra GoDaddy name servers are worth checking.

#### WhatWeb: web technology fingerprint

**Command:** `whatweb networkwalks.com` (Figure 3)

- `http://` returns 301 Moved Permanently and redirects to `https://networkwalks.com/`.
- HTTPS returns 200 OK. Detected: Apache, WordPress 7.0.4, WordPress Download Manager 3.3.58, jQuery 3.7.1, HTML5, Open Graph; page title "Networkwalks Academy".
- Also exposed: IP 192.232.216.135 (United States), cookie `__wpdm_client` (HttpOnly) and the email address `info@networkwalks.com`.
- WhatWeb also reported Bootstrap 7.0.4, which is the same number as the WordPress version. That looks like a detection false positive, so I did not treat it as a real finding without manual verification.

**Why it matters:** exact CMS and plugin versions let an attacker look up known weaknesses for those versions.

#### nslookup: DNS resolution

**Command:** `nslookup networkwalks.com` (Figure 4)

- Resolver used: 8.8.8.8 (Google Public DNS). The answer was non-authoritative.
- `networkwalks.com` resolves to **192.232.216.135**.

**Why it matters:** it gives the IP address of the web server, which is the starting point for network-level reconnaissance. A non-authoritative answer comes from the resolver's cache rather than the domain's own name server.

#### curl -I: HTTP response headers

**Command:** `curl -I https://networkwalks.com` (Figure 5)

- `HTTP/1.1 200 OK`, `Date: Fri, 13 Nov 2026 14:45:27 GMT`, `Server: Apache`.
- `Link` header advertises the WordPress REST API at `/wp-json/` (and `/wp-json/wp/v2/pages/53`).
- `Set-Cookie: __wpdm_client` (secure; HttpOnly). Other headers: `Referrer-Policy: no-referrer-when-downgrade`, `Permissions-Policy`, `X-Endurance-Cache-Level: 0` and `X-nginx-cache: WordPress`, which hint at the hosting and caching stack.
- No `Strict-Transport-Security`, `Content-Security-Policy`, `X-Frame-Options` or `X-Content-Type-Options` header appeared in this response.

**Why it matters:** headers confirm WordPress, reveal caching software and show which browser protections are missing.

#### wafw00f: Web Application Firewall detection

**Command:** `wafw00f networkwalks.com` (Figure 6)

- wafw00f v2.4.2 checked `https://networkwalks.com` using 2 requests.
- Result: the site is behind **ModSecurity (SpiderLabs) WAF**.

**Why it matters:** knowing a WAF is present tells an attacker that simple attacks will likely be filtered, and the product name helps them research bypass methods. The detection is a fingerprint based on how the site responds, not proof of how the WAF is configured.

#### dnsrecon: DNS enumeration

**Command:** `dnsrecon -d networkwalks.com` (Figure 7)

- SOA and NS records point to `ns6135.hostgator.com` (50.87.144.87) and `ns6136.hostgator.com` (192.232.216.131).
- Both name servers disclosed their software version: BIND `"9.16.23-RH"`.
- A record: `networkwalks.com` = 192.232.216.135, matching nslookup and WhatWeb.
- DNSSEC query returned "No answer", consistent with WHOIS showing the zone as unsigned.
- 8 SRV records for `_autodiscover._tcp.networkwalks.com` pointing to `cpanelemaildiscovery.cpanel.net` on port 443 (184.94.203.x and 184.94.204.x addresses), which shows the domain uses cPanel email autodiscovery.
- The output did not list MX or TXT/SPF records. I did not assume they do not exist; they should be checked separately with `dig MX` and `dig TXT`.

**Why it matters:** DNS data maps the hosting provider, mail setup and DNS software without touching the website at all.

### 4.2 Maltego (W2-PM3)

I used Maltego Graph (Desktop) 4.14.0 to find email addresses related to `networkwalks.com`. I created a graph with the domain as a starting entity and ran the Utilities transforms **To Email address [From whois info]**, **To Emails @domain [Search Engine]**, **To Email Addresses [Search Engine]** and **To Email Addresses [PGP]** (Figure 8).

- Two email entities were returned: **`info@networkwalks.com`** and **`abuse@godaddy.com`**.
- Only `info@networkwalks.com` belongs to the target. `abuse@godaddy.com` is the registrar's abuse contact copied from the WHOIS record, so it says nothing about Networkwalks staff.
- `info@networkwalks.com` was also seen in the WhatWeb output, so two independent tools confirm it.
- The PGP key-server transform finished without adding new addresses. The transform log shows 21 credits used and 179 remaining.

**Why it matters:** email addresses are the usual first target for phishing, and Maltego shows how different sources link back to one domain.

### 4.3 theHarvester (W2-PM4)

I used theHarvester 4.11.1 in Kali Linux to gather emails, hosts and IP addresses for **`microsoft.com`**, the target set by the module.

- **Task 1:** Baidu source, limit 1000: `theHarvester -d microsoft.com -l 1000 -b baidu`. Result: no IPs, no emails, no people and no hosts found (Figure 9).
- **Task 2:** all sources, limit 50: `theHarvester -d microsoft.com -l 50 -b all` (Figures 10 and 10b). Many sources printed "Missing API key" errors (for example BeVigil, BuiltWith, Censys, Hunter, Intelx, GitHub) and returned nothing, but sources that need no key (such as crt.sh, CertSpotter, HackerTarget, RapidDNS and Hudson Rock) still produced results.
- **Results (second run, Figures 10b and 10c):** **3 emails** (`dotnet-docker-bot@microsoft.com`, `opencode@microsoft.com`, `secure@microsoft.com`), **158 IP addresses** (IPv4 and IPv6), **6 ASNs** (including AS8075, which is Microsoft) and **6 interesting URLs**, mostly Microsoft sign-in links with OAuth parameters. No people or LinkedIn profiles were found.
- The first run of the same command returned slightly different totals: 167 IPs, 7 ASNs, 12 interesting URLs and **1,360 hosts**. Totals differ between runs because the sources are live services and some time out or rate-limit. The host total is not visible in the screenshots.
- Many host names look internal or test-related (for example names containing `corp.microsoft.com`, `selfhost`, `ppe` or `test`). Certificate and DNS datasets commonly record such names even when the systems are not reachable from the internet.
- The Hudson Rock source also reported statistics about credentials exposed by info-stealer malware for the domain. I did not collect or use any credentials.
- The `-l 50` limit clearly did not cap the totals, which suggests it only applies to some sources. Some entries (for example `2Fblogs.microsoft.com`) are URL-encoding artifacts, not real hosts, so raw counts need cleaning before they are reported.

Baidu and the all-sources run gave very different results, which shows why source choice matters. A source returning nothing does not prove nothing is exposed: Baidu may have blocked the automated query or may not index the domain well (I did not test which). Adding free API keys to `/etc/theHarvester/api-keys.yaml` would let more sources respond.

### 4.4 Network Scanning with Zenmap (W2-PM5)

I installed Zenmap on my Windows PC and used the **Ping scan** profile (`nmap -sn`) to discover live hosts, then opened the **Topology** tab with the legend shown (Figure 11).

- Local addresses from `ipconfig` (Figure 12): Wi-Fi adapter **10.205.228.131**, subnet mask 255.255.255.0 (/24), default gateway 10.205.228.16, so my LAN subnet is **10.205.228.0/24**. A second adapter, "Ethernet 3" (VirtualBox host-only), has 192.168.56.1/24 and no gateway.
- First scan (Figure 11): `nmap -sn 192.168.56.1`, a single address. The topology showed 192.168.56.1 and 10.205.228.131, both addresses of my own PC (host-only adapter and Wi-Fi), so it found no other devices.
- Second scan: Ping scan of the whole LAN subnet, `nmap -sn 10.205.228.0/24` (Figure 13), run on 14 November 2026 at 13:04 (+0200) with Nmap 7.993. This network is my phone's mobile hotspot, which I own, so it is within scope. Result: 256 addresses scanned in 4.08 seconds, **2 hosts up**.

| Host | Device | MAC address |
|---|---|---|
| `10.205.228.16` | Default gateway (my phone's hotspot), latency 0.019 s (Figure 14) | `5A:E6:F3:5D:98:F7` (vendor "Unknown") |
| `10.205.228.131` | My own Windows PC (Figure 15) | Not shown: Nmap does not report the scanning machine's own MAC. It can be read with `ipconfig /all` |

- The gateway's MAC starts with 5A. A second hex digit of 2, 6, A or E marks a locally administered address, which suggests a private or randomized MAC and would explain why no vendor was identified.
- Topology (Figure 16): both hosts appear connected to localhost with dotted lines (no traceroute information). Topology saved as PDF. txt files and screenshots/zenmap_topology.pdf

**Why it matters:** a ping scan finds live devices and MAC addresses without scanning any ports, which is why every host shows 0 open ports and 0 scanned ports. That does not mean the hosts have no open ports, only that ports were not checked. On a larger network the same scan would list every connected device, which helps an attacker map targets and helps a defender spot unknown devices.

## 5. Risk Analysis / Impact

| # | Risk / Finding | Evidence / Observation | Potential Impact | Risk Level |
|---|---|---|---|---|
| 1 | Web technology and version information exposed | WhatWeb: WordPress 7.0.4, WordPress Download Manager 3.3.58, jQuery 3.7.1, Apache | Attackers can match exact versions against known vulnerability lists to decide what to test next | 🟠 **Medium** |
| 2 | WordPress REST API advertised | curl -I: the `Link` header points to `/wp-json/` and `/wp-json/wp/v2/pages/53` | Confirms WordPress and shows where content or user data may be enumerated if the API is not restricted | 🟢 **Low** |
| 3 | Common security headers not seen | curl -I response showed no HSTS, CSP, X-Frame-Options or X-Content-Type-Options header | Weaker browser-side protection against downgrade, clickjacking and content-sniffing attacks | 🟢 **Low** |
| 4 | DNS software version disclosed; DNSSEC unsigned | dnsrecon: both name servers reported BIND "9.16.23-RH". WHOIS shows DNSSEC unsigned and dnsrecon got no DNSSEC answer | Version banners help target DNS software; unsigned zones cannot be cryptographically validated | 🟢 **Low** |
| 5 | Name servers differ between registry and registrar | Registry record lists 2 HostGator name servers; registrar record also lists NS29/NS30.DOMAINCONTROL.COM | Leftover or mismatched name servers can point to stale DNS configuration worth verifying | 🟢 **Low** |
| 6 | Public email address exposed | `info@networkwalks.com` found by WhatWeb and by Maltego (search-engine transforms) | Expected for a contact mailbox, but it is a target for phishing and spam | 🟢 **Low** |
| 7 | Hosting infrastructure identifiable | IP 192.232.216.135 (nslookup, WhatWeb, dnsrecon); HostGator name servers; cPanel autodiscover SRV records | Helps an attacker profile the hosting provider and management stack | 🟢 **Low** |
| 8 | Local host discovery | Zenmap ping scan of 10.205.228.0/24 found 2 live hosts: the hotspot gateway 10.205.228.16 and my PC 10.205.228.131 | On a larger network the same scan would list every device, so unknown or unauthorized devices would show up | 🟢 **Low** |
| 9 | WAF product identifiable | wafw00f identified ModSecurity (SpiderLabs) in front of the site | A WAF is a good control, but knowing the product helps an attacker plan around it | 🟢 **Low** |

**Risk level key:** 🔴 Critical, 🟠 Medium, 🟢 Low (no Critical findings)

The risks above are observations from information gathering, not confirmed vulnerabilities. No exploitation or vulnerability validation was performed. A software version, IP address or DNS record does not by itself mean a system is vulnerable; further authorized testing would be required to confirm any weakness.

## 6. Recommendations

1. **Reduce technology and version disclosure.** Remove the WordPress generator tag, hide plugin version strings where possible, and keep WordPress core and plugins (including WordPress Download Manager) updated against current advisories.
2. **Restrict the WordPress REST API where it is not needed.** Limit anonymous access to endpoints that list users or content.
3. **Add standard security headers.** Enable Strict-Transport-Security, X-Content-Type-Options, a frame-blocking policy (X-Frame-Options or CSP `frame-ancestors`) and a stricter Referrer-Policy.
4. **Harden DNS.** Hide the BIND version banner, confirm the DNS software is patched (the "RH" build may carry backported fixes), and consider enabling DNSSEC.
5. **Reconcile name server records.** Confirm whether the DOMAINCONTROL.COM name servers listed at the registrar are still intended.
6. **Protect public mailboxes.** Check that SPF, DKIM and DMARC are in place for the domain and that spam filtering is active for `info@networkwalks.com`.
7. **Keep the WAF (ModSecurity) enabled and its rules up to date.** It already filters basic attacks, but it should not be the only defense, so keep patching the application behind it.
8. **Scan your own networks regularly and investigate unknown devices.** Keep network topology and device lists documented and up to date.
9. **Test only with authorization.** Reconnaissance and scanning should only be done on systems where written permission exists.

## 7. Conclusion

During Week 2 of my Cybersecurity & Ethical Hacking internship, I completed practical activities covering footprinting, OSINT and network scanning.

With the six Kali tools I learned what each one reveals: WHOIS shows registration and hosting details, WhatWeb and curl show the web technology, nslookup and dnsrecon show addresses and DNS setup, and wafw00f checks for a firewall. Maltego and theHarvester showed that email addresses and hosts can be found from public sources, and that results depend on the source: Baidu returned nothing, while the all-sources run returned 3 emails and over 150 IP addresses. Zenmap showed how hosts are discovered and drawn as a topology.

The most useful lesson was cross-checking: the same IP address and the same email address showed up in several tools, which makes a finding more trustworthy, and a result such as the Bootstrap version needed a second look before I could trust it. I also learned that a good report states what was run, what was seen, why it matters and what to do about it, without calling an observation a vulnerability.

Finally, I learned that reconnaissance and scanning must stay within an authorized scope. These activities were completed as part of an assigned educational cybersecurity lab.

## 8. Evidences Collected

**Figure 1 - `whois networkwalks.com` (registry record)**

![1_whois](screenshots/1_whois.png)

**Figure 2 - `whois networkwalks.com` (registrar record, GoDaddy)**

![2_whois_registrar](screenshots/2_whois_registrar.png)

**Figure 3 - `whatweb networkwalks.com`**

![3_whatweb](screenshots/3_whatweb.png)

**Figure 4 - `nslookup networkwalks.com`**

![4_nslookup](screenshots/4_nslookup.png)

**Figure 5 - `curl -I https://networkwalks.com`**

![5_curl](screenshots/5_curl.png)

**Figure 6 - `wafw00f networkwalks.com`**

![6_wafw00f](screenshots/6_wafw00f.png)

**Figure 7 - `dnsrecon -d networkwalks.com`**

![7_dnsrecon](screenshots/7_dnsrecon.png)

**Figure 8 - Maltego graph and transform output for `networkwalks.com`**

![8_maltego](screenshots/8_maltego.png)

**Figure 9 - theHarvester, `microsoft.com`, Baidu source, limit 1000**

![9_theharvester_baidu](screenshots/9_theharvester_baidu.png)

**Figure 10 - theHarvester, `microsoft.com`, all sources, limit 50 (start of run)**

![10_theharvester_all](screenshots/10_theharvester_all.png)

**Figure 10b - theHarvester, `microsoft.com`, all sources, results part 1 (ASNs, interesting URLs, start of IP list)**

![10b_theharvester_results_1](screenshots/10b_theharvester_results_1.png)

**Figure 10c - theHarvester, `microsoft.com`, all sources, results part 2 (end of IP list, emails)**

![10c_theharvester_results_2](screenshots/10c_theharvester_results_2.png)

**Figure 11 - Zenmap ping scan and topology with legend**

![11_zenmap](screenshots/11_zenmap.png)

**Figure 13 - Zenmap Nmap Output for `nmap -sn 10.205.228.0/24` (2 hosts up)**

![13_zenmap_subnet_output](screenshots/13_zenmap_subnet_output.png)

**Figure 14 - Zenmap Host Details for `10.205.228.16` (gateway, with MAC)**

![14_zenmap_host_16](screenshots/14_zenmap_host_16.png)

**Figure 15 - Zenmap Host Details for `10.205.228.131` (my PC)**

![15_zenmap_host_131](screenshots/15_zenmap_host_131.png)

**Figure 16 - Zenmap Topology tab after scanning `10.205.228.0/24`**

![16_zenmap_subnet_topology](screenshots/16_zenmap_subnet_topology.png)

---

-End-

## 👤 Author

**Kochiwe Anthony Tembo**
Cybersecurity intern, Networkwalks 

LinkedIn: <https://www.linkedin.com/in/kochiwe-tembo-0a52a4400>
GitHub: <https://github.com/Kochiweatembo>

## 📌 Project Information

**Program Name:** Cybersecurity Internship 2026 at Networkwalks | **Week:** 02 | **Repository:** GitHub
