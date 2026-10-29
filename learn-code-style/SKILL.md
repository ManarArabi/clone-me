---
name: learn-code-style
description: |
  Learns the user's personal code style from their past PRs and from every correction they make, and writes it into their style skill on top of a baseline of language best practices, secure and robust code, and no useless comments, so the same correction is never needed twice. Use after implementing or changing code whenever the user corrects the result — a requested change ("make this simpler", "move it to utils", "drop this test"), a review comment on your work, or the user silently editing or reverting what you wrote. Also use when the user says "learn my style", "learn from my PRs", "remember how I like this", or "update my style skill", or when they have no style skill yet.
---

# Learn code style from corrections

Build the user's style skill from their past code and PR review comments, then keep it current: each correction they make is a style signal. Capture it as a rule so later sessions start from it.

## Find the style skill

Write rules into the skill the user already uses for implementing or reviewing code, for example `~/.claude/skills/implement-task/SKILL.md`. If there are several, put each rule in the one it governs: how to write code goes in the implementation skill, and what to flag goes in the review skill. A rule can go in both.

If no style skill exists, create `~/.claude/skills/my-code-style/SKILL.md` with a `name`, a `description` saying it holds the user's personal code style and should be used when writing or reviewing code, a `## Baseline` section holding the rules below, and one section per topic (naming, structure, tests, comments, errors, security, and so on). Tell the user you created it. If the user already has a style skill without a baseline, add the `## Baseline` section to it and say so.

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

Start with the most recent few hundred comments and commits. Count how often each pattern appears, and keep only those that repeat across several PRs or files. Write them into the style skill the same way as rules from corrections (see below), and keep project-specific conventions apart from personal ones. Show the user a short summary of the rules you found, grouped by topic, and ask them to remove any that are wrong before you rely on them.

Never post, comment, or push anything while reading their history.

## What counts as a correction

- A change the user asks for after you have written code.
- A review comment on your work, from the user or relayed by them.
- The user editing or reverting your code themselves. Before you continue after they have touched files, run `git diff` (or compare with what you left) on the files you wrote. A silent edit counts as much as a spoken one, and the same silent edit made twice is a strong signal.

## Turn it into a rule

1. **Generalise.** State the rule so it applies beyond this task. "Use `findOne` with a projection here" becomes "Project only the fields you read in Mongo queries." If the correction is about this task's business logic and not about style, skip it.
2. **Check existing rules.** If a rule already covers it, sharpen that rule instead of adding a second one. If the correction contradicts a rule, replace the old rule.
3. **Write it in the matching section**, one line per rule, in the voice of the rest of the file. Add an example only when the rule is unclear without one.
4. **Keep the why out of the code.** The rule lives in the skill, not in a code comment.
5. **Check it against the baseline.** If the rule would weaken a security rule, don't add it (see Baseline).

## Report it

Do this without asking first. The user can revert the edit. Then tell them in one line what changed, for example:

> Style skill: added "Return early instead of nesting `if` blocks" under Structure.

When you replaced a rule, name the rule you replaced.

## Don't

- Don't record one-off choices, ticket details, or anything already stated in the project's own docs (README, AGENTS.md, CLAUDE.md, lint config).
- Don't add a rule from a single ambiguous edit. Ask the user which way they meant it.
- Don't let the file grow without limit. When two rules overlap, merge them.
