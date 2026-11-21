# Chain-of-thought

## What it is

Chain-of-thought (CoT) prompting asks the model to produce intermediate reasoning
before its final answer. Instead of jumping to a conclusion, it works through
steps, and the answer is derived from those steps.

Implicit CoT:

```text
A store had 120 apples. It sold 45% on Monday and 30 of the remainder on
Tuesday. How many apples are left?
```

Explicit CoT:

```text
A store had 120 apples. It sold 45% on Monday and 30 of the remainder on
Tuesday. How many apples are left?

Work through this step by step before giving your final answer.
```

## Why it works

For tasks that require composition of multiple operations, the answer is not
directly retrievable — it must be computed. CoT helps because:

1. **It allocates computation.** Each reasoning token is an opportunity for the
   model to do work that a single output token cannot.
2. **It exposes intermediate state.** Errors become visible at the step where they
   occur, not only at the end.
3. **It resolves ambiguity earlier.** Writing out the interpretation of
   "45% of the remainder" forces the model to commit, which constrains the answer.

Empirically, CoT produces large gains on arithmetic, multi-hop question answering,
and symbolic manipulation — and much smaller gains, or none, elsewhere.

## Where it helps

- Multi-step arithmetic and unit conversion
- Multi-hop questions requiring chained facts
- Planning and constraint satisfaction
- Anything where a human would need scratch paper

## Where it hurts

This is the part most guides omit.

**Simple lookups.** For "what is the capital of France," CoT adds latency and
tokens for no gain, and gives the model an opportunity to talk itself out of a
correct answer.

**When you want only the answer.** If your parser expects a label and the model
emits reasoning first, you now have a format problem. This is a solvable
engineering issue — request the reasoning in a separate field, or parse the tail —
but it must be handled deliberately.

**When reasoning is unfaithful.** Models can produce reasoning that looks
compelling and does not reflect the actual basis of the answer. The written steps
are generated text conditioned on the input, not a trace of internal computation.
A confident chain of reasoning is therefore *not* evidence the answer is right,
and can create false confidence in human reviewers. Treat CoT as a compute aid,
not an audit trail.

**When it invites over-exploration.** Given room to reason, a model sometimes
invents complications the input does not support, then answers the complicated
question it created.

## Variants

**Zero-shot CoT** — just the instruction ("think step by step"). Cheap and often
surprisingly effective. Use this as your first attempt.

**Few-shot CoT** — examples that include reasoning, not only answers. Better when
you want a specific reasoning *style*, not merely reasoning. Combining with
[few-shot](few-shot.md) means your examples must show the steps.

**Structured CoT** — an explicit reasoning schema, which is more parseable and more
consistent:

```text
Answer in this format:
{"steps": ["...", "..."], "answer": "..."}
```

This is usually preferable in production because it separates reasoning from the
final value and keeps the answer machine-readable.

**Decomposition** — split the task into explicit sub-questions and answer each.
Useful when the reasoning has a natural sequence you want enforced.

## A worked example

Prompt without CoT, on a task that needs composition:

```text
Is this invoice compliant with the NET-30 policy?

Invoice: Net-45 terms, line items total $4,200, partial payment of $1,000
received on day 20.
```

A model may answer "no" — terms violate NET-30 — which is correct but hides the
harder question of whether the partial payment was properly allocated.

With structured CoT:

```text
Analyse the invoice against the policy. Answer in this format:

{
  "payment_terms_check": "...",
  "partial_payment_check": "...",
  "total_check": "...",
  "compliant": true | false,
  "reason": "..."
}

Policy: payment terms must be NET-30 or shorter; partial payments must be
allocated to line items before the due date; totals above $5,000 require a
purchase order.

Invoice: {...}
```

Now each dimension is checked separately. Failures are attributable: the output
says *which* check failed, which is what you need for remediation and for
evaluation.

## Costs

CoT is not free:

- **Tokens.** Reasoning can triple output length; cost scales accordingly.
- **Latency.** Users waiting on a chat response notice.
- **Evaluation difficulty.** More output means more places to be wrong — and
  judging reasoning quality is harder than judging answers.

For interactive products, a common pattern is to reason internally but return only
the conclusion, letting structured output format do the separation.

## Key takeaways

- CoT buys computation for tasks that need composition
- It is unnecessary for simple tasks and can reduce accuracy there
- Written reasoning is not a faithful audit trail; do not treat it as evidence
- Prefer structured CoT in production so reasoning and answer stay separable
- Measure the gain; it is task-dependent, not a universal improvement

Next: [Role prompting](role-prompting.md).