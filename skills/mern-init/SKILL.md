---
name: mern-init
description: >-
  Initializes new MERN projects by cloning existing GitHub repos and
  scaffolding a Vite React client and/or Express Mongo server (TypeScript,
  feature-based, no monorepo). Always install npm @latest and never scaffold
  deprecated APIs (not moduleResolution node, not raw Mongoose Document as id).
  Use when creating a new MERN app, initializing client or server, or the user
  says init, scaffold, new project, or fullstack.
---

# MERN init

Not a monorepo. Client and server are separate git repos.

## Must not finish init until all of these are true

- Every package installed with `@latest` (no remembered version numbers). `npm outdated` is empty.
- `tsc` / `lint` / `build` succeed with **zero** errors.
- Server `tsconfig` uses `"moduleResolution": "nodenext"` — **never** `"node"` (deprecated).
- Vite client keeps `"moduleResolution": "bundler"`.
- Repository `.lean()` (or mapper) turns `_id` into `id: string`. Service never passes a Mongoose `Document` into `toUser` / `signToken`.
- No `ReactDOM.render`, no Mongoose callbacks, no Express `req.param`.
- `npm install` did not leave a `deprecated` warning unfixed.

Read before scaffolding:

- [01-project-structure.md](01-project-structure.md)
- [02-client-structure.md](02-client-structure.md) — client
- [03-server-structure.md](03-server-structure.md) — server
- [04-api-contract.md](04-api-contract.md)
- [05-dependency-updates.md](05-dependency-updates.md) — Cursor-only version bumps
- After new server routes: `postman-sync` skill (collection + JWT env + refresh)

## Rules

- Ask where to put the project. Use `<parent>/<project-name>/<repo-name>`.
- The GitHub repo **already exists**. Clone it. Never create a remote.
- Separate client and server repos.
- TypeScript both sides. Vite + React client. Express + Mongoose server.
- Feature-based folders. Server includes a repository layer.
- If the target folder exists, stop and ask. Do not overwrite.
- Never commit `.env`. Add `.gitignore` first.
- After the folder exists, move the agent to that project (parent if both repos, else the one repo).
- Do not scaffold from a random home folder after the project exists.
- **Latest packages at init.** Do not pin versions from memory, old apps, or this skill. New projects must install **current npm latest** (including current majors). `dep-updates` is only for apps that already exist.
- **No deprecated APIs.** Scaffold only current TypeScript, Mongoose, Express, and React APIs. If `npm install` prints `deprecated`, or `tsc`/ESLint flags a deprecated option, fix it before handing off. Never use `moduleResolution: "node"`. Never pass a raw Mongoose `Document` into a DTO that requires `id: string`.

## Workflow

1. Ask **parent folder** and **project name**.
2. Ask **client / server / both**. If they skip → **both**.
3. Ask existing GitHub repo URL(s) or `owner/name`.
4. Create the project folder. Clone into `<parent>/<project-name>/<repo-name>`.
5. Move the agent to that root.
6. Scaffold only **missing** files.
7. Write README and `.env.example`. Install **latest** deps (see below). Later bumps on an existing app: `dep-updates` skill, Cursor only.
8. Do not push unless they ask.

## Install latest (required at create)

In each new repo, do **not** write `"react": "19.1.1"` (or any remembered number) into `package.json`.

1. Prefer official current scaffolds where they exist (`npm create vite@latest`, then add the extra packages).
2. Install by name with `@latest`, no version pin:

```bash
npm install react@latest react-dom@latest react-router-dom@latest
npm install -D typescript@latest vite@latest @vitejs/plugin-react@latest eslint@latest prettier@latest
```

Same idea on the server (`express@latest`, `mongoose@latest`, …).

3. Run `npm outdated` in that repo. If anything is listed, install `@latest` again until outdated is empty (or only optional extras).
4. Then `npm run lint` and `npm run build`. If a brand-new major breaks the template, fix the template to the new API — do not downgrade to an old major on a new project.
5. `tsc` and lint must be clean: no deprecated `tsconfig` values, no Mongoose `Document` vs DTO errors.

## No deprecated APIs (required)

| Do not scaffold | Use instead |
|---|---|
| `moduleResolution: "node"` | Server: `"nodenext"`. Vite client: `"bundler"` (from `create vite`) |
| `module: "CommonJS"` + old `node` resolution | `"module": "NodeNext"` with `"moduleResolution": "nodenext"` (or keep the Vite client defaults) |
| Pass Mongoose `Document` into `toUser({ id, name, email })` | Repository `.lean()` (or `toObject()`), map `_id` → `id: String(doc._id)` in the repository or a mapper. Service never sees a raw Document |
| `user.id` on a Document (optional in TS) | `id` only on the DTO after mapping |
| Mongoose callbacks | Promises / `async` |
| `ReactDOM.render` | `createRoot` in `main.tsx` |
| `req.param` (Express) | `req.params` / `req.query` |

If install logs `deprecated <package>`, switch to the replacement or the current API of that package before you finish init.

## After scaffold

- Client: `npm run dev` → `5173`
- Server: `npm run dev` → `5000`
- Say the paths. `.env` stays local.
