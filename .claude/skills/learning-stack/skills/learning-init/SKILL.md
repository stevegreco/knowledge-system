---
name: learning-init
description: Set up a new repo as a learning project the user builds slowly, piece by piece. Turns a project idea or plan (notes, a pasted plan, a claude.ai doc link) into a README plan, a docs/learning-path.md of ordered learning-sized issues, an AGENTS.md that stops agents from building ahead or writing the learner's code, a CLAUDE.md that imports it, and a .gitignore. Use when the user wants to "set up a learning repo", "init a project I want to learn", "break this into learning issues", "make agents go slow in this repo", or runs /learning-init. Not for scaffolding application code.
---

# Learning Init

Set up a repo whose purpose is for the user to learn by building it. The output is planning and agent-behavior files only. No application code, no scaffolding, no git commands.

## Workflow

### 1. Get the plan

Find the source material, in this order:

- A plan the user pasted or linked. For a claude.ai artifact link, read it with the docs tools (load the docs skill first); other links with the matching reader.
- Existing files in the target directory (a README or notes).
- Otherwise interview the user. Ask at most three questions per message, starting with: what they're building, what they most want to understand by the end, and their language/stack if they know it.

Also learn, from the source or by asking: the learner's current experience with the stack, and anything explicitly out of scope. Don't ask about details the plan already settles. Leave genuinely open decisions as open questions in the README, and make settling them the first issue.

### 2. Check the target directory

List it. Never overwrite an existing file without asking. If a README already exists, offer to merge the plan into it.

### 3. Write README.md

The project plan, in this order (drop sections the project doesn't need):

1. One-paragraph purpose, stating that learning is the primary goal
2. Learning goals: "By the end, I should be able to explain and demonstrate:" + bullets
3. Scope: in scope / out of scope
4. Architecture: one paragraph + a mermaid diagram if there is more than one component
5. Components, data model, tech stack with a "why" column, deliberate decisions
6. Roadmap: phases table (phase · deliverables · exit gate) + a link to docs/learning-path.md
7. Evaluation / success criteria, if the project has a measurable output
8. Open questions as `- [ ]` tasks
9. Working principles

Keep the source plan's substance. Write plainly: short sentences, no filler.

### 4. Write docs/learning-path.md

Read [references/issue-design.md](references/issue-design.md) before writing it. Start from the skeleton in [assets/learning-path-template.md](assets/learning-path-template.md).

### 5. Write AGENTS.md and CLAUDE.md

Copy [assets/AGENTS-template.md](assets/AGENTS-template.md) to `AGENTS.md` and fill every `{{placeholder}}` with project-specific content. The build-ahead examples must name real later issues from the learning path, so agents can see concretely what "getting ahead" means in this project.

Write `CLAUDE.md` containing only:

```
@AGENTS.md
```

### 6. Write .gitignore

Match the stack: language build/cache dirs, virtualenvs or `node_modules`, `.env` and `.env.*` (but keep `.env.example`), local data and model dirs the plan mentions, and editor/OS files.

### 7. Hand off

Don't scaffold code, run `git init` or commit. Summarize the files created and list the first issue. Give the git setup as one command in its own bash block:

```bash
git init && git add . && git commit -m "Add learning plan"
```

When the user has a GitHub repo, offer to create one issue per learning-path section with `gh issue create`, in order, titled `NN · Title` with the section text as the body. List the titles and get confirmation before creating any.
