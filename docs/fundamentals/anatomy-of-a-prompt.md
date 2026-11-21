# Anatomy of a prompt

Robust prompts share five components. Not every prompt needs all five, but when a
prompt fails it is usually because one is missing. Treat this as a checklist to
diagnose, not a template to fill mechanically.

## The five components

### 1. Task

What the model should do, stated as an imperative.

```text
Summarize the document below.
```

Weak versions are questions (`Can you summarize this?`) or descriptions of a
desired outcome (`I need a summary`). Imperatives are unambiguous about what
action is required.

### 2. Input

The material the task operates on, clearly delimited.

```text
<document>
{content}
</document>
```

Delimiters matter more than they appear to. Without them, the model must infer
where your instructions end and the data begins — and input text that happens to
look like instructions (a document containing "ignore the above") is genuinely
confusable. Explicit delimiters remove the ambiguity.

Use consistent, unlikely-to-collide markers. XML-style tags work well because they
rarely appear in natural text.

### 3. Constraints

The boundaries of acceptable output.

```text
- Maximum 3 bullet points
- One sentence each
- No information not present in the document
```

Constraints do the heavy lifting. Every constraint you omit is a decision the
model makes on your behalf, possibly differently across runs.

### 4. Output format

The exact shape you need to parse.

```text
Respond with JSON only:
{"summary": ["...", "..."]}
```

If downstream code parses the output, the format is part of the contract, not a
suggestion. State it explicitly and consider providing a schema. Models are
generally good at following a concrete schema and less good at guessing your
implicit expectations.

### 5. Examples

Optional worked input/output pairs. Covered in detail in
[few-shot prompting](../techniques/few-shot.md).

## Assembling them

Most of this guide's prompts follow task → constraints → format → input, with
examples before the input when used. A complete example:

```text
Summarize the document below into exactly 3 bullet points.

Constraints:
- Each bullet is one sentence of at most 20 words
- Bullets cover: the main claim, the strongest evidence, the key limitation
- Include nothing not stated in the document

Format:
{"bullets": ["...", "...", "..."]}

<document>
{content}
</document>
```

Note that input comes last. This is deliberate: instructions placed after the
input are easier to overlook, and instructions placed *before* the input set up
the frame in which the model reads the data.

## Diagnosing failures with the anatomy

When output is wrong, identify which component was underspecified:

| Symptom | Missing component |
| --- | --- |
| Output ignores part of your request | Constraints |
| Output format breaks your parser | Output format |
| Model summarises its own opinions | Constraints (scope) |
| Model treats input text as instructions | Input (delimiters) |
| Results vary wildly between runs | Constraints, or too little structure |
| Model does the wrong task entirely | Task (too vague) |

This table resolves a surprising fraction of "the model is bad at this" reports.

## Common mistakes

**Burying constraints in prose.** A constraint embedded in a paragraph gets
dropped under long context. Put constraints in a list, near the end.

**Contradictory instructions.** "Be concise" plus "explain your reasoning in
detail" is a contradiction the model resolves arbitrarily. Contradictions between
the system prompt and the user prompt are worse: re-read the system prompt when
debugging.

**Over-constraining.** Every constraint is a stated requirement; thirty
constraints make a prompt brittle and hard to satisfy. Move must-haves into the
prompt and leave nice-to-haves to evaluation, or split the task.

**Assuming format stability.** A prompt that "usually" returns JSON will
eventually return prose. If you parse it, demand the format explicitly and handle
the violation.

## Key takeaways

- Five components: task, input, constraints, output format, examples
- Delimiters prevent input from being read as instructions
- Omitted constraints become the model's arbitrary decisions
- Use the symptom table to locate the weak component before rewriting blindly

Next: [Few-shot prompting](../techniques/few-shot.md).