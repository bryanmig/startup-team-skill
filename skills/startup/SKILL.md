---
name: startup
description: Use when evaluating a startup or product idea with an adversarial team simulation, before committing to building. Triggers on /startup or when user wants structured multi-perspective analysis of a business idea.
---

# Startup Team Simulation

Simulate an adversarial leadership team discussion to evaluate a product or business idea. 15 built-in domain-expert personas debate, challenge, and refine the idea across multiple rounds until consensus or clear disagreement emerges. When the idea touches a domain no built-in persona covers, the skill detects the gap, creates a new persona matching the team's depth and format, and saves it to the project for reuse. If the team recommends proceeding, a complete product requirements document is generated — ready for engineers or agent teams to begin implementing.

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
3. Set variable `personas_dir` to the `personas/` directory located in this plugin's root (two levels up from this skill file: `../../personas/`)
4. Read `startup.prose` from this skill directory
5. Become the OpenProse VM (read `prose.md` from the open-prose skill) and execute `startup.prose`
   - During execution, the selector scans `.claude/personas/` in the current project for previously-created dynamic personas and adds them to the available roster
   - If domain gaps are detected, the user is asked to confirm before new personas are created

## Requirements

- The open-prose skill must be installed and available for VM execution
- This plugin ships with all 15 persona definitions and decision frameworks in `personas/`
- Dynamically created personas are saved to `.claude/personas/` in the project where the skill runs
