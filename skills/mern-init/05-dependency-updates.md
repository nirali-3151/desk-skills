# Dependency updates

Cursor only. Use the `dep-updates` skill.

**New project (`mern-init`):** install `@latest` for every package, including current majors. Run `npm outdated` before handing off; a day-old app should not already be 30 packages behind.

**Existing project:** list each package → email the list → click **Yes** in Cursor → agent updates. Yes = patch + minor only. Majors need a second Yes.

---

## Steps

1. `npm outdated` in each repo (client and server separately).
2. Ask which email to use. Send: `package: current → latest (patch|minor|major)`.
3. AskQuestion: **Yes — apply patch + minor** / **No** / **Majors too** (second confirm).
4. On Yes: `npx npm-check-updates --target minor -u` then `npm install`, `lint`, `build`.
5. No commit/push unless they ask.

Come back later: “apply dep updates”.

Report: `<project>/.cursor/dep-update-report.md`

---

## Deprecated or changed functions

Patches/minors rarely remove APIs. **Majors** rename or delete them.

| Signal | Meaning |
|---|---|
| `npm install` says `deprecated` | Still works — plan a swap |
| `lint` / `build` fails | Renamed or removed — fix before you finish |
| Runtime warning | Works now, gone later |

On a **major** (only after the second Yes):

1. Read the changelog Breaking / Deprecated section.
2. `npm install` then `lint` and `build` in that repo only.
3. Search for the old function name and replace it in the same change.
4. If the migration is large, skip the bump.

One major at a time. If `build` fails, stop. If `mern-init` templates still use the old API, update that skill too.
