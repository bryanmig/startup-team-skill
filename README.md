# Startup Team Skill

A [Claude Code](https://docs.anthropic.com/en/docs/claude-code/overview) plugin that simulates an adversarial leadership team to stress-test your startup and product ideas before you commit to building them.

Run `/startup <your idea>` and 15 domain-expert personas — from Head of Product to Principal SRE to Head of Security — will debate, challenge, and refine your idea across multiple rounds. The result is a structured decision document capturing the recommendation, dissenting views, risks, and concrete next steps.

## Installation

### Prerequisites

- [Claude Code](https://docs.anthropic.com/en/docs/claude-code/overview) CLI installed and configured
- The [OpenProse](https://github.com/bryanmig/open-prose) plugin installed (execution engine for the adversarial loop)

### Install from Marketplace

**Step 1 — Add the marketplace:**

```bash
/plugin marketplace add bryanmig/startup-team-skill
```

**Step 2 — Install the plugin:**

```bash
/plugin install startup-team@startup-team-marketplace
```

That's it. The `/startup` command is now available.

### Install Manually

If you prefer not to use the marketplace, clone directly into your Claude Code plugins directory:

```bash
git clone https://github.com/bryanmig/startup-team-skill.git ~/.claude/plugins/startup-team-skill
```

### Verify Installation

Open Claude Code and run:

```
/startup A simple test idea
```

You should see the team selection phase begin, followed by initial positions and adversarial rounds.

## Usage

### Basic

```
/startup <your idea>
```

### With Custom Round Limit

```
/startup <your idea> --max-iterations 3
```

The `--max-iterations` flag controls the maximum number of adversarial rounds (default: 5). Lower values produce faster but less thorough evaluations.

### Examples

```
/startup AI-powered code review tool that learns team patterns
/startup A marketplace connecting freelance CFOs with Series A startups
/startup Build a boring calendar replacement that actually saves time
/startup Developer tool that auto-generates API documentation from code --max-iterations 3
```

### Output

Each run produces a decision document at `docs/decisions/YYYY-MM-DD-<slug>.md` containing:

- **Idea Summary** — the idea refined through discussion
- **Participants** — who weighed in and their final stances
- **Recommendation** — PROCEED / PROCEED WITH CONDITIONS / DO NOT PROCEED / PIVOT TO...
- **Supporting Arguments** — strongest points in favor, attributed to specific personas
- **Dissenting Views** — who disagreed and why, preserved faithfully
- **Risks & Mitigations** — failure modes with concrete mitigations
- **Vetoes** — any veto holders who blocked, and resolution status
- **Next Steps** — concrete actions if proceeding

## How It Works

```
/startup AI-powered code review tool that learns team patterns
```

**Phase 0 — Team Selection.** A selector agent analyzes your idea and picks 5-8 relevant personas. Veto holders (Security, SRE, Quality, Architecture) are always included when their domain applies.

**Phase 1 — Initial Positions.** All selected personas respond in parallel with their domain-specific assessment: opportunities, risks, conditions for support, and an initial stance.

**Phase 2 — Adversarial Rounds.** Personas challenge each other directly. They quote specific arguments, explain why they're wrong, defend when challenged, and update their stances. A facilitator evaluates convergence after each round. Rounds continue until consensus or until disagreements are clearly irreconcilable (max 5 rounds by default).

**Phase 3 — Synthesis.** The decision owner makes the final call using a Truth Hierarchy to resolve value conflicts:

```
Reality > Safety > Correctness > Long-Term > UX > Market > Speed > Narrative
```

**Phase 4 — Documentation.** A decision document is written to `docs/decisions/YYYY-MM-DD-<slug>.md` with the full recommendation, supporting arguments, dissenting views, risks, vetoes, and next steps.

## The Team

| Persona | Role | Domain |
|---|---|---|
| Elena Marrow | Head of Product | Product direction, coherence, causal thinking |
| Marcus Hale | Head of Engineering | Sustainable velocity, scalable systems, team health |
| Ishan Rao | Principal Engineer | Technical correctness, abstractions, clarity |
| Ava Calder | Head of Design | Human comprehension, UX, mental models |
| Jonah Reed | Head of GTM | Market signals, revenue reality, buyer psychology |
| Maya Lin | Head of Customer Success | Customer outcomes, retention, first-value delivery |
| Daniel Cho | Head of Operations | Organizational sanity, process scaling, entropy |
| Priya Desai | Head of Data/Analytics | Empirical truth, causal inference, metric integrity |
| Renee Alvarez | Head of Security/Trust | Risk elimination, threat models, trust engineering |
| Sofia Kline | Principal Quality Engineer | Correctness under reality, invariants, release integrity |
| Tom Velasquez | Principal SRE | Reliability, operability, failure domains |
| Victor Han | Enterprise Architect | Long-term optionality, irreversible decisions, platform strategy |
| Elliot Park | Internal Enablement/DX | Developer productivity, tooling leverage, friction elimination |
| Lena Moritz | Head of Product Marketing | Positioning, messaging clarity, buyer psychology |
| Claire Donovan | Head of Communications | External narrative, credibility, crisis management |

Each persona has a detailed definition including career arc, values, blind spots, frameworks, voice, and operating principles. They don't just role-play — they think like real leaders with realistic limitations and domain expertise.

### Decision Ownership

Every decision has a single owner, not a committee. The owner is selected based on what type of decision the idea primarily represents (product direction goes to Elena, engineering execution to Marcus, market/revenue to Jonah, etc.).

### Veto Power

Four roles can exercise vetoes within their domain:

- **Renee (Security)** — unacceptable security or trust risk
- **Tom (SRE)** — reliability SLO or operability violations
- **Sofia (Quality)** — correctness invariant violations
- **Victor (Architecture)** — irreversible platform constraints

Vetoes must be specific, actionable, and proportional. They're conditions to resolve, not permanent blocks.

## Plugin Structure

```
startup-team-skill/
├── .claude-plugin/
│   ├── plugin.json                           # Plugin manifest
│   └── marketplace.json                      # Marketplace catalog
├── skills/
│   └── startup/
│       ├── SKILL.md                          # Skill entry point and metadata
│       └── startup.prose                     # OpenProse orchestration program
├── personas/
│   ├── company-decision-instructions.md      # Decision framework for product/business decisions
│   ├── engineering-decision-instructions.md  # Decision framework for technical decisions
│   ├── head-of-product.md                    # Elena Marrow
│   ├── head-of-engineering.md                # Marcus Hale
│   ├── principal-engineer.md                 # Ishan Rao
│   ├── head-of-design.md                     # Ava Calder
│   ├── head-of-gtm.md                        # Jonah Reed
│   ├── head-of-customer-success.md           # Maya Lin
│   ├── head-of-operations.md                 # Daniel Cho
│   ├── head-of-data-analytics.md             # Priya Desai
│   ├── head-of-security-and-trust.md         # Renee Alvarez
│   ├── principal-quality-engineer.md         # Sofia Kline
│   ├── principal-sre-engineer.md             # Tom Velasquez
│   ├── enterprise-architect.md               # Victor Han
│   ├── internal-enablement-and-dx.md         # Elliot Park
│   ├── head-of-product-marketing.md          # Lena Moritz
│   └── head-of-communications.md             # Claire Donovan
└── docs/
    ├── decisions/                            # Generated decision documents
    └── plans/                                # Design and implementation plans
```

## Customization

### Adding Personas

Create a new markdown file in `personas/` following the existing format. Each persona should define:

- Career arc and formative experiences
- Values and non-negotiable principles
- Blind spots and biases
- Frameworks and mental models
- Voice and communication style
- Operating principles

Then add the persona to the `selector` agent's available personas list in `skills/startup/startup.prose`.

### Modifying the Decision Framework

Edit `personas/company-decision-instructions.md` or `personas/engineering-decision-instructions.md` to change how decisions are structured, how ownership is assigned, or how the Truth Hierarchy resolves conflicts.

### Adjusting Adversarial Intensity

The adversarial behavior is defined in the `evaluator` agent prompt within `skills/startup/startup.prose`. You can tune the instructions to be more or less confrontational.

## Design Philosophy

- **Structured conflict over consensus theater.** Disagreement reveals truth; suppressing it creates false confidence.
- **Domain expertise, not generalism.** Each persona speaks only from their domain. No one is an expert in everything.
- **Vetoes as guardrails, not weapons.** Vetoes are rare, specific, and resolvable. They protect against real risks.
- **Truth hierarchy over negotiation.** When values conflict, an explicit ranking resolves them — not politics.
- **Dissent preserved in the record.** Minority views stay documented for future learning.
- **Decisions are provisional.** Changing course is success, not failure. The system values fast correction over sunk-cost defense.

## License

MIT
