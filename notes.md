
## 1. What is included in my Claude.md, and why?

**Included:**

- **Exact commands**, including how to run one test file and one test by name. Future sessions need these, and they aren't obvious from the code.
- **Non-obvious architecture**, such as the `require.main` guard in `server.js`, the shared in-memory state in `db/store.js` that makes tests affect each other, and the fact that IDs are numeric. These are the facts most likely to cause a wrong edit.
- **Conventions that differ from defaults**, like CommonJS instead of ES modules and the ESLint allowance for unused Express arguments.

**Omitted:**

- **Endpoint details and response shapes.** The code is small and readable, so Claude can look them up when needed. Copying them would only go stale.
- **General Node or Express advice.** Claude already knows it, and it would dilute the project-specific facts.
- **Secrets and environment values.** They don't belong in a file committed to git.
- **Course background and history.** It doesn't help with the code and changes over time.

**Why this approach:** CLAUDE.md loads into every session, so I'd keep it short and limit it to facts a newcomer would get wrong without being told.

## 2. What permission rules does `.claude/settings.json` have, and what could go wrong without the deny rules?

- **Allow (no prompt):** `npm test`, `npm run lint`, `git status`
- **Ask (confirm each time):** `git push`, `git commit`, `npm install`
- **Deny (blocked):** `Read(./.env)`, `git push --force`, `rm -rf`

Without the deny rules:

- **`Read(./.env)`:** Claude could read secrets from `.env` into the conversation or a transcript. This is the most serious gap.
- **`git push --force`:** Claude could overwrite remote history and remove teammates' commits. The rule only matches that exact prefix, so other force-push spellings may not be caught.
- **`rm -rf`:** Claude could delete a directory tree with no undo. Like the force-push rule, it matches only a specific spelling, so `rm -r -f` and similar forms may get through.
