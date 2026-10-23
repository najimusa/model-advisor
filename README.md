# Model Advisor

A Claude skill that checks, on every message, whether the Claude model you picked fits the task. If the gap is real (a quick lookup on Opus, a deep high-stakes analysis on Haiku), Claude tells you which model it recommends, why, and how confident it is, then asks whether you want to switch. If the model fits, it stays silent.

## What it does

- Assesses every message for reasoning depth, stakes, scope, repetition and ambiguity.
- Uses a normal bar for new tasks and a higher bar for follow-ups, so long tasks are not interrupted every turn.
- Never claims to switch models. You switch in the model picker yourself.
- Say "stop checking this" to mute it for the session. It still speaks up once for an extreme mismatch.

## Install

Download `model-advisor.zip` from the [latest release](https://github.com/najimusa/model-advisor/releases/latest), or clone this repository.

**Claude app (web, desktop, mobile)**
1. Open Settings and find the Skills section.
2. Upload `model-advisor.zip`.
3. Turn the skill on.

**Claude Code**
```bash
git clone https://github.com/najimusa/model-advisor.git
cp -r model-advisor/model-advisor ~/.claude/skills/
```
For a single project, copy it into `.claude/skills/` inside the project instead.

## Keeping it current

The model lineup in `SKILL.md` reflects October 2026 (Haiku 5.5, Sonnet 5.5, Opus 5.5, Fable 5.1). When Anthropic releases new models, edit the table. Pull requests welcome.

## Repository layout

```
model-advisor/
  SKILL.md      the skill itself
README.md
LICENSE
```

## License

MIT. See [LICENSE](LICENSE).
