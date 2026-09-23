---
description: Pressure-test a plan, decision, or approach before committing to it, surfacing failure modes and unstated assumptions.
argument-hint: "<plan-or-decision>"
---

Pressure-test a plan, decision, or approach before you commit to it. Surface the
*genuine* holes — wrong assumptions, failure modes, a simpler path — and propose a
fix for each. Stay out of the way when the plan is already sound.

## Arguments

`$ARGUMENTS` is the plan or decision to harden. If empty, use the most recent
concrete plan or decision from the conversation. If there's nothing concrete to
test, ask the user for the decision in one line — don't guess and don't harden a
blur.

Reach for this right before committing to a direction: an architecture call, an
approach to a feature, a tradeoff you've half-decided.

## The one rule that matters

**Earn every concern.** This skill is only useful if it is willing to say "this is
solid — go." Do not manufacture issues to look thorough. If the plan is sound, say
so in one line and stop. A skill that always finds three problems is noise, and the
user will stop trusting it. Silence on a point is a valid, common outcome.

## Steps

1. **Restate the decision in one line** — what's being committed to and the goal
   behind it. If you can't, ask. You can't harden what you can't state.

2. **Probe these axes silently** — only surface what actually fires:
   - **Hidden assumptions** — what must be true for this to work that nobody stated?
   - **Failure modes** — what breaks, and what's the blast radius when it does?
   - **Reversibility** — if this is wrong, how expensive is it to undo? One-way
     doors deserve more scrutiny than easily-reverted ones.
   - **Simpler path** — is there a materially smaller or safer way to get ~90% of
     the value?
   - **The blind spot** — the thing the plan doesn't mention at all.

3. **Triage what you find** into three tiers:
   - **🛑 Blocker** — genuinely likely to fail, cost, or hurt. Worth stopping for.
     Rare.
   - **⚠ Risk** — a real hole, not fatal. Worth a fix or a conscious accept.
   - **(nothing)** — sound. The expected outcome for a well-thought-out plan.

4. **Report — short.** One-word verdict, then at most the top 1–3 items. For each:
   the concern in one line, *why it's real* in one line, and **a concrete suggested
   fix**. Never a bare criticism without a proposed path forward.

5. **Push only on blockers.** For a 🛑, say plainly that you think the current path
   is wrong, why, and what you'd do instead — this is the "we need to discuss it"
   case, so make the case. For ⚠ risks, present and let the user decide; don't
   litigate. Close with the *single* question whose answer would most change your
   read — and only if one genuinely exists.

## Format

```
**Verdict: <Solid / One fix / Hold up>**

🛑/⚠ <concern> — <why it's real>
→ <suggested fix>

<optional: the one question that would change my read>
```

If the verdict is **Solid**, that's the entire output plus one line of why. Done.

## Notes

- Fast by design. No preamble, no "great question," no exhaustive checklist dump.
  Signal, not coverage. One round by default — go deeper only if the user pulls a
  thread.
- A second set of eyes, not a gate. The user owns the call. Your job is to make
  sure they're choosing with their eyes open, not to win the argument.
- Match the stakes. A throwaway script doesn't earn the scrutiny of a schema
  migration — calibrate depth to reversibility and blast radius.
