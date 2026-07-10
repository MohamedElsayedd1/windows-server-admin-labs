# 🌍 Lecture: DNS — Domain Name System

**Topic:** DNS — Concepts, Resolution Workflow, Records & Enterprise Configuration  
**Platform:** Windows Server 2016 / 2019  
**Difficulty:** Beginner–Intermediate

---

## 📋 Table of Contents

1. [What is DNS?](#1-what-is-dns)
2. [Before DNS — The Hosts File](#2-before-dns--the-hosts-file)
3. [DNS Records](#3-dns-records)
4. [DNS Resolution Process](#4-dns-resolution-process)
5. [Authoritative vs Non-Authoritative Answers](#5-authoritative-vs-non-authoritative-answers)
6. [DNS Caching](#6-dns-caching)
7. [DNS in Enterprise Networks](#7-dns-in-enterprise-networks)
8. [DNS Tools & Commands](#8-dns-tools--commands)
9. [Key Terms Glossary](#9-key-terms-glossary)

---

## 1. What is DNS?

**Domain Name System (DNS)** is the protocol that translates human-readable domain names into IP addresses — and vice versa — because networks only understand IP addresses, not names.

DNS is considered the **heartbeat of any network**. Every action that involves a name (opening a website, sending an email, logging into a domain) depends on DNS resolving that name to an address first.

| Function | Description |
|---|---|
| **Forward Lookup** | Domain name → IP address (e.g., `example.com` → `93.184.216.34`) |
| **Reverse Lookup** | IP address → domain name (e.g., `192.168.1.2` → `pdc.dc.local`) |
| **Resource Locator** | Directs users to websites, servers, email services, and internal resources |

---

## 2. Before DNS — The Hosts File

Before DNS existed, devices relied on a **hosts file** — a local, manually maintained text file mapping hostnames to IP addresses.

- **Location:** `C:\Windows\System32\drivers\etc\hosts`
- **Format:**

```
192.168.1.2    pdc.dc.local
192.168.1.10   webserver.dc.local
```

### Why it failed at scale

- Had to be manually updated on every device
- Error-prone and inconsistent across machines
- Completely impractical for large or growing networks

**DNS centralized these mappings** into servers hosting **zones**, making name resolution automatic, consistent, and scalable across entire networks.

---

## 3. DNS Records

DNS zones store **resource records** — each record type serves a specific purpose:

| Record Type | Purpose | Example |
|---|---|---|
| **A** | Maps a hostname to an IPv4 address | `webserver.dc.local → 192.168.1.10` |
| **AAAA** | Maps a hostname to an IPv6 address | `webserver.dc.local → fe80::1` |
| **MX** | Specifies the mail server for a domain | Email to `user@example.com` → mail server |
| **SRV** | Locates a service within a network | Kerberos authentication server location |
| **PTR** | Reverse lookup — IP to hostname | `192.168.1.2 → pdc.dc.local` |
| **CNAME** | Alias — maps one name to another | `www.dc.local → webserver.dc.local` |

### MX Record example

When sending an email to `user@example.com`, the sending mail server queries DNS for the **MX record** of `example.com` to discover which mail server should receive the message.

### SRV Record example

When a Windows client joins a domain, it queries DNS for **SRV records** to locate the Kerberos authentication server and domain controller for the domain.

---

## 4. DNS Resolution Process

When a client needs to resolve a domain name, the following steps occur in order — each step only proceeds if the previous one fails:

```
┌─────────────────────────────────────────────────────────────────┐
│                    DNS Resolution Flow                          │
│                                                                 │
│  1. LOCAL HOSTS FILE       C:\Windows\System32\drivers\etc\hosts│
│          ↓ (not found)                                          │
│  2. LOCAL DNS CACHE        ipconfig /displaydns                 │
│          ↓ (not found)                                          │
│  3. PREFERRED DNS SERVER   (e.g., 192.168.1.2)                  │
│          ↓ (not found / unreachable)                            │
│  4. ALTERNATE DNS SERVER   (e.g., 192.168.1.3)                  │
│          ↓ (not found)                                          │
│  5. CONDITIONAL FORWARDER  (specific domain → specific server)  │
│          ↓ (no match)                                           │
│  6. FORWARDER              (e.g., 8.8.8.8 / 1.1.1.1)           │
│          ↓ (not configured)                                     │
│  7. ROOT HINTS             (13 global root DNS servers)         │
└─────────────────────────────────────────────────────────────────┘
```

### Step-by-step breakdown

**Step 1 — Local hosts file**
The client checks its local hosts file first. If a matching entry exists, resolution stops here — no DNS query is sent.

**Step 2 — Local DNS cache**
The client checks its local DNS resolver cache for a previously resolved answer. If found and still within TTL, resolution stops here.

**Step 3 — Preferred DNS server**
The client sends a query to its configured **preferred DNS server**. This is the primary path for all DNS queries.

**Step 4 — Alternate DNS server**
The alternate DNS server is only contacted if the preferred server is **unreachable or times out** — not if it simply returns "not found."

**Step 5 — Conditional forwarders**
If the DNS server has a **conditional forwarder** configured for the queried domain (e.g., all queries for `zahdi.net` → `10.0.0.5`), it forwards the query to that specific server.

**Step 6 — Forwarders**
If no conditional forwarder matches, the DNS server forwards the query to a configured external DNS server such as:
- Google DNS: `8.8.8.8` / `8.8.4.4`
- Cloudflare: `1.1.1.1`
- ISP-provided DNS

**Step 7 — Root hints**
If no forwarders are configured, the DNS server uses **root hints** — a preconfigured list of the 13 global root DNS servers — to begin walking down the DNS hierarchy (root → TLD → authoritative server).

---

## 5. Authoritative vs Non-Authoritative Answers

| Answer Type | Description |
|---|---|
| **Authoritative** | The responding DNS server **owns the zone** for the queried domain — the answer is definitive |
| **Non-Authoritative** | The DNS server **obtained the answer** from another server via recursive query or cache — may be slightly stale |

When using `nslookup`, the response indicates whether the answer is authoritative, helping verify whether the correct DNS server is being reached.

---

## 6. DNS Caching

DNS responses are cached at both the **client** and **server** level to reduce query traffic and improve speed.

- Cache entries are stored for a duration defined by the record's **TTL (Time To Live)** — typically around **1 hour**
- After TTL expires, the cache entry is discarded and a fresh query is made
- Stale cache entries can cause connectivity issues after DNS changes — use `ipconfig /flushdns` to clear them

### Cache lifecycle

```
Query resolved → Cached with TTL → TTL expires → Entry removed → Next query hits DNS again
```

---

## 7. DNS in Enterprise Networks

In enterprise environments, DNS requires careful planning beyond just pointing to `8.8.8.8`.

### Why public DNS breaks domain environments

Setting a domain-joined machine's DNS to a public server (e.g., Google `8.8.8.8`) **instead of the internal DNS server** causes:

- Failure to resolve internal hostnames (`pdc.dc.local`, `fileserver.dc.local`)
- Inability to authenticate against domain controllers (SRV record lookups fail)
- Loss of access to internal applications and shares

### Recommended enterprise DNS architecture

| Layer | Purpose |
|---|---|
| **Internal DNS server** | Hosts zones for internal domains (`dc.local`, `company.local`) |
| **Conditional forwarders** | Route queries for partner/external domains to their specific DNS servers |
| **Forwarders** | Send unresolved external queries to `8.8.8.8` or ISP DNS |
| **Root hints** | Fallback if no forwarders are configured |

### DNS Firewall

Some security appliances include a **DNS firewall module** that monitors, filters, and logs DNS traffic — protecting against DNS-based attacks (tunneling, hijacking, exfiltration).

---

## 8. DNS Tools & Commands

```cmd
nslookup <hostname>             # Query DNS to resolve a name — shows server used and result
nslookup <hostname> <dns-ip>    # Query a specific DNS server directly

ipconfig /displaydns            # Display the current local DNS resolver cache
ipconfig /flushdns              # Clear the local DNS cache (force fresh resolution)

ping <hostname>                 # Tests name resolution + connectivity in one step
```

### nslookup examples

```cmd
nslookup pdc.dc.local           # Resolve internal hostname
nslookup google.com 8.8.8.8     # Query Google DNS directly
nslookup -type=MX example.com   # Look up MX record for a domain
nslookup -type=SRV _kerberos._tcp.dc.local   # Look up Kerberos SRV record
```

---

## 9. Key Terms Glossary

| Term | Definition |
|---|---|
| **DNS** | Domain Name System — translates names to IPs and vice versa |
| **Zone** | A portion of the DNS namespace managed by a specific server, containing DNS records |
| **Hosts file** | Local file mapping hostnames to IPs — predecessor to DNS |
| **Forward Lookup** | Resolving a domain name to an IP address |
| **Reverse Lookup** | Resolving an IP address to a domain name |
| **A Record** | DNS record mapping a hostname to an IPv4 address |
| **MX Record** | DNS record identifying the mail server for a domain |
| **SRV Record** | DNS record locating a specific service within a network |
| **Conditional Forwarder** | Forwards DNS queries for a specific domain to a designated server |
| **Forwarder** | External DNS server used when the local server cannot resolve a query |
| **Root Hints** | Preconfigured list of 13 global root DNS servers used as last-resort resolution |
| **TTL** | Time To Live — duration a cached DNS record remains valid |
| **Authoritative Answer** | Response from the DNS server that owns the zone |
| **Non-Authoritative Answer** | Response obtained through recursive query or cache |
| **nslookup** | Command-line tool for manually querying DNS servers |
| **`ipconfig /flushdns`** | Clears the local DNS resolver cache |

---

## 👨‍💻 Author

> Lecture notes prepared for Windows Server administration coursework.  
> Topic: DNS — Domain Name System, Resolution & Enterprise Configuration.
