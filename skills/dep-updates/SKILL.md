---
name: dep-updates
description: >-
  Lists npm patch/minor/major updates in Cursor, emails the list, then asks
  Yes/No before applying. Use when checking outdated packages, applying
  patches, or the user says check deps, update versions, or apply the list.
---

# Dep updates

Cursor only. List, email, then apply after a Yes click in this session. Do not apply updates any other way.

Do **not** apply any bump until the user clicks an option (AskQuestion).

## Modes

**Check** (default) — list → email → AskQuestion.  
**Apply** — user came back (“yes”, “apply the updates”) → re-scan or re-read the last report → AskQuestion → apply only if Yes.

## 1. Find repos

Ask which folder to scan if it is not obvious. Then scan each child (or the current app) that has a `package.json`. Skip `node_modules`.

Client and server stay separate. Never one lockfile for both.

## 2. List each update

In each repo:

```bash
npm outdated --json || true
```

If that is empty, also try `npx npm-check-updates --format json`.

| Kind | Rule | Auto on Yes? |
|---|---|---|
| **patch** | same major.minor, newer patch | Yes |
| **minor** | same major, newer minor | Yes |
| **major** | new major | No — second question only |

Write `.cursor/dep-update-report.md` in the project folder (create `.cursor` if needed): date, repo paths, table of package / current / latest / kind.

If everything is current: say so, skip email, stop.

## 3. Email the list

Ask which address to send to if it is not already known in this chat.

- **subject:** `Dep updates: <project> — <N> packages (<date>)`
- **body:** list grouped by repo, then patch / minor / major. Each line: `package: current → latest (patch|minor|major)`
- End with: `Open Cursor and say: apply dep updates`

Send with the Gmail tool in Cursor. If send fails, still show the list in chat and continue. Do not pretend the email was sent.

## 4. Click Yes (AskQuestion)

Always use AskQuestion. Never bump before they choose.

- **Yes — apply patch + minor**
- **No — leave versions**
- **Majors too** — only if majors exist; then a second question listing each major by name.

If they type “yes” in chat, still run AskQuestion once.

## 5. Apply (only after Yes)

Each repo, one at a time:

```bash
npx npm-check-updates --target minor -u
npm install
npm run lint
npm run build
```

Majors only after the second Yes:

```bash
npx npm-check-updates --target latest -u
npm install
npm run lint
npm run build
```

If `lint` or `build` fails, stop that repo and report it. Do not commit or push unless they ask. Do not update client and server in one command.

When done: what changed, what failed, whether majors were skipped.

## Rules

- Cursor only.
- Yes = patch + minor only.
- Majors need a second click.
- Email is notify-only. Applying still needs Yes in Cursor.
