# Project setup

Separate client and server git repos. Not a monorepo.

**Must (or init is not done):** `@latest` packages · no `moduleResolution: "node"` · no raw Mongoose `Document` as `{ id }` · `tsc`/`lint`/`build` clean.

- Client folders: [02-client-structure.md](02-client-structure.md)
- Server folders: [03-server-structure.md](03-server-structure.md)
- Response and status: [04-api-contract.md](04-api-contract.md)
- Dep updates (Cursor): [05-dependency-updates.md](05-dependency-updates.md)
- Postman collection: [06-postman-collection.md](06-postman-collection.md)

---

## Path and clone

```
<parent>/<project-name>/<repo-name>
```

Example (both):

```
<parent>/<project-name>/
├── <project>-client/     own git repo
└── <project>-server/     own git repo
```

- Ask parent folder and project name.
- New project: install every package with `@latest` (including current majors). Do not pin old versions. `dep-updates` is for existing apps only.
- No deprecated APIs: not `moduleResolution: "node"`, not raw Mongoose `Document` as `{ id: string }`. See [03-server-structure.md](03-server-structure.md).
- GitHub repo **already exists**. Clone only. Never create a remote.
- Ask: **client / server / both**. Default if skipped: **both**.
- Both → two repo URLs, two clones under the same project folder.
- If the target folder exists → stop. Do not overwrite.
- Prefer `gh repo clone` when signed in; otherwise `git clone`.
- Never commit `.env`. Add `.gitignore` first.

---

## Stack

| Side | Stack |
|---|---|
| Client | Vite + React + TypeScript |
| Server | Express + Mongoose + TypeScript |

---

## How the two repos talk

No shared package.

- Server: CORS + `/api` + response envelope — [04-api-contract.md](04-api-contract.md)
- Client: `VITE_API_URL` or Vite proxy — [02-client-structure.md](02-client-structure.md)
- `.env.example` in each repo
