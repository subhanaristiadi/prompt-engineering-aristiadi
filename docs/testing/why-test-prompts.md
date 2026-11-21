# Why test prompts?

## The problem

A prompt that works on the inputs you tried is a prompt that works on the inputs
you tried. That is a weaker statement than it feels like.

Prompt failures are silent. A conventional program with a bug crashes; a prompt
with a bug returns fluent, confident, plausible output that happens to be wrong.
Nothing in the response signals the defect. You discover it when a user reports it,
or when it reaches a decision that matters.

Testing is the only mechanism that converts "I tried it and it seemed fine" into
"it works, within these bounds, on this distribution."

## What "working" means

Before testing, define success. A pass criterion must be:

- **Observable** — a judgment someone or something can make about the output
- **Binary or scored** — so results aggregate
- **Tied to the task, not the prompt** — "returns valid JSON" is necessary but not
  sufficient; "extracts the correct total" is the real question

Vague criteria ("the answer should be good") produce untestable systems. Spend the
time to write the criterion down; it forces the task definition to become concrete,
which improves the prompt independently.

## The common failure modes

These recur across domains. Recognising yours is usually the fastest path to a fix.

### 1. Overfitting to hand-picked examples

You iterate against five inputs until they pass. The prompt now contains specific
phrasings tuned to those five — and generalises to nothing.

**Fix:** hold out a set you do not look at while iterating.

### 2. Distribution shift

The prompt was tested on well-formed inputs. Production has typos, empty fields,
mixed languages, 40,000-token documents, and the occasional pasted spreadsheet.

**Fix:** sample your evaluation set from real production data, not from inputs you
wrote.

### 3. Format drift

The prompt usually returns JSON. Occasionally it returns JSON with a preamble, or
prose explaining why it cannot answer. The parser breaks.

**Fix:** evaluate format compliance as its own metric, at volume, not by eye.
Handle violations in code rather than hoping.

### 4. Contradictory instructions

The system prompt says "be concise"; the user prompt says "explain each step."
The model picks one arbitrarily and picks differently next run.

**Fix:** review the assembled prompt end to end, not just the part you edited.

### 5. Non-determinism mistaken for unreliability

The same input gives different outputs. Sometimes this is a real bug; sometimes
it is a task with no defined correct answer, and the prompt was never going to be
stable.

**Fix:** decide whether variance is acceptable. If not, constrain harder. If yes,
measure *how much* rather than treating each variation as a defect.

### 6. Regression by edit

You fix case A and silently break cases B through F. Without a test set you will
not notice until much later, and you will not know which edit caused it.

**Fix:** re-run the full set after every change. This is the entire reason
regression suites exist.

## What a minimal evaluation looks like

It does not need infrastructure. It needs four things:

1. **A dataset.** Twenty to fifty cases drawn from realistic inputs. More is
   better; twenty is far better than none.
2. **A pass criterion.** Per case, pass or fail.
3. **A harness.** A script that runs every case and tallies results. Thirty lines
   of code.
4. **A baseline.** The current prompt's score, so changes are measurable.

That is the entire minimum viable evaluation. Everything beyond it — LLM-as-judge,
confidence intervals, stratified analysis — improves resolution but is optional.

## On LLM-as-judge

Using a model to grade outputs is common and useful, with caveats:

- It is a measurement instrument, so validate it — grade a sample by hand and
  check agreement
- Judges have biases: toward length, toward their own vocabulary, toward the
  first option shown
- It is cheapest for format and factuality checks, weakest for nuanced quality
  judgments
- Never use the same model both to generate and to judge without a hand-checked
  sample

For a first evaluation, deterministic checks (valid JSON, contains expected value,
matches regex) are more reliable than any judge and should be used wherever the
criterion allows.

## The discipline

Evaluation fails as a habit more than as a technique. Prompts drift because
editing is easy and re-testing is not. The countermeasures are mundane:

- Version prompts in source control, like code
- Re-run the suite on every change, in CI if the prompt ships in software
- Record scores over time, so degradation is visible
- Re-evaluate when the model or its version changes — a prompt is tuned to a
  model, and upgrading is a breaking change until proven otherwise

That last point is worth emphasising: **swapping models silently invalidates your
evaluation.** Re-run everything before trusting a prompt on a new model.

## Key takeaways

- Prompt failures are silent, which makes testing the only reliable signal
- Define the pass criterion before writing the prompt
- Use realistic, held-out inputs sampled from production
- Minimal evaluation is small: a dataset, a criterion, a script, a baseline
- Measure format compliance separately from correctness
- Treat model upgrades as breaking changes

Next: [Evaluation checklist](evaluation-checklist.md) — a reusable procedure.