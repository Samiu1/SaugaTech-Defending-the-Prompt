---
name: ai-red-team
description: Threat-model an AI feature, agent, or LLM application for security and misuse risks. Use whenever the user wants to red-team, security-review, threat-model, or audit an AI system. Trigger on phrases like "red team this", "what could go wrong with this agent", "prompt injection risk", "lethal trifecta", "tool misuse", "jailbreak", "AI security review", "guardrails check". Especially relevant when reviewing agentic features that combine untrusted input, sensitive data access, and external action capability. Push to use this skill any time an AI feature touches user input plus tools plus data.
---

# AI Red Team

Structured threat modeling for AI features. Built around the "lethal trifecta" framing: untrusted input + sensitive data access + external action capability.

## When to use

- Pre-launch security review of an AI feature
- Adding tools or data sources to an existing agent
- Investigating an incident
- Preparing a talk or doc on AI security risks
- Vendor evaluation (is this third-party agent safe to integrate?)

## Output

A markdown threat model with the sections below. Each identified risk gets a severity, likelihood, and mitigation owner.

## Workflow

### Step 1: Map the system

Draw or list:
- **Input sources**: where does data enter the model context? (user input, retrieved docs, tool outputs, web pages, emails, files)
- **Trust level** of each source: trusted (your DB), semi-trusted (your users), untrusted (the open web, user-uploaded files)
- **Data the model can read**: what's in scope, what's sensitive
- **Tools the model can call**: with what permissions, against what systems
- **Outputs**: where the model's response goes (user UI, API, automated action, downstream system)

### Step 2: Check the lethal trifecta

The system is high-risk if it has all three:
1. Processes untrusted input
2. Has access to private/sensitive data
3. Can communicate externally (tool calls, network, sending messages)

Any AI system with all three needs strong mitigations. Document which apply.

### Step 3: Walk the threat categories

For each, ask: can this happen? how bad? what stops it?

#### Prompt injection
- Direct: user types "ignore previous instructions"
- Indirect: malicious instructions hidden in a retrieved doc, web page, email, image, file metadata
- Multi-step: instructions accumulate across tool calls
- **Test**: can you make the agent do something the system prompt forbids by putting instructions in a document it reads?

#### Data exfiltration
- Sensitive data leaked in model output
- Sensitive data sent to attacker-controlled URL via tool call
- Sensitive data embedded in image/markdown that triggers a fetch
- Cross-tenant leakage (user A sees user B's data)
- **Test**: can you craft an input that makes the model reveal or transmit data it shouldn't?

#### Tool / action misuse
- Model called the wrong tool
- Model called the right tool with wrong arguments
- Model chained tools in an unintended way
- Model performed an irreversible action without confirmation
- **Test**: can you make the agent delete, send, pay, or publish without explicit user consent?

#### Jailbreak / policy bypass
- Roleplay attacks
- Encoding attacks (base64, leetspeak, foreign language)
- Persona injection
- Multi-turn priming
- **Test**: can you get the model to produce content it should refuse?

#### Denial of service / cost attacks
- Token bombs (huge inputs)
- Recursive tool calls
- Infinite agent loops
- Expensive model calls triggered by free user inputs
- **Test**: what's the most expensive single request a free user can trigger?

#### Supply chain
- Compromised model provider
- Compromised prompt template / config
- Compromised tool / MCP server
- Compromised retrieval source
- Dependency on a third-party agent framework with known CVEs
- **Test**: if a dependency was malicious, what's the blast radius?

#### Identity and authorization confusion
- Model acts on behalf of user A using user B's permissions
- Model accepts authorization claims from untrusted input
- Tool calls inherit broader permissions than the user has
- **Test**: does every tool call carry the right user context?

### Step 4: Score and prioritize

For each risk found:

| Field | Values |
|---|---|
| Severity | Critical / High / Medium / Low |
| Likelihood | Likely / Possible / Unlikely |
| Detectability | Detected / Late / Never |
| Mitigation | What stops it now |
| Gap | What's still missing |
| Owner | Name |
| Deadline | Date |

Critical + Likely = ship blocker.

### Step 5: Mitigations checklist

Standard defenses to consider:

- [ ] Tagged trust boundaries on all inputs
- [ ] System prompt cannot be overridden (tested)
- [ ] Tool calls require explicit user confirmation for destructive actions
- [ ] Output filters for sensitive patterns (PII, secrets, URLs to external domains)
- [ ] Egress allowlist on tool calls
- [ ] Per-user rate limits
- [ ] Per-user cost ceilings
- [ ] Audit log of every tool call
- [ ] Kill switch
- [ ] Incident runbook
- [ ] Eval cases for every known attack pattern (and they run in CI)

### Step 6: Write the report

Sections:
1. System summary (one paragraph)
2. Trust boundary diagram (or list)
3. Lethal trifecta assessment
4. Risks found (table from step 4)
5. Mitigations in place
6. Mitigations recommended
7. Ship recommendation (ship / ship with conditions / do not ship)

## References to internalize

- The lethal trifecta framing (Simon Willison)
- OWASP Top 10 for LLM Applications
- Recent prompt injection research and CVEs in agent frameworks
