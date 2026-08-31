# Client structure

Vite + React + TypeScript. Own git repo.

See also: [01-project-structure.md](01-project-structure.md) · [04-api-contract.md](04-api-contract.md)

---

## Folder tree

```
<client-repo>/
├── .env.example
├── .gitignore
├── .prettierrc
├── eslint.config.js
├── index.html
├── package.json
├── vite.config.ts
├── tsconfig.json
├── README.md
└── src/
    ├── main.tsx
    ├── app/
    │   ├── App.tsx
    │   ├── providers.tsx
    │   └── router.tsx
    ├── constants/
    ├── types/
    ├── shared/
    │   ├── ui/
    │   ├── lib/
    │   │   ├── api.ts
    │   │   └── storage.ts
    │   └── hooks/
    │       └── useAuth.ts
    └── features/
        └── auth/
            ├── LoginPage.tsx
            ├── RegisterPage.tsx
            ├── auth.api.ts
            └── auth.types.ts
```

**Shared:** `app`, `constants`, `types`, `shared/ui`, `shared/lib`, `shared/hooks`  
**Per feature:** pages, `*.api.ts`, `*.types.ts`

First feature: `features/auth/` (Login + Register).

`tsconfig` comes from `npm create vite@latest` — keep `"moduleResolution": "bundler"`. Never set `"node"`. Use `createRoot` in `main.tsx`, never `ReactDOM.render`.

---

## Env and ports

- `.env.example`: `VITE_API_URL=http://localhost:5000/api`
- Vite proxy: `/api` → `http://localhost:5000`
- Dev: `npm run dev` → `5173`

How `shared/lib/api.ts` reads responses: [04-api-contract.md](04-api-contract.md)

---

## Libraries

**Always:** `react` `react-dom` `react-router-dom` `typescript` `vite` `@vitejs/plugin-react` `eslint` `prettier` `babel-plugin-react-compiler`

Install each with `@latest` on a **new** project. Never copy version numbers from an old repo or from memory.

Use `fetch` in `shared/lib/api.ts`. No Axios, Redux, or UI kit unless asked.

**Add when needed:** `@tanstack/react-query`, `react-hook-form` + `zod`

Scripts: `dev` `build` `lint` `format`
