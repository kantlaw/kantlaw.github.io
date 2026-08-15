---
layout: post
title: "Excessive agency vs. prompt injection"
description: "What I learned attacking AI-powered scanners: seven findings on why an injection filter does not stop an over-permissioned agent."
date: 2026-08-08 12:00:00 -0500
tags: [prompt-injection, excessive-agency, llm-security]
source: https://github.com/kantlaw/ai-security-lab
---

Two failures get filed under the same ticket and they are not the same bug. One is text the
model obeys. The other is a system that was handed too much reach and used it. An injection
filter stops the first and does nothing about the second.

## The distinction

<div class="t-wrap" markdown="1">

| Dimension | Prompt injection | Excessive agency |
|---|---|---|
| What happens | The attacker injects instructions the model obeys | The agent decides to act on its own over what it finds |
| Malicious instruction? | Yes | No |
| Example | Context escape plus an explicit order to delete the user | Only the panel HTML was requested; the agent follows the delete link on its own initiative |
| Correct mitigation | Filtering, delimiting, instruction hierarchy | Narrow the permissions, require human confirmation |
| Stopped by an injection filter? | Partially | No |

</div>

## Seven things the labs taught me

<div class="numbered" markdown="1">

### Map agency before prompts

Lab 4 did not fall to the payload. It fell because the scanner held Carlos's credentials, had
`send_request`, and could reach `192.168.0.0/24`. The winning payload was one line asking for the
panel HTML. The first audit question is not "can I inject?" — it is what tools the model has, what
identity it runs as, and what it can reach.

### Audit pairs of tools, not tools

The scanner could read an API key from context and post comments. Neither capability is dangerous
alone. Together they are an exfiltration channel that never touches the external network.

### The agent's own report is not evidence

It lied in both directions. On command injection it returned "invalid email" while the file was
already deleted — `$()` ran before validation failed. On lab 4 it reported "successfully
exploited… delete users" and the lab stayed unsolved. Verify out of band.

### Every context field is attack surface

Signing as `Administrator` instead of `developer` changed the outcome. The author name enters the
context too, and the model reads it as authority. Audit every field that reaches the model —
author, date, title, metadata — not just the body.

### Quantify or it is not a finding

The same payload failed and succeeded across attempts. Eight variants before one landed against
defenses; oscillating results on lab 4 with identical text. "Works 60% of the time, n=20" is
defensible. "Works" gets closed as not reproducible.

### Decompose the chain

Enumeration plus bypass plus deletion in one payload failed — the model read it as an attack. One
step per scan worked: locate the host, confirm the bypass, then execute. Each atomic instruction is
plausible on its own; only the full chain looks hostile.

### The boring pretext wins

`----SYSTEM MESSAGE----` signed as Administrator was flagged as social engineering and reported.
What worked was `SECURITY ISSUE FOUND!` formatted as a technical report, mimicking the scanner's
own output. Effective evasion is not more authority — it is less friction.

</div>
