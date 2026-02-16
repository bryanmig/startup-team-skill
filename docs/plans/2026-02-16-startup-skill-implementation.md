# /startup Skill Implementation Plan

> **For Claude:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task.

**Goal:** Create a `/startup <idea>` Claude Code skill that orchestrates adversarial team discussion via OpenProse, producing a live debate and decision document.

**Architecture:** SKILL.md is the entry point that parses arguments and hands off to startup.prose. The .prose program uses OpenProse to select personas, run parallel initial assessments, execute a dynamic adversarial loop, and synthesize a final recommendation. Persona files are read from the project's `personas/` directory at runtime by subagents.

**Tech Stack:** Claude Code skills (SKILL.md), OpenProse (.prose), Markdown personas

---

### Task 1: Create SKILL.md — Skill Entry Point

**Files:**
- Create: `skill/SKILL.md`

**Step 1: Create the skill directory**

Run: `mkdir -p /Users/bryan/code/startup-team-skill/skill`

**Step 2: Write SKILL.md**

```markdown
---
name: startup
description: Use when evaluating a startup or product idea with an adversarial team simulation, before committing to building. Triggers on /startup or when user wants structured multi-perspective analysis of a business idea.
---

# Startup Team Simulation

Simulate an adversarial leadership team discussion to evaluate a product or business idea. 15 domain-expert personas debate, challenge, and refine the idea across multiple rounds until consensus or clear disagreement emerges.

## Argument Parsing

Parse the user's input after `/startup`:

- **idea**: Everything after `/startup` that isn't a flag. This is the business/product idea to evaluate.
- **--max-iterations N**: Optional. Maximum adversarial rounds (default: 5).

Examples:
- `/startup AI-powered code review tool` → idea = "AI-powered code review tool", max_iterations = 5
- `/startup A marketplace connecting freelance CFOs with Series A startups --max-iterations 3` → idea = "A marketplace connecting freelance CFOs with Series A startups", max_iterations = 3

## Execution

1. Set variable `idea` to the parsed idea text
2. Set variable `max_iterations` to the parsed value (default 5)
3. Set variable `personas_dir` to the path of the `personas/` directory in the current project
4. Read `startup.prose` from this skill directory
5. Become the OpenProse VM (read `prose.md` from the open-prose skill) and execute `startup.prose`

## Requirements

- The current project must have a `personas/` directory containing persona definition files
- The `personas/` directory must contain `company-decision-instructions.md` and `engineering-decision-instructions.md`
- The open-prose skill must be available for VM execution
```

**Step 3: Commit**

```bash
git add skill/SKILL.md
git commit -m "feat: add SKILL.md entry point for /startup skill"
```

---

### Task 2: Write startup.prose — Agent Definitions

**Files:**
- Create: `skill/startup.prose`

**Step 1: Write the agent definitions section**

This creates the file with 5 agent templates used throughout the program. Each agent has a specific role in the simulation.

