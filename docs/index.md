# Documentation

This index is the guided path through the repository. Each section builds on the
one before it, but you can jump to whatever you need.

## 1. Fundamentals

Read these first if prompt engineering is new to you, or if you want a shared
vocabulary for the rest of the guide.

- **[What is a prompt?](fundamentals/what-is-a-prompt.md)** — The prompt as a
  program, the role of context, and why "just ask better" is incomplete advice.
- **[Anatomy of a prompt](fundamentals/anatomy-of-a-prompt.md)** — The five
  components every robust prompt has, and what goes wrong when one is missing.

## 2. Techniques

Specific patterns, each with when it helps, when it does not, and how to tell the
difference.

- **[Few-shot prompting](techniques/few-shot.md)** — Teaching by example;
  choosing examples is most of the work.
- **[Chain-of-thought](techniques/chain-of-thought.md)** — Eliciting reasoning,
  and the cases where it actively hurts.
- **[Role prompting](techniques/role-prompting.md)** — What a persona actually
  changes, and the placebo effect around it.

## 3. Testing

The section that makes the rest of the repository credible.

- **[Why test prompts?](testing/why-test-prompts.md)** — The failure modes that
  motivate evaluation, and what "working" even means.
- **[Evaluation checklist](testing/evaluation-checklist.md)** — A concrete,
  reusable checklist you can apply to any prompt before shipping it.
- **[Sandbox notes](testing/sandbox-notes.md)** — Environment notes for running a
  Git/GitHub workflow inside a confined agent sandbox, including the failures that
  masquerade as credential problems.

## Suggested reading order

**If you are shipping something this week:** Fundamentals →
[Anatomy of a prompt](fundamentals/anatomy-of-a-prompt.md) →
[Evaluation checklist](testing/evaluation-checklist.md). Skip the theory, apply
the checklist, move on.

**If you are building a system with many prompts:** read in order. The testing
section will make more sense once you have seen the techniques and understand
which ones are fragile.

**If a prompt of yours is misbehaving:** start with
[Why test prompts?](testing/why-test-prompts.md) — it catalogues the common
failure modes and is usually enough to identify yours.

## A note on scope

This guide covers prompting for large language models in general. It avoids
model-specific folklore ("this magic phrase works on model X") because such
advice ages badly. Where a technique depends on model capability, that is called
out explicitly.

Techniques described here are not neutral about risk: the testing section
includes guidance on prompts that produce confidently wrong output, which is the
most common way a working prompt becomes a production incident.