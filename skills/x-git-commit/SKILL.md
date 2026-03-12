name: x-git-commit
description: Git commit message format and style (feat/fix/refactor, short + detailed)

# Git Commit Rules

When writing or suggesting commit messages, follow these rules.

## Structure

- **Short line**: First line must be under 50 characters, then a blank line.
- **Body**: Optional detailed description after the blank line.

## Prefix

Use exactly one of these prefixes on the first line:

- `feat: ` — new feature
- `fix: ` — bug fix
- `refactor: ` — refactor (no behavior change)

## Style

- Use **markdown** in the body (lists, bold, etc.).
- Write **concisely**, **informal** tone.
- List **significant changes** only.
- Put each sentence on its **own line** in the body.
- Mention **implications and possible usages** of the changes.

## Avoid

- Do **not** mention specific file names or paths from the code.
- Do **not** use phrases like "this commit", "this change", "this PR".
- Do **not** wrap the message in a code block (no ``` at the end).

## Example

```
feat: add token refresh with configurable expiry

- TokenManager now refreshes before expiry using Configuration.ttl
- Enables long-running clients without manual re-auth
- Backward compatible when ttl is not set
```