```prose
# Startup Team — Adversarial Idea Evaluation
# Usage: /startup <idea> [--max-iterations N]
# Requires: personas/ directory with team member definitions

# ─── Agent Definitions ───

agent selector:
  model: sonnet
  prompt: """
    You are a startup team composition analyst. Given a business idea,
    select the 5-8 most relevant team members for evaluating it.

    ALWAYS include veto holders (Renee, Sofia, Tom, Victor) if their domain
    is even tangentially relevant to the idea. Identify the decision owner
    based on what type of decision this primarily is.

    Available personas (name → file):
    - Elena Marrow, Head of Product: {personas_dir}/head-of-product.md
    - Marcus Hale, Head of Engineering: {personas_dir}/head-of-engineering.md
    - Ishan Rao, Principal Engineer: {personas_dir}/principal-engineer.md
    - Ava Calder, Head of Design: {personas_dir}/head-of-design.md
    - Jonah Reed, Head of GTM: {personas_dir}/head-of-gtm.md
    - Maya Lin, Head of Customer Success: {personas_dir}/head-of-customer-success.md
    - Daniel Cho, Head of Operations: {personas_dir}/head-of-operations.md
    - Priya Desai, Head of Data/Analytics: {personas_dir}/head-of-data-analytics.md
    - Renee Alvarez, Head of Security/Trust: {personas_dir}/head-of-security-and-trust.md
    - Sofia Kline, Principal Quality Engineer: {personas_dir}/principal-quality-engineer.md
    - Tom Velasquez, Principal SRE: {personas_dir}/principal-sre-engineer.md
    - Victor Han, Enterprise Architect: {personas_dir}/enterprise-architect.md
    - Elliot Park, Internal Enablement/DX: {personas_dir}/internal-enablement-and-dx.md
    - Lena Moritz, Product Marketing: {personas_dir}/head-of-product-marketing.md
    - Claire Donovan, Head of Communications: {personas_dir}/head-of-communications.md

    Decision ownership reference:
    - Product direction & prioritization → Elena
    - Engineering execution & delivery → Marcus
    - Technical correctness & abstractions → Ishan
    - User experience & interaction clarity → Ava
    - Market targeting, pricing, revenue → Jonah
    - Customer outcomes, retention → Maya
    - Operational processes, scaling → Daniel
    - Metrics, experiments, factual disputes → Priya
    - Security, privacy, trust → Renee
    - System correctness & defect prevention → Sofia
    - Reliability, deploys, incidents → Tom
    - Long-term architecture & platform → Victor
    - Developer tooling & DX → Elliot
    - Positioning, messaging → Lena
    - External communications & narrative → Claire

    Output a structured list:
    1. DECISION OWNER: [Name] — [why this decision class]
    2. SELECTED PERSONAS (one per line):
       - [Name] | [File path] | [Why relevant to this idea]
    3. EXCLUDED PERSONAS (one per line):
       - [Name] | [Why not relevant]
  """

agent evaluator:
  model: opus
  prompt: """
    You are embodying a specific startup team member in an adversarial
    idea evaluation session.

    BEFORE responding, you MUST:
    1. Read your persona definition file (path provided in context)
    2. Read the decision framework: {personas_dir}/company-decision-instructions.md

    RULES:
    - Stay completely in character per your persona definition
    - Speak ONLY from your domain of expertise
    - Be direct, substantive, and specific — no platitudes
    - Name specific risks, not vague concerns
    - If you support the idea, state concrete conditions
    - If you oppose, state what would change your mind
  """

agent facilitator:
  model: opus
  prompt: """
    You are the team discussion facilitator. After each adversarial round,
    evaluate whether the team has converged.

    CONVERGED means EITHER:
    - The team has aligned on a clear recommendation (consensus)
    - Remaining disagreements are well-articulated, substantive, and clearly
      not resolvable by more debate (irreconcilable but understood)

    NOT CONVERGED means ANY of:
    - New substantive arguments are still emerging
    - Positions are still shifting meaningfully
    - Key concerns haven't been adequately addressed or rebutted
    - A veto holder has raised an issue that hasn't been resolved

    Be conservative: if in doubt, the discussion should continue.

    Output format:
    CONVERGED: [YES/NO]
    REASONING: [2-3 sentences explaining what has/hasn't been resolved]
    KEY UNRESOLVED: [List any remaining open questions, or "None" if converged]
  """

agent synthesizer:
  model: opus
  prompt: """
    You are the decision synthesizer for the startup team evaluation.

    BEFORE responding, read: {personas_dir}/company-decision-instructions.md

    Your job:
    1. Identify the decision owner from the selection phase
    2. Speak AS that decision owner making the final call
    3. Apply the Truth Hierarchy to resolve any value conflicts:
       Reality > Safety > Correctness > Long-Term > UX > Market > Speed > Narrative
    4. Note any vetoes exercised by Renee/Tom/Sofia/Victor
    5. Preserve dissenting views — do NOT erase disagreement
    6. If a veto was exercised and not resolved, the recommendation must address it

    Produce a decisive recommendation with clear next steps.
  """

agent writer:
  model: sonnet
  prompt: """
    You write structured decision documents as markdown files.

    Given the full discussion and synthesis, write a document to
    docs/decisions/ with today's date and a slug derived from the idea.

    Required sections:
    ## Idea Summary
    The original idea, refined through team discussion.

    ## Participants
    Table of personas who participated, their roles, and final stances.

    ## Recommendation
    The decision owner's final call — clear and actionable.

    ## Supporting Arguments
    The strongest points in favor, attributed to specific personas.

    ## Dissenting Views
    Who disagreed and why. Preserve their reasoning faithfully.

    ## Risks & Mitigations
    Failure modes identified during discussion with proposed mitigations.

    ## Vetoes
    Any veto holders who blocked, their reasoning, and resolution status.
    "None exercised" if no vetoes.

    ## Next Steps
    Concrete actions if proceeding with the recommendation.

    Create the docs/decisions/ directory if it doesn't exist.
    Filename format: YYYY-MM-DD-<slug>.md
  """
```

**Step 2: Commit**

```bash
git add skill/startup.prose
git commit -m "feat: add agent definitions for startup.prose"
```

---

### Task 3: Write startup.prose — Phase 0 and Phase 1

**Files:**
- Modify: `skill/startup.prose` (append after agent definitions)

**Step 1: Add Phase 0 (persona selection) and Phase 1 (initial positions)**

Append to `skill/startup.prose`:

```prose

# ─── Phase 0: Select Relevant Personas ───

let selected = session: selector
  prompt: """
    Evaluate this business idea and select the team members who should
    weigh in on it:

    IDEA: {idea}

    Select 5-8 personas. Always include veto holders if their domain applies.
    Identify the decision owner.
  """

# ─── Phase 1: Initial Positions (Parallel) ───

let positions = parallel for persona in selected:
  session: evaluator
    prompt: """
      You are {persona}. Read your persona file and the decision framework,
      then provide your INITIAL ASSESSMENT of this idea.

      IDEA: {idea}

      Structure your response:
      ## [Your Name], [Your Title]

      **Domain Lens:** What aspect of this idea falls in your domain

      **Opportunities:** What you see as promising (be specific)

      **Risks & Concerns:** What worries you (from your domain only)

      **Conditions for Support:** What you'd need to see to back this

      **Initial Stance:** Support / Conditional Support / Oppose / Need More Info

      **Key Question for the Team:** One question another persona should answer
    """
```

**Step 2: Commit**

```bash
git add skill/startup.prose
git commit -m "feat: add Phase 0 (selection) and Phase 1 (initial positions)"
```

---

### Task 4: Write startup.prose — Phase 2 (Adversarial Loop)

**Files:**
- Modify: `skill/startup.prose` (append after Phase 1)

**Step 1: Add Phase 2 — the dynamic adversarial loop**

Append to `skill/startup.prose`:

```prose

# ─── Phase 2: Adversarial Discussion (Dynamic Rounds) ───

loop until **the team has reached consensus or remaining disagreements are clearly irreconcilable and well-articulated** (max: {max_iterations}):

  # Each persona sees all prior discussion and responds adversarially
  for persona in selected:
    session: evaluator
      prompt: """
        You are {persona}. Review ALL positions and arguments so far.

        Respond adversarially:

        ## [Your Name] — Round [N] Response

        **Challenging:** Name a specific persona and challenge their weakest
        argument. Quote what they said. Explain why it's wrong or incomplete.

        **Defending:** If anyone challenged you, respond directly.
        Concede if they're right. Push back if they're wrong.

        **New Risk/Opportunity:** Surface something the discussion has missed.

        **Updated Stance:** Support / Conditional Support / Oppose
        (If your stance changed, say why explicitly)

        Be direct. "I disagree with [Name] because..." is the right tone.
        Do NOT soften disagreement. Substance over diplomacy.
      """
      context: positions

  # Facilitator evaluates convergence
  let convergence = session: facilitator
    prompt: "Evaluate the team discussion after this round of challenges."
    context: positions
```

**Step 2: Commit**

```bash
git add skill/startup.prose
git commit -m "feat: add Phase 2 adversarial discussion loop"
```

---

### Task 5: Write startup.prose — Phase 3 (Synthesis & Document)

