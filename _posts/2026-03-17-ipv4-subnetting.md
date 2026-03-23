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

## Why Does Subnetting Exist?

Imagine a company with 1,000 devices — servers, laptops, printers, phones — all on one flat network. Every time any device sends a broadcast, every other device has to process it. Traffic is noisy, troubleshooting is a nightmare, and a single compromised device can talk directly to anything else.

**Subnetting** is the solution. You take a large block of addresses and divide it into smaller, isolated networks. Each subnet is its own contained space. Traffic stays local unless it needs to cross a router. Security boundaries become real. It's the difference between one big open office floor and a building with proper rooms and locked doors.

That's the *why*. Now let's build the *how* from scratch.

---

## What Even is an IP Address?

Think of the postal system. Every house has a unique address so letters are delivered to exactly the right place. Networks work the same way — every device needs a unique address so data knows where to go. That address is its **IP address**.

One quick distinction worth knowing: there are actually *two* kinds of addresses at play when data moves across a network.

- **IP address** — a *logical* address assigned by software. It can change. It identifies where a device sits in the network topology. This operates at **Layer 3** of the network model.
- **MAC address** — a *physical* address burned into the hardware of your network card. It identifies the device itself, not its location. This operates at **Layer 2**.

The postal analogy maps cleanly: your **IP address** is like your home address (which changes when you move), and your **MAC address** is like your passport number (permanently tied to you, the person). IP gets the data to the right neighbourhood; MAC gets it to the exact door. We could go much deeper on this — and we will in a future post — but for now, IP = logical = Layer 3 is all you need.

IPv4 addresses are **32 bits long**, written as four numbers separated by dots — called **octets** because each one is 8 bits. Each octet ranges from 0 to 255, giving roughly **4.3 billion unique addresses**.

---

## Binary — The Only Language That Matters

Here's what nobody tells beginners soon enough: **your router doesn't think in decimal**. It thinks in binary. Every IP address, every routing decision, every subnet calculation happens in binary. If you skip this section, nothing later will make real sense.

Binary is **base-2**. Normal decimal is base-10 — each position is worth 10× the one to its right (ones, tens, hundreds…). Binary works the same way, but with 2× instead of 10×.

In an 8-bit octet, the positions are worth:

```
Position:   8    7    6    5    4    3    2    1
Value:     128   64   32   16    8    4    2    1
```

Each value is exactly **double** the one to its right. That's it — that's the whole system. To read a binary number, add up the position values wherever there's a `1`.

So `10110000` means: 128 + 32 + 16 = **176**. Try it below — click any bit to flip it and watch the decimal update live:

