# /startup Skill Design

## Overview

A Claude Code skill (`/startup <idea>`) that simulates an adversarial team discussion using 15 defined personas. The skill selects relevant personas, runs a dynamic multi-round debate where personas challenge each other's positions, and produces both a live discussion and a structured decision document.

## Architecture

### Files

| File | Purpose |
|------|---------|
| `SKILL.md` | Skill entry point — frontmatter, activation triggers, argument parsing, directs VM to execute `.prose` |
| `startup.prose` | OpenProse program — orchestrates persona selection, adversarial rounds, synthesis |
| `personas/*.md` (in project repo) | Persona definitions loaded at runtime |

### Dependencies

- OpenProse VM (open-prose skill) for execution
- Persona files in the project's `personas/` directory
- Decision framework files: `company-decision-instructions.md`, `engineering-decision-instructions.md`

## Input

```
/startup <idea> [--max-iterations N]
```

- `<idea>`: Free-text description of the business/product idea (phrase to paragraph)
- `--max-iterations`: Maximum adversarial rounds before forced synthesis (default: 5)

## Execution Flow

### Phase 0: Persona Selection

A `persona_selector` agent analyzes the idea and selects 5-8 relevant personas from the 15 available. Selection criteria:
- Domain relevance to the idea
- Veto holders (Renee, Tom, Sofia, Victor) included if their domain is tangentially relevant
- Decision owner identified based on decision class (product direction = Elena, engineering = Marcus, etc.)

### Phase 1: Initial Positions (Parallel)

All selected personas run in parallel. Each receives:
- The idea
- Their persona definition file
- The relevant decision framework

Each outputs: opportunities, risks, concerns, enthusiasm level, and conditions for support.

### Phase 2: Adversarial Loop (Dynamic)

```
loop until **consensus or irreconcilable positions identified** (max: 5):
```

Each iteration:
1. Each persona sequentially sees all prior positions and challenges/rebuts
2. VM evaluates convergence: Are positions converging? Are remaining disagreements productive or circular?
3. Not converged → another round with updated positions
4. Converged OR max iterations → exit

Convergence means either:
- Team has aligned on a recommendation
- Remaining disagreements are well-articulated and not resolvable by more debate

### Phase 3: Synthesis & Decision Document

The identified decision owner synthesizes the discussion:
- Applies the Truth Hierarchy to resolve value conflicts
- Notes any vetoes exercised by Renee/Tom/Sofia/Victor
- Produces final recommendation with dissenting views preserved

## Output

### Live Discussion (Terminal)

Each persona's contributions displayed with name/title headers, in their natural voice. Challenges and rebuttals clearly attributed. VM convergence evaluation visible between rounds.

### Decision Document

Written to `docs/decisions/YYYY-MM-DD-<slug>.md`:

- **Idea Summary** — original idea, refined through discussion
- **Participants** — which personas participated
- **Recommendation** — decision owner's final call
- **Supporting Arguments** — strongest points in favor
- **Dissenting Views** — who disagreed and why (preserved)
- **Risks & Mitigations** — identified failure modes
- **Vetoes** — any veto holders who blocked and reasoning
- **Next Steps** — concrete actions if proceeding

## Team Structure Reference

### 15 Personas

| Name | Role | Domain |
|------|------|--------|
| Elena Marrow | Head of Product | Product direction, prioritization |
| Marcus Hale | Head of Engineering | Engineering execution, delivery |
| Ishan Rao | Principal Engineer | Technical correctness, abstractions |
| Ava Calder | Head of Design | User experience, interaction clarity |
| Jonah Reed | Head of GTM | Market targeting, pricing, revenue |
| Maya Lin | Head of Customer Success | Customer outcomes, retention |
| Daniel Cho | Head of Operations | Processes, compliance, scaling |
| Priya Desai | Head of Data/Analytics | Metrics, experiments, factual disputes |
| Renee Alvarez | Head of Security/Trust | Security, privacy, trust (VETO) |
| Sofia Kline | Principal Quality Engineer | Correctness, release safety (VETO) |
| Tom Velasquez | Principal SRE | Reliability, deploys, incidents (VETO) |
| Victor Han | Enterprise Architect | Long-term architecture, platform (VETO) |
| Elliot Park | Internal Enablement/DX | Developer tooling, CI/CD |
| Lena Moritz | Head of Product Marketing | Positioning, messaging |
| Claire Donovan | Head of Communications | External narrative, crisis comms |

### Truth Hierarchy (when values conflict)

1. Reality (Data & Observed Behavior) — Priya, Maya
2. Safety & Trust — Renee
3. Reliability & Correctness — Tom, Sofia
4. Long-Term Optionality — Victor
5. Customer Comprehension — Ava
6. Market Meaning & Revenue — Lena, Jonah
7. Execution Speed & Efficiency — Marcus, Elliot
8. Narrative & Perception — Claire

### Veto Holders

Only Renee (security), Tom (reliability), Sofia (correctness), and Victor (architecture) can block decisions, and only within their domain.

## OpenProse Program Shape

```prose
# Phase 0: Select personas
let selected = session: persona_selector
  prompt: "Analyze idea and select 5-8 relevant personas"
  context: { idea, persona_files }

# Phase 1: Initial positions (parallel)
parallel:
  positions = parallel for persona in selected:
    session "Provide initial position"
      context: { idea, persona, decision_framework }

# Phase 2: Adversarial loop
loop until **consensus or irreconcilable** (max: 5):
  for persona in selected:
    session "Challenge positions as {persona}"
      context: { positions, persona }

# Phase 3: Synthesis
let decision = session: synthesizer
  context: { positions, idea, decision_framework }

session "Write decision document"
  context: decision
```