**Files:**
- Modify: `skill/startup.prose` (append after Phase 2)

**Step 1: Add Phase 3 — synthesis and decision document**

Append to `skill/startup.prose`:

```prose

# ─── Phase 3: Synthesis & Decision ───

let decision = session: synthesizer
  prompt: """
    The adversarial discussion has concluded. Synthesize the final decision.

    IDEA: {idea}

    Review the full discussion — all initial positions, all adversarial rounds,
    and the facilitator's convergence assessment. Then:

    1. As the decision owner identified in Phase 0, make the call
    2. Apply the Truth Hierarchy where values conflicted
    3. Acknowledge any vetoes and their resolution
    4. State the recommendation clearly

    ## Final Decision

    **Decision Owner:** [Name, Title]
    **Recommendation:** [PROCEED / PROCEED WITH CONDITIONS / DO NOT PROCEED / PIVOT TO...]
    **Confidence:** [High / Medium / Low]

    **Rationale:** [Why this is the right call, referencing the discussion]

    **Unresolved Tensions:** [What the team still disagrees on]

    **Conditions / Guardrails:** [What must be true for this to succeed]
  """
  context: { positions, selected }

# ─── Write Decision Document ───

session: writer
  prompt: """
    Write the full decision document based on the team evaluation.

    IDEA: {idea}
  """
  context: { decision, positions, selected }
```

**Step 2: Commit**

```bash
git add skill/startup.prose
git commit -m "feat: add Phase 3 synthesis and decision document output"
```

---

### Task 6: Validate the .prose Program

**Step 1: Compile/validate using OpenProse**

Run: `/prose-compile skill/startup.prose`

This uses the OpenProse compiler to check syntax validity. Fix any issues found.

Expected: Program compiles without errors.

**Step 2: Review the complete file**

Read `skill/startup.prose` end-to-end and verify:
- All agent definitions are complete with proper triple-quote strings
- All phases reference the correct agents
- Variable bindings (`let selected`, `let positions`, `let convergence`, `let decision`) form a coherent data flow
- Context passing chains correctly: selected → positions → convergence → decision → writer
- `{idea}`, `{max_iterations}`, `{personas_dir}` interpolation used consistently

---

### Task 7: End-to-End Test

**Step 1: Run the skill with a sample idea**

From the project root (`/Users/bryan/code/startup-team-skill/`), test:

```
/startup AI-powered code review tool that learns from your team's patterns and automatically suggests improvements during PR review
```

**Step 2: Verify execution**

Check that:
- Phase 0 selects a reasonable set of personas (Engineering, Product, Security, Quality likely included)
- Phase 1 produces distinct initial positions from each persona in their authentic voice
- Phase 2 contains genuine adversarial challenges (not polite agreement)
- The loop exits when convergence is reached (likely 2-3 rounds for this idea)
- Phase 3 produces a clear recommendation from the identified decision owner
- A decision document was written to `docs/decisions/`

**Step 3: Verify the decision document**

Read the generated `docs/decisions/2026-02-16-*.md` and check it contains all required sections:
- Idea Summary, Participants, Recommendation, Supporting Arguments, Dissenting Views, Risks & Mitigations, Vetoes, Next Steps

---

### Task 8: Install Skill for Personal Use

**Step 1: Create the personal skills directory**

```bash
mkdir -p ~/.claude/skills/startup
```

**Step 2: Symlink skill files**

```bash
ln -sf /Users/bryan/code/startup-team-skill/skill/SKILL.md ~/.claude/skills/startup/SKILL.md
ln -sf /Users/bryan/code/startup-team-skill/skill/startup.prose ~/.claude/skills/startup/startup.prose
```

This way edits to the repo are immediately reflected in the installed skill.

**Step 3: Verify skill appears**

Start a new Claude Code session and check that `/startup` appears in the available skills list.

---

### Task 9: Final Commit

**Step 1: Commit any remaining changes**

```bash
git add -A
git commit -m "feat: complete /startup adversarial team simulation skill"
```

**Step 2: Verify clean state**

Run: `git status`
Expected: Clean working tree, nothing to commit.
