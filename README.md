# Smartplan

**The structured plan before your next big process rollout. Who does what, where to stop, how to roll back.**

A [Hermes Agent](https://hermes-agent.nousresearch.com) skill for creating structured non-code process plans — business workflows, automation setups, data migrations, product launches, and operational rollouts.

## What it does

When you ask your agent to plan a process, Smartplan ensures the plan includes:

- **Context & prerequisites** — what exists, what's needed, what's assumed
- **Sequenced execution steps** — each with a named responsible, a concrete action, a deliverable, and an executable verification
- **GO gates** — explicit human validation points before any external mutation
- **Rollback** — 3 levels (immediate cancel, restore from backup, full rollback) with observable triggers
- **Final verification** — independently checkable completion criteria + proof of success

## Why

Most agent planning skills are code-oriented (TDD, file paths, git commits). There was no structured skill for **business process deployment** — the kind of plan where you need "who does what, in what order, with what GO gates, and how to roll back if it breaks."

Smartplan fills that gap.

## Installation

### Manual install

Copy the `smartplan` folder into your Hermes skills directory:

```
~/.hermes/skills/productivity/smartplan/
├── SKILL.md
├── templates/
│   └── plan-template.md
└── references/
    └── example-plan.md
```

### Verify

Your agent should be able to load the skill:

```
skill_view(name='smartplan')
```

## Usage

Ask your agent to plan a process:

> "Plan the deployment of our post-purchase survey: webhook, storage, Slack notification."

The agent will:
1. Load this skill
2. Inspect the context (read-only)
3. Produce a structured plan saved to `.hermes/plans/`
4. List the GO gates and offer execution — but **not** execute without your explicit go-ahead

## What it does NOT do

- **Code implementation plans** → use `software-delivery-workflow` or `superpowers:writing-plans`
- **Recurring automated routines** → use a cronjob
- **Simple 2-3 action tasks** → just do it, no plan needed

## Plan structure

Every plan produced by Smartplan follows the same structure:

```
Header (objective, scope, owner, date, status)
1. Context & prerequisites
2. Execution steps (responsible + action + deliverable + verification + risk)
3. GO gates (gate / after step / condition / who validates)
4. Rollback (3 levels + observable triggers)
5. Final verification (completion criteria + proof of success)
6. Risks (optional)
7. Notes (optional)
```

## License

MIT

## Author

David Schkiwisk — [kavyro.com](https://kavyro.com)