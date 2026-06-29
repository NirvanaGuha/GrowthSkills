---
name: custom-ai-agent-builder
description: >
  Takes a task description, tool list, and brand voice and produces a fully
  configured system prompt, tool definitions, and agent scaffold ready to
  deploy on any platform (Claude, OpenAI, LangChain, n8n, Make, Zapier
  AI, custom API). Outputs are real, deployable artifacts — not diagrams or
  general advice. Two modes: Greenfield (design and build a new agent from
  scratch) and Audit+Rebuild (review an existing system prompt or agent config
  and harden it). Invokes brand-brain so the agent's voice and persona match
  the brand. Composes automation-workflow-designer-debugger for the surrounding
  trigger/routing logic; compose prompt-engineering-library-suite to version and
  iterate the resulting prompt over time. Use when the user says "build me an
  agent," "write a system prompt for," "configure an AI assistant that,"
  "I need a custom GPT / Claude agent / AI workflow for," "automate this
  task with AI," "deploy an agent," or hands over an existing system prompt and
  asks for improvements.
---

# Custom AI Agent Builder

Hand it a task. Get back a working agent — system prompt, tool definitions, and
a scaffold you can paste into Claude Projects, the Anthropic API, an OpenAI
Custom GPT, n8n, or any agent runtime. This is an engineering skill, not a
consulting deck: every output is a real artifact you can deploy in minutes, not
guidance you need to go interpret.

Our working framework underneath is **PACT** — Purpose, Actions, Constraints, Tone —
a house mnemonic: a minimal four-axis design space that forces every agent decision to be
deliberate. A senior operator works through PACT once per agent, then writes
the system prompt, tools, and test suite as a direct translation of those
decisions.

---

## Skills this calls

- **`brand-brain`** (required, Step 0) — resolves the active brand's voice,
  ICP, banned words, and proof so the agent persona is on-brand. Do not guess
  voice or write the system prompt before this returns.
- **`automation-workflow-designer-debugger`** (compose when asked) — designs
  the surrounding multi-step trigger / routing workflow that the agent lives
  inside. This skill owns the agent's internals; that skill owns the plumbing.
- **`prompt-engineering-library-suite`** (compose when asked) — versions and
  iterates the resulting system prompt over time, maintaining a regression-case
  library. Call after the initial build if the user wants a versioned library.

---

## How a run works

```
Step 0   Load the brand      ──► call brand-brain; get voice, ICP, banned words
Step 1   Pick the mode       ──► Greenfield | Audit+Rebuild
Step 2   PACT design pass    ──► resolve Purpose, Actions, Constraints, Tone
Step 3   Write the artifacts ──► system prompt + tool definitions + scaffold
Step 4   Generate test cases ──► 3 happy-path + 2 adversarial prompts
Step 5   Self-review + save  ──► quality checklist; save to ./agents/
```

---

## Step 0 — Load the brand (always first)

Invoke the `brand-brain` skill (Skill tool, `skill: brand-brain`), passing the
user's request and any named brand. It returns: voice adjectives, banned words,
offer mechanics, real proof, positioning, ICP. Obey voice + banned-words as
hard overrides; use only real proof (mark anything unconfirmed `[verify]`).

**Fallback if brand-brain is not installed:** read `~/.brandbrain/brands/.active`
and that brand's `brand.md` directly; if none exists, ask the user four
questions (product/service · ICP + awareness · 3 voice adjectives + banned
words · primary task the agent must perform), then proceed.

---

## Step 1 — Pick the mode

| Mode | Trigger |
|---|---|
| **Greenfield** (default) | New agent from scratch — task description in, full scaffold out. |
| **Audit+Rebuild** | Existing system prompt / config in — hardened version out with annotated change log. |

When input includes an existing system prompt, default to Audit+Rebuild and
offer a Greenfield alternative at the end. When input is purely descriptive,
default to Greenfield.

---

## Step 2 — PACT design pass (the framework)

Work through all four axes explicitly before writing a single line of system
prompt. Show the user the resolved PACT table so they can correct
misunderstandings before the build.

### P — Purpose
One crisp sentence: what job does this agent perform, for whom, and what does
success look like? If the user gives a vague goal ("help with support"), push
for the atomic job ("triage inbound support emails and draft first-response
copy in under 90 words, escalating billing issues to Tier 2"). Vague Purpose
produces vague agents.

### A — Actions
Enumerate every action the agent must be able to take. Split into:
- **LLM-only** (reasoning, drafting, classifying, extracting — no tools needed)
- **Tool-required** (web search, CRM read/write, API calls, code execution,
  file I/O) — each tool gets a name, a single-sentence description, and the
  input/output contract

Cap tools at what the task actually needs. Every extra tool is a hallucination
vector and a permission surface.

### C — Constraints
Explicit guardrails that ship inside the system prompt — not hopes:
- **Scope:** what the agent refuses to do, stated as "you do not [X]"
- **Output format:** JSON schema, max length, required fields, prohibited
  phrases (pull banned words from brand-brain)
- **Escalation rules:** when to stop and hand off to a human or another system
- **Data handling:** PII you don't echo, secrets you don't log
- **Fallback behavior:** when uncertain, say so + ask; never invent

### T — Tone
Pull from brand-brain's returned digest: voice adjectives, register (formal /
conversational / technical), persona name if the agent needs one. Translate into
two to three concrete system-prompt directives ("Write at a 10th-grade reading
level. Never use passive voice. Do not use any of these words: [banned list].").

---

## Step 3 — Write the artifacts

### 3a. System prompt
Write the complete system prompt using the PACT decisions above. Required
sections in order:

```
## Role & Purpose
[One-sentence job + persona name if applicable.]

## Behavior rules
[Numbered constraints from C — always enforced, no exceptions.]

## Tone & voice
[2–3 concrete directives from T; pull voice from brand-brain.]

## Actions & tools
[For each tool: name, when to call it, what to pass, what to do with the result.]

## Output format
[Exact format spec — JSON schema, markdown template, plain prose with length cap, etc.]

## Escalation & fallback
[Explicit trigger conditions; what to say when escalating or uncertain.]
```

Write the system prompt in the second person (addressing the model as "you").
Every rule is imperative. No hedging ("try to," "if possible") — state the
behavior as absolute.

### 3b. Tool definitions
For each tool-required action, produce a JSON tool definition in the format of
the target platform:

- **Anthropic / Claude API:** `tools[]` array with `name`, `description`,
  `input_schema` (JSON Schema)
- **OpenAI / Custom GPT:** same structure; note any `strict: true` requirement
- **n8n / Make / Zapier AI:** map to HTTP Request node or native integration;
  specify endpoint, auth type, request body schema, response mapping
- If platform is unstated, default to Anthropic tool-use format and note it

Keep descriptions under 60 words. The model reads these at inference time;
every word counts.

### 3c. Agent scaffold
A minimal runnable configuration the user can copy into their platform:

```
Platform: [Anthropic API | OpenAI | Claude Projects | n8n | Make | other]
Model: [recommended model ID and why — check the claude-api skill or current
        docs if unsure; mark with [verify] if uncertain on latest IDs]
Temperature: [0.0–0.3 for deterministic tasks; 0.5–0.7 for creative]
Max tokens: [task-appropriate cap]
System prompt: [paste the Step 3a output here]
Tools: [paste the Step 3b JSON here]
Memory / context strategy: [stateless | rolling window N turns | external
        vector store; brief rationale]
```

For Greenfield, the scaffold is a ready-to-paste block. For Audit+Rebuild,
the scaffold replaces the submitted config with annotated diffs.

---

## Step 4 — Test cases

Generate five prompts to validate the agent before deploying:
- **3 happy-path:** cover the core use cases; verify the output format and tone
- **2 adversarial:** one out-of-scope request the agent should refuse; one
  ambiguous input that should trigger the fallback/escalation rule

For each, write: the input, the expected output shape, and the specific PACT
rule it validates. These become the regression suite if the user composes
`prompt-engineering-library-suite`.

---

## Step 5 — Save

Save all artifacts to `./agents/<agent-slug>/`:
- `system-prompt.md` — the full system prompt
- `tools.json` — all tool definitions
- `scaffold.md` — the platform scaffold block
- `test-cases.md` — the five test prompts + expected outputs

Confirm the save path and offer to compose `prompt-engineering-library-suite`
for versioning.

---

## Principles (Non-Negotiable)

- **PACT before prose.** Never write a single line of system prompt before
  resolving all four PACT axes. Unexamined purpose = hallucination-prone agents.
- **Brand-brain first.** The agent's tone and voice come from the brand, not
  from defaults. Every banned word in the brand is a banned word in the agent.
- **Explicit over hopeful.** State every constraint as an absolute imperative.
  "Never" and "always" outperform "try to" and "if possible."
- **Minimum viable tool surface.** Add tools only when LLM-only reasoning
  cannot do the job. Every extra tool is an attack vector.
- **Real artifacts.** The output is deployable code/config, not advice. If the
  user can't paste it directly into their platform, keep writing.
- **Truth only.** Use real proof and real platform behavior; `[verify]` anything
  uncertain (model IDs, API limits, pricing). Do not invent capabilities.

---

## What Not to Do

- Do not write the system prompt before `brand-brain` returns (or the fallback
  mini-setup completes).
- Do not produce diagrams, slide decks, or narrative memos as the primary
  output — the deliverable is the config, not a plan about a config.
- Do not redesign the surrounding workflow or pipeline — that is
  `automation-workflow-designer-debugger`'s job. Call it; don't duplicate it.
- Do not add tools speculatively. If the task can be done in the system prompt
  alone, do not add a tool.
- Do not hallucinate platform-specific behavior (rate limits, context windows,
  tool-calling quirks). Mark uncertain platform facts `[verify]`.
- Do not commit secrets, API keys, or real credentials into any artifact.
  Use `<YOUR_API_KEY>` placeholders.

---

## Quality Checklist (self-review before presenting)

- `brand-brain` invoked and the brand digest received before any artifact was
  written?
- PACT table shown to the user (or confirmed internally) before the system
  prompt was drafted?
- System prompt has all six sections; every rule is imperative (no hedging)?
- Tool definitions have name, description under 60 words, and a complete
  input/output schema?
- Scaffold specifies model, temperature, max tokens, system prompt, tools, and
  memory strategy?
- Five test cases present (3 happy-path + 2 adversarial), each with expected
  output and the PACT rule it validates?
- All artifacts saved to `./agents/<agent-slug>/`?
- No invented proof, no hardcoded secrets, uncertain platform facts marked
  `[verify]`?
