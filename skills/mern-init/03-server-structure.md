# Server structure

Express + Mongoose + TypeScript. Feature-based. Own git repo.

See also: [01-project-structure.md](01-project-structure.md) · [04-api-contract.md](04-api-contract.md)

---

## Folder tree

```
<server-repo>/
├── .env.example
├── .gitignore
├── .prettierrc
├── eslint.config.js
├── package.json
├── tsconfig.json
├── README.md
└── src/
    ├── server.ts
    ├── app.ts
    ├── config/
    ├── constants/
    ├── utils/
    ├── middleware/
    ├── types/
    ├── errors/
    └── modules/
        └── auth/
            ├── auth.model.ts
            ├── auth.repository.ts
            ├── auth.validation.ts
            ├── auth.service.ts
            ├── auth.controller.ts
            └── auth.routes.ts
```

**Shared:** `config`, `constants`, `utils`, `middleware`, `types`, `errors`  
**Per feature:** model, repository, validation, service, controller, routes

First feature: `modules/auth/`.

Listen on `5000`. Mount routes under `/api`. CORS for the Vite origin.

**Skip until needed:** `jobs/`, `seeds/`, uploads, Swagger, tests, Docker, CI.

---

## Call chain

```
routes → controller → service → repository → model
```

| File | Job |
|---|---|
| Controller | `req` / `res` only. Calls `ok` / `created` / `noContent`. |
| Service | Business rules. Throws `AppError`. No `res.json`. |
| Repository | Mongo queries only (`findOne`, `create`, `update`). |
| Model | Schema / collection. |

Response helpers and status: [04-api-contract.md](04-api-contract.md)

---

## tsconfig (not deprecated)

```json
{
  "compilerOptions": {
    "target": "ES2022",
    "module": "NodeNext",
    "moduleResolution": "nodenext",
    "outDir": "dist",
    "rootDir": "src",
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true,
    "resolveJsonModule": true
  },
  "include": ["src"]
}
```

Never `"moduleResolution": "node"` — TypeScript marks it deprecated.

---

## Repository DTOs (Mongoose)

Do not pass a Mongoose `Document` into `toUser` / `signToken` types that require `{ id: string }`. `Document.id` is optional in current types (TS2345).

Repository returns a **plain object**:

```ts
const doc = await User.findOne({ email }).lean();
if (!doc) return null;
return { id: String(doc._id), name: doc.name, email: doc.email, passwordHash: doc.passwordHash };
```

Service uses `user.id` only on that DTO. `signToken(user.id)` is fine after the map.

---

## Libraries

**Always:** `express` `mongoose` `cors` `dotenv` `zod` `jsonwebtoken` `bcryptjs` `typescript` `tsx` `eslint` `prettier`

Install each with `@latest` on a **new** project. Never copy version numbers from an old repo or from memory.

**Add when needed:** `helmet`, `express-rate-limit`, `multer`, `nodemailer`, `pino` or `morgan`

Scripts: `dev` (`tsx watch src/server.ts`) `build` `start` `lint` `format`
