<div align="center">

# 🛡 Named Extended ACL: One Host, One Server, One Service

### ACL Series · Lab 4 · Cisco Packet Tracer

![Cisco](https://img.shields.io/badge/Cisco-IOS-1BA0D7?style=for-the-badge&logo=cisco&logoColor=white)
![Packet Tracer](https://img.shields.io/badge/Packet%20Tracer-Lab-049fd9?style=for-the-badge)
![ACL](https://img.shields.io/badge/Extended-ACL-red?style=for-the-badge)
![Named](https://img.shields.io/badge/Named-ACL-purple?style=for-the-badge)
![Direction](https://img.shields.io/badge/Direction-Outbound-orange?style=for-the-badge)
![Level](https://img.shields.io/badge/Level-CCNA-green?style=for-the-badge)
![Status](https://img.shields.io/badge/Status-Verified-success?style=for-the-badge)

*Match the source AND the destination AND the protocol AND the port.*

</div>

---

## 📑 Table of Contents
1. [Overview](#-overview)
2. [Topology](#-topology)
3. [Addressing Table](#-addressing-table)
4. [Standard vs Extended ACL](#-standard-vs-extended-acl)
5. [Configuration](#-configuration)
6. [ACL Logic: Line by Line](#-acl-logic-line-by-line)
7. [Verification: Stage by Stage](#-verification-stage-by-stage)
8. [Test Matrix](#-test-matrix)
9. [Placement Logic](#-placement-logic-why-r1-serial-outbound)
10. [Named vs Numbered ACLs](#-named-vs-numbered-acls)
11. [Key Takeaways](#-key-takeaways)
12. [Troubleshooting Notes](#-troubleshooting-notes)
13. [Next Labs](#-next-labs-in-the-acl-series)
14. [Author](#-author)

---

## 🎯 Overview

Labs 1–3 used **Standard ACLs**, which only see the source IP. This lab introduces the **Extended ACL**, which can also match the **destination**, the **protocol** and the **port**.

**Goal:** Only PC1 (`192.168.1.3`) may open the web server at `200.100.100.2`. Every other host is blocked from the web service, but **all other traffic** (ping, Telnet and so on) must keep working.

| Item | Detail |
|---|---|
| ACL type | **Extended**, **named** (`cisco`) |
| Applied on | R1, Serial0/0/0 |
| Direction | **Outbound** |
| Protected service | HTTP (TCP 80) and HTTPS (TCP 443) on 200.100.100.2 |
| Only permitted web client | 192.168.1.3 (PC1) |
| Everything else | Permitted (`permit ip any any`) |

---

## 🗺 Topology

```
                  ACL OUT SIDE ──► (ip access-group cisco out)
                         1.0.0.0 (Serial WAN)
        1.1.1.1  ●━━━━━━━━━━━━━━⚡━━━━━━━━━━━━━━●  1.1.1.2
              ┌──────┐                          ┌──────┐
              │  R1  │                          │  R2  │
              │ 1941 │                          │ 1941 │
              └──┬───┘                          └──┬───┘
        192.168.1.1/24                       200.100.100.1/24
                 │                                 │
            ┌────┴────┐                       ┌────┴────┐
            │ Switch0 │                       │ Switch1 │
            └─┬──┬──┬─┘                       └─┬──┬──┬─┘
              │  │  │                           │  │  │
           PC0  PC1  PC2                       PC    PC   Server-PT
      192.168.1.2 .1.3 .1.4              200.100.100.4 .100.3  200.100.100.2
       (🚫 web)  (✅ web) (🚫 web)                              🌐 Web server
```

---

## 📋 Addressing Table

| Device | Interface | IP Address | Role |
|---|---|---|---|
| R1 (1941) | LAN interface | 192.168.1.1/24 | Gateway, LAN 1 |
| R1 (1941) | Serial0/0/0 | 1.1.1.1 | WAN link (**ACL applied here**) |
| R2 (1941) | Serial | 1.1.1.2 | WAN link |
| R2 (1941) | LAN interface | 200.100.100.1/24 | Gateway, LAN 2 |
| PC0 | NIC | 192.168.1.2/24 | LAN 1, 🚫 web denied |
| **PC1** | NIC | **192.168.1.3/24** | ✅ **Only web-permitted host** |
| PC2 | NIC | 192.168.1.4/24 | LAN 1, 🚫 web denied |
| PC | NIC | 200.100.100.4/24 | LAN 2 |
| PC | NIC | 200.100.100.3/24 | LAN 2 |
| **Server-PT** | NIC | **200.100.100.2/24** | 🌐 **Protected web server** |

---

## 🔄 Standard vs Extended ACL

| | Standard ACL (Labs 1–3) | **Extended ACL (this lab)** |
|---|---|---|
| Matches on | Source IP only | **Source + destination + protocol + port** |
| Number ranges | 1–99, 1300–1999 | 100–199, 2000–2699 |
| Granularity | Whole host | **One service on one host** |
| Best placement | Close to the **destination** | Close to the **source** |
| Example | `permit host 192.168.1.3` | `permit tcp host 192.168.1.3 host 200.100.100.2 eq 80` |

---

## ⚙️ Configuration

### R1: remove the leftover ACL from Lab 1 (if still present)
> A leftover inbound ACL on the serial interface would block the server's replies.

```
R1(config)# interface serial 0/0/0
R1(config-if)# no ip access-group 1 in
R1(config-if)# exit
```

### R1: the Named Extended ACL

```
R1(config)# ip access-list extended cisco
R1(config-ext-nacl)# permit tcp host 192.168.1.3 host 200.100.100.2 eq 80
R1(config-ext-nacl)# deny tcp any host 200.100.100.2 eq 80
R1(config-ext-nacl)# deny tcp any host 200.100.100.2 eq 443
R1(config-ext-nacl)# permit ip any any
R1(config-ext-nacl)# exit

R1(config)# interface serial 0/0/0
R1(config-if)# ip access-group cisco out
R1(config-if)# end
```

### Routing between the two LANs
> Replace this block with the routing you actually used.

```
R1(config)# ip route 200.100.100.0 255.255.255.0 1.1.1.2
R2(config)# ip route 192.168.1.0   255.255.255.0 1.1.1.1
```

### Config breakdown

| Command | Purpose |
|---|---|
| `ip access-list extended cisco` | Create a **named** extended ACL called `cisco` |
| `permit tcp host 192.168.1.3 host 200.100.100.2 eq 80` | PC1 may open HTTP on the web server |
| `deny tcp any host 200.100.100.2 eq 80` | Everyone else is denied HTTP to that server |
| `deny tcp any host 200.100.100.2 eq 443` | Everyone is denied HTTPS to that server |
| `permit ip any any` | Allow all other traffic (ping, Telnet and so on) |
| `ip access-group cisco out` | Check packets **leaving** R1 through Serial0/0/0 |

---

## 🧩 ACL Logic: Line by Line

| Seq | Rule | Who it matches | Action |
|---|---|---|---|
| 10 | `permit tcp host 192.168.1.3 host 200.100.100.2 eq 80` | PC1's web requests to the server | ✅ Permit |
| 20 | `deny tcp any host 200.100.100.2 eq 80` | **Everyone else's** web requests (PC0, PC2) | ❌ Deny |
| 30 | `deny tcp any host 200.100.100.2 eq 443` | Anyone's HTTPS requests, **including PC1** | ❌ Deny |
| 40 | `permit ip any any` | Everything else (ICMP, Telnet and so on) | ✅ Permit |

```
 PC1 ──HTTP──► line 10 matches ──► ✅ page loads
 PC0 ──HTTP──► line 10 no ► line 20 matches ──► ❌ timeout
 PC0 ──ping──► 10 no ► 20 no ► 30 no ► line 40 matches ──► ✅
```

### Order matters

| Config | Result |
|---|---|
| Permit PC1 **before** the deny | ✅ PC1 gets in, others blocked |
| Deny **before** the permit | ❌ PC1 is blocked too, because the deny matches first |
| No `permit ip any any` at the end | ❌ The implicit `deny any` kills ping and Telnet too |

---

## ✅ Verification: Stage by Stage

### 🔴 Stage 1: PC0 (192.168.1.2) → Web browser → BLOCKED

```
URL: http://200.100.100.2
→ Request Timeout
```

**Annotations**
- Source 192.168.1.2 does **not** match line 10, so it falls to line 20 (`deny tcp any ... eq 80`).
- The browser shows a **timeout** because the HTTP request never completes.

---

### 🔴 Stage 2: PC2 (192.168.1.4) → Web browser → BLOCKED

```
URL: http://200.100.100.2
→ Request Timeout
```

**Annotation:** The same result as PC0. Only the exact host in line 10 gets through.

---

### 🟢 Stage 3: PC1 (192.168.1.3) → Web browser → ALLOWED

```
URL: http://200.100.100.2
→ Page loads:
   "Maaz Khan"
   "Welcome to Cisco Packet Tracer. Opening doors to new opportunities. Mind Wide Open."
   Quick Links: A small page · Copyrights · Image page · Image
```

**Annotation:** Source 192.168.1.3 + destination 200.100.100.2 + TCP 80 matches **line 10** exactly, so the packet is permitted. This is the Extended ACL working: **source AND destination AND port**.

---

### 🟢 Stage 4: PC0 → Ping the same server → ALLOWED

```
C:\> ping 200.100.100.2

Reply from 200.100.100.2: bytes=32 time=1ms TTL=126  (x4)

Packets: Sent = 4, Received = 4, Lost = 0 (0% loss)
Minimum = 1ms, Maximum = 1ms, Average = 1ms
```

**Annotations**
- The **same PC** that cannot browse the server can still ping it. ICMP is not TCP 80, so it skips lines 10–30 and hits `permit ip any any`.
- **TTL = 126**: 128 minus 2 routers.

---

### 🟢 Stage 5: PC0 → Telnet to R2 → ALLOWED

```
C:\> telnet 1.1.1.2
Trying 1.1.1.2 ...Open

User Access Verification

Password:
R2>en
Password:
R2#
```

**Annotation:** Telnet is TCP 23, so it is not matched by the web deny lines. It passes through line 40. **Only web access to that server is restricted.**

---

### 📊 Stage 6: ACL hit counters (run on R1)

```
R1# show access-lists
Extended IP access list cisco
    10 permit tcp host 192.168.1.3 host 200.100.100.2 eq www (10 match(es))
    20 deny tcp any host 200.100.100.2 eq www (87 match(es))
    30 deny tcp any host 200.100.100.2 eq 443
    40 permit ip any any (154 match(es))
```

**Annotations**

| Line | Matches | What it shows |
|---|---|---|
| 10 | 10 | PC1's web sessions (permitted) |
| 20 | **87** | PC0 and PC2 browser attempts and retries (denied) |
| 30 | none | **No HTTPS test was run yet**, so nothing hit this line |
| 40 | 154 | Ping, Telnet and all other permitted traffic |

> IOS displays `eq 80` as `eq www`. They are the same.

---

## 🧪 Test Matrix

| # | Source | Destination / Service | Result | Matched line |
|---|---|---|---|---|
| 1 | PC1 (192.168.1.3) | Web server, HTTP | ✅ Page loads | 10 |
| 2 | PC0 (192.168.1.2) | Web server, HTTP | ❌ Request Timeout | 20 |
| 3 | PC2 (192.168.1.4) | Web server, HTTP | ❌ Request Timeout | 20 |
| 4 | PC0 (192.168.1.2) | Web server, ping | ✅ 4/4, TTL 126 | 40 |
| 5 | PC0 (192.168.1.2) | R2 (1.1.1.2), Telnet | ✅ R2# | 40 |
| 6 | Any PC, including PC1 | Web server, HTTPS | ❌ Expected | 30 |

> Row 6 follows from the ACL logic, because no permit exists for 443. Test it with `https://200.100.100.2` from PC1 and the counter on line 30 will increase.

---

## 📍 Placement Logic: Why R1 Serial Outbound?

| ACL type | Where to place it | Why |
|---|---|---|
| Standard | Close to the **destination** | It only sees the source, so it would over-block near the source |
| **Extended** | Close to the **source** | It can name the exact destination and port, so unwanted traffic can be stopped **before it crosses the WAN** |

```
 PC0 ──HTTP──► R1 ──► [ACL cisco OUT on Se0/0/0] ──✖ dropped here
                          (never reaches the WAN link or R2)
 PC1 ──HTTP──► R1 ──► [ACL cisco OUT on Se0/0/0] ──✔──► R2 ──► Web server
```

- One ACL on R1's serial interface covers **every** host in LAN 1 at a single point.
- The server's **reply** comes back **inbound** on Serial0/0/0. This ACL is `out` only, so replies are not filtered.

---

## 🏷 Named vs Numbered ACLs

| | Numbered | **Named (this lab)** |
|---|---|---|
| Create | `access-list 101 permit ...` | `ip access-list extended cisco` |
| Readability | Just a number | **Descriptive name** |
| Editing a line | Hard, often rebuild the list | **Insert, delete or replace by sequence number** |
| Sequence numbers | Auto, hidden | **10, 20, 30, 40** (visible, editable) |

Example of editing later without rebuilding:

```
R1(config)# ip access-list extended cisco
R1(config-ext-nacl)# no 30
R1(config-ext-nacl)# 25 deny tcp any host 200.100.100.2 eq 22
```

---

## 💡 Key Takeaways

| # | Takeaway |
|---|---|
| 1 | Extended ACLs match **source, destination, protocol and port** |
| 2 | Place Extended ACLs close to the **source**, and Standard ACLs close to the destination |
| 3 | Named ACLs use **sequence numbers**, so editing is easy |
| 4 | Specific permit first, then the denies, then `permit ip any any` |
| 5 | Without the final permit, the implicit `deny any` breaks ping and Telnet |
| 6 | Deny HTTPS (443) as well as HTTP (80), or the service stays reachable |
| 7 | One host can be blocked from a service but still ping and Telnet |
| 8 | `show access-lists` counters show which line handled which traffic |

---

## 🛠 Troubleshooting Notes

| Symptom | Check |
|---|---|
| PC1 cannot browse either | Is the permit line (10) **above** the deny? Is the server's HTTP service on? |
| Everyone can browse | Is the ACL applied to Serial0/0/0 in the `out` direction? |
| Ping and Telnet also fail | Is `permit ip any any` missing at the end? |
| Nothing works at all | Is an old ACL still on the serial interface? |
| Counters stay at 0 | Is the traffic really leaving R1 through that interface? |

---

## 🚀 Next Labs in the ACL Series

- [x] Lab 1: Standard ACL, whitelist a host ✅
- [x] Lab 2: Standard ACL, blacklist a host with placement logic ✅
- [x] Lab 3: Standard ACL, restrict Telnet (VTY) to one admin PC ✅
- [x] Lab 4: Named Extended ACL, one host to one web server ✅
- [ ] Editing a named ACL with sequence numbers
- [ ] Extended ACL with ranges, ICMP and `established`
- [ ] Reflexive / time-based ACLs

---

## 👨‍💻 Author

**Maaz Khan**
CCNA-certified Network & NOC Engineer · Lower Dir, KPK, Pakistan

[![LinkedIn](https://img.shields.io/badge/LinkedIn-maazkhanms-0A66C2?style=for-the-badge&logo=linkedin)](https://www.linkedin.com/in/maazkhanms)
[![GitHub](https://img.shields.io/badge/GitHub-maazkhanms-181717?style=for-the-badge&logo=github)](https://github.com/maazkhanms)

⭐ If this lab helped you, give the repo a star!
