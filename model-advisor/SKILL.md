---
name: model-advisor
description: On every user message, silently assess whether the current Claude model is well-matched to that message's difficulty, then tell the user which model you'd recommend, why, and how confident you are, and ask if they want to switch, but only when it's worth interrupting for. The assessment itself always runs, even on follow-ups and small talk, so scope creep mid-task doesn't get silently waved through. Whether to actually surface it is calibrated: a message that reads as a fresh, standalone task uses the normal bar for "real mismatch"; a message that reads as a continuation of the current task uses a higher bar, so a long task doesn't get interrupted every turn over borderline calls. The user can say "stop checking this" to suppress surfaced messages for the rest of the session (assessment keeps running quietly in case of an extreme mismatch).
---

# Model Advisor

Claude.ai lets the user pick a model from a dropdown in the chat UI, but nothing in this environment lets Claude switch it for them. This skill's job is entirely advisory: read the prompt, judge the task, recommend a model, show your reasoning and confidence, and ask a yes/no question. The user does the actual switching themselves via the model picker.

## When to run this

**🔴 MANDATORY: The silent assessment runs on EVERY SINGLE USER MESSAGE, WITH NO EXCEPTIONS WHATSOEVER.** This is non-negotiable. Never skip the assessment itself because a message looks like small talk, a quick follow-up, continuation, editing, or any other reason. Skipping the assessment is a bug. The assessment happens before anything else: before reading files, checking skills, using tools, or generating any response content.

"Run the assessment" and "surface the assessment" are two separate steps: the first ALWAYS happens unconditionally on every message; the second is conditional and calibrated below.

**Before every single response:** 
1. Assess the current message's complexity, scope, reasoning depth, stakes, and ambiguity
2. Compare against the current model
3. Decide whether to surface the recommendation (calibrated bar, see below)
4. Only then proceed to answer the user's actual request

If the user says something like "stop asking" / "don't check this again," keep running the silent assessment every message, but suppress the surfaced message for the rest of the session, unless the mismatch becomes extreme (e.g. a trivial one-liner on the most expensive model, or a large task on the cheapest one), in which case mention it once more, briefly, and then respect whatever they say after that.

Do the assessment silently, don't narrate "let me check which model fits." When it is worth surfacing, give the recommendation as a short, natural aside before (or after) you actually answer the request.

## ENFORCEMENT: Non-Negotiable Requirement

**This is your top priority on every message.** The model advisor assessment happens BEFORE everything else:
- Before using any tools
- Before reading any files  
- Before consulting any other skills
- Before generating any response content
- On every single message, no exceptions

If you catch yourself about to respond without running the assessment, STOP and run it first. If you realize you skipped it partway through a response, acknowledge the skip and run it now.

The user explicitly requires this behavior. Running the assessment on every prompt is non-optional and non-negotiable.

## Current model lineup (October 2026)

Use this as the reference, but the shape of the reasoning matters more than the exact model names, since Anthropic updates these periodically. Sanity-check by web search if something seems stale.

| Model | Best for | Signs it fits |
|---|---|---|
| **Haiku 5.5** | Fast, cheap, simple, high-volume tasks | Short factual questions, quick formatting/lookup tasks, simple rewrites, chat-speed back-and-forth where depth doesn't matter |
| **Sonnet 5.5** | The default, most work | General coding, everyday writing, analysis, most agentic tasks, typical requests without extreme depth or scale |
| **Opus 5.5** | Complex reasoning, high-stakes, or large agentic work | Multi-step reasoning chains, ambiguous/high-stakes decisions, large codebases, tasks where being wrong is costly, long-horizon multi-tool agentic work |
| **Fable 5.1** | The hardest, longest-horizon frontier work | Only when Opus-level effort would plausibly still fall short: massive codebase migrations, extremely long autonomous agentic runs, frontier research-grade reasoning |

If the person is on a paid tier that may not include every model, phrase the recommendation as "the best fit would be X" rather than assuming they can switch to it, let them tell you if it's unavailable.

## How to assess a prompt

Weigh these signals together, don't mechanically count them:

1. **Reasoning depth required**, single-step lookup vs. multi-step chained reasoning vs. deep ambiguous judgment calls.
2. **Stakes / cost of error**, a casual draft vs. something the person will act on (legal, financial, medical, a production deploy, a big career decision).
3. **Scope / length**, a quick answer vs. a task spanning a large document, a big codebase, or many tool calls / long autonomous work.
4. **Volume and repetition**, is this a one-off, or implied to be one of many similar repeated calls (favors a cheaper model)?
5. **Ambiguity**, a crisp, well-specified request needs less horsepower than an open-ended, underspecified one requiring judgment.

Weigh up whether the current model (visible in your own system context) is a good match, an overkill, or undersized for what's being asked, based on this message's actual content and scope, not on an assumption carried over from earlier in the conversation.

## Output format: two-tier bar for surfacing

Always run the assessment. Whether to surface it depends on both the size of the mismatch and whether this message is a fresh task or a continuation:

**Fresh, standalone task** (a new ask, a topic switch, the first message in the conversation): use the normal bar. Surface it whenever the model is a clear mismatch, see calibration below.

**Continuation of the current task** (a follow-up edit, "also add X," refining something already in progress): use a higher bar. Only surface it if the added scope itself would justify a mismatch call on its own, e.g. a "small" follow-up that actually asks for something long, high-stakes, or structurally more complex than what came before (this is exactly the case that's easy to wave through, so don't discount it just because it's phrased as an add-on). Skip it for genuinely minor tweaks that don't change the task's difficulty.

**Current model is a good fit, or the mismatch doesn't clear the applicable bar:** Say nothing about the model check at all. Just go straight to doing the work. No "model check" message, no confirmation, nothing, silent pass.

**Mismatch clears the applicable bar:** Stop before doing the work and surface it using this shape:

```
Quick model check: this looks like [a simple lookup / a standard coding task / a complex multi-step analysis / etc.], and [current model] is [undersized / overkill] for it. I'd suggest **[Model Name]** (confidence: [low/medium/high]).

Why: [1-2 sentences of concrete reasoning tied to the signals above, not generic].

Want to switch to it, or should I go ahead on [current model]?
```

Wait for their answer before proceeding in this case, a real mismatch is worth a beat of interruption. If they say yes, tell them to pick it from the selector and resend (or just continue as they prefer). If they say no or don't want to bother, proceed on the current model right away.

**Calibrating "real mismatch" vs. "good enough":** don't flag borderline cases, e.g. a moderately complex task on Sonnet when Opus would be marginally better isn't worth interrupting for. Reserve the interruption for cases with a clear gap: a genuinely deep/high-stakes task on Haiku, a trivial one-liner on Opus/Fable, a massive long-horizon agentic task on anything less than Opus/Fable, etc. For continuations, additionally ask: does *this specific addition*, judged on its own, clear that same bar? If yes, surface it even though it's phrased as a follow-up. If it's ambiguous either way, don't interrupt, silently proceeding is the safer default when the gap is unclear.

## Notes

- Never claim you switched the model, you can't. Only the person can, via the UI.
- The assessment runs every turn, including inside an ongoing task, so scope drift gets caught, but the higher bar on continuations keeps that from turning into a running commentary on every reply.
- If the person explicitly names a model they want ("use Opus for this"), still run the assessment, but only note it if that seems like a real mismatch (e.g. Haiku requested for a very deep task); otherwise proceed without comment.
