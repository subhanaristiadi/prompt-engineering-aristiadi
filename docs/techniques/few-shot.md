# Few-shot prompting

## What it is

Few-shot prompting supplies worked examples of the task before asking for a new
answer. Instead of describing the transformation in prose, you demonstrate it.

Zero-shot:

```text
Convert the following sentence to past tense: She runs to the store.
```

Few-shot:

```text
She runs to the store.  ->  She ran to the store.
They build houses.      ->  They built houses.
He writes a letter.     ->  He wrote a letter.

We go home.
```

## When it helps

Few-shot is most effective when the *task* resists description but is easy to
demonstrate:

- **Idiosyncratic formats** — your house style for error messages or commit titles
- **Judgement calls with fuzzy boundaries** — what counts as a "critical" bug
- **Classification with a nuanced label** — where the label definition is a matter
  of degree, not kind
- **Edge-case handling** — showing how to treat the exception rather than
  describing it

In these cases an example is worth a paragraph, because the model copies a pattern
more reliably than it applies a rule.

## When it does not help

Few-shot does poorly when:

- **The task is already well-specified.** If your instructions are clear, examples
  add tokens and bias without improving accuracy.
- **Examples are unrepresentative.** Three examples that all look alike teach the
  model that all inputs look alike.
- **The task is reasoning-heavy.** Showing the final answer of a three-step
  derivation teaches the format of an answer, not the reasoning. Pair with
  [chain-of-thought](chain-of-thought.md) or show the reasoning.
- **You need guaranteed correctness.** Examples constrain style, not entailment.
  A model can copy the form of an example and violate its logic.

## The real work: choosing examples

Example selection dominates prompt quality. The instruction text is often less
important than which eight examples you chose.

**Cover the input distribution.** If 30% of your inputs are malformed, roughly
30% of your examples should be. Demonstrating the clean path teaches nothing about
the dirty one.

**Include the hard cases deliberately.** The example that took you longest to
write is usually the most valuable one, because it shows the boundary.

**Mix label frequencies to match reality.** For classification, if POSITIVE
outnumbers NEGATIVE 9:1 in production, an evenly balanced example set will
over-predict NEGATIVE. Match the deployment distribution, or accept the bias
knowingly.

**Prefer diversity over redundancy.** Eight near-identical examples convey less
than eight that each teach something different. If two examples support the same
lesson, drop one.

**Place examples near the question.** In long contexts, examples far from the
final query are easier to lose. Order: instructions, examples, input.

## Ordering effects

Models are measurably sensitive to example order. Two consequences:

1. **Position bias.** The first and last examples tend to exert more influence.
   Put your most important examples at extremes.
2. **Instability.** Reordering examples can shift accuracy by several percentage
   points with no other change. This is a useful, cheap experiment — and a reason
   not to trust a single evaluation run.

Because order matters, shuffle it on every evaluation to measure real performance
rather than the performance of one lucky ordering.

## Few-shot as retraining

A practical framing: few-shot prompting lets you "train" a model on your task
using tokens instead of gradients. The parallel holds further than it first
appears:

- Examples are training data; the same quality concerns apply
- Format inconsistencies in examples teach the model to be inconsistent
- Leakage between examples and test inputs inflates measured accuracy — if an
  evaluation case appears in the examples, exclude it

That leakage point is the most common evaluation error in few-shot systems, and
[why test prompts](../testing/why-test-prompts.md) treats it in detail.

## A concrete example

Task: classify support tickets by urgency.

```text
Classify each support ticket as URGENT, NORMAL, or LOW.

<examples>
Ticket: "Payment failed and I'm being charged twice."
Label: URGENT

Ticket: "How do I change my notification preferences?"
Label: LOW

Ticket: "The export button returns a blank file for reports over 1000 rows."
Label: NORMAL

Ticket: "I can't log in, I've tried resetting and it still fails."
Label: NORMAL
</examples>

Ticket: "{new_ticket}"
Label:
```

Note the fourth example. "I can't log in" *sounds* urgent, and a model without
that example tends to over-predict URGENT on login complaints. The example
encodes a policy decision (account lockout is NORMAL because self-service reset is
available) that would take a paragraph to state and probably still be applied
inconsistently. This is the archetypal few-shot win.

The third example similarly teaches the boundary between NORMAL and URGENT for
functional-but-degraded issues.

## Key takeaways

- Few-shot teaches by demonstration; use it when description is hard and
  demonstration is easy
- Choose examples to cover the input distribution, not to look tidy
- Example order introduces bias and variance — shuffle when evaluating
- Treat examples as training data, including its leakage hazards
- Do not use few-shot to paper over an underspecified task; fix the specification
  first

Next: [Chain-of-thought](chain-of-thought.md).