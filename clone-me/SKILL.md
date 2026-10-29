---
name: clone-me
description: |
  Codes and reviews like the user. Learns their style from past commits and PR review comments, applies it with a baseline of language best practices, secure code and no useless comments, and turns every correction or review discussion into a rule so it is never needed twice. Use when reviewing code ("review my PR", "review PR 123", "review these changes"); after writing code whenever the user corrects it (a requested change, a review comment, or a silent edit or revert); when the user agrees, disagrees with or adds to your review findings; and when the user says "clone me", "code like me", "learn my style", "learn from my PRs", "remember how I like this" or "update my style skill", or has no style skill yet.
---

# Clone me: code and review the way the user does

Build the user's style skill from their past code and PR review comments, use it to write and review code the way they would, and keep it current. Each correction to your code and each reaction to your review findings is a signal. Capture it as a rule so later sessions start from it.

## Find the style skill

Write rules into the skill the user already uses for implementing or reviewing code, for example `~/.claude/skills/implement-task/SKILL.md`. If there are several, put each rule in the one it governs: how to write code goes in the implementation skill, and what to flag goes in the review skill. A rule can go in both.

If no style skill exists, create `~/.claude/skills/my-code-style/SKILL.md` with a `name`, a `description` saying it holds the user's personal code style and should be used when writing or reviewing code, a `## Baseline` section holding the rules below, one section per topic (naming, structure, tests, comments, errors, security, and so on), and a `## Review` section with three lists: `Flag`, `Don't flag` and `How to write findings`. Tell the user you created it. If the user already has a style skill without a baseline, add the `## Baseline` section to it and say so.

## Baseline: language, security and comments

These hold for all code written under the style skill. Learned rules add to them and may override a style choice, but never a security rule.

- **Write in the user's language.** Use the language, version, framework and tooling the repo already uses, detected from its files (`package.json`, `tsconfig.json`, `pyproject.toml`, `go.mod`, `Cargo.toml`, `pom.xml`, and so on). Don't switch languages or add a new framework or dependency unless asked.
- **Follow that language's current best practices:** its idiomatic style and naming, its official style guide, the repo's linter and formatter config, strict types where the language has them (no `any` in TypeScript, type hints in Python), and its standard library before new dependencies.
- **Write secure, robust code by default:**
  - Validate and type-check all external input at the boundary: request bodies, query params, headers, files, env vars and third-party responses.
  - Use parameterised queries and safe APIs. Never build SQL, NoSQL queries, shell commands, file paths or HTML from raw user input.
  - Never hard-code secrets, tokens or credentials. Read them from config or a secret store, and keep them out of logs and error messages.
  - Check authorisation on every protected action, not only authentication.
  - Handle errors explicitly. Don't swallow exceptions, don't leak stack traces or internals to clients, and fail closed.
  - Set timeouts on network calls, bound loops, batch sizes and uploads, and clean up resources (connections, files, locks).
  - Guard against races where state is shared: use transactions, atomic updates or locks instead of read-then-write.
  - Prefer well-maintained, pinned dependencies, and don't add one for something a few lines can do.
- **No useless comments.** Don't write comments that restate the code, narrate a change ("added X", "fixed bug"), or leave TODOs without an owner. Make the code explain itself through names and structure. Put the reasoning in the reply or the commit message instead. Keep a comment only when the code can't show why: a non-obvious constraint, a workaround for an external bug, or a security reason. If the user's own code has no comments, write none.

When a correction or a pattern in the user's history conflicts with a security rule, don't record it. Tell the user which rule it conflicts with and let them decide.

## Learn from past code first

Ask this once per style skill. Ask when the style skill has no `Learned from:` line, even if it already has some rules. Create or update the style skill and its Baseline first, then ask. If the user asked for a coding task in the same message, don't hold it up: do the task under the Baseline, then ask at the end of your reply. The question is:

> Which repos or PRs should I learn your style from? Give me repo names (`owner/repo`), local paths, or PR links, and your Git or GitHub username if it differs from the local git config.

Wait for the answer. When it comes, add a `Learned from:` line under the style skill's title. It lists the sources you read, or says `none (declined)` if they decline, so you never ask again. If they decline, skip to corrections. Otherwise read their history with read-only commands (`gh` for GitHub, `git` for local repos):

- **Their own code:** `git log --author=<user> --no-merges -p` on each repo, newest first. Look for what they do the same way every time: naming, file layout, error handling, how they write tests, what they leave out.
- **Their review comments on other people's PRs:** `gh api "repos/<owner>/<repo>/pulls/comments?per_page=100" --paginate`, then keep the comments where `.user.login` is theirs. Also read the review bodies on PRs they reviewed (`gh pr list --search "reviewed-by:<user>" --state all`, then `gh api repos/<owner>/<repo>/pulls/<n>/reviews`). This is the best source, because each comment is a correction they made.
- **Comments others left on their PRs that they accepted:** the change they pushed after the comment shows a rule they agree with.

