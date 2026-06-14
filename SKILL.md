---
name: prompt-compression
description: Decode and respond to messages written in PCP (Prompt Compression Protocol) shorthand — tags like FIX/ADD/RM/REFAC/OPT/TEST/DOC/ASK/EXPL, @file/@file#L10-40/@file#fn:name references, $VAR session variables, EXPL:0/1/2 and FMT:diff/snippet/full output directives, and ~ / Δ scope shortcuts. Use this skill whenever a message starts with one of these tags, contains @file# or $VAR= syntax, or the user asks to "use PCP", "compress prompts", or "save tokens". Expand the shorthand to its full meaning, complete the task at normal quality, and shape the response per the output directives — without echoing the rules back or narrating the decoding.
---

# Prompt Compression Protocol (PCP) — decoder

The user writes terse, tagged requests to save input tokens. Your job: silently
expand each message to what it would mean in full English, do the work at
normal quality, and format the *output* per the directives. Never explain that
you're "decoding" — just respond as if the full-length request had been sent.

## Decoding table

**Intent tags (start of message):**
- `FIX` — fix a bug
- `ADD` — add a feature/function
- `RM` — remove code
- `REFAC` — refactor, preserve behavior
- `OPT` — optimize (perf/memory/etc.)
- `TEST` — write or extend tests
- `DOC` — write docs/report text/comments
- `ASK` — question; don't change code
- `EXPL` — explain something; no code change

**Scope references** — read the referenced file/section yourself (use file
tools) rather than asking the user to paste it:
- `@path/to/file` — whole file
- `@path/to/file#L10-40` — line range
- `@path/to/file#fn:name` — a specific function/method/class
- `~` — same file/scope as the user's previous message
- `Δ` — "only what changed since last time" — diff against the prior version
  you produced, not the whole thing

**Session variables** — `$NAME=value` defines a variable (e.g. project
context, language, style preferences). Later messages may just say `$NAME`;
substitute the stored value mentally. If `$NAME` is used before being defined
in this conversation, ask the user to define it once.

**Output directives** — apply to your response, don't mention them:
- `EXPL:0` — code/diff/snippet only, no prose
- `EXPL:1` — 1–3 sentence explanation alongside the result
- `EXPL:2` — full explanation
- `FMT:diff` — respond with a unified diff only
- `FMT:snippet` — show only the changed function/block, not the whole file
- `FMT:full` — show the whole file
- `LANG:xx` — write any prose in the given language

If a message has no `EXPL:`/`FMT:` directive, default to `EXPL:1` and
`FMT:snippet` for edits to existing code, and `FMT:full` for brand-new files —
this matches what most people want without being asked.

## Working with files

- If the user references `@file...` and you have file tools (Claude Code,
  Cowork, computer use), read the file yourself — don't ask them to paste it.
- If you're in a plain chat with no file access and the user defined
  `$F1 = <pasted content>` earlier in the conversation, treat later
  `$F1#fn:name` references as pointing at that earlier pastes — it's already
  in context, no need to ask for it again.
- For `FMT:diff`, produce a standard unified diff (`--- a/...` / `+++ b/...`
  with `@@` hunks) so it can be applied directly.

## First use in a conversation

If this is the first PCP-style message in the conversation and the user
hasn't pasted the `[PCP-1]` setup block from the protocol doc, you don't need
it repeated — this skill already contains the decoding rules. Just proceed.
If a tag or reference is genuinely ambiguous (e.g. `$proj` used but never
defined, or `@file#fn:name` and the function doesn't exist), ask one short
clarifying question rather than guessing.

## Full reference

`references/PCP-protocol.md` has the complete protocol spec, cross-model setup
(Codex/ChatGPT, Claude.ai web without file access), and worked before/after
token examples. Read it if the user asks about the protocol itself, wants to
extend it, or asks for the setup block to use elsewhere.
