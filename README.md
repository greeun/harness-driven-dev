# Harness-Driven Dev

A Claude Code skill that turns Anthropic's [*Harness Design for Long-Running Application Development*](https://www.anthropic.com/engineering/harness-design-long-running-apps) into ready-to-dispatch English system prompts for building apps and services.

> **[한국어 README](README.ko.md)**

---

## Why This Exists

Long-running AI coding agents suffer from two structural failures:

```
┌──────────────────────┬──────────────────────────────┐
│  Context Anxiety     │  Self-Evaluation Bias        │
│                      │                              │
│  As context fills,   │  When grading its own work,  │
│  the agent rushes    │  the agent praises mediocre  │
│  to wrap up early    │  output confidently          │
│                      │                              │
│  Compaction does     │  Especially severe for       │
│  NOT fix this        │  subjective qualities        │
└──────────────────────┴──────────────────────────────┘
```

**Solo agent result (Retro Game Maker):** 20 min, ~$9 — central feature shipped broken.  
**With harness (same app):** 6 hours, ~$200 — polished, functional application.

The difference is architecture, not intelligence.

## Architecture

A GAN-inspired three-agent separation where generation and evaluation live in independent contexts, communicating only through files:

```
              ┌──────────┐
              │  User    │
              │  Brief   │
              └────┬─────┘
                   │
                   ▼
          ┌────────────────┐
          │    PLANNER     │  Idea → spec.md
          └───────┬────────┘
                  │
             spec.md
                  │
    ┌─────────────┼─────────────┐
    ▼             │             ▼
┌──────────┐     │     ┌──────────────┐
│GENERATOR │◄────┘     │  EVALUATOR   │
│          │           │              │
│ Build    │           │ Playwright   │
│ code     │──────────►│ real tests   │
│          │ report.md │ rubric grade │
└──────────┘           └──────┬───────┘
                              │
                         critique.md
                              │
                    ┌─────────▼─────────┐
                    │ PASS → next sprint │
                    │ FAIL → fresh retry │
                    └───────────────────┘
```

### Key Patterns

| Pattern | What it does |
|---|---|
| **Context resets** | Fresh session per agent. No compaction — clean slate only. |
| **File-based handoff** | `spec.md`, `sprint_contract.md`, `generator_report.md`, `critique.md`, `handoff.md` |
| **Sprint contract** | Generator and Evaluator negotiate testable success criteria before coding starts |
| **Adversarial evaluation** | Evaluator is the Generator's adversary, not teammate. Evidence required. |
| **7-criteria rubric** | Design Quality, Originality, Craft, Functionality + Correctness, Robustness, Usability |
| **5-15 iteration range** | Per design slice. Fewer underfits; more thrashes. |

## What's in the Skill

| Component | Description |
|---|---|
| **PROMPT 1** | Planner — turns 1-4 sentence ideas into `spec.md` |
| **PROMPT 2** | Generator — implements code, negotiates sprint contracts |
| **PROMPT 3** | Evaluator — Playwright-based adversarial verification with rubric grading |
| **PROMPT 4** | Orchestrator — controls the Planner→Generator→Evaluator loop |
| **PROMPT 5** | Simplified single-session harness for strong models (Opus 4.5/4.6) |
| **10 Principles** | Non-negotiable rules derived from the article |
| **Evaluator Tuning** | Step-by-step workflow to calibrate the Evaluator |
| **Model Guide** | Sonnet 4.5 (heavy scaffolding) vs Opus 4.5/4.6 (simplified) |
| **Case Studies** | Retro Game Maker ($9 vs $200), DAW ($124.70) |

## Installation

```bash
# Clone
git clone https://github.com/withwiz/harness-driven-dev.git

# Symlink to Claude Code skills directory
ln -s "$(pwd)/harness-driven-dev" ~/.claude/skills/harness-driven-dev
```

## Usage

The skill activates automatically when you mention related topics in Claude Code:

```
"harness design prompts"
"planner generator evaluator loop"
"앱 만들 때 프롬프트"
"하네스 프롬프트"
"코딩 에이전트 루프 설계"
```

Or invoke directly:

```
/harness-driven-dev
```

### Full Harness (PROMPT 1-4)

Best for:
- Complex apps beyond solo model capability
- Design-critical frontend work
- Sonnet-class models

### Simplified Harness (PROMPT 5)

Best for:
- Opus 4.5/4.6
- Tasks within solo model range
- Rapid prototyping

## Core Insight: Harness Shifts, Not Shrinks

> "Every component in a harness encodes an assumption about what the model can't do on its own, and those assumptions are worth stress testing."

As models improve, some scaffolding drops away — but the design space for new agent combinations expands. The harness doesn't disappear; its complexity migrates toward harder tasks.

## Source

Based on: [Harness Design for Long-Running Application Development](https://www.anthropic.com/engineering/harness-design-long-running-apps) — Anthropic Engineering Blog

## License

MIT
