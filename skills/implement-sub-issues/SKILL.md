---
name: implement-sub-issues
description: "Implement every open sub-issue of one GitHub parent issue with subagents, one issue at a time: each on its own branch and PR into the parent's integration branch, one CodeRabbit review per sub-issue PR, merged there; then one draft PR to main and preview links for Ovi's visual check. Use in Codex, in the project folder, when Ovi names a parent issue (the spec issue that $to-tickets filled with sub-issues). Usage: $implement-sub-issues 84 (a number, #84, or the issue URL)."
disable-model-invocation: true
---

# Implement sub-issues

You are the orchestrator. Follow `process.md` in this skill folder, step by
step, in order. You do not write code yourself: every piece of work is done
by a subagent you spawn, one at a time, and you wait for each one before you
start the next. You run only read-only `git` and `gh` commands to check what
the subagents report.

The parent issue number comes from the argument after `$implement-sub-issues`
(a number, `#84`, or an issue URL). If there is none, ask for it and stop.

How to write, test and commit code is defined by the project's own
instructions (AGENTS.md), which every subagent loads. Do not add instructions
about how to do the work to any subagent's prompt. Pass each prompt in
`process.md` exactly as written, with the values filled in.

In Claude Code the same procedure exists as the saved workflow
`/implement-sub-issues`; use that there, not this skill.
