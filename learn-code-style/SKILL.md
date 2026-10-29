---
name: learn-code-style
description: |
  Turns every correction the user makes to code you wrote into a durable rule in their personal style skill, so the same correction is never needed twice. Use after implementing or changing code whenever the user corrects the result — a requested change ("make this simpler", "move it to utils", "drop this test"), a review comment on your work, or the user silently editing or reverting what you wrote. Also use when the user says "learn my style", "learn from my PRs", "remember how I like this", or "update my style skill", or when they have no style skill yet.
---

# Learn code style from corrections

Build the user's style skill from their past code and PR review comments, then keep it current: each correction they make is a style signal. Capture it as a rule so later sessions start from it.

## Find the style skill

Write rules into the skill the user already uses for implementing or reviewing code, for example `~/.claude/skills/implement-task/SKILL.md`. If there are several, put each rule in the one it governs: how to write code goes in the implementation skill, and what to flag goes in the review skill. A rule can go in both.

If no style skill exists, create `~/.claude/skills/my-code-style/SKILL.md` with a `name`, a `description` saying it holds the user's personal code style and should be used when writing or reviewing code, and one section per topic (naming, structure, tests, comments, errors, and so on). Tell the user you created it.

## Learn from past code first

The first time the skill runs for a user, or when their style skill is empty or new, ask:

> Which repos or PRs should I learn your style from? Give me repo names (`owner/repo`), local paths, or PR links, and your Git or GitHub username if it differs from the local git config.

Wait for the answer. If they decline, skip to corrections. Otherwise read their history with read-only commands (`gh` for GitHub, `git` for local repos):

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

## Report it

Do this without asking first. The user can revert the edit. Then tell them in one line what changed, for example:

> Style skill: added "Return early instead of nesting `if` blocks" under Structure.

When you replaced a rule, name the rule you replaced.

## Don't

- Don't record one-off choices, ticket details, or anything already stated in the project's own docs (README, AGENTS.md, CLAUDE.md, lint config).
- Don't add a rule from a single ambiguous edit. Ask the user which way they meant it.
- Don't let the file grow without limit. When two rules overlap, merge them.
