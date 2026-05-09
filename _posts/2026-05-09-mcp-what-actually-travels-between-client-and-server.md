---
title: "MCP — What Actually Travels Between Client and Server"
date: 2026-05-09 00:00:00 +0000
categories: [AI Engineering, MCP]
tags: [mcp, json-rpc, claude-code, stdio, streamable-http, sse]
description: MCP looks like a platform until you watch the bytes move. Here's the same tool call over stdio and Streamable HTTP — same JSON-RPC, different wire.
---

**MCP** — short for **Model Context Protocol** — is how AI assistants like Claude talk to outside tools: GitHub, your filesystem, a database, a calendar, anything. If you've used Claude Code with a tool plugged in, you've already used MCP, even if you didn't notice.

For a protocol that's becoming the standard plumbing of AI tooling, the actual MCP "thing" is surprisingly small. Most explanations make it sound bigger and more architectural than it is.

This post does the opposite. We're going to look at the actual bytes on the wire — twice — and watch the whole protocol collapse into a single idea.

---

## The Punchline First

MCP isn't a platform. It isn't an architecture. It's just **JSON-RPC 2.0** — a convention for JSON messages shaped like requests and matching responses — sent over a pipe between two programs. The whole protocol surface fits on a napkin.

Everything that feels confusing about MCP — local servers, remote servers, stdio, Streamable HTTP, SSE, "hosted MCP" — collapses into one decision:

**Which pipe are the JSON-RPC messages travelling through?**

I'm going to prove it by watching the same tool call cross the wire twice. Same JSON. Different pipe.

For the whole post, our actors are:

- **Client:** Claude Code
- **Server:** the GitHub MCP server
- **Tool:** `create_issue`

Same two characters the whole way through. One mental model.

---

## What MCP Actually Says

First, the language. MCP messages are written in **JSON-RPC 2.0** — a small, decades-old convention for "request/response, both as JSON." Each request carries an `id`. Each response carries the same `id` so the client can tell which answer goes with which question. That's the whole trick.

So: Claude Code wants to file a GitHub issue called *"MCP post draft"* in `cotneok/blog`. Here's the entire tool-call exchange it has with the GitHub MCP server to make that happen:

<style>
.mcp-wire-widget{background:rgba(128,128,128,.06);border:1px solid rgba(128,128,128,.22);border-radius:12px;padding:1.25rem;margin:1.5rem 0;font-family:var(--font-family-monospace,monospace);color:inherit;overflow-x:auto}
.mcp-title{font-size:11px;color:var(--text-muted-color,#888);letter-spacing:.12em;text-transform:uppercase;margin-bottom:1rem}
.mcp-pair{display:grid;grid-template-columns:minmax(240px,1fr) auto minmax(240px,1fr);gap:1rem;align-items:center}
.mcp-card{background:rgba(128,128,128,.08);border:1px solid rgba(128,128,128,.22);border-radius:8px;overflow:hidden}
.mcp-card-label{padding:.55rem .75rem;border-bottom:1px solid rgba(128,128,128,.18);font-size:10px;letter-spacing:.1em;text-transform:uppercase;color:var(--text-muted-color,#888)}
.mcp-code{margin:0;padding:.85rem;font-size:12px;line-height:1.45;white-space:pre;overflow-x:auto;background:transparent;color:inherit}
.mcp-arrow{display:flex;flex-direction:column;align-items:center;gap:.4rem;color:#6e7fff;font-size:26px;font-weight:700;min-width:58px}
.mcp-id{font-size:10px;letter-spacing:.08em;text-transform:uppercase;color:#6e7fff;border:1px solid rgba(110,127,255,.5);border-radius:999px;padding:.2rem .5rem;background:rgba(110,127,255,.12)}
.mcp-flow{display:grid;grid-template-columns:minmax(190px,1fr) minmax(220px,1.25fr) minmax(190px,1fr);gap:1rem;align-items:center}
.mcp-node{border:1px solid rgba(128,128,128,.25);border-radius:8px;background:rgba(128,128,128,.08);padding:1rem;text-align:center;min-height:96px;display:flex;flex-direction:column;justify-content:center}
.mcp-node strong{display:block;font-size:14px;margin-bottom:.35rem}
.mcp-node span{font-size:11px;color:var(--text-muted-color,#888);line-height:1.4}
.mcp-lanes{border-radius:8px;background:rgba(110,127,255,.05);border:1px dashed rgba(128,128,128,.24);padding:1rem;display:grid;gap:.85rem}
.mcp-lane-row{display:grid;gap:.3rem}
.mcp-lane-label{font-size:10px;letter-spacing:.1em;text-transform:uppercase;color:var(--text-muted-color,#888)}
.mcp-packet{padding:.45rem .6rem;border-radius:6px;background:rgba(110,127,255,.12);border:1px solid rgba(110,127,255,.4);font-size:11px;line-height:1.4;color:inherit;white-space:nowrap;overflow:hidden;text-overflow:ellipsis;font-family:var(--font-family-monospace,monospace)}
.mcp-packet.response{background:rgba(127,212,154,.12);border-color:rgba(127,212,154,.5)}
.mcp-controls{display:flex;align-items:center;gap:.65rem;flex-wrap:wrap;margin-top:1rem}
.mcp-btn{padding:.5rem .85rem;border-radius:6px;border:1px solid #6e7fff;background:rgba(110,127,255,.12);color:inherit;font-family:inherit;font-size:13px;cursor:pointer;transition:background .15s}
.mcp-btn:hover,.mcp-btn.active{background:rgba(110,127,255,.28)}
.mcp-status{font-size:11px;color:var(--text-muted-color,#888)}
.mcp-http-response{margin-top:1rem;border:1px solid rgba(128,128,128,.22);border-radius:8px;background:rgba(128,128,128,.08);overflow:hidden}
.mcp-event-list{padding:.85rem;display:grid;gap:.55rem}
.mcp-event{border:1px solid rgba(127,212,154,.4);border-radius:6px;padding:.55rem .65rem;background:rgba(127,212,154,.08);font-size:12px;line-height:1.45;color:inherit}
.mcp-event.dim{border-color:rgba(110,127,255,.4);background:rgba(110,127,255,.08)}
.mcp-compare{display:grid;grid-template-columns:1fr 1fr;gap:1rem;margin-top:1rem}
.mcp-compare-card{border:1px solid rgba(128,128,128,.22);border-radius:8px;background:rgba(128,128,128,.08);padding:1rem}
.mcp-compare-card h3{font-size:14px;margin:0 0 .8rem}
.mcp-compare-card ul{margin:.5rem 0 0;padding-left:1.1rem;font-size:13px;line-height:1.65}
.mcp-invariant{border:1px solid rgba(110,127,255,.4);background:rgba(110,127,255,.08);border-radius:8px;padding:.75rem;font-size:12px;line-height:1.5}
@media (max-width: 760px){
  .mcp-pair,.mcp-flow,.mcp-compare{grid-template-columns:1fr}
  .mcp-arrow{transform:rotate(90deg);margin:.25rem 0}
}
</style>

<div class="mcp-wire-widget">
  <div class="mcp-title">JSON-RPC exchange — the invariant layer</div>
  <div class="mcp-pair">
    <div class="mcp-card">
      <div class="mcp-card-label">Claude Code → GitHub MCP server</div>
      <pre class="mcp-code"><code>{
  "jsonrpc": "2.0",
  "id": 7,
  "method": "tools/call",
  "params": {
    "name": "create_issue",
    "arguments": {
      "owner": "cotneok",
      "repo": "blog",
      "title": "MCP post draft"
    }
  }
}</code></pre>
    </div>
    <div class="mcp-arrow">→<span class="mcp-id">id: 7</span></div>
    <div class="mcp-card">
      <div class="mcp-card-label">GitHub MCP server → Claude Code</div>
      <pre class="mcp-code"><code>{
  "jsonrpc": "2.0",
  "id": 7,
  "result": {
    "content": [
      { "type": "text",
        "text": "Created issue #42" }
    ]
  }
}</code></pre>
    </div>
  </div>
</div>

Three things to notice in those two objects:

- **`id: 7`** — the matching number. Client picks it, server echoes it back. That's how the client knows which response belongs to which request when several are in flight at once.
- **`method: "tools/call"`** — *what* the client wants the server to do. MCP defines other methods too — for setup, tool discovery, prompts, resources, notifications, and more — but for this post we only need this one: `tools/call`.
- **`params`** — the arguments. Same shape as calling a function.

That's it. For this exchange, that's MCP, end to end. A request with an `id`, a response with the matching `id`. Two JSON objects.

The rest of this post is about how those two objects physically move from one process to another. Because that — and only that — is where stdio and Streamable HTTP differ.

---

## Wire 1: stdio

When you install an MCP server locally — an executable, a Python script, a Docker container Claude Code launches itself — the transport is usually `stdio`.

Quick definition, in case "stdio" is jargon: every program on your computer gets three built-in text channels — `stdin` for input, `stdout` for output, `stderr` for errors. "stdio" is just shorthand for "send the messages over those built-in channels." Nothing more exotic than that.

Here's what physically happens when Claude Code uses stdio:

1. Claude Code spawns the GitHub MCP server as a child process.
2. The OS wires the child's `stdin` and `stdout` to Claude Code.
3. Claude Code writes the request JSON to the child's `stdin`, terminated by `\n`.
4. The server reads one line from its own `stdin`, parses the JSON, runs `create_issue`, writes the response JSON to `stdout`, terminated by `\n`.
5. Claude Code reads one line from the child's `stdout` and matches `id: 7`.

<div class="mcp-wire-widget">
  <div class="mcp-title">stdio transport — parent process to child process</div>
  <div class="mcp-flow">
    <div class="mcp-node"><strong>Claude Code</strong><span>parent process<br>owns the pipes</span></div>
    <div class="mcp-lanes">
      <div class="mcp-lane-row">
        <div class="mcp-lane-label">stdin → request line</div>
        <div class="mcp-packet">{"jsonrpc":"2.0","id":7,"method":"tools/call",...}\n</div>
      </div>
      <div class="mcp-lane-row">
        <div class="mcp-lane-label">stdout → response line</div>
        <div class="mcp-packet response">{"jsonrpc":"2.0","id":7,"result":{...}}\n</div>
      </div>
    </div>
    <div class="mcp-node"><strong>GitHub MCP Server</strong><span>child process<br>reads stdin, writes stdout</span></div>
  </div>
</div>

And here's what one of those stdio frames actually looks like on the wire:

```
{"jsonrpc":"2.0","id":7,"method":"tools/call","params":{...}}\n
```

On stdio, this is literally one UTF-8 line. The `\n` is the frame delimiter — read until newline, parse the buffer as JSON, and you have a JSON-RPC message. No length-prefix, no fancy framing protocol. Once MCP's brief setup exchange at startup is out of the way, the transport really is this boring. That's it.

That's the whole transport. Two unidirectional byte streams between a parent and a child process. Newline-delimited JSON-RPC.

This is fast. There's no networking. No ports. No load balancer. Authentication is usually whatever environment variables the parent passed into the child — the GitHub MCP server, for example, just reads `GITHUB_TOKEN` out of its own environment when Claude Code spawns it.

It's also only local. There's no way for Claude Code on your laptop to "stdio into" a server running on someone else's machine. `stdio` requires a parent-child process relationship, and processes don't span hosts.

That's not an MCP limitation. That's how operating systems work.

So the moment you want a remote MCP server — one running in someone else's data center, not on your laptop — you need a different wire. The internet's default wire is HTTP. So MCP uses HTTP.

---

## Wire 2: Streamable HTTP

When the GitHub MCP server runs somewhere else — a hosted deployment, a container behind a load balancer, a third-party service — the transport has to be the network.

MCP's current answer is **Streamable HTTP**. The server exposes a single endpoint, usually `/mcp`. The client sends JSON-RPC messages by `POST`-ing them to that endpoint. The server can answer in one of two shapes — either a normal JSON body, or an **SSE stream**.

(SSE = **Server-Sent Events**, a long-standing web standard for keeping a single HTTP response open and trickling data through it event-by-event, like a live feed. We'll see why MCP uses it in a moment.)

Here's our same `create_issue` call over Streamable HTTP:

```http
POST /mcp HTTP/1.1
Host: github-mcp.example.com
Content-Type: application/json
Accept: application/json, text/event-stream
Mcp-Session-Id: 9f3c...

{"jsonrpc":"2.0","id":7,"method":"tools/call","params":{...}}
```

Two headers on this request are doing real work:

- **`Accept: application/json, text/event-stream`** — required by the spec. The client commits, up front, to handling either response shape. That's what gives the server license to pick.
- **`Mcp-Session-Id`** — if the server established a session during `initialize`, Claude Code echoes the id on every later request so the server can route it to the right session.

The server has two ways to respond.

Option A — a single JSON response:

```http
HTTP/1.1 200 OK
Content-Type: application/json

{"jsonrpc":"2.0","id":7,"result":{...}}
```

Option B — an SSE stream:

```http
HTTP/1.1 200 OK
Content-Type: text/event-stream

event: message
data: {"jsonrpc":"2.0","method":"notifications/progress",...}

event: message
data: {"jsonrpc":"2.0","id":7,"result":{...}}
```

<div class="mcp-wire-widget">
  <div class="mcp-title">Streamable HTTP — same request, two response shapes</div>
  <div class="mcp-flow">
    <div class="mcp-node"><strong>Claude Code</strong><span>HTTP client<br>sends POST /mcp</span></div>
    <div class="mcp-lanes">
      <div class="mcp-lane-row">
        <div class="mcp-lane-label">POST https://github-mcp.example.com/mcp</div>
        <div class="mcp-packet">{"jsonrpc":"2.0","id":7,"method":"tools/call",...}</div>
      </div>
      <div class="mcp-lane-row">
        <div class="mcp-lane-label" id="http-wire-label">response: application/json</div>
        <div class="mcp-packet response" id="http-wire-packet">200 OK {"jsonrpc":"2.0","id":7,...}</div>
      </div>
    </div>
    <div class="mcp-node"><strong>GitHub MCP Server</strong><span>remote host<br>behind an HTTP endpoint</span></div>
  </div>
  <div class="mcp-controls">
    <button class="mcp-btn active" id="http-json">Single JSON response</button>
    <button class="mcp-btn" id="http-sse">SSE stream</button>
    <span class="mcp-status" id="http-status">One HTTP response carries the matching JSON-RPC result.</span>
  </div>
  <div class="mcp-http-response">
    <div class="mcp-card-label" id="http-response-label">Content-Type: application/json</div>
    <div class="mcp-event-list" id="http-events">
      <div class="mcp-event dim">{"jsonrpc":"2.0","id":7,"result":{"content":[...]}}</div>
    </div>
  </div>
</div>

**Why two response shapes?** Because some tool calls finish instantly and some don't. Creating an issue is one quick answer — a single JSON body is fine. But running a long build, scraping a bunch of pages, or generating a big document might take half a minute, and the server might want to stream progress along the way. The `notifications/progress` line in the SSE example above is exactly that — the server saying *"I'm partway through"* mid-call. SSE lets one HTTP response carry that whole back-and-forth instead of forcing the client to poll.

Worth being precise on the part people most often tangle:

> **SSE is not the transport. Streamable HTTP is the transport.** SSE is one response shape inside Streamable HTTP, used when the server has more than one message to send back.
{: .prompt-info }

Small bonus, when servers opt in: if the server attaches SSE event ids to its messages, Claude Code can reconnect after a drop and send `Last-Event-ID`, and the server *may* replay whatever was missed. It's not automatic — it's a hook MCP leaves open for servers that need resumable streams.

(Historical footnote, in case you read older docs: an earlier MCP version used a separate "HTTP+SSE" transport. The 2025-03-26 spec revision replaced it with Streamable HTTP, which is what's described above. Modern MCP defines exactly two standard transports: `stdio` and Streamable HTTP. That's the whole list.)

---

## So What Actually Changed?

Look at what the two transports have in common, and what they don't.

<div class="mcp-wire-widget">
  <div class="mcp-title">Local MCP vs remote MCP — same protocol, different wire</div>
  <div class="mcp-invariant">
    <strong>This layer is invariant:</strong>
    <code>{"jsonrpc":"2.0","id":7,"method":"tools/call",...}</code>
    →
    <code>{"jsonrpc":"2.0","id":7,"result":{...}}</code>
  </div>
  <div class="mcp-compare">
    <div class="mcp-compare-card">
      <h3>stdio</h3>
      <ul>
        <li>Newline-delimited JSON over child-process pipes</li>
        <li>Parent spawns child</li>
        <li>Local only</li>
        <li>No network addressing</li>
        <li>Great for tools Claude Code runs on your machine</li>
      </ul>
    </div>
    <div class="mcp-compare-card">
      <h3>Streamable HTTP</h3>
      <ul>
        <li>JSON-RPC over an HTTP endpoint</li>
        <li>Usually a single `/mcp` route</li>
        <li>Works across hosts</li>
        <li>Can stream responses with SSE</li>
        <li>Great for hosted or multi-client deployments</li>
      </ul>
    </div>
  </div>
</div>

The JSON didn't change. The protocol didn't change. The tool didn't change. The thing on top of the wire is exactly the same in both pictures.

What changed was the wire.

This is why people get confused about "local MCP" and "remote MCP" as if they were different products. They aren't. The same MCP server idea can run under `stdio` for local Claude Code and under Streamable HTTP for a hosted deployment. Only the transport wiring differs.

---

## The Reframe

Once you can picture the wire, MCP stops being a platform and becomes plumbing.

There's no magic `MCPServer` class doing clever things. There's a process — or a host — that reads JSON-RPC frames off some channel, runs a function, writes a JSON-RPC frame back. Every "MCP integration" you'll ever read about is a variant of that.

If you followed this far, you now hold the whole protocol in your head:

- **JSON-RPC 2.0** is the language. Request, response, matched by `id`.
- **stdio** is for local servers. Client launches the server as a child process; they talk over `stdin`/`stdout`, one JSON message per line.
- **Streamable HTTP** is for remote servers. Client `POST`s JSON-RPC to `/mcp`. Server answers with either a single JSON body or an SSE stream (when there's more than one message to send back).

The choice between `stdio` and Streamable HTTP isn't a protocol decision. It's a deployment decision: *where does the server live?*

Local? `stdio`. Remote? HTTP. Same JSON either way.

That's the whole map.

---

If you want to see real frames flying past instead of taking my word for it, point the [MCP Inspector](https://github.com/modelcontextprotocol/inspector) at any MCP server and watch the actual JSON-RPC traffic in your browser. It's the fastest way to make this post stop being theory.

---

*Tsotne · tsotne.blog · AI engineering series, post #2*

<script>
(function(){
  var jsonBtn = document.getElementById('http-json');
  var sseBtn = document.getElementById('http-sse');
  var httpStatus = document.getElementById('http-status');
  var httpLabel = document.getElementById('http-response-label');
  var wireLabel = document.getElementById('http-wire-label');
  var wirePacket = document.getElementById('http-wire-packet');
  var events = document.getElementById('http-events');
  function setMode(mode){
    var sse = mode === 'sse';
    jsonBtn.classList.toggle('active', !sse);
    sseBtn.classList.toggle('active', sse);
    if (sse) {
      httpStatus.textContent = 'One HTTP response stays open and carries multiple SSE message events.';
      httpLabel.textContent = 'Content-Type: text/event-stream';
      wireLabel.textContent = 'response: text/event-stream';
      wirePacket.textContent = 'event: message ... event: message ...';
      events.innerHTML = [
        '<div class="mcp-event dim">event: message<br>data: {"jsonrpc":"2.0","method":"notifications/progress","params":{...}}</div>',
        '<div class="mcp-event">event: message<br>data: {"jsonrpc":"2.0","id":7,"result":{"content":[...]}}</div>'
      ].join('');
    } else {
      httpStatus.textContent = 'One HTTP response carries the matching JSON-RPC result.';
      httpLabel.textContent = 'Content-Type: application/json';
      wireLabel.textContent = 'response: application/json';
      wirePacket.textContent = '200 OK {"jsonrpc":"2.0","id":7,...}';
      events.innerHTML = '<div class="mcp-event dim">{"jsonrpc":"2.0","id":7,"result":{"content":[...]}}</div>';
    }
  }
  if (jsonBtn && sseBtn) {
    jsonBtn.onclick = function(){ setMode('json'); };
    sseBtn.onclick = function(){ setMode('sse'); };
  }
})();
</script>