Start with the most recent few hundred comments and commits. Count how often each pattern appears, and keep only those that repeat across several PRs or files. Write them into the style skill the same way as rules from corrections (see below), and keep project-specific conventions apart from personal ones. What they flag in others' PRs goes under `Review > Flag`. How they phrase comments (length, tone, whether they suggest code) goes under `Review > How to write findings`. Show the user a short summary of the rules you found, grouped by topic, and ask them to remove any that are wrong before you rely on them.

Never post, comment, or push anything while reading their history.

## Review code like the user

When asked to review a PR, a branch or local changes, review it the way the user would.

1. **Get the change.** For a PR: `gh pr view <n>` and `gh pr diff <n>`. For local work: `git diff` against the base branch. Read the surrounding code as well as the diff, so you understand what changed.
2. **Read the discussion so far.** For a PR, read the existing review threads and comments (`gh api repos/<owner>/<repo>/pulls/<n>/comments` and `gh pr view <n> --comments`). Don't repeat a point someone already raised. Report only new findings, and say when you agree with an open thread.
3. **Check it in this order:**
   - **Correctness and common sense:** does it do what the PR or ticket says? Look for logic errors, wrong conditions, off-by-one errors, missing awaits, unhandled null or empty cases, broken edge cases, wrong error handling, data loss and races. Check for behaviour changes for existing callers, and missing migrations or backfills. Check for leftover debug code, dead code, and code that duplicates an existing helper.
   - **Security and robustness:** every Baseline security rule.
   - **The user's learned rules:** everything in the style skill, especially `Review > Flag`. Skip anything listed under `Review > Don't flag`.
   - **Language best practices and the repo's conventions:** idioms, types, naming, structure, and tests for the new behaviour.
   - **Comments:** flag useless comments, as the Baseline defines them.
4. **Verify before reporting.** For each finding, re-read the code and confirm it's real: name the input or state that breaks it. Drop anything you can't back up, or mark it as a question.
5. **Report.** Rank findings from most severe to least. Give each one a `file:line`, a one-line problem and a one-line fix. Follow `Review > How to write findings`, and keep each finding to one or two lines unless the user's rules say otherwise. Put the evidence in your reply to the user, not in the comment text.
6. **Never post** review comments, approvals or change requests on the PR unless the user asks you to. When they do, post exactly what they approved.

## What counts as a correction

When coding:
- A change the user asks for after you have written code.
- A review comment on your work, from the user or relayed by them.
- The user editing or reverting your code themselves. Before you continue after they have touched files, run `git diff` (or compare with what you left) on the files you wrote. A silent edit counts as much as a spoken one, and the same silent edit made twice is a strong signal.

When reviewing, treat the user's reaction to each finding as a signal:
- **They reject a finding** ("that's fine", "not an issue here", "we don't care about that", "too picky"): add the general case to `Review > Don't flag`, unless it's a security or correctness issue. In that case, keep flagging it and say so once.
- **They point out something you missed:** add it to `Review > Flag`.
- **They agree, or ask you to fix or post it:** if a rule already covers it, it's confirmed, so keep it. If no rule covers it and it's about style, add it to `Review > Flag`.
- **They change how a finding is worded** (shorter, softer, no code suggestion, a different format), or rewrite your drafted comment before posting: add it to `Review > How to write findings`.
- **A back-and-forth that ends in a decision:** record the decision, not the discussion. For example, after "is this N+1?", "no, it's cached", record "Don't flag repeated lookups on cached repositories" only if it's general, and skip it if it's specific to this PR.

## Turn it into a rule

1. **Generalise.** State the rule so it applies beyond this task. "Use `findOne` with a projection here" becomes "Project only the fields you read in Mongo queries." If the correction is about this task's business logic and not about style, skip it.
2. **Check existing rules.** If a rule already covers it, sharpen that rule instead of adding a second one. If the correction contradicts a rule, replace the old rule.
3. **Write it in the matching section**, one line per rule, in the voice of the rest of the file. Add an example only when the rule is unclear without one.
4. **Keep the why out of the code.** The rule lives in the skill, not in a code comment.
5. **Check it against the baseline.** If the rule would weaken a security rule, don't add it (see Baseline). The same goes for a `Don't flag` rule that would hide a security or correctness bug.
6. **Mirror it.** A coding rule usually implies a review rule, and the reverse. For example, "Return early" also means "Flag deeply nested `if`/`else`". Put each in its section.

## Report it

Do this without asking first. The user can revert the edit. Then tell them in one line what changed, for example:

> Style skill: added "Return early instead of nesting `if` blocks" under Structure.

> Style skill: added "Don't flag missing JSDoc on internal helpers" under Review > Don't flag.

When you replaced a rule, name the rule you replaced.

## Don't

- Don't record one-off choices, ticket details, or anything already stated in the project's own docs (README, AGENTS.md, CLAUDE.md, lint config).
- Don't add a rule from a single ambiguous edit. Ask the user which way they meant it.
- Don't let the file grow without limit. When two rules overlap, merge them.
