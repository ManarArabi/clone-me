# 🧬 clone-me

**Teach Claude to code and review like you. Then never repeat yourself again.**

You've fixed the same thing in Claude's code five times. Nested `if`s. Pointless comments. A query built with string glue. And its reviews flag nitpicks you don't care about while missing the things you always catch. `clone-me` makes the sixth time never happen.

It studies how you code and review, listens every time you push back, and turns each correction into a rule. Session after session, Claude writes and reviews more like you and less like a stranger.

## ⚡ What it does

**🔍 Studies your past work.** On day one it asks which repos or PRs to learn from. Then it reads, without touching anything:
- your commits;
- the review comments you left on teammates' PRs (the best source, because every comment is a correction you made);
- the feedback you accepted on your own PRs.

Only patterns that show up again and again make the cut. You see the list first and remove anything that isn't you.

**👂 Learns from every correction.** "Make this simpler." "Move it to utils." A review comment. Even you quietly fixing its code without saying a word, because it checks `git diff`. Each one becomes a one-line rule.

**🕵️ Reviews like you.** Ask it to review a PR or your local changes and it checks, in order:
1. **Logic and common sense:** bugs, edge cases, races, broken callers, dead code.
2. **Security.**
3. **Your rules,** including the things you always flag in reviews.
4. **Language best practices.**
5. **Useless comments.**

It reads the existing PR threads first so it never repeats a teammate. It confirms every finding is real before reporting it. And it never posts anything unless you say so.

**💬 Learns from review arguments too.**
- You say "nah, that's fine": the clone stops flagging it.
- You point out something it missed: it flags it next time.
- You reword its comment: it writes the next one your way.

Security and correctness bugs can't be argued away.

**🧹 Keeps the rulebook tidy.** It sharpens existing rules instead of piling up duplicates and replaces rules you've outgrown. It never saves one-off business logic.

**📣 Tells you what it learned.** One line, no fuss:

> Style skill: added "Return early instead of nesting `if` blocks" under Structure.

## 🛡️ The baseline: your clone has standards

Every clone starts from three rules. Your habits build on top of them.

- **Speaks your language.** It uses the language, framework and tooling already in your repo, and follows its best practices and your linter config. No surprise rewrites or dependencies.
- **Secure by default:**
  - checks all input at the boundary;
  - uses parameterised queries;
  - keeps secrets out of the code;
  - checks permissions on every protected action;
  - handles errors and fails safe;
  - sets timeouts and limits;
  - makes race-safe writes.
- **Zero useless comments.** No comments that repeat the code, no "added X", no orphan TODOs. A comment survives only when the code can't explain itself.

> 🚫 Your clone copies your style, not your shortcuts. Teach it to glue SQL strings together and it will refuse, tell you which security rule that breaks, and offer a safe alternative.

## 🚀 Get started

**Claude Code:** copy the folder into your skills directory.

```bash
cp -R clone-me ~/.claude/skills/
```

**Claude app:** upload `clone-me.skill` under Settings → Capabilities → Skills.

Then say:

> Clone me. Learn my style from my PRs.

Or:

> Review PR 123 like me.

From then on it runs on its own whenever you correct Claude's code or push back on its review findings. You can also say "code like me", "remember how I like this" or "update my style skill".

Your rules live in your existing implementation or review skill. If you don't have one, it creates `~/.claude/skills/my-code-style/SKILL.md`.

## 🧰 Requirements

- `git`, for local repos.
- [GitHub CLI](https://cli.github.com/) (`gh`), signed in, to read PRs and review comments.

## ✅ Tested

It was run in throwaway repos on four scenarios:
- the first run;
- a silent edit by the user;
- writing a new endpoint;
- an unsafe "remember this" request.

All four passed.

Review mode was tested on two more:
- a full review of a PR seeded with ten bugs;
- the user pushing back on a review.

Both passed.

## 📁 What's inside

| Path | What it is |
| --- | --- |
| `clone-me/SKILL.md` | The skill |
| `clone-me.skill` | Bundled file for the Claude app |

After editing `SKILL.md`, rebuild the bundle:

```bash
rm -f clone-me.skill && zip -qr clone-me.skill clone-me
```
