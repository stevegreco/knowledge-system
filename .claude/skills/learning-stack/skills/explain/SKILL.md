---
name: explain
description: Explain a concept in depth to a beginner, grounded in what just happened in the session. Use when the user runs /explain, or asks to explain, break down or unpack a term, tool, pattern or step that came up ("explain that", "what is X actually doing?", "I don't get why we did Y", "explain like I'm new to this", "go deeper on Z"). Explanation only. It doesn't change code or move the task forward.
---

# Explain

Teach one concept so a beginner understands it, using the current session's work as the example.

## Pick the concept

- If the user named it, explain that.
- If they said "that", "this" or nothing, use the most recent concept from the last agent turn: a term, a command, a design choice, or a line of code. If two or more are equally likely, ask which one, listing them in one line.
- If the concept depends on another one the user probably doesn't know, explain the prerequisite briefly first, or say it's a prerequisite and offer to explain it next.

## Shape of the explanation

Use these parts, in order. Skip a part only when it truly adds nothing.

1. **The short answer.** One or two plain sentences, with no jargon.
2. **The problem it solves.** What goes wrong without it. People learn a concept faster once they know why it exists.
3. **An everyday analogy.** Then one sentence on where the analogy breaks down.
4. **How it works.** Step by step, with a tiny concrete example: real numbers, a three-line snippet, or a small before/after. Use a short ASCII or mermaid diagram when the concept is a flow or structure.
5. **Where it shows up here.** Tie it to this session's actual work, linking files as `path:line` where possible. Use the real names from the code.
6. **Common mix-ups.** One to three misconceptions or gotchas beginners hit.
7. **Check yourself.** One question the user can answer in their own words. Then offer two related concepts to go deeper on.

## Rules

- Define every technical term the first time it appears, in plain words. Don't use a term before it's defined.
- Assume no background beyond basic programming unless the user says otherwise.
- Be thorough, not padded. Depth comes from steps and examples, not from repeating ideas.
- Don't edit files, run the task forward, or suggest next implementation steps. If the explanation exposes a bug or a better approach, mention it in one line at the end and leave the decision to the user.
- For library- or version-specific details, check the docs (for example with context7) instead of relying on memory, and say when something is uncertain.
- If the user asks a follow-up, keep the same beginner level unless they ask to go more advanced.
