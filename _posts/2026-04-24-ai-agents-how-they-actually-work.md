---
title: "AI Agents — What Actually Happens When 'Claude Calls a Tool'"
date: 2026-04-24 00:00:00 +0000
categories: [AI Engineering, Agents]
tags: [ai-agents, llm, claude, tool-use, react, python]
description: LLMs don't execute code. They generate text. So when an agent "checks the weather" or "calls an API", something else is doing the work. Here's the loop behind every agent framework — built from scratch, with an interactive walkthrough.
---

Most of what I build in my own time is AI agents — MCP servers, Claude-powered pipelines, tools that do real work. The single thing that changed how I think about all of it was realising something very uncomfortable:

**LLMs don't do anything. They generate text. That's the entire capability surface.**

So when an "agent" books a flight, queries a database, or runs a command — that's not the model doing those things. Something else is doing them. An invisible layer you've probably never thought about carefully.

This post is about that layer. The actual machinery behind every agent system you've ever used — LangChain, Claude's tool-use API, OpenAI function calling, Cursor, Claude Code. Same loop, every time. Once you see it, every framework becomes transparent.

Let me walk through it the way it finally clicked for me.

---

## The Wall — Text In, Text Out

Try asking a base LLM:

> What's the weather in Paris right now?

A model trained on the internet knows what weather is, where Paris is, maybe what the weather looked like on some day inside its training cut-off. But **right now**? No chance. Not because it's dumb — because it **can't**.

An LLM takes text and produces text. Full stop. It cannot:

- Make HTTP requests
- Query a database
- Execute a function
- Read a file
- Check the clock

It receives tokens, predicts tokens, returns tokens. Everything else is outside its reach.

This is the wall. Agents exist to go around it.

---

## The Trick — Orchestration, Not Magic

Here's what actually happens when an "agent" answers that weather question:

1. The LLM generates text that **describes** a tool call — something like `{"tool": "get_weather", "args": {"city": "Paris"}}`.
2. A framework **parses** that text and **stops** the model before it keeps generating.
3. The framework **calls the real API**.
4. The framework **injects the result** back into the conversation as new text.
5. The LLM reads the updated context and **generates a natural-language answer**.

The user sees a fluid conversation. What actually happened was **structured text → parse → real execution → result back as text → next prediction**.

```
User question
     ↓
  [LLM]  →  '{"tool":"get_weather","args":{"city":"Paris"}}'
     ↓
  [Framework parses + validates]
     ↓
  [Real function executes]   →   "Sunny, 18°C"
     ↓
  [Result injected as text]
     ↓
  [LLM reads context, answers]
     ↓
  "It's sunny and 18°C in Paris."
```

> **The LLM is the brain. The framework is the body.**  
> The model decides *what* to do. The framework does it. Neither half works alone.
{: .prompt-tip }

That separation — reasoning on one side, execution on the other — is the whole game. Every agent framework is a different dressing on this same mechanism.

---

## The ReACT Loop

Every agent I've built or read follows the same fundamental pattern. It's called **ReACT** (Reasoning + Acting):

```
Thought:      What do I need to do next?
Action:       Call a specific tool with specific arguments.
Observation:  Here's what the tool returned.
[Repeat until the task is complete]
```

The cycle continues until the model stops emitting tool calls and just answers.

Here's the step-by-step for *"What's the weather in London and Paris?"* — click through and watch each phase:

<style>
.react-widget{background:var(--card-bg,rgba(128,128,128,.05));border:1px solid var(--border-color,rgba(128,128,128,.2));border-radius:12px;padding:1.5rem;margin:1.5rem 0;font-family:var(--font-family-monospace,monospace);color:var(--text-color,inherit);overflow-x:auto}
.react-title{font-size:11px;color:var(--text-muted-color,#888);letter-spacing:.12em;text-transform:uppercase;margin-bottom:1rem}
.react-stage{display:flex;gap:6px;margin-bottom:1rem;flex-wrap:wrap}
.react-pill{padding:.35rem .7rem;border-radius:999px;font-size:10px;letter-spacing:.08em;text-transform:uppercase;background:var(--body-bg,rgba(128,128,128,.1));border:1px solid var(--border-color,rgba(128,128,128,.2));color:var(--text-muted-color,#888);transition:all .2s}
.react-pill.active{background:rgba(110,127,255,.85);border-color:#8fa0ff;color:#fff}
.react-pill.done{background:rgba(110,127,255,.15);border-color:rgba(110,127,255,.4);color:#8fa0ff}
.react-body{min-height:110px;padding:1rem;border-radius:8px;background:var(--body-bg,rgba(128,128,128,.08));margin-bottom:1rem;font-size:13px;line-height:1.55}
.react-body .label{font-size:10px;letter-spacing:.1em;text-transform:uppercase;color:var(--text-muted-color,#888);margin-bottom:.5rem;display:block}
.react-body .text{white-space:pre-wrap;word-break:break-word}
.react-controls{display:flex;gap:8px;align-items:center;flex-wrap:wrap}
.react-btn{padding:.5rem 1rem;border-radius:6px;border:1px solid #6e7fff;background:rgba(110,127,255,.15);color:inherit;font-family:inherit;font-size:13px;cursor:pointer;transition:all .15s}
.react-btn:hover{background:rgba(110,127,255,.3)}
.react-btn:disabled{opacity:.4;cursor:not-allowed}
.react-btn.ghost{border-color:var(--border-color,rgba(128,128,128,.3));background:transparent}
.react-step{font-size:11px;color:var(--text-muted-color,#888);margin-left:auto}
.react-log{margin-top:1rem;padding:.8rem 1rem;border-radius:8px;background:var(--body-bg,rgba(128,128,128,.05));border:1px dashed var(--border-color,rgba(128,128,128,.2));font-size:12px;line-height:1.6;max-height:220px;overflow-y:auto}
.react-log-entry{margin-bottom:.5rem;display:flex;gap:.6rem;align-items:flex-start}
.react-log-entry .src{font-size:9px;letter-spacing:.1em;text-transform:uppercase;color:var(--text-muted-color,#888);min-width:52px;padding-top:2px}
.react-log-entry.user .msg{color:#8fbfff}
.react-log-entry.llm .msg{color:#c8a6ff}
.react-log-entry.tool .msg{color:#7fd49a}
.react-log-entry.final .msg{color:#ffc87f;font-weight:600}
.react-empty{color:var(--text-muted-color,#888);font-style:italic}
</style>

<div class="react-widget">
  <div class="react-title">ReACT Loop — Step through a real agent run</div>
  <div class="react-stage" id="react-stage"></div>
  <div class="react-body" id="react-body"></div>
  <div class="react-controls">
    <button class="react-btn" id="react-next">Start ▶</button>
    <button class="react-btn ghost" id="react-reset">Reset</button>
    <span class="react-step" id="react-step">0 / 11</span>
  </div>
  <div class="react-log" id="react-log"><div class="react-empty">Conversation history will build here as the loop runs.</div></div>
</div>

<script>
(function(){
  var steps = [
    {stage:'user', label:'User asks', text:"What's the weather in London and Paris?", log:{src:'USER', cls:'user', text:"What's the weather in London and Paris?"}},
    {stage:'llm', label:'LLM generates text', text:'Thought: I need the weather in London first.\n{"tool":"get_weather","arguments":{"location":"London"}}', log:{src:'LLM', cls:'llm', text:'{"tool":"get_weather","arguments":{"location":"London"}}'}},
    {stage:'llm', label:'Stop sequence fires', text:'The stop sequence "Observation:" triggers.\nGeneration halts *before* the model can invent a fake result.'},
    {stage:'parse', label:'Framework parses', text:'JSON.parse succeeds.\ntool = "get_weather", args = {location: "London"}.\nAllowlist check passes.'},
    {stage:'exec', label:'Real tool executes', text:'get_weather("London") → "Partly cloudy, 12°C"', log:{src:'TOOL', cls:'tool', text:'get_weather("London") → "Partly cloudy, 12°C"'}},
    {stage:'inject', label:'Result injected', text:'Observation: Partly cloudy, 12°C\n↳ appended to the conversation. LLM is called again.'},
    {stage:'llm', label:'LLM generates again', text:'Thought: Got London. Need Paris next.\n{"tool":"get_weather","arguments":{"location":"Paris"}}', log:{src:'LLM', cls:'llm', text:'{"tool":"get_weather","arguments":{"location":"Paris"}}'}},
    {stage:'exec', label:'Real tool executes', text:'get_weather("Paris") → "Sunny, 18°C"', log:{src:'TOOL', cls:'tool', text:'get_weather("Paris") → "Sunny, 18°C"'}},
    {stage:'inject', label:'Result injected', text:'Observation: Sunny, 18°C\n↳ appended. LLM is called again.'},
    {stage:'llm', label:'LLM generates final answer', text:"London is partly cloudy at 12°C. Paris is sunny at 18°C.", log:{src:'LLM', cls:'final', text:"London is partly cloudy at 12°C. Paris is sunny at 18°C."}},
    {stage:'done', label:'No tool call → loop ends', text:'The framework sees plain text (no JSON), stops looping, returns the answer to the user.'}
  ];
  var stages = [
    {key:'user',   label:'User'},
    {key:'llm',    label:'LLM'},
    {key:'parse',  label:'Parse'},
    {key:'exec',   label:'Execute'},
    {key:'inject', label:'Inject'},
    {key:'done',   label:'Done'}
  ];
  var idx = -1;
  var stageEl = document.getElementById('react-stage');
  var bodyEl  = document.getElementById('react-body');
  var stepEl  = document.getElementById('react-step');
  var logEl   = document.getElementById('react-log');
  var nextBtn = document.getElementById('react-next');
  var resetBtn= document.getElementById('react-reset');
  function esc(s){return s.replace(/&/g,'&amp;').replace(/</g,'&lt;').replace(/>/g,'&gt;');}
  function renderStages(currentKey){
    stageEl.innerHTML = '';
    var seenCurrent = false;
    stages.forEach(function(s){
      var el = document.createElement('div');
      el.className = 'react-pill';
      if (s.key === currentKey) { el.className += ' active'; seenCurrent = true; }
      else if (!seenCurrent && currentKey) { el.className += ' done'; }
      el.textContent = s.label;
      stageEl.appendChild(el);
    });
  }
  function render(){
    if (idx < 0) {
      renderStages('');
      bodyEl.innerHTML = '<span class="label">Ready</span><div class="text">Click "Start" to simulate a full ReACT agent loop answering a two-city weather question. Watch how the LLM and the framework hand off to each other.</div>';
      stepEl.textContent = '0 / ' + steps.length;
      logEl.innerHTML = '<div class="react-empty">Conversation history will build here as the loop runs.</div>';
      nextBtn.textContent = 'Start ▶';
      nextBtn.disabled = false;
      return;
    }
    var s = steps[idx];
    renderStages(s.stage);
    bodyEl.innerHTML = '<span class="label">' + esc(s.label) + '</span><div class="text">' + esc(s.text) + '</div>';
    stepEl.textContent = (idx + 1) + ' / ' + steps.length;
    logEl.innerHTML = '';
    var any = false;
    for (var i = 0; i <= idx; i++) {
      if (steps[i] && steps[i].log) {
        any = true;
        var e = document.createElement('div');
        e.className = 'react-log-entry ' + steps[i].log.cls;
        e.innerHTML = '<span class="src">' + steps[i].log.src + '</span><span class="msg">' + esc(steps[i].log.text) + '</span>';
        logEl.appendChild(e);
      }
    }
    if (!any) logEl.innerHTML = '<div class="react-empty">Conversation history will build here as the loop runs.</div>';
    logEl.scrollTop = logEl.scrollHeight;
    if (idx >= steps.length - 1) {
      nextBtn.disabled = true;
      nextBtn.textContent = 'Complete ✓';
    } else {
      nextBtn.disabled = false;
      nextBtn.textContent = 'Next step ▶';
    }
  }
  nextBtn.onclick = function(){ if (idx < steps.length - 1) { idx++; render(); } };
  resetBtn.onclick = function(){ idx = -1; render(); };
  render();
})();
</script>

Here's the insight most people miss:

> The "thoughts" aren't hidden cognition inside the model. They're just tokens appended to the conversation. On every turn the model re-reads the entire history — its own previous thoughts, tool calls, and observations — and uses that text to decide what to do next.
{: .prompt-info }

There is no hidden scratch pad. The reasoning *is* the conversation. That's why long-running agents blow out their context window — every iteration carries the whole trail forward.

Which means **prompt design isn't cosmetic**. You're shaping the text the model will later rely on to make decisions. A loose system prompt produces confused loops and wrong tool choices. A disciplined one produces reasoning chains that actually converge.

---

## Building the Smallest Agent That Works

Let's build one end-to-end so you can see every piece. Python, one file, a fake weather API. Every agent framework in the world does some version of this under its abstractions.

### 1. Define real tools

```python
def get_weather(location: str) -> str:
    """Current weather for a location."""
    fixtures = {
        "London": "Partly cloudy, 12°C",
        "Paris":  "Sunny, 18°C",
        "Tokyo":  "Rainy, 15°C",
    }
    return fixtures.get(location, "Weather data not available")
```

Nothing magical — just a Python function. The LLM will **never run this code**. It will generate text that asks for it to run.

### 2. Describe the tools to the model

```python
import inspect

def describe(func):
    sig = inspect.signature(func)
    params = [f"{n}: {p.annotation.__name__}" for n, p in sig.parameters.items()]
    return (
        f"Tool: {func.__name__}\n"
        f"Description: {func.__doc__.strip()}\n"
        f"Parameters: {', '.join(params)}"
    )

tools = {"get_weather": get_weather}
tool_docs = "\n\n".join(describe(f) for f in tools.values())
```

This produces the text the model actually sees — it never reads your Python source.

### 3. Write a system prompt that teaches the loop

```python
SYSTEM_PROMPT = f"""You are an agent that can use tools.

Available tools:
{tool_docs}

To use a tool, output JSON in this exact format:
{{"tool": "<name>", "arguments": {{"<param>": "<value>"}}}}

After each tool call you will receive an Observation with the result.
Then decide: use another tool, or output a final answer in plain text.
"""
```

### 4. The hallucination trap

This is where people get burned. If you let the model keep generating after it outputs a tool call, it will **invent the observation**:

```
{"tool":"get_weather","arguments":{"location":"London"}}
Observation: Partly cloudy, 12°C    ← fabricated by the model
```

The model has seen thousands of ReACT examples in training. It knows `Action` is followed by `Observation`, so it just writes one. Plausible-looking. Entirely fake.

The fix is a **stop sequence** — tell the API to halt generation the moment it hits `"Observation:"`:

```python
import anthropic
client = anthropic.Anthropic()

def call_llm(messages):
    resp = client.messages.create(
        model="claude-sonnet-4-6",
        max_tokens=1024,
        system=SYSTEM_PROMPT,
        messages=messages,
        stop_sequences=["Observation:"],
    )
    return resp.content[0].text
```

> Stop sequences aren't a performance tweak — they're a **correctness boundary**. Without them the model will fabricate tool outputs that look identical to real ones, and you will debug it for hours wondering why the answers are slightly wrong.
{: .prompt-warning }

### 5. The loop itself

```python
import json

def run_agent(query, max_iters=8):
    messages = [{"role": "user", "content": query}]
    for _ in range(max_iters):
        out = call_llm(messages).strip()

        # Plain text → final answer, stop looping
        if not out.startswith("{"):
            return out

        # Parse and execute the tool call
        call = json.loads(out)
        name, args = call["tool"], call["arguments"]
        if name not in tools:
            result = f"Error: unknown tool '{name}'"
        else:
            result = tools[name](**args)

        # Inject the *real* observation back
        messages.append({"role": "assistant", "content": out})
        messages.append({"role": "user", "content": f"Observation: {result}"})

    return "Max iterations reached."
```

That's the whole thing. Forty lines of Python. Every production agent framework is fundamentally this loop with more error handling around it.

---

## Security — Where This Stops Being Safe

The moment your tools touch the real world, your agent stops being a chat toy and becomes a process with privileges. I do volunteer vulnerability research outside work, and this is the part I see underestimated most.

Three rules I follow on every agent I ship:

1. **Don't trust the LLM's output.** The model might emit `delete_all_users()` because a prompt injection buried in some scraped page told it to. Validate tool calls against an allowlist *before* execution. Schema-check the arguments.
2. **Treat tool outputs as untrusted input.** If a tool reads from anywhere a user can reach — web pages, PDFs, emails, tickets — that content can contain instructions for the next turn. The model will happily read them as authoritative.
3. **Gate the dangerous stuff.** Tools that delete, send, or spend money should require explicit human approval or sit behind a capability token. Default every new tool to read-only and promote explicitly.

> Prompt injection is the XSS of the agent era. Same pattern: untrusted input reaching an execution context that treats it as trusted. Same fix: separate the two, hard.
{: .prompt-danger }

---

## What the Frameworks Actually Hide

Now you know what LangChain, LangGraph, smolagents, CrewAI — plus the native tool-use APIs from Anthropic, OpenAI, and Google — are really doing under the hood. They automate:

| Concern | What they do for you |
|---|---|
| Tool registration | Turn Python functions into LLM-readable schemas |
| System prompt | Inject tool descriptions + ReACT workflow |
| Structured output | Return validated JSON, not raw text to parse |
| Stop handling | Halt generation at the right boundary |
| Execution | Dispatch to the real function |
| History | Append observations to conversation state |
| Loop control | Iterate until the model stops calling tools |

Native tool-calling APIs are **standardised parsing**. They give you validated JSON calls and handle the stop-sequence dance. They do **not** build the loop, execute your tools, or manage the conversation — that's still your code.

That's freeing once you see it. When your agent loops forever, picks the wrong tool, or misses that it's done — you know exactly which part of the loop to look at:

- **Loops forever?** → Termination logic in the system prompt.
- **Wrong tool?** → Tool descriptions are too vague, or too similar to each other.
- **Hallucinated results?** → Stop sequence isn't firing, or you're not using one.
- **Garbage arguments?** → Schema validation is missing between parse and execute.

---

## The Mental Model That Sticks

Everything in this post reduces to one sentence:

> **Agents don't execute. They describe. A framework executes the description.**

LLM generates structured text. Framework parses, validates, executes, captures the result, injects it back. Repeat until the model stops emitting tool calls.

That's it. That's every agent. Multi-agent systems are multiple loops passing text between each other. Autonomous agents are loops whose tools spawn more loops. Cursor editing your code, Claude Code running shell commands, a research agent browsing the web — same pattern, different tools.

The pattern scales because it's boring. Getting the prompts right, handling edge cases, managing context, containing the blast radius — those are the hard parts. The mechanism itself is just disciplined text manipulation with a parser in the middle.

---

## What to Do With This

- **Using a framework?** Read its source once. You'll never be confused by it again.
- **Building your own?** Start from the forty-line version above. Add only what you actually need.
- **Shipping to production?** Treat the tool boundary as a security perimeter — because it is.

Next post in this series: prompt caching, and how to stop paying for the same tokens twenty times. I've been running real workloads on Claude for a while and the cost delta from getting caching right is not small.

If something didn't click or you've hit a failure mode I haven't covered, drop it in the comments — those are exactly the prompts worth expanding next.

---

*Tsotne · tsotne.blog · AI engineering series, post #1*
