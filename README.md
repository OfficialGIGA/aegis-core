# Aegis Core

**An AI-orchestrated cyber-defense operations console you can open in a browser — built to learn and teach defensive security.**

Aegis Core is a single self-contained HTML file. Double-click it and it runs: a live defense dashboard with a local detection engine, an automated containment playbook, a Zero Trust posture view, a hands-on attack lab, and an optional AI analyst layer. No build step, no install, no server.

---

## What this is (and what it isn't)

This project is deliberately honest about its scope, because security tools that overstate themselves are worse than useless.

**What's real:**
- The **detection engine** — ten readable, auditable rules that score events for brute force, impossible travel, port scans, data exfiltration, privilege escalation, known-bad file hashes, lateral movement, DNS tunneling / C2, ransomware bursts, and admin logins from new countries.
- The **response logic** — a containment-only playbook (throttle, MFA-challenge, isolate) where high-impact actions wait for human approval.
- The **log parser** — paste real `sshd` auth logs or connection logs into the Telemetry tab and the *same* engine parses and scores them.
- The **teaching material** — every rule is explained in plain language with the defender's question it answers and the right next step.

**What's simulated:**
- The **live global feed.** A web page is sandboxed by the browser and cannot read your real network, OS, or "the internet." The live stream you see is a realistic traffic simulator with coherent attacker campaigns. It exists so the console has something to detect and teach against.
- The **AI analyst** lights up only where the Anthropic API is reachable. Opened straight from disk (`file://`), the API can't be called, so the analyst shows a graceful fallback. **This is by design** — detection and response never depend on the AI being available.

If you want a tool that genuinely monitors live infrastructure, you need real sensors (syslog collectors, netflow, an EDR/eBPF agent) running on the hosts themselves. The engine in this file is structured so that same logic could sit behind those real feeds — the simulator is just standing in for them.

---

## Quick start

1. Download `aegis-core.html`.
2. Double-click it. It opens in your default browser and starts running.

That's the whole setup.

### Optional: enable the AI analyst
The AI triage layer calls the Anthropic Messages API. To use it, serve the file from an environment where that endpoint is reachable and authenticated (for example, a small local proxy that injects your API key). Without it, everything else still works — the engine is the heartbeat, the AI is an analyst on top.

---

## The tabs

| Tab | What it does |
|---|---|
| **Operations** | The "are we okay right now?" view — system posture, severity counts, blocks applied, and recent activity. |
| **Alert stream** | Every detection, newest first. Click one for full evidence, the readable detection logic, AI analysis, and the response taken. Identical alerts collapse into one row with a count. Hover the list to pause it while you read. |
| **Response log** | Every containment action, its target, and whether it auto-applied or is awaiting your approval. |
| **Telemetry** | Paste real log lines and run them through the live engine. This is the seam to real data. |
| **Zero Trust** | Live maturity across the seven NIST / CISA pillars. |
| **Lab mode** | Trigger synthetic attacks safely and watch the engine catch them. Built for teaching. |
| **Learn** | Every detection rule opened up as plain-language logic — no black boxes. |
| **How it works** | The architecture and the design principles behind it. |

---

## Architecture

```
  Telemetry  ->  Local engine  ->  Auto-response
 (logs/feed)    (rules, always-on)  (throttle/challenge/isolate)
                      |
                      v
                AI triage layer   <- async, non-blocking. If it's down,
                (Claude analyst)     detection and response keep running.
```

The design rule is simple: **the part that protects you must never depend on a network call.** Detection and response are local and synchronous. The AI is called asynchronously, *after* an alert already exists, purely to explain and prioritize. Pull the network and the console keeps detecting and responding — you only lose the plain-English commentary until it returns.

---

## Design principles

- **Defense only.** Throttle, challenge, isolate, verify. There is no offensive capability anywhere in this project, by design.
- **Human in the loop.** High-impact actions like host isolation always wait for approval.
- **Auditable rules.** Every detection is readable logic, not an opaque model. Read it, question it, fork it, improve it.
- **Fail open for visibility.** If a single rule or the AI errors, the engine logs it and keeps watching everything else.
- **Honest scope.** The live global feed is simulated; the engine, rules, response logic, and log parsing are real.

---

## Extending it

The engine is a pure function over events, so it's easy to grow:

- **Add a detection rule:** append an object to the `RULES` array with an `evaluate(event, state)` function. If it returns a result, an alert fires. That's the whole contract.
- **Add a real-log parser:** extend `parseLogLine()` in the Telemetry path with a pattern for your log format (Windows Event logs, firewall syslog, JSON, etc.). The same rules score whatever you feed in.
- **Wire real responses:** the `PLAYBOOK` entries currently log proposed actions. In a real deployment these map to your firewall / EDR APIs.

---

## A note on teaching

This was built to be taught from. Open **Learn** to walk through how each detection works, then use **Lab mode** to fire a matching attack and watch it get caught in real time. The pairing — read the logic, then see it trigger — is the point.

---

## License

MIT. See `LICENSE`.

## Disclaimer

Aegis Core is an educational and demonstration tool. It is not a substitute for production security software, and its simulated feed does not monitor real systems. Use it to learn, teach, and prototype detection logic — not as your actual line of defense.