<style>
/* Responsive & Theme-Adaptive styles for Chirpy */
.widget{background:var(--card-bg, rgba(128,128,128,0.05));border:1px solid var(--border-color, rgba(128,128,128,0.2));border-radius:12px;padding:1.5rem;margin:1.5rem 0;font-family:var(--font-family-monospace, monospace);color:var(--text-color, inherit);overflow-x:auto}
.widget-title{font-size:11px;color:var(--text-muted-color, #888);letter-spacing:.12em;text-transform:uppercase;margin-bottom:1rem}
.bit-row{display:flex;gap:4px;justify-content:center;flex-wrap:wrap;margin-bottom:.8rem}
.fbit{width:38px;height:44px;border-radius:6px;display:flex;flex-direction:column;align-items:center;justify-content:center;font-size:18px;font-weight:600;cursor:pointer;transition:all .15s;border:2px solid transparent;user-select:none;gap:2px}
.fbit-label{font-size:9px;color:var(--text-muted-color, #888)}
.fbit.on{background:rgba(110,127,255,0.8);color:#fff;border-color:#8fa0ff}
.fbit.off{background:var(--body-bg, rgba(128,128,128,0.1));color:var(--text-muted-color, #888);border-color:var(--border-color, rgba(128,128,128,0.2))}
.fbit:hover{transform:translateY(-2px);filter:brightness(1.2)}
.bit-result{text-align:center;font-size:2rem;color:#6e7fff;font-weight:600;margin-top:.5rem}
.bit-result span{color:var(--text-muted-color, #888);font-size:1rem}
.slider-row{display:flex;align-items:center;gap:12px;margin-bottom:1rem;flex-wrap:wrap}
.slider-row label{color:var(--text-muted-color, #888);font-size:12px;min-width:50px}
.slider-row input[type=range]{flex:1;min-width:150px;accent-color:#6e7fff;cursor:pointer}
.slider-val{color:#6e7fff;font-size:1.2rem;font-weight:600;min-width:40px;text-align:right}
.cidr-split-bar{height:36px;border-radius:8px;overflow:hidden;display:flex;border:1px solid var(--border-color, rgba(128,128,128,0.2));margin-bottom:1rem}
.cidr-net{background:rgba(110,127,255,.3);display:flex;align-items:center;justify-content:center;font-size:11px;color:#6e7fff;border-right:2px solid #6e7fff;transition:width .4s ease;overflow:hidden;white-space:nowrap;padding:0 4px}
.cidr-host{background:rgba(86,227,159,.15);display:flex;align-items:center;justify-content:center;font-size:11px;color:#56e39f;flex:1;overflow:hidden;white-space:nowrap;padding:0 4px}
.cidr-stats{display:grid;grid-template-columns:repeat(auto-fit,minmax(100px,1fr));gap:8px}
.stat-card{background:var(--body-bg, rgba(128,128,128,0.1));border:1px solid var(--border-color, rgba(128,128,128,0.2));border-radius:8px;padding:10px;text-align:center}
.stat-val{font-size:1rem;color:#6e7fff;font-weight:600;margin-bottom:3px;word-break:break-all}
.stat-lbl{font-size:9px;color:var(--text-muted-color, #888);text-transform:uppercase;letter-spacing:.08em}
.and-rows{display:flex;flex-direction:column;gap:6px;overflow-x:auto;padding-bottom:8px}
.and-row{display:flex;align-items:center;gap:6px;flex-wrap:nowrap;width:max-content}
.and-label{font-size:11px;color:var(--text-muted-color, #888);min-width:70px;text-align:right;font-family:var(--font-family-monospace, monospace)}
.and-bits{display:flex;gap:2px;flex-wrap:nowrap}
.abit{width:22px;height:24px;border-radius:4px;display:flex;align-items:center;justify-content:center;font-size:10px;font-weight:600;font-family:var(--font-family-monospace, monospace)}
.abit-ip{background:rgba(110,127,255,.25);color:#6e7fff;border:1px solid rgba(110,127,255,.4)}
.abit-mask{background:rgba(255,209,102,.2);color:#ffd166;border:1px solid rgba(255,209,102,.35)}
.abit-r1{background:rgba(86,227,159,.25);color:#56e39f;border:1px solid rgba(86,227,159,.4)}
.abit-r0{background:var(--body-bg, rgba(128,128,128,0.1));color:var(--text-muted-color, #888);border:1px solid var(--border-color, rgba(128,128,128,0.2))}
.and-divider{border-top:1px solid var(--border-color, rgba(128,128,128,0.2));margin:4px 0 4px 76px;width:100%}
.and-result-label{font-size:11px;color:#56e39f;min-width:70px;text-align:right;font-family:var(--font-family-monospace, monospace)}
.and-legend{display:flex;gap:16px;margin-top:1rem;flex-wrap:wrap;font-size:11px;color:var(--text-muted-color, #888)}
.legend-dot{display:inline-block;width:10px;height:10px;border-radius:2px;margin-right:4px;vertical-align:middle}
.calc-inputs{display:flex;gap:8px;margin-bottom:1rem;align-items:flex-end;flex-wrap:wrap}
.calc-input-group{display:flex;flex-direction:column;gap:4px;flex:1;min-width:120px}
.calc-input-group label{font-size:11px;color:var(--text-muted-color, #888);letter-spacing:.08em}
.calc-input-group input,.calc-input-group select{background:var(--body-bg, rgba(128,128,128,0.1));border:1px solid var(--border-color, rgba(128,128,128,0.2));border-radius:6px;color:var(--text-color, inherit);font-family:var(--font-family-monospace, monospace);font-size:14px;padding:8px 10px;outline:none;transition:border-color .2s;width:100%}
.calc-input-group input:focus,.calc-input-group select:focus{border-color:#6e7fff}
.calc-btn{background:#6e7fff;color:#fff;border:none;border-radius:6px;padding:9px 18px;font-family:var(--font-family-monospace, monospace);font-size:13px;cursor:pointer;transition:all .2s;align-self:flex-end;white-space:nowrap}
.calc-btn:hover{background:#8fa0ff;transform:translateY(-1px)}
.calc-results{display:none}
.calc-results.show{display:grid;grid-template-columns:repeat(auto-fit,minmax(140px,1fr));gap:8px}
.result-card{background:var(--body-bg, rgba(128,128,128,0.1));border:1px solid var(--border-color, rgba(128,128,128,0.2));border-radius:8px;padding:12px 14px}
.result-label{font-size:10px;color:var(--text-muted-color, #888);text-transform:uppercase;letter-spacing:.08em;margin-bottom:4px}
.result-val{font-size:14px;color:var(--text-color, inherit);font-weight:500;font-family:var(--font-family-monospace, monospace)}
.result-val.accent{color:#6e7fff}
.result-val.green{color:#56e39f}
.result-val.orange{color:#ff7a5c}
.result-val.amber{color:#ffd166}
.calc-error{color:#ff7a5c;font-size:12px;margin-top:6px;display:none;font-family:var(--font-family-monospace, monospace)}
.challenge-box{background:var(--card-bg, rgba(128,128,128,0.05));border:1px solid var(--border-color, rgba(128,128,128,0.2));border-left:3px solid #6e7fff;border-radius:0 10px 10px 0;padding:1.2rem 1.5rem;margin:1.5rem 0}
.challenge-box h4{color:#6e7fff;font-family:var(--font-family-monospace, monospace);font-size:13px;margin-bottom:.8rem}
.challenge-reveal{background:var(--body-bg, rgba(128,128,128,0.1));border:1px solid var(--border-color, rgba(128,128,128,0.2));border-radius:6px;padding:8px 14px;font-family:var(--font-family-monospace, monospace);font-size:12px;color:var(--text-muted-color, #888);cursor:pointer;margin-top:1rem;display:inline-block;transition:all .2s}
.challenge-reveal:hover{border-color:#6e7fff;color:#6e7fff}
.challenge-answer{display:none;margin-top:1rem;background:var(--body-bg, rgba(128,128,128,0.1));border:1px solid var(--border-color, rgba(128,128,128,0.2));border-radius:8px;padding:1rem;font-family:var(--font-family-monospace, monospace);font-size:12px;line-height:2;color:var(--text-color, inherit)}
.challenge-answer .hl{color:#56e39f}
.info-box{background:var(--card-bg, rgba(128,128,128,0.05));border:1px solid var(--border-color, rgba(128,128,128,0.2));border-left:3px solid #ffd166;border-radius:0 10px 10px 0;padding:1rem 1.5rem;margin:1.5rem 0;font-size:13px}
.step-box{background:var(--card-bg, rgba(128,128,128,0.05));border:1px solid var(--border-color, rgba(128,128,128,0.2));border-radius:12px;padding:1.5rem;margin:1.5rem 0}
.step-box ol{margin:0;padding-left:1.4rem;line-height:2.2;font-family:var(--font-family-monospace, monospace);font-size:13px;color:var(--text-color, inherit)}
.step-box ol li strong{color:#6e7fff}
</style>

<div class="widget">
  <div class="widget-title">Binary Bit Flipper — click any bit to flip it</div>
  <div class="bit-row" id="flipper-bits"></div>
  <div class="bit-result" id="flipper-result">0 <span>decimal</span></div>
</div>

<script>
(function(){
  var bits=[0,0,0,0,0,0,0,0];
  var places=[128,64,32,16,8,4,2,1];
  var container=document.getElementById('flipper-bits');
  var result=document.getElementById('flipper-result');
  function render(){
    container.innerHTML='';
    var val=0;
    bits.forEach(function(b,i){
      if(b) val+=places[i];
      var el=document.createElement('div');
      el.className='fbit '+(b?'on':'off');
      el.innerHTML=b+'<span class="fbit-label">'+places[i]+'</span>';
      (function(idx){ el.onclick=function(){ bits[idx]=bits[idx]?0:1; render(); }; })(i);
      container.appendChild(el);
    });
    result.innerHTML=val+' <span>decimal</span>';
  }
  render();
})();
</script>

**Quick sanity check:** flip on *only* the leftmost bit (128). Result: 128. Now flip on the next one too (64). Result: 192. That's exactly the first octet of `192.168.1.1` — you just read a real IP address in binary.

> 8 bits → values 0–255. Four octets → 32 bits total → roughly 4.3 billion addresses.

---

## IP Classes — Why the Old System Broke

When IPv4 launched in the early 1980s, the designers divided addresses into classes based on the first octet. The idea was simple: small organisations get a Class C (254 hosts), big ones get a Class A (16 million hosts).

| Class | First Octet | Hosts per Network |
|-------|-------------|-------------------|
| A | 1–126 | ~16.7 million |
| B | 128–191 | ~65,534 |
| C | 192–223 | 254 |
| D | 224–239 | Multicast only |
| E | 240–255 | Experimental |

This seems fine until you notice the problem: **there's no middle ground.** A company needs 1,000 hosts — they're too big for a Class C (254) but must take an entire Class B (65,534), wasting 64,000+ addresses. Multiply this across thousands of organisations and you've burned through the address space in years.

The solution was **CIDR** (Classless Inter-Domain Routing) — which we'll cover next. Classes are largely obsolete today, but CCNA still tests them, and understanding *why they failed* is exactly what makes CIDR click.

> [!NOTE]
> `127.x.x.x` is reserved for **loopback** — when a device talks to itself. `ping 127.0.0.1` always responds, even with no network connection.

---

## Private vs. Public Addresses

Three ranges are permanently reserved for private networks — they're never routed on the public internet:

| Range | Addresses | Common Use |
|-------|-----------|------------|
| `10.0.0.0/8` | ~16.7 million | Large enterprises |
| `172.16.0.0/12` | ~1 million | Medium organisations |
| `192.168.0.0/16` | ~65K | Your home Wi-Fi — right now |

Your home router uses **NAT** (Network Address Translation) to represent all your private devices behind a single public IP. Like an apartment building — hundreds of flats, one street address.

---

## Subnet Masks — Drawing the Boundary

An IP address contains two pieces of information packed together: **which network** the device is on, and **which host** it is within that network. A subnet mask is what splits them apart.

It's always a solid block of 1s followed by 0s. The 1s cover the network portion; the 0s cover the host portion:

```
255.255.255.0  =  11111111.11111111.11111111.00000000
                  |_______ network _________|_ host _|
```

More 1s = more bits locked to the network = fewer addresses left for hosts = **smaller subnet**.  
Fewer 1s = more host bits free = **larger subnet**.

---

## The AND Operation — How Routers Actually Think

When your computer sends a packet, it needs to know: is the destination on *my* network, or does this need to go to a router? It figures this out by ANDing the destination IP with the subnet mask. **This single operation is the foundation of all routing.**

AND rules: `1 AND 1 = 1`. Anything else = 0. Applying a mask to an IP zeroes out all the host bits, leaving only the network address.

**Full example — 192.168.1.100 AND 255.255.255.0:**

<div class="widget">
  <div class="widget-title">AND Operation Visualizer</div>
  <div class="and-rows" id="and-vis"></div>
  <div class="and-legend">
    <span><span class="legend-dot" style="background:#6e7fff"></span>IP bits</span>
    <span><span class="legend-dot" style="background:#ffd166"></span>Mask bits</span>
    <span><span class="legend-dot" style="background:#56e39f"></span>Result = 1</span>
    <span><span class="legend-dot" style="background:rgba(128,128,128,0.2);border:1px solid rgba(128,128,128,0.3)"></span>Result = 0</span>
  </div>
</div>

<script>
(function(){
  var ip=[192,168,1,100];
  var mask=[255,255,255,0];
  var res=ip.map(function(o,i){ return o&mask[i]; });
  function toBits(n){ return n.toString(2).padStart(8,'0').split('').map(Number); }
  var ipBits=ip.reduce(function(a,o){ return a.concat(toBits(o)); },[]);
  var maskBits=mask.reduce(function(a,o){ return a.concat(toBits(o)); },[]);
  var resBits=res.reduce(function(a,o){ return a.concat(toBits(o)); },[]);
  function makeRow(bits, cls, label, labelClass){
    var row=document.createElement('div');
    row.className='and-row';
    var lbl=document.createElement('div');
    lbl.className=labelClass||'and-label';
    lbl.textContent=label;
    row.appendChild(lbl);
    var bd=document.createElement('div');
    bd.className='and-bits';
    bits.forEach(function(b,i){
      var el=document.createElement('div');
      el.className='abit '+cls;
      el.textContent=b;
      bd.appendChild(el);
      if(i===7||i===15||i===23){
        var dot=document.createElement('span');
        dot.style.cssText='color:rgba(128,128,128,0.4);margin:0 2px;font-family:monospace';
        dot.textContent='.';
        bd.appendChild(dot);
      }
    });
    row.appendChild(bd);
    return row;
  }
  var container=document.getElementById('and-vis');
  container.appendChild(makeRow(ipBits,'abit-ip',ip.join('.')));
  container.appendChild(makeRow(maskBits,'abit-mask',mask.join('.')));
  var div=document.createElement('div');
  div.className='and-divider';
  container.appendChild(div);
  container.appendChild(makeRow(resBits,'',res.join('.'),'and-result-label'));
  var lastRow=container.lastChild;
  var lastBits=lastRow.querySelector('.and-bits');
  lastBits.innerHTML='';
  resBits.forEach(function(b,i){
    var el=document.createElement('div');
    el.className='abit '+(b?'abit-r1':'abit-r0');
    el.textContent=b;
    lastBits.appendChild(el);
    if(i===7||i===15||i===23){
      var dot=document.createElement('span');
      dot.style.cssText='color:rgba(128,128,128,0.4);margin:0 2px;font-family:monospace';
      dot.textContent='.';
      lastBits.appendChild(dot);
    }
  });
})();
</script>

The result — **192.168.1.0** — is the network address. Any device whose IP ANDs to the same result is on the same network and can be reached directly.

---

## CIDR — Slicing Networks into Blocks

Writing `255.255.255.0` every time is tedious. Worse, class-based addressing wasted millions of IPs because you had to take a fixed-size block — no middle ground.

CIDR (Classless Inter-Domain Routing) solves both. The idea: **you take a pool of 32 bits and decide how many bits belong to the network.** The rest belong to hosts. You write this as a slash followed by the bit count:

```
192.168.1.0/24  →  24 bits = network,  8 bits = hosts  →  256 addresses
192.168.1.0/25  →  25 bits = network,  7 bits = hosts  →  128 addresses
192.168.1.0/26  →  26 bits = network,  6 bits = hosts  →   64 addresses
```

**The key insight:** host bits are all 0s or all 1s for the special addresses (network and broadcast), so usable hosts = 2ⁿ − 2, where n is the number of host bits.

**Block size** = 2^(32 − prefix). A /24 has a block size of 256. A /25 splits that block in two: 256/2 = 128. A /26 splits again: 64. Every time you add one bit to the prefix, you halve the block size and double the number of subnets.

This is why CIDR replaced classful addressing — you can right-size any allocation. Need 500 hosts? Take a /23 (512 addresses, 510 usable). Need 6 hosts? Take a /29 (8 addresses, 6 usable). No waste.

**Drag the slider to see how prefix length controls block size:**

<div class="widget">
  <div class="widget-title">CIDR Prefix Explorer</div>
  <div class="slider-row">
    <label>Prefix</label>
    <input type="range" id="cidr-slider" min="1" max="30" value="24">
    <div class="slider-val" id="cidr-val">/24</div>
  </div>
  <div class="cidr-split-bar">
    <div class="cidr-net" id="cidr-net-bar" style="width:75%">NETWORK (24)</div>
    <div class="cidr-host" id="cidr-host-bar">HOST (8)</div>
  </div>
  <div class="cidr-stats">
    <div class="stat-card"><div class="stat-val" id="s-mask">255.255.255.0</div><div class="stat-lbl">Subnet Mask</div></div>
    <div class="stat-card"><div class="stat-val" id="s-hosts">254</div><div class="stat-lbl">Usable Hosts</div></div>
    <div class="stat-card"><div class="stat-val" id="s-block">256</div><div class="stat-lbl">Block Size</div></div>
  </div>
</div>

<script>
(function(){
  function cidrToMask(p){
    return [0,1,2,3].map(function(i){
      var b=Math.min(8,Math.max(0,p-i*8));
      return 256-Math.pow(2,8-b);
    });
  }
  function fmt(n){
    if(n>=1000000) return (n/1000000).toFixed(1)+'M';
    if(n>=1000) return (n/1000).toFixed(0)+'K';
    return n.toString();
  }
  function update(p){
    document.getElementById('cidr-val').textContent='/'+p;
    var mask=cidrToMask(p);
    var hosts=Math.max(0,Math.pow(2,32-p)-2);
    var block=Math.pow(2,32-p);
    document.getElementById('s-mask').textContent=mask.join('.');
    document.getElementById('s-hosts').textContent=fmt(hosts);
    document.getElementById('s-block').textContent=fmt(block);
    var netPct=(p/32*100).toFixed(1);
    var netBar=document.getElementById('cidr-net-bar');
    var hostBar=document.getElementById('cidr-host-bar');
    netBar.style.width=netPct+'%';
    netBar.textContent=p>5?'NETWORK ('+p+')':'';
    hostBar.textContent=(32-p)>2?'HOST ('+(32-p)+')':'';
  }
  var slider=document.getElementById('cidr-slider');
  slider.addEventListener('input',function(){ update(parseInt(slider.value)); });
  update(24);
})();
</script>

Quick reference:

| CIDR | Subnet Mask | Block Size | Usable Hosts | Use Case |
|------|-------------|------------|--------------|----------|
| `/8` | 255.0.0.0 | 16,777,216 | 16,777,214 | ISP / large org |
| `/16` | 255.255.0.0 | 65,536 | 65,534 | Campus / enterprise |
| `/24` | 255.255.255.0 | 256 | 254 | Office LAN (most common) |
| `/25` | 255.255.255.128 | 128 | 126 | Half of a /24 |
| `/30` | 255.255.255.252 | 4 | 2 | Point-to-point links |
| `/32` | 255.255.255.255 | 1 | 1 | Loopback / host route |

---

## How to Subnet — Step by Step

This is the skill. Given any IP and prefix, you need four values: **network address, broadcast address, first usable host, last usable host.** Here's the method that works every time:

<div class="step-box">
  <ol>
    <li><strong>Find the block size.</strong> Subtract the last non-255 octet of the mask from 256. For /26: mask = 255.255.255.192, block = 256 − 192 = 64.</li>
    <li><strong>Find the network address.</strong> Round the interesting octet down to the nearest multiple of the block size. For 192.168.1.100 /26: 100 rounds down to 64 → network = 192.168.1.64</li>
    <li><strong>Find the broadcast address.</strong> Add block size − 1 to the network address. 64 + 64 − 1 = 127 → broadcast = 192.168.1.127</li>
    <li><strong>Usable host range.</strong> First host = network + 1 = 192.168.1.65. Last host = broadcast − 1 = 192.168.1.126.</li>
  </ol>
</div>

That's it. Four steps, works for any prefix. Let's practice.

---

## Subnet Calculator

<div class="widget">
  <div class="widget-title">Subnet Calculator</div>
  <div class="calc-inputs">
    <div class="calc-input-group">
      <label>IP ADDRESS</label>
      <input type="text" id="calc-ip" placeholder="192.168.1.0" value="192.168.1.0">
    </div>
    <div class="calc-input-group" style="max-width:100px">
      <label>PREFIX</label>
      <select id="calc-prefix">
        <option>/8</option><option>/9</option><option>/10</option><option>/11</option>
        <option>/12</option><option>/13</option><option>/14</option><option>/15</option>
        <option>/16</option><option>/17</option><option>/18</option><option>/19</option>
        <option>/20</option><option>/21</option><option>/22</option><option>/23</option>
        <option selected>/24</option><option>/25</option><option>/26</option><option>/27</option>
        <option>/28</option><option>/29</option><option>/30</option><option>/32</option>
      </select>
    </div>
    <button class="calc-btn" onclick="calcSubnet()">Calculate</button>
  </div>
  <div class="calc-error" id="calc-error">Invalid IP address</div>
  <div class="calc-results" id="calc-results">
    <div class="result-card"><div class="result-label">Network Address</div><div class="result-val accent" id="r-network">—</div></div>
    <div class="result-card"><div class="result-label">Broadcast Address</div><div class="result-val orange" id="r-broadcast">—</div></div>
    <div class="result-card"><div class="result-label">First Usable Host</div><div class="result-val green" id="r-first">—</div></div>
    <div class="result-card"><div class="result-label">Last Usable Host</div><div class="result-val green" id="r-last">—</div></div>
    <div class="result-card"><div class="result-label">Subnet Mask</div><div class="result-val" id="r-mask">—</div></div>
    <div class="result-card"><div class="result-label">Usable Hosts</div><div class="result-val amber" id="r-hosts">—</div></div>
  </div>
</div>

<script>
function calcSubnet(){
  var ipStr=document.getElementById('calc-ip').value.trim();
  var prefix=parseInt(document.getElementById('calc-prefix').value.replace('/',''));
  var err=document.getElementById('calc-error');
  var results=document.getElementById('calc-results');
  var parts=ipStr.split('.').map(Number);
  if(parts.length!==4||parts.some(function(p){ return isNaN(p)||p<0||p>255; })){
    err.style.display='block'; results.classList.remove('show'); return;
  }
  err.style.display='none';
  function cidrToMask(p){
    return [0,1,2,3].map(function(i){ var b=Math.min(8,Math.max(0,p-i*8)); return 256-Math.pow(2,8-b); });
  }
  var mask=cidrToMask(prefix);
  var network=parts.map(function(o,i){ return o&mask[i]; });
  var wildcard=mask.map(function(m){ return 255-m; });
  var broadcast=network.map(function(o,i){ return o|wildcard[i]; });
  var first=network.slice(); first[3]+=1;
  var last=broadcast.slice(); last[3]-=1;
  var hosts=Math.max(0,Math.pow(2,32-prefix)-2);
  function fmt(n){ if(n>=1000000) return (n/1000000).toFixed(2)+'M'; if(n>=1000) return (n/1000).toFixed(0)+'K'; return n.toString(); }
  document.getElementById('r-network').textContent=network.join('.');
  document.getElementById('r-broadcast').textContent=broadcast.join('.');
  document.getElementById('r-first').textContent=prefix<=30?first.join('.'):'N/A';
  document.getElementById('r-last').textContent=prefix<=30?last.join('.'):'N/A';
  document.getElementById('r-mask').textContent=mask.join('.');
  document.getElementById('r-hosts').textContent=fmt(hosts)+' hosts';
  results.classList.add('show');
}
</script>

---

## Wildcard Masks — A Quick Note

> [!NOTE]
> **You need this for CCNA, but it's a narrow concept.** Skip it if you're not there yet.

Cisco ACLs (Access Control Lists) and OSPF use **wildcard masks** — the bitwise inverse of a subnet mask. The logic flips: `0` means "this bit must match," `1` means "don't care." To get one, subtract the subnet mask from `255.255.255.255`.

```
Subnet mask:   255.255.255.0
Wildcard:        0.0.0.255     ← 255 − each octet
Meaning: match the first three octets exactly, ignore the last
```

```
! Permit all traffic from 192.168.1.x
access-list 10 permit 192.168.1.0 0.0.0.255

! Permit only one specific host
access-list 10 permit 10.10.10.5 0.0.0.0
```

That's the whole concept. We'll cover ACLs properly in a dedicated post.

---

## Practice Challenges

Work these out by hand using the four-step method above before revealing the answers. The calculator is for checking, not shortcutting.

<div class="challenge-box">
  <h4>// CHALLENGE 1 — Warm-up</h4>
  <p style="color:var(--text-color, inherit);font-family:var(--font-family-monospace, monospace);font-size:13px">Given: <strong style="color:#6e7fff">172.16.5.0 /27</strong></p>
  <ol style="color:var(--text-muted-color, #888);font-size:13px;margin-top:.5rem;padding-left:1.2rem;line-height:2.2">
    <li>What is the subnet mask?</li>
    <li>What is the block size?</li>
    <li>What is the broadcast address?</li>
    <li>What is the usable host range?</li>
  </ol>
  <div class="challenge-reveal" onclick="document.getElementById('ch1-answer').style.display='block';this.style.display='none'">Reveal answer ↓</div>
  <div class="challenge-answer" id="ch1-answer">
    Subnet mask:&nbsp;&nbsp;&nbsp;&nbsp; <span class="hl">255.255.255.224</span><br>
    Block size:&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hl">32 (256 − 224)</span><br>
    Network address: <span class="hl">172.16.5.0</span><br>
    Broadcast:&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hl">172.16.5.31</span><br>
    Host range:&nbsp;&nbsp;&nbsp;&nbsp; <span class="hl">172.16.5.1 – 172.16.5.30</span><br>
    Usable hosts:&nbsp;&nbsp; <span class="hl">30 (2⁵ − 2)</span>
  </div>
</div>

<div class="challenge-box">
  <h4>// CHALLENGE 2 — The boundary test</h4>
  <p style="color:var(--text-color, inherit);font-family:var(--font-family-monospace, monospace);font-size:13px">Given: <strong style="color:#6e7fff">10.0.0.200 /26</strong></p>
  <ol style="color:var(--text-muted-color, #888);font-size:13px;margin-top:.5rem;padding-left:1.2rem;line-height:2.2">
    <li>What network is this host on?</li>
    <li>What is the broadcast address of that network?</li>
    <li>Is 10.0.0.191 on the same subnet? Why or why not?</li>
  </ol>
  <div class="challenge-reveal" onclick="document.getElementById('ch2-answer').style.display='block';this.style.display='none'">Reveal answer ↓</div>
  <div class="challenge-answer" id="ch2-answer">
    Block size: <span class="hl">64 (256 − 192)</span><br>
    Multiples of 64: 0, 64, 128, 192 → 200 falls in the 192 block<br>
    Network address: <span class="hl">10.0.0.192</span><br>
    Broadcast:&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hl">10.0.0.255</span><br>
    Host range:&nbsp;&nbsp;&nbsp;&nbsp; <span class="hl">10.0.0.193 – 10.0.0.254</span><br>
    Is 10.0.0.191 on the same subnet? <span class="hl">No.</span> It falls in the 10.0.0.128/26 subnet (128–191).
  </div>
</div>

<div class="challenge-box">
  <h4>// CHALLENGE 3 — Design task</h4>
  <p style="color:var(--text-color, inherit);font-family:var(--font-family-monospace, monospace);font-size:13px">You have the block <strong style="color:#6e7fff">192.168.10.0 /24</strong> and need to divide it into 4 equal subnets.</p>
  <ol style="color:var(--text-muted-color, #888);font-size:13px;margin-top:.5rem;padding-left:1.2rem;line-height:2.2">
    <li>What prefix do you use?</li>
    <li>List all four subnet network addresses.</li>
    <li>How many usable hosts does each subnet have?</li>
  </ol>
  <div class="challenge-reveal" onclick="document.getElementById('ch3-answer').style.display='block';this.style.display='none'">Reveal answer ↓</div>
  <div class="challenge-answer" id="ch3-answer">
    4 subnets requires 2 extra bits → /24 + 2 = <span class="hl">/26</span><br>
    Block size: 64<br>
    Subnet 1: <span class="hl">192.168.10.0/26</span>&nbsp;&nbsp; (0–63)<br>
    Subnet 2: <span class="hl">192.168.10.64/26</span>&nbsp; (64–127)<br>
    Subnet 3: <span class="hl">192.168.10.128/26</span> (128–191)<br>
    Subnet 4: <span class="hl">192.168.10.192/26</span> (192–255)<br>
    Usable hosts each: <span class="hl">62 (2⁶ − 2)</span>
  </div>
</div>

Done all three? Drop your working in the comments below — it's good practice to write it out, and I'll check your method.

---

## What You Now Know

- **IP vs MAC** — IP is a logical Layer 3 address; MAC is a physical Layer 2 address. IP gets data to the right network, MAC gets it to the right device.
- **Binary** — 8 bits per octet, place values double right-to-left: 1, 2, 4, 8, 16, 32, 64, 128. Add the 1-positions to get decimal.
- **IP classes** — the original fixed-size system. Its rigidity caused address waste, which is exactly why CIDR replaced it.
- **Private ranges** — 10.x, 172.16.x, 192.168.x never appear on the public internet.
- **Subnet masks** — draw the network/host boundary in binary. All 1s = network bits, all 0s = host bits.
- **AND operation** — IP AND mask = network address. This is the core routing decision.
- **CIDR** — instead of fixed classes, you choose how many bits belong to the network. Block size = 2^(32 − prefix). More bits = smaller block = more subnets.
- **Wildcard masks** — inverse of subnet masks, used in Cisco ACLs and OSPF.

The fastest way to get good at this: practice on paper. Pick a random IP and prefix, work out the four values by hand using the four-step method. After 20 problems it becomes second nature — the numbers just fall into place.

---

*Written by Tsotne · tsotne.blog · Post #001*
