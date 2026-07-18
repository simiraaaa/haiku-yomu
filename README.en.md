# haiku-yomu

*俳句詠む 弦の調べの 五月雨に*

[日本語版 (Japanese)](./README.md)

A Claude Code / Codex CLI skill (plugin) that has Haiku read your document and verifies it can be understood **without deep thinking**. Either way, the `claude` CLI is required — it is what runs the Haiku reader.

By deliberately giving the document to a reader with limited reasoning, it avoids the problem that "a reader capable of deep inference fills the gaps", and measures how clear the document is on its own. (*詠む (= yomu)* is Japanese for "to read / to compose a poem".)

## Install

In a Claude Code session, run these **one at a time, in order** (pasting them together can leave the later lines unexecuted):

1. Register the marketplace

   ```
   /plugin marketplace add simiraaaa/haiku-yomu
   ```

2. Install the plugin

   ```
   /plugin install haiku@haiku-yomu
   ```

3. Reload if you want to use it in the same session (a fresh session does not need this)

   ```
   /reload-plugins
   ```

This installs both `haiku:retell` (for English documents) and `haiku:yomu` (for Japanese documents).

### Using with Codex CLI

The same repository also works as a Codex CLI plugin. Run these in a terminal, one at a time:

```
codex plugin marketplace add simiraaaa/haiku-yomu
```

```
codex plugin add haiku@haiku-yomu
```

The skill names are the same as in Claude Code (`haiku:yomu` / `haiku:retell`). The reading engine is still Haiku launched via the `claude` CLI, so that requirement stays the same.

## Usage

In a Claude Code session:

- Type `/haiku:retell` (you can also specify which file)
- Or just ask: "Is this document clear?", "Have Haiku review this", "Run a readability check with a lightweight model"

*Note: the session running this skill needs a model with strong reasoning (Opus etc.) to work well.*

1. The main session reads the file and pins down the intent
2. Compare with what Haiku understood from a blind read
3. Report the gaps and fix candidates
4. You decide whether to fix (the skill stops at the report)
5. If you request fixes, steps 2–4 run again (the intent is reused; Haiku rereads fresh every time)

You can also use the reading primitive directly via the bundled `haiku-read.sh`:

```bash
./skills/retell/haiku-read.sh <document file> "<premise knowledge text (optional)>"
```

(Example when running from the repository root. When installed as a plugin, the skill resolves the location.)

## How it works

1. **The main session articulates the "answer key" first** — key points (3–5) and post-reading actions
2. **An isolated Haiku receives only the document body and retells the key points in its own words** — no tools, no customization, low effort
3. **Inspect the gaps between the retelling and the answer key** — misunderstandings and omissions surface directly as the document's unclarity

### Notes

If you ask Haiku directly "point out the unclear parts", it tends to reply "it's clear" and the check fails — hence this design.

Misreadings are **treated as the document's problem** by default.

## Where it helps

- **Explanatory text**: READMEs, design docs, runbooks, skill definitions, PR descriptions
- Anything meant to make a reader understand something — not just manuals (tech blogs, shared notes)
- Multiple documents can be inspected in parallel (see SKILL.md)
- **Pick the skill that matches the document's language**: English documents → `haiku:retell` (report in English); Japanese documents → `haiku:yomu` (report in Japanese). It still works across languages, but the report may come back coarser

## What it cannot detect

- **Correctness or completeness of the content** — Haiku knows nothing outside the document
- **Consistency with other documents or the implementation** — each inspection sees only its own document body
- **Being interesting, style, persuasiveness** — the only thing measured is "does the intent come across"

A pass shows only that "**given the stated premise, Haiku could retell the key points and post-reading actions**".

## Layout

| File | Role |
|---|---|
| `skills/retell/SKILL.md` | The skill itself (procedure, judgment criteria, Red Flags) |
| `skills/retell/haiku-read.sh` | Reading primitive (prompt assembly + launching an isolated Haiku) |
| `.claude-plugin/plugin.json` | Plugin definition (plugin name `haiku`) |
| `skills/yomu/` | Japanese version of the skill (`haiku:yomu`) |
| `.claude-plugin/marketplace.json` | Marketplace definition for distribution (marketplace name `haiku-yomu`) |
| `.codex-plugin/plugin.json` / `.agents/plugins/marketplace.json` | Plugin & marketplace definitions for Codex CLI (skills are shared) |
| `LICENSE` | MIT |

## License

MIT
