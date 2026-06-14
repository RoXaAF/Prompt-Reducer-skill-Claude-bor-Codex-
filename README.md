# Prompt-Reducer-skill-Claude-bor-Codex-
# PCP — Prompt Compression Protocol

A tiny shorthand syntax for talking to Claude, Codex, and ChatGPT that cuts
your **input tokens by 40–75%** on iterative coding/writing sessions, without
losing output quality.

It doesn't make the model smarter it removes the three biggest sources of
wasted tokens in a normal chat:

1. Re-pasting code the model can already read from disk
2. Re-explaining project context it already has from earlier in the session
3. Writing full sentences for things that are really just flags
   ("explain briefly and just show me the diff" → `EXPL:1 FMT:diff`)

You write terse, tagged messages. The model expands them internally and
responds exactly as if you'd typed the long version.

```diff
- "Hey, in my TigerGame.java file, in the movePiece method, there's a bug
-  where the Tiger can jump over a Dog even when the three squares aren't
-  in a straight line — the collinearity check isn't working. Can you look
-  at the file, fix it, briefly explain what was wrong, and just show me
-  the fixed method?"                                          (~70 tokens)
+ FIX @TigerGame.java#fn:movePiece — collinearity check broken
+ for jumps. EXPL:1 FMT:snippet                                 (~14 tokens)
```

---

## Quickstart

### Claude (Claude Code / Claude.ai)

1. Download [`prompt-compression-skill.skill`](./prompt-compression-skill.skill)
   (or the `prompt-compression-skill/` folder).
2. **Claude Code**: unzip into `.claude/skills/` in your project (or
   `~/.claude/skills/` for all projects).
   **Claude.ai**: upload it as a custom skill in settings.
3. Done — Claude now auto-decodes PCP shorthand in any conversation. No setup
   message needed.

### Codex / Codex CLI

1. Copy the **setup block** below into your repo's `AGENTS.md`. Codex reads
   this automatically as project instructions.

### ChatGPT (web/app)

1. Copy the setup block into **Settings → Custom Instructions**. It applies
   to all future chats.

### Any other model / one-off chat

1. Paste the setup block as your first message in the conversation.

```
[PCP-1] Compression mode on. Rules for this session:
- @file or @file#L10-40 or @file#fn:name = read/operate on that file/range/function
  directly (use your file tools); don't ask me to paste it.
- $NAME=value defines a session variable; $NAME reuses it later.
- Message prefix sets intent: FIX / ADD / RM / REFAC / OPT / TEST / DOC / ASK / EXPL
- EXPL:0 = code/diff only, no prose. EXPL:1 = 1-3 sentence explanation. EXPL:2 = full explanation.
- FMT:diff = unified diff only. FMT:snippet = only the changed function/block. FMT:full = whole file.
- ~ = same file/scope as my previous message. Δ = "only what changed since last time".
- Expand all shorthand silently. Don't restate or echo these rules back to me.
```

That block is ~120 tokens, paid once per project/session.

---

## Syntax reference

### Intent tags (start of message)

| Tag | Meaning |
|---|---|
| `FIX` | fix a bug |
| `ADD` | add a feature/function |
| `RM` | remove code |
| `REFAC` | refactor, no behavior change |
| `OPT` | optimize (perf/memory/etc.) |
| `TEST` | write/extend tests |
| `DOC` | docs / report text / comments |
| `ASK` | question — don't change code |
| `EXPL` | explain something — no code change |

### Scope (instead of pasting code)

| Syntax | Means |
|---|---|
| `@path/to/file` | whole file |
| `@path/to/file#L10-40` | line range |
| `@path/to/file#fn:name` | a specific function/method/class |
| `~` | same file/scope as your previous message |
| `Δ` | "only what changed since last time" |
| `$NAME=value` / `$NAME` | define / reuse a session variable |

### Output control

| Syntax | Means |
|---|---|
| `EXPL:0` | code/diff/snippet only, no prose |
| `EXPL:1` | 1–3 sentence explanation |
| `EXPL:2` | full explanation |
| `FMT:diff` | unified diff only |
| `FMT:snippet` | only the changed function/block |
| `FMT:full` | whole file |
| `LANG:xx` | language for prose (e.g. `LANG:zh`) |

If you omit `EXPL:`/`FMT:`, models following this protocol default to
`EXPL:1 FMT:snippet` for edits and `FMT:full` for new files.

---

## More examples

**Defining context once:**
```
$proj="Java Swing board game, 23-node board, Tiger vs Dog pieces"
```
Every later message can just reference `$proj` instead of re-explaining the
project.

**Follow-up edit (after a fix above):**
```
ADD ~ same collinearity check for Dog.move FMT:diff EXPL:0
```
~6 tokens vs. ~40 for the equivalent full sentence.

**Working without file access (plain chat, no tools):**
```
$F1 = <paste file once>
...later...
FIX $F1#fn:movePiece — off-by-one in loop bound. FMT:snippet
```
The model already has `$F1` in context you're just pointing at it.

---

## Why this works

- **Code dominates token counts.** `@file` references (when the model has
  file tools) eliminate re-pasting entirely usually the single biggest win.
- **Output tokens become input tokens next turn.** `FMT:diff`/`FMT:snippet`
  shrink responses, and those savings compound over a long session.
- **Context repetition is pure waste.** `$VAR`s let you state project context
  once instead of every message.
- **Politeness markers cost tokens for zero signal.** The model doesn't need
  "could you please" to behave well.

---

## Caveats

- This compresses *your* tokens, not the model's reasoning for genuinely
  novel/ambiguous tasks, don't use `EXPL:0`; give the model room to think.
- 75% savings is realistic for **iterative coding sessions** with lots of
  repeated file/context. One-shot questions with little redundancy will see
  smaller gains (10–30%).
- This is a convention, not a guarantee. If a model misreads a tag, just
  spell it out that once — clarity beats compression.

---

## Repo contents

- [`SKILL/`](./SKILL) — Claude skill
  (also packaged as `prompt-compression-skill.skill`) that auto-decodes PCP.
- [`PCP-protocol.md`](./PCP-protocol.md) — full protocol spec.

## Contributing

PRs welcome for: additional intent tags, language-specific abbreviation
tables, integrations for other tools/agents. Keep the core syntax small
the point is that it's easy to memorize.

## License

MIT — see [`LICENSE`](./LICENSE).
