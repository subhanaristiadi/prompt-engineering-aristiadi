# What is a prompt?

## The short answer

A prompt is the text a model conditions on when generating a response. That
sounds trivial, and it is — but the consequences are not.

The model has no memory of your intent, your product, or the previous
conversation unless it is in the prompt (or the context the system assembles
around it). Everything the model knows about your task at generation time is
either in the prompt or absent. This is why prompt writing feels unlike writing
documentation for a human: a human reader can ask a question, notice a gap, or
infer from tone. The model does none of these things. It produces the single most
plausible continuation of what you gave it.

## The prompt as a program

The most useful mental model: **a prompt is a program written in natural
language, executed by an interpreter that never errors.**

Consider this prompt:

```text
Extract the names of all the people mentioned.
```

A conventional program with an equivalent bug would crash — `which people?`,
`from what?`, `what counts as a name?`. The model instead does *something*
reasonable, which is worse, because the failure is silent. You get output that
looks like a correct answer to a question you did not quite ask.

This is the central asymmetry of prompt engineering. Ambiguity does not produce
an error; it produces a plausible guess. Your job is to shrink the space of
guesses until the output is the one you wanted.

## Context is part of the prompt

Strictly, the prompt is only what you write. In practice the model also sees:

- A **system prompt** describing its role and constraints
- **Retrieved documents** injected by your application
- **Conversation history** from earlier turns

These are part of the effective prompt whether or not you authored them. Many
"prompt engineering problems" are actually context problems — the instruction was
fine, but it was buried under 40,000 tokens of retrieved text, or contradicted by
a stale system prompt nobody re-read.

When a prompt misbehaves, the first question is not "how do I phrase this better"
but **"what exactly did the model see?"** Dump the full assembled prompt once. The
answer is frequently embarrassing and immediately actionable.

## Why "just write it better" is incomplete advice

The intuition that better writing yields better output is correct but
insufficient, for three reasons:

**1. The target is narrower than human communication.**
Good human prose is evocative and lets the reader fill gaps. A good prompt is
precise and closes gaps. Fluency is not the goal; disambiguation is.

**2. The failure modes are different.**
A human who is confused asks. A model that is confused answers confidently. You
cannot rely on the reader's metacognition as an error signal, so prompts must
encode assumptions explicitly.

**3. Testing is not optional.**
Because failures are silent, you cannot know a prompt works by reading it. You
have to run it, on inputs you did not choose to flatter it.

That last point is the thesis of this repository's [testing
section](../testing/why-test-prompts.md).

## A worked comparison

Weak prompt:

```text
Is this review positive or negative?
```

This leaves open: what to output (word? label? score?), what to do with mixed
reviews, whether to explain, and how long an answer is acceptable. Each is a
coin flip at inference time.

Stronger prompt:

```text
Classify the sentiment of the review below as exactly one of:
POSITIVE, NEGATIVE, MIXED.

Use MIXED only when the review contains both specific praise and specific
criticism. Do not explain your reasoning. Output only the label.

<review>
{review_text}
</review>
```

The second prompt is not "more politely written." It makes four decisions
explicit that the first one delegates to chance: the label set, the definition of
the ambiguous label, the required output format, and the absence of explanation.

## Key takeaways

- A prompt is everything the model sees, not only what you typed
- Ambiguity does not cause errors; it causes confident guesses
- Fluency is not precision — the goal is closing gaps, not sounding good
- Silent failure is the default, which makes evaluation mandatory

Next: [Anatomy of a prompt](anatomy-of-a-prompt.md) — the components that close
those gaps systematically.