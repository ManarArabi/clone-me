# learn-code-style

A Claude skill that learns your personal code style and keeps it current.

It builds a style skill from your past commits and PR review comments. After that, it turns every correction you make to Claude's code into a rule. Over time Claude writes code the way you would and stops repeating the mistakes you already fixed.

## Baseline

Every style skill it builds starts from three rules. Learned rules add to them.

- **Your language, done right.** Claude uses the language, version, framework and tooling your repo already uses, and follows that language's current best practices and your linter config.
- **Secure and robust by default.** This covers:
  - input validation at the boundary;
  - parameterised queries;
  - no hard-coded secrets;
  - authorisation checks;
  - explicit error handling that fails closed;
  - timeouts and limits;
  - race-safe writes;
  - few, pinned dependencies.

  A learned habit can never override a security rule. If one conflicts, Claude asks you instead.
- **No useless comments.** No comments that restate the code, narrate a change, or leave ownerless TODOs. A comment stays only when the code can't show the reason.

## How it works

1. **Learns from your history.** On first use it asks which repos or PRs to learn from, and for your GitHub username. It reads them without changing anything:
   - your own commits;
   - the review comments you left on other people's PRs;
   - comments others left on your PRs that you then fixed.

   It keeps only patterns that repeat across several PRs or files. Then it shows you the rules it found so you can remove any that are wrong.
2. **Learns from your corrections.** After Claude writes code, any change you ask for, review comment you give, or edit you make yourself becomes a one-line rule. Claude checks `git diff` to catch silent edits too.
3. **Keeps the rules clean.**
   - A new rule sharpens an existing one instead of duplicating it.
   - A rule that contradicts an old one replaces it.
   - Business logic and one-off choices are never recorded.
4. **Tells you what changed.** One line per update, for example:

   > Style skill: added "Return early instead of nesting `if` blocks" under Structure.

Rules go into the implementation or review skill you already use. If you have none, the skill creates `~/.claude/skills/my-code-style/SKILL.md`.

## Install

**Claude Code:** copy the folder into your skills directory.

```bash
cp -R learn-code-style ~/.claude/skills/
```

**Claude app:** upload `learn-code-style.skill` under Settings → Capabilities → Skills.

## Usage

Start with:

> Learn my code style from my PRs.

After that it runs on its own whenever you correct code Claude wrote. You can also trigger it with "remember how I like this" or "update my style skill".

## Requirements

- `git` for local repos.
- [GitHub CLI](https://cli.github.com/) (`gh`), signed in, to read PRs and review comments.

## Files

| Path | What it is |
| --- | --- |
| `learn-code-style/SKILL.md` | The skill itself |
| `learn-code-style.skill` | Bundled file for uploading to the Claude app |

## Updating the bundle

After editing `SKILL.md`, rebuild the bundle:

```bash
rm learn-code-style.skill && zip -qr learn-code-style.skill learn-code-style
```
