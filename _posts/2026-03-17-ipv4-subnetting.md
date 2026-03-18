---
title: "IPv4 Subnetting Explained Simply"
date: 2026-03-17 00:00:00 +0000
categories: [Networking, Beginner]
tags: [ipv4, subnetting, ccna, networking, binary]
description: A beginner's guide to IP addresses, binary, subnet masks, CIDR, and step-by-step subnetting calculations. If you're studying for CCNA or just trying to finally get it — this one's for you.
---

I'll be honest — subnetting broke my brain for weeks when I first started studying for CCNA. I'd stare at numbers like `192.168.1.0/26` and feel completely lost. Everyone online either went too fast or drowned me in formulas without explaining *why* any of it works.

So this is the post I wished existed. No shortcuts, no memorisation tricks — just the actual logic, built from the ground up. By the end you'll be able to subnet any network by hand, and more importantly, you'll *understand* what you're doing.

---

## What Even is an IP Address?

Think of the postal system. Every house has a unique address so the postal service knows exactly where to deliver a letter. A network works the same way — every device needs a unique address so data knows where to go. That address is its **IP address**.

IPv4 addresses are **32 bits long**, written as four numbers separated by dots — called **octets** because each one is 8 bits. Each octet can be any value from 0 to 255.

```
192   .   168   .   1   .   100
octet 1   octet 2  octet 3  octet 4
 8 bits    8 bits   8 bits   8 bits  =  32 bits total
```

Four numbers, each 0–255, giving roughly **4.3 billion unique addresses**. That felt infinite in the 1980s. Today we've nearly exhausted them — which is why IPv6 exists, but that's a story for another post.

---

## Binary — The Only Language Computers Speak

Here's the thing nobody tells beginners early enough: **your router doesn't think in decimal**. It thinks in binary — 1s and 0s — because its circuits are either on or off. Every IP address, every routing decision, every subnet calculation happens in binary.

Binary is base-2. Each position is worth 2× the one to its right:

| Position | 128 | 64 | 32 | 16 | 8 | 4 | 2 | 1 |
|----------|-----|----|----|----|----|---|---|---|
| Binary   |  1  |  1 |  0 |  0 |  0 | 1 | 0 | 0 |
| Value    | 128 | 64 |  – |  – |  – | 4 | – | – |

**128 + 64 + 4 = 196**

To convert any decimal to binary: divide by 2 repeatedly, read remainders bottom to top. To go binary → decimal: add up the place values wherever there's a `1`.

Each octet in an IP address is just a decimal number stored as 8 bits. When your router looks at `192.168.1.100`, it sees four groups of 8 bits — and all its decisions happen on those bits directly.

---

## IP Classes — The Original Design

When IPv4 was first designed, addresses were divided into classes based on the first octet:

| Class | First Octet | Scale |
|-------|-------------|-------|
| A | 1–126 | Huge — ~16M hosts/network |
| B | 128–191 | Medium — ~65K hosts |
| C | 192–223 | Small — 254 hosts |
| D | 224–239 | Multicast only |
| E | 240–255 | Experimental |

> **Note:** `127.x.x.x` is reserved for **loopback** — when a device communicates with itself. Try pinging `127.0.0.1` — it'll always respond, because you're literally talking to your own network stack.

Classes are mostly historical now. Modern networks use CIDR (covered below), which is far more flexible. But you'll still see class-based language in documentation and CCNA material — so it's worth knowing.

---

## Private vs. Public Addresses

Your device's IP right now is almost certainly *not* visible on the internet. Three ranges are permanently reserved for private networks:

| Range | Addresses | Common Use |
|-------|-----------|------------|
| `10.0.0.0/8` | ~16.7 million | Large enterprises, data centres |
| `172.16.0.0/12` | ~1 million | Medium organisations |
| `192.168.0.0/16` | ~65K | Your home Wi-Fi — right now |

Your home router gives local devices addresses in the `192.168.x.x` range, then uses **NAT** (Network Address Translation) to represent all of them behind a single public IP. Think of it like an apartment building — hundreds of flats, one street address.

---

## Subnet Masks — Drawing the Boundary

Every IP address has two parts: the **network** portion (which network?) and the **host** portion (which device on that network?). A subnet mask is what tells you where that boundary falls.

A subnet mask is always a solid block of 1s followed by a solid block of 0s. The 1s cover the network bits; the 0s expose the host bits:

```
255.255.255.0  =  11111111.11111111.11111111.00000000  (/24)
255.255.0.0    =  11111111.11111111.00000000.00000000  (/16)
255.0.0.0      =  11111111.00000000.00000000.00000000  (/8)
```

More 1s in the mask = smaller network, fewer hosts. Fewer 1s = larger network, more hosts. This trade-off is the entire point of subnetting.

---

## The AND Operation — How Routers Actually Think

When your computer sends a packet, it first asks: *is this destination on my network, or do I send it to the router?* To answer, it does one operation: **bitwise AND** between the destination IP and the subnet mask.

AND is simple — it only outputs `1` if *both* inputs are `1`:

```
1 AND 1 = 1
1 AND 0 = 0
0 AND 0 = 0
```

Full example with `192.168.1.100` and mask `255.255.255.0`:

```
IP:     11000000.10101000.00000001.01100100
Mask:   11111111.11111111.11111111.00000000
        ────────────────────────────────────
Result: 11000000.10101000.00000001.00000000  =  192.168.1.0
```

**This is the one thing to remember from this entire post.** Every routing decision — in your home router, a Cisco switch, or an internet backbone — is this exact AND operation. The result is the **network address**. Master this and you have the key to all of networking.

---

## CIDR Notation — The Shorthand

Writing `255.255.255.0` every time is tedious. CIDR notation uses a slash followed by the number of 1-bits in the mask:

```
192.168.1.0/24   →   255.255.255.0    (24 ones, 8 zeros)
10.0.0.0/8       →   255.0.0.0        (8 ones, 24 zeros)
172.16.0.0/16    →   255.255.0.0      (16 ones, 16 zeros)
```

Quick reference for the most common prefix lengths:

| CIDR | Subnet Mask | Usable Hosts | Use Case |
|------|-------------|--------------|----------|
| `/8` | 255.0.0.0 | 16,777,214 | ISP / large org |
| `/16` | 255.255.0.0 | 65,534 | Campus / enterprise |
| `/24` | 255.255.255.0 | 254 | Office LAN (most common) |
| `/25` | 255.255.255.128 | 126 | Half of a /24 |
| `/30` | 255.255.255.252 | 2 | Point-to-point links |
| `/32` | 255.255.255.255 | 1 (host route) | Loopback / static host |

The number after the slash tells you everything. More bits = smaller network, fewer hosts. Fewer bits = larger network, more hosts.

---

## Worked Examples

Theory is fine. But subnetting only clicks when you work through real problems.

### Example 1 — `192.168.1.0/26`

**Step 1:** `/26` means 26 network bits, 6 host bits. The last octet splits at bit 6: `11000000` = 192. Subnet mask = `255.255.255.192`.

**Step 2:** Block size = 2⁶ = **64**. Networks increment by 64 in the last octet: .0, .64, .128, .192.

**Step 3:** Calculate the four key values:

```
Network address:    192.168.1.0
Broadcast address:  192.168.1.63
Usable host range:  192.168.1.1 – 192.168.1.62
Usable hosts:       62  (2⁶ − 2)
```

---

### Example 2 — `10.0.0.0/22`

**Step 1:** `/22` means 22 network bits, 10 host bits. The boundary falls in the *third* octet. Mask third octet: `11111100` = 252. Full mask = `255.255.252.0`.

**Step 2:** Block size = 2¹⁰ = **1024** addresses. Networks increment by 4 in the *third* octet: `10.0.0.0`, `10.0.4.0`, `10.0.8.0`...

**Step 3:**

```
Network address:    10.0.0.0
Broadcast address:  10.0.3.255
Usable host range:  10.0.0.1 – 10.0.3.254
Usable hosts:       1,022  (2¹⁰ − 2)
```

---

## Wildcard Masks — The Inverse

Cisco ACLs and OSPF use **wildcard masks** — the bitwise inverse of a subnet mask. The rule is simple: `0` = "this bit must match exactly", `1` = "any value is fine."

To get a wildcard mask: subtract the subnet mask from `255.255.255.255`.

```
Subnet mask:   255.255.255.0    (/24)
Wildcard mask:   0.0.0.255      (NOT of the subnet mask)
Meaning:       Match 192.168.1.0 → 192.168.1.255  (entire /24 range)
```

In a Cisco ACL it looks like this:

```cisco
! Permit all traffic from 192.168.1.x
access-list 10 permit 192.168.1.0 0.0.0.255

! Permit only the host 10.10.10.5
access-list 10 permit 10.10.10.5 0.0.0.0
```

---

## Putting It All Together

When your computer sends a packet to `192.168.1.55`, here is the exact decision chain — every time, at wire speed:

```
Step 1 — AND destination with my subnet mask:
  192.168.1.55 AND 255.255.255.0 = 192.168.1.0

Step 2 — Does result match my network address?
  My network: 192.168.1.0  ✓ Match → send directly

Step 3 — No match? Send to default gateway (router)
  Router repeats this vs. its routing table — longest prefix wins
```

> **The whole point of subnetting** is to break a large address space into smaller, manageable segments — reducing broadcast traffic, improving security, and making networks easier to manage. Every subnet has one network address (first), one broadcast address (last), and everything in between is usable by hosts.

That's the foundation. Master these concepts and you'll be able to read any network diagram, understand any routing table, and write Cisco ACLs with confidence. Everything else — VLSM, OSPF, BGP — builds on exactly this.

---

## Practice Challenge

Given: **172.16.5.0/27**

1. What is the subnet mask?
2. What is the block size?
3. What is the network address?
4. What is the broadcast address?
5. What is the usable host range?

<details>
<summary>Reveal answer</summary>

```
Subnet mask:       255.255.255.224
Block size:        32 addresses (2⁵)
Network address:   172.16.5.0
Broadcast address: 172.16.5.31
Usable host range: 172.16.5.1 – 172.16.5.30
Usable hosts:      30  (2⁵ − 2)
```

Key steps: `/27` = 5 host bits → mask last octet is `11100000` = 224. Block size = 2⁵ = 32. Network .0, broadcast .31, hosts .1–.30.

</details>

---

*Written by Tsotne · tsotne.blog · Post #001*
