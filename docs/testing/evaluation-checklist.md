# Evaluation checklist

A reusable procedure for taking a prompt from "seems to work" to "known to work."
Apply it before shipping, and re-apply after every edit.

Copy this into your repository's PR template if prompts are part of your codebase.

## Before you write the prompt

- [ ] **Task is stated in one sentence.** If you cannot write it, the task is not
      defined yet.
- [ ] **Success is defined.** A specific, observable criterion: what must an output
      contain or match to pass?
- [ ] **Failure cost is understood.** Is a wrong answer embarrassing, or expensive?
      This sets how much evaluation is warranted.
- [ ] **The task is genuinely a prompt problem.** Some tasks are retrieval problems,
      data problems, or impossible. Confirm prompting is the lever before tuning it.

## Building the dataset

- [ ] **Cases come from realistic inputs.** Sampled from production or representative
      of it — not invented to illustrate the happy path.
- [ ] **Minimum 20 cases**, or all cases if the domain is small. Fewer than 20 makes
      results indistinguishable from noise.
- [ ] **Held-out set is separate.** You iterate on a dev split; the held-out split is
      only touched for final judgment.
- [ ] **Distribution matches production.** Include the messy inputs at roughly their
      real frequency: malformed, empty, adversarial, very long.
- [ ] **Edge cases are explicitly included.** Empty input, maximum length, wrong
      language, input that resembles instructions.
- [ ] **No leakage between examples and test cases.** For few-shot prompts, ensure no
      test case appears in your examples.

## Writing the prompt

- [ ] **All five components present:** task, input, constraints, output format,
      examples where useful. See [anatomy](../fundamentals/anatomy-of-a-prompt.md).
- [ ] **Input is delimited.** Data is clearly separated from instructions.
- [ ] **Output format is explicit and parseable.** If code parses it, the format is a
      contract.
- [ ] **Constraints are in a list**, not buried in prose.
- [ ] **No contradictions** between system prompt and prompt, or among constraints.
- [ ] **Assembled prompt reviewed end to end**, including injected context.

## Measuring

- [ ] **Baseline recorded.** The current version's score, per metric.
- [ ] **Metrics are separated:** correctness, format compliance, and latency/cost are
      three measurements, not one.
- [ ] **Pass criterion is applied mechanically**, by script, not by reading outputs.
- [ ] **Deterministic checks used where possible** — schema validation, substring
      match, numeric comparison — before reaching for LLM-as-judge.
- [ ] **LLM-as-judge (if used) is validated** against a hand-graded sample.

## Interpreting results

- [ ] **Variance is estimated.** Run the suite more than once; temperature and
      sampling mean a single run is one sample, not the truth.
- [ ] **Improvements are significant.** A jump from 82% to 85% on 20 cases is
      noise. Scale the dataset or accept the uncertainty.
- [ ] **Failures are examined individually**, not only counted. The *kind* of failure
      determines the fix.
- [ ] **Example ordering is shuffled** for few-shot prompts, since order alone moves
      scores.
- [ ] **Results are attributable.** For each failing case, you can name the prompt
      component that caused the failure.

## Before shipping

- [ ] **Full suite passes at the accepted threshold.**
- [ ] **Format violations handled in code**, not merely assumed absent.
- [ ] **Prompt is in version control**, with the dataset it was evaluated on.
- [ ] **Evaluation is re-runnable** by someone who did not write it — scripted, no
      manual steps.
- [ ] **CI runs the evaluation** if the prompt ships in software.
- [ ] **Monitoring exists for the deployed prompt**: error rates, refusal rates,
      parse failures, a sample of outputs reviewed periodically.

## Ongoing

- [ ] **Every prompt change re-runs the suite.** No exceptions; this is the habit
      that matters.
- [ ] **Scores are recorded over time**, so slow degradation is visible.
- [ ] **Model or version upgrades trigger a full re-evaluation.** Prompts are tuned to
      a model; an upgrade is a breaking change until proven otherwise.
- [ ] **Production samples are periodically added** to the dataset as the input
      distribution shifts.

## Quick reference: what to try first

When a case fails, check in this order:

| Check | Likely fix |
| --- | --- |
| Was the input within the prompt's stated scope? | Add scope constraints, or handle outside the prompt |
| Is the failing component identifiable? | Use the anatomy symptom table |
| Does it fail consistently or intermittently? | Consistent → specification; intermittent → constraint tightening |
| Do similar cases pass? | Add an example covering the boundary |
| Does reasoning help? | Try [chain-of-thought](../techniques/chain-of-thought.md) |
| Is the answer wrong but the reasoning plausible? | The task may exceed the model's capability — reconsider the design |

## The honest summary

A prompt without an evaluation is a hypothesis. This checklist is how you
convert it into a claim you can defend — and how you notice, months later, when
something has changed.