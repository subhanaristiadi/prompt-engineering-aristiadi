# Prompt Engineering

A practical, opinionated guide to writing prompts that work — and to knowing
*when* they work.

Most prompt engineering material is a list of tricks. This repository is
organised around a single idea: **a prompt is a specification, and a
specification you cannot test is a guess.** Every technique here is paired with
a way to evaluate it, because the difference between a clever prompt and a good
prompt is measured, not asserted.

## What's here

| Section | Contents |
| --- | --- |
| [Fundamentals](docs/fundamentals/) | What a prompt actually is, and the anatomy of one |
| [Techniques](docs/techniques/) | Few-shot, chain-of-thought, role prompting |
| [Testing](docs/testing/) | Why prompts need tests, and how to build them |

Start with the [documentation index](docs/index.md) if you want the guided path.

## Who this is for

- Engineers integrating an LLM into a product and needing predictable behaviour
- Analysts writing prompts they will run more than once
- Anyone who has had a prompt "stop working" and wants to know why

No prior prompt engineering experience is assumed. Familiarity with calling an
LLM API helps for the testing section, which includes runnable examples.

## The three principles

Everything in this repository derives from three claims:

**1. Prompts are specifications.**
A prompt defines a task the model should perform. Like any specification, it can
be ambiguous, incomplete, or internally inconsistent — and those defects cause
failures far more often than "the model isn't smart enough."

**2. Structure beats cleverness.**
Explicit instructions, clear input/output boundaries, and worked examples
outperform rhetorical tricks. If you find yourself writing "you are a genius
world-class expert," you are compensating for a structure problem.

**3. Untested equals unknown.**
A prompt that works on three hand-picked inputs has demonstrated very little.
Reliability requires a held-out set, a defined pass criterion, and a habit of
re-running after every edit.

## Quick example

Before:

```text
Summarize this article
```

After:

```text
Summarize the article below in exactly 3 bullet points.
Each bullet must be one sentence, under 20 words.
Cover: the main claim, the strongest evidence, the key limitation.
Do not include opinions or recommendations not present in the text.

<article>
{article_text}
</article>
```

The second prompt is longer, but it removes decisions the model would otherwise
make for you: length, granularity, coverage, and the temptation to editorialise.
That is the pattern the rest of this repository generalises.

## Status

Active work in progress. The [testing](docs/testing/) section is the most mature;
the techniques pages are being expanded with more worked examples.

## Contributing

Corrections and counterexamples are welcome. If a technique here fails for your
use case, that is interesting — open an issue describing the task and the
failure, ideally with the prompt and a sample of inputs.

## License

MIT