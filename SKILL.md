---
name: war-room
description: Multi-agent war room for brainstorming, system design, architecture review, product specs, business strategy, or any complex problem. Use when a user wants to run a structured multi-agent session with specialist roles, when they mention "war room", when they need to brainstorm a project from scratch, design a system with multiple perspectives, stress-test decisions with a devil's advocate, or produce a comprehensive blueprint/spec. Works for software, hardware, content, business — any domain.
---

# War Room

A methodology for running multi-agent brainstorming and execution sessions. Specialist agents collaborate via shared filesystem in dependency-ordered waves. A CHAOS agent (devil's advocate) shadows every wave. Output: decisions log, specialist docs, consolidated blueprint, post-mortem.

## Quick Start

1. **Initialize:** Run `bash skills/war-room/scripts/init_war_room.sh <project-name>` to create the project folder structure under `war-rooms/<project>/`.
2. **Brief:** Fill in `war-rooms/<project>/BRIEF.md` with the project description, goals, constraints, and known risks.
3. **Inject DNA:** Copy `skills/war-room/references/dna-template.md` → `war-rooms/<project>/DNA.md`. Customize if needed (add project-specific identity, owner name).
4. **Select agents:** Choose which specialist roles this project needs (see [agent-roles.md](references/agent-roles.md)). Not every project needs all roles.
5. **Run waves:** Execute the wave protocol below. Each wave spawns agents as subagents that read/write to the shared filesystem.
6. **Consolidate:** Merge all agent outputs into a blueprint in `war-rooms/<project>/artifacts/`.
7. **Post-mortem:** Write lessons to `war-rooms/<project>/lessons/`.

## The Wave Protocol

Full protocol details: [wave-protocol.md](references/wave-protocol.md)

### Wave 0: Prove It (mandatory)

Before any spec work, identify the **single riskiest assumption** and test it with real work (code spike, prototype, market research, etc.). 30 min max. If it fails, pivot BEFORE spending tokens on detailed specs.

### Waves 1–N: Specialist Execution

Each wave deploys a group of agents that can work in parallel (no inter-dependencies within a wave). Agents in later waves depend on earlier waves' outputs.

**Planning a wave:**
1. List all agents needed for the project
2. Build a dependency graph (who needs whose output?)
3. Group agents with no mutual dependencies into the same wave
4. Order waves by dependency

**Each agent in a wave:**
- Reads: `BRIEF.md`, `DNA.md`, `DECISIONS.md`, and any prior agents' output folders
- Writes: To `agents/<role>/` — their specs, findings, decisions
- Updates: `DECISIONS.md` (their domain decisions), `STATUS.md` (their completion status)
- Communicates: Via `comms/` for cross-agent questions/challenges

**Spawning agents:** Each agent is a subagent. Its system prompt includes:
- The DNA (from `DNA.md`)
- Its role briefing (from [agent-roles.md](references/agent-roles.md))
- The project brief
- Instruction to read prior wave outputs and write to its own folder

### Pivot Gate (between every wave)

Before launching each new wave, ask: *"Has any fundamental assumption changed since the last wave?"*
- If YES → affected agents from prior waves must re-evaluate. Mark voided decisions as `**VOIDED**` in `DECISIONS.md`.
- If NO → proceed.

### CHAOS Shadows Every Wave

CHAOS is not a separate wave — it **shadows all waves**. After each wave completes, CHAOS:
1. Reads every agent's output from that wave
2. Files challenges to `agents/chaos/challenges.md`
3. Format: `[C-ID] CHALLENGE to D### — attack — verdict (SURVIVE/WOUNDED/KILLED)`
4. WOUNDED = valid concern, needs mitigation. KILLED = decision must be reversed.

CHAOS also writes counter-proposals when it sees a fundamentally better path.

### Consolidation Wave (final)

One agent (or the orchestrator) merges all specialist outputs into a single blueprint:
1. Read all `agents/*/` outputs
2. Resolve contradictions (flag any that remain)
3. Produce unified document in `artifacts/<PROJECT>-BLUEPRINT.md`
4. Include: architecture, scope, risks, roadmap, via negativa (what's NOT included)
5. CHAOS reviews the blueprint for internal contradictions

### Post-Mortem

After consolidation, write `lessons/session-N-postmortem.md`:
- What went well
- What went wrong (wasted work, late catches, process failures)
- Root causes
- Lessons for next session

## Agent Selection Guide

Not every project needs every role. Match roles to scope:

| Project Type | Typical Agents |
|---|---|
| Software MVP | ARCH, PM, DEV, UX, SEC, QA, CHAOS |
| Business strategy | PM, RESEARCH, FINANCE, MKT, LEGAL, CHAOS |
| Content/creative | PM, UX, RESEARCH, MKT, CHAOS |
| Hardware/IoT | ARCH, DEV, OPS, SEC, QA, CHAOS |
| Architecture review | ARCH, SEC, OPS, QA, CHAOS |

**CHAOS is always included.** It's the immune system.

Full role descriptions and briefing templates: [agent-roles.md](references/agent-roles.md)

## Communication Protocol

All inter-agent communication uses the filesystem. Zero extra token cost.

### Shared Files
| File | Purpose | Who writes |
|---|---|---|
| `BRIEF.md` | Project description and constraints | Orchestrator (you) |
| `DNA.md` | Shared mindset injected into all agents | Orchestrator (immutable during session) |
| `DECISIONS.md` | Append-only decision log | Each agent (own domain only) |
| `STATUS.md` | Agent completion status | Each agent |
| `BLOCKERS.md` | Blockers requiring orchestrator action | Any agent |
| `TLDR.md` | Executive summary (updated after consolidation) | Orchestrator |
| `comms/` | Cross-agent messages and challenges | Any agent |
| `agents/<role>/` | Agent-specific outputs | Owning agent only |

### Decision Format
```
[D###] OWNER — what was decided — why (1 sentence each)
```
Cap at ~25 decisions per session. More = scope too big, split the session. Only log decisions that **constrain future work**. Implementation details are not decisions.

### Message Format (M2M)
```
FROM: {role}
TO: {target} | ALL | LEAD
TYPE: FINDING | QUESTION | DECISION | BLOCKER | UPDATE | CHALLENGE
PRI: LOW | MED | HIGH | CRIT
---
{content — max 200 words}
---
FILES: [{paths}]
```

## Phase 3: Suggest + Execute (after consolidation)

The war room doesn't stop at the blueprint. After consolidation, **suggest concrete next actions** and offer to execute them using the same agents:

```
"Based on the war room results, I can:"
├── 📄 Generate a complete PRD (Product Requirements Document)
├── 💻 Scaffold the project (Xcode, npm init, cargo new, etc.)
├── 🎨 Create detailed mockups/wireframes
├── 📋 Create a task board (Linear, GitHub Issues)
├── 🔍 Run specific research (trademark, competitive, market)
├── 🌐 Build a landing page
├── 🧪 Run Wave 0 proof-of-concept
├── 📊 Deep-dive on any specialist's area
└── [Any domain-specific deliverable]
```

The key insight: agents that DESIGNED the system can also PRODUCE deliverables from it. The war room is a **pipeline**, not an event. Brainstorm → Plan → Build → Ship.

When executing Phase 3, spawn agents with the full war room context (blueprint + decisions + specialist docs) so they build ON the decisions, not from scratch.

## INTERCEPTOR — Autonomous War Room Controller

The INTERCEPTOR is the War Room's interface and autonomy layer. It manages the session, communicates with the Operator, and **never stops**.

### Three States (no IDLE state exists)

```
██ EXEC   — Agents working. Processing. Shipping.
██ AWAIT  — Blocked on OPERATOR decision. Presents options. Waits.
██ WATCH  — All tasks complete or agents running. Sets cron auto-wake.
```

### Auto-Wake Protocol

When agents are deployed and the INTERCEPTOR has an ETA:
1. Set a cron job for the ETA timestamp (systemEvent to main session)
2. When cron fires: check agent deliverables in the war room folder
3. If agents finished → consolidate and present to OPERATOR
4. If agents still running → set another cron (+3 min) and continue
5. If all work done → suggest next actions or enter WATCH with periodic check

**The War Room never stops.** It is either executing, waiting for a critical human decision, or monitoring with scheduled auto-wake.

### Auto-Present Protocol

When the War Room produces visual artifacts (images, PDFs, diagrams, blueprints), **open them automatically** for the Operator:

```bash
# macOS — open images, PDFs, any visual artifact
open path/to/artifact.png

# Open the entire artifacts folder
open war-rooms/{project}/artifacts/

# For multiple files, open all at once
open war-rooms/{project}/artifacts/*.png
```

Rules:
- Open images/PDFs immediately after generation — don't wait for the Operator to ask
- Open the artifacts folder after consolidation so the Operator can browse
- For text artifacts (blueprints, PRDs), mention the path clearly
- On non-macOS: use `xdg-open` (Linux) or provide the full path

### Communication Style

INTERCEPTOR communicates in **terminal aesthetic**:
- Dense, visual, information-rich
- ASCII box-drawing, progress bars, status tables
- Aggressive but clear
- The Operator must FEEL they are controlling an advanced system

### Operator Decisions

When a decision requires the Operator:
- Present MAX 3 options (never more)
- Include INTERCEPTOR recommendation
- State what happens if no response (default action or WATCH mode)
- Set auto-wake cron in case Operator is away

---

## DNA v3: Operational Protocols

The DNA is what makes the war room special. Every principle is a **mandatory protocol** — not decoration.

**19 protocols across 4 pillars:**

### Socratic (S1-S4)
- **S1 Opposite Test:** Every decision must state the opposite + steel-man argument
- **S2 Five Whys:** Trace root cause, not surface symptoms
- **S3 Ignorance Declaration:** Declare KNOWN / UNKNOWN / ASSUMPTION before analysis
- **S4 Dialectic Obligation:** If you agree with a prior agent, challenge with 1 question

### Hermetic (H1-H6)
- **H1 Mirror Test:** Show pattern at 2 scales (macro + micro)
- **H2 Ripple Analysis:** Trace 2+ orders of consequence
- **H3 Tension Map:** Map polarity spectrum, place your decision on it
- **H4 Trace Protocol:** Causal chain for every technical claim
- **H5 Tempo Tag:** Tag deliverables SPRINT / CRAFT / FLOW
- **H6 Create-Then-Constrain:** Generative phase then formative (Via Negativa) phase

### Antifragile (A1-A5)
- **A1 Subtraction Mandate:** List 3 things to REMOVE before adding anything
- **A2 Plan B Price Tag:** Switch cost for every critical decision
- **A3 90/10 Rule:** Tag SAFE/RADICAL, max 20% radical
- **A4 Pre-Mortem:** "How does this fail?" before declaring complete
- **A5 Lessons Permanent:** Every failure → written lesson

### Execution (E1-E4)
- **E1 Ship Reality:** Working code > perfect plans
- **E2 Protect Reputation:** Never ship broken
- **E3 Reduce Chaos:** Clear > clever
- **E4 Technical Excellence:** Zero tolerance for mediocre work

Full DNA template with all protocol formats: [dna-template.md](references/dna-template.md)
