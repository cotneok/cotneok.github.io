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

IPv4 addresses are **32 bits long**, written as four numbers separated by dots — called **octets** because each one is 8 bits. Each octet can be any value from 0 to 255, giving roughly **4.3 billion unique addresses**.

---

## Binary — The Only Language Computers Speak

Here's the thing nobody tells beginners early enough: **your router doesn't think in decimal**. It thinks in binary. Every IP address, every routing decision, every subnet calculation happens in binary.

Binary is base-2. Each position is worth 2× the one to its right. To read a binary number, add up the place values wherever there is a `1`.

**Try it — click any bit to flip it:**

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

> [!NOTE]
> `127.x.x.x` is reserved for **loopback** — when a device talks to itself. Try `ping 127.0.0.1` — it always responds.

---

## Private vs. Public Addresses

Three ranges are permanently reserved for private networks — they're never routed on the public internet:

| Range | Addresses | Common Use |
|-------|-----------|------------|
| `10.0.0.0/8` | ~16.7 million | Large enterprises |
| `172.16.0.0/12` | ~1 million | Medium organisations |
| `192.168.0.0/16` | ~65K | Your home Wi-Fi — right now |

Your home router uses **NAT** to represent all your private devices behind a single public IP. Like an apartment building — hundreds of flats, one street address.

---

## Subnet Masks — Drawing the Boundary

A subnet mask tells you where the **network** bits end and the **host** bits begin. It's always a solid block of 1s followed by 0s:

```
255.255.255.0  =  11111111.11111111.11111111.00000000  (/24)
255.255.0.0    =  11111111.11111111.00000000.00000000  (/16)
255.0.0.0      =  11111111.00000000.00000000.00000000  (/8)
```

More 1s = smaller network, fewer hosts. Fewer 1s = larger network, more hosts.

---

## The AND Operation — How Routers Actually Think

When your computer sends a packet, it ANDs the destination IP with the subnet mask to find the network address. **This is the core of all routing.**

**Full example — 192.168.1.100 AND 255.255.255.0:**

<div class="widget">
  <div class="widget-title">AND Operation Visualizer</div>
  <div class="and-rows" id="and-vis"></div>
  <div class="and-legend">
    <span><span class="legend-dot" style="background:#6e7fff"></span>IP bits</span>
    <span><span class="legend-dot" style="background:#ffd166"></span>Mask bits</span>
    <span><span class="legend-dot" style="background:#56e39f"></span>Result = 1</span>
    <span><span class="legend-dot" style="background:#1c1c22;border:1px solid #2a2a35"></span>Result = 0</span>
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
        dot.style.cssText='color:#2a2a35;margin:0 2px;font-family:monospace';
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
  container.appendChild(makeRow(resBits.map(function(b){ return b; }),'',res.join('.'),'and-result-label'));
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
      dot.style.cssText='color:#2a2a35;margin:0 2px;font-family:monospace';
      dot.textContent='.';
      lastBits.appendChild(dot);
    }
  });
})();
</script>

The result — **192.168.1.0** — is the network address. Any device whose IP ANDs to the same result is on the same network.

---

## CIDR Notation — The Shorthand

Writing `255.255.255.0` every time is tedious. CIDR uses a slash + the count of 1-bits:

```
192.168.1.0/24  →  255.255.255.0   (24 ones)
10.0.0.0/8      →  255.0.0.0       (8 ones)
```

**Drag the slider to explore prefix lengths:**

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

| CIDR | Subnet Mask | Usable Hosts | Use Case |
|------|-------------|--------------|----------|
| `/8` | 255.0.0.0 | 16,777,214 | ISP / large org |
| `/16` | 255.255.0.0 | 65,534 | Campus / enterprise |
| `/24` | 255.255.255.0 | 254 | Office LAN (most common) |
| `/25` | 255.255.255.128 | 126 | Half of a /24 |
| `/30` | 255.255.255.252 | 2 | Point-to-point links |
| `/32` | 255.255.255.255 | 1 | Loopback / host route |

---

## Wildcard Masks — The Inverse

Cisco ACLs and OSPF use **wildcard masks** — the bitwise inverse of a subnet mask. `0` = "must match", `1` = "don't care." To get one: subtract the subnet mask from `255.255.255.255`.

```
Subnet mask:   255.255.255.0   (/24)
Wildcard mask:   0.0.0.255
Meaning:       Match 192.168.1.0 → .255 (entire /24)
```

```
! Permit all traffic from 192.168.1.x
access-list 10 permit 192.168.1.0 0.0.0.255

! Permit only host 10.10.10.5
access-list 10 permit 10.10.10.5 0.0.0.0
```

---

## Subnet Calculator

Enter any IP and prefix length to get all four key values instantly:

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

## Practice Challenge

<div class="challenge-box">
  <h4>// CHALLENGE — Can you subnet this?</h4>
  <p style="color:var(--text-color, inherit);font-family:var(--font-family-monospace, monospace);font-size:13px">Given: <strong style="color:#6e7fff">172.16.5.0 /27</strong></p>
  <ol style="color:var(--text-muted-color, #888);font-size:13px;margin-top:.5rem;padding-left:1.2rem;line-height:2.2">
    <li>What is the subnet mask?</li>
    <li>What is the block size?</li>
    <li>What is the network address?</li>
    <li>What is the broadcast address?</li>
    <li>What is the usable host range?</li>
  </ol>
  <div class="challenge-reveal" onclick="document.getElementById('ch-answer').style.display='block';this.style.display='none'">Reveal answer ↓</div>
  <div class="challenge-answer" id="ch-answer">
    Subnet mask:&nbsp;&nbsp;&nbsp;&nbsp; <span class="hl">255.255.255.224</span><br>
    Block size:&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hl">32 addresses (2⁵)</span><br>
    Network address: <span class="hl">172.16.5.0</span><br>
    Broadcast:&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hl">172.16.5.31</span><br>
    Host range:&nbsp;&nbsp;&nbsp;&nbsp; <span class="hl">172.16.5.1 – 172.16.5.30</span><br>
    Usable hosts:&nbsp;&nbsp; <span class="hl">30 hosts (2⁵ − 2)</span>
  </div>
</div>

---

## What You Now Know

- **IPv4 addresses** are 32 bits, four decimal octets (0–255 each)
- **Binary** is how computers actually process every address — 8 bits per octet
- **IP classes** (A/B/C) defined the original split — mostly replaced by CIDR
- **Private ranges** (10.x, 172.16.x, 192.168.x) never appear on the public internet
- **Subnet masks** draw the boundary between network and host bits
- **AND operation** — the fundamental routing decision: IP AND mask = network address
- **CIDR /notation** counts the 1-bits. More bits = smaller network
- **Wildcard masks** are the inverse — used in Cisco ACLs and OSPF

The fastest way to get good at this: practice on paper. Pick a random IP and prefix, work out the four values by hand. After 20 problems it becomes second nature.

---

*Written by Tsotne · tsotne.blog · Post #001*
