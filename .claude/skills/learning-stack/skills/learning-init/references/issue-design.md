# Designing learning issues

Use these principles to split a plan into issues that teach, not just deliver.

## Each issue

- **One concept per issue.** Name it first. If an issue teaches two things, split it.
- **One or two sittings.** If the build list has more than about four bullets, split it.
- **Smallest concrete build.** The build exists to make the concept tangible, not to finish a feature.
- **Observable "done when".** Things the learner can run, print or show: a command's output, a passing test, a number, a before/after sample. Never "understands X".
- **Two or three "check your understanding" questions.** Ask why and what-if, answerable in the learner's own words. Avoid trivia or questions answered by reading the issue text.

Format:

```markdown
### NN · Title

**Concept:** One or two sentences: the idea, and why it matters.

**Build:**
- ...

**Done when:** ...

**Check your understanding:**
- ...
```

## Ordering

- **Foundations before features.** Environment, data store, and the yardstick (eval set, test fixtures) come first.
- **Do the core idea by hand once before using it in the pipeline.** For example, compute similarity with numpy before using a vector database, or write a raw SQL query before using an ORM, or handle a request with the standard library before a framework.
- **Write the yardstick early.** If the project has a measurable output, write the eval set before building what it measures, so it isn't biased toward the implementation.
- **Measure parts separately.** Put an issue that measures an intermediate stage (retrieval, parsing, a single service) before the stage that depends on it, so failures can be blamed on the right half.
- **Baseline, then analysis, then improvement.** After a phase reaches its baseline, put a failure-analysis issue before any optimization issue. Each later improvement issue records its result next to the baseline.
- **Simplest working end-to-end path first.** The first phase ends with something that works badly but completely.
- **Defer abstractions.** Introduce frameworks, libraries that hide the core mechanism, and generalization only after the learner has done it by hand. Defer production concerns (auth, scaling, deployment) unless they are the learning goal.
- **Gates.** End each phase on an issue whose "done when" is the phase's exit gate from the README, labeled "(gate)" in the overview table.
- **Stretch last.** Put optional experiments in one final issue as a bullet list.

## The file

- An intro saying what each issue contains and that each section maps to one GitHub issue.
- An overview table: `# · Issue · Phase`.
- A `## Phase ...` section per phase with a one-line intro, then its issues.
- Number issues `01`, `02`, ... continuously across phases. Use the `### NN · Title` heading format so each section maps cleanly to one GitHub issue.
- Typical size: 15 to 25 issues. Fewer than 10 means issues are too big; more than 30 means the project is too big or the issues are too thin.
