# Role prompting

## What it is

Role prompting assigns the model a persona or expertise before the task:

```text
You are a senior security engineer performing a code review.
```

The intent is to shift the model toward the vocabulary, priorities, and standards
associated with that role.

## What it actually does

A role is a compact bundle of correlated expectations. Saying "senior security
engineer" implies:

- What to look for (injection, authz gaps, secret handling)
- What to ignore (formatting, naming conventions)
- How to phrase findings (severity, exploitability, remediation)
- What level of detail is appropriate

This is genuine and useful. The model has observed enough text authored by and
about such people that the phrase reliably conditions its output toward those
patterns.

What a role does **not** do is grant capability. Telling a model it is a
mathematician does not make it better at arithmetic; telling it it is an expert
does not make it correct. The persona changes emphasis and register, not
competence.

## Useful role prompts

Roles earn their place when they encode a *selection* of concerns:

```text
You are reviewing this pull request as a database administrator.
Prioritise: index usage, query plans, migration reversibility, lock duration.
Deprioritise: code style, naming, test coverage.
```

Here the role's value is the prioritisation list — the persona is shorthand for a
particular set of concerns. Note that the explicit priority list does most of the
work. The role sentence orients; the list specifies.

## The placebo problem

A large share of role prompting advice is superstition:

```text
You are a world-class genius expert with 20 years of experience.
```

This is an attempt to buy capability with flattery. There is no reliable evidence
that it improves answer quality, and some reason to think it is counterproductive:
it consumes tokens, and it encourages the model toward confident, flowing prose —
which is exactly the register in which errors are least visible.

The test is straightforward. Take a prompt, add a grandiose role line, and evaluate
both on the same held-out set. If the difference is within noise, the role is
decoration. Run this once per task type and you will stop adding these lines.

## Roles and safety

Roles interact with safety behaviour in ways worth understanding:

- A role can make a model more willing to engage with a topic in a technical
  register
- A role does not reliably override safety training, and prompts that assume it
  will are both unreliable and a sign of poor task framing
- Framing a task as legitimate (a security engineer auditing their own system)
  sometimes helps with false refusals *when the task genuinely is legitimate*

If you find yourself searching for a persona that lets the model do something it
keeps declining, the honest question is whether the task should be done, not which
framing prevails.

## Practical guidance

**Use a role to narrow scope.** "Review as a DBA, prioritising X" is better than
"Review as an expert."

**State the priority explicitly.** Do not rely on the persona to imply it. The
persona is a hint; explicit instructions are a specification.

**Keep it short.** One sentence plus a prioritisation list. Long persona
backstories are mostly tokens you could spend on constraints.

**Test it.** The evaluation for a role change is identical to any other prompt
change: same held-out set, compare. See the
[evaluation checklist](../testing/evaluation-checklist.md).

**Do not stack roles.** "You are a senior engineer and a lawyer and a designer" is
a model that is mediocredly three things. Use separate calls.

## A comparison

Weak:

```text
You are an expert technical writer with decades of experience. Write
documentation for this API.
```

Stronger:

```text
You are writing reference documentation for an internal HTTP API.

Audience: backend engineers familiar with REST, new to this service.
Prioritise: exact request/response schemas, error codes and their meanings,
authentication requirements.
Omit: tutorials, marketing language, rationale for design decisions.
Tone: declarative, present tense, no second person.
```

The second prompt has no grandeur and is far more useful. Every sentence removes
a degree of freedom the first one left open.

## Key takeaways

- A role is shorthand for a bundle of concerns — useful for orientation, not
  capability
- Pair every role with an explicit priority list; the list does the work
- Grandiose persona lines are unproven decoration and can encourage unfounded
  confidence
- Never stack roles; use separate prompts

Next: [Why test prompts?](../testing/why-test-prompts.md).