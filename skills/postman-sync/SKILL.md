---
name: postman-sync
description: >-
  Keeps a Postman collection in sync with Express modules: project → module →
  API, env vars, login token scripts, refresh on 401, sockets. Creates and
  updates the collection and environment directly in Postman via MCP (not
  disk import). Use when adding a route, new module, Postman collection,
  JWT env, refresh token, or WebSocket request.
---

# Postman sync

Cursor only. When a server route is added or changed, update the collection **in Postman** (same account as MCP). Do not invent APIs that are not in `src/modules`.

Read [scripts.md](scripts.md) for pre/post script text. Read [scenarios.md](scenarios.md) for extra cases.

## Destination (required)

Write **directly into Postman Cloud** with the `user-postman` MCP. The desktop app shows the same collection after it syncs (same Google account). **No File → Import. No `~/Postman/Collections` folder.**

Do **not** write JSON on disk unless the user asks for a git copy, or MCP is unavailable after auth.

### MCP steps (every run)

1. Inspect `user-postman`. If `needsAuth`, call `mcp_auth` and wait until `ready`. If auth fails, stop and tell them to connect Postman in Cursor Settings → MCP.
2. `getWorkspaces`. If this chat already has a workspace id, reuse it. If several personal/team workspaces, ask which one. If they skip, use the first **personal** workspace you own.
3. `getCollections` with that workspace and `name` = project name (exact match).
4. **Missing collection** → `createCollection` in that workspace. Collection v2.1.0: folders, requests, collection pre-request + 401 test from [scripts.md](scripts.md).
5. **Existing collection** → `getCollection` (full model, so scripts are visible). Add only what is missing:
   - `createCollectionFolder` for a new module folder
   - `createCollectionRequest` in that folder for a new `METHOD` + path
   - Do **not** `putCollection` the whole collection (that recreates ids and wipes edits) unless the user asks for a full replace
   - Do **not** overwrite a request that already has a pre or test script
   - Do **not** `deleteCollection` / delete folders or requests unless the user asks
6. Environment named `<project> local` in the same workspace:
   - `getEnvironments` → if missing, `createEnvironment`
   - If present, add **missing keys only**. Never overwrite `accessToken` or `refreshToken` if they already have a value
   - Token keys: type `secret`, empty string on create
7. After sync, tell them: workspace name, collection name, environment name, what was added, what was left alone. Open or refresh the Postman app.

Optional disk copy (`<server>/postman/` or a folder they name) **only if they ask**. Never treat disk as the destination.

## Folder shape

```
<project>                         collection name
  health                          GET /health (no auth)
  auth
    POST register
    POST login
    POST refresh
    POST logout
  <module>
    METHOD  short-name
  realtime                        only if the app has sockets
    WS connect
    WS <event>
```

- Collection = project  
- Folder = `src/modules/<name>`  
- Request = one route (`METHOD` + path)  
- Path: `{{baseUrl}}/api/...`  
- Protected requests: header `Authorization: Bearer {{accessToken}}`  
- Login / register / refresh: **no** Bearer header  

## Env vars

| Variable | Purpose |
|---|---|
| `baseUrl` | API origin (no trailing slash) |
| `wsUrl` | Socket origin, if any |
| `accessToken` | Set by login / refresh scripts |
| `refreshToken` | Set by login / refresh scripts |
| `userId` | Set by login |
| `createdId` | Set by successful POST create |
| `adminToken` | Optional second role |

Never commit real tokens. New env values for tokens are empty.

## When to run this skill

1. New or changed file in `src/modules/*/ *.routes.ts` (or equivalent).  
2. User says add API, Postman, collection, JWT env, refresh, or socket.  
3. New module scaffolded.

## Sync rules

1. Read all route files.  
2. Ensure a folder per module and a request per `METHOD` + path.  
3. **Do not overwrite** a request that already has a pre or test script (login, refresh, createdId). Only add missing requests.  
4. New protected requests get: Bearer header + default test (assert `success` / `error` envelope and status).  
5. New POST creates get the **save `createdId`** test from [scripts.md](scripts.md).  
6. Auth requests get the scripts in [scripts.md](scripts.md) **once** (create if missing, never regenerate blindly).  
7. Collection **pre-request**: attach Bearer if `accessToken` exists and the request is not auth-public.  
8. Collection **test**: on `401`, refresh once, save tokens, do not loop on `/refresh`.  
9. If the app has a socket gateway, add `realtime` WS items; connect uses `{{wsUrl}}` and `{{accessToken}}`.  
10. Always push with MCP. Disk import is not the workflow.

## Auth behavior

- **Login / register** (if tokens in `data`): post-script writes `accessToken`, `refreshToken`, `userId`.  
- **Refresh**: public; post-script updates tokens; never call refresh from the refresh request’s 401 handler.  
- **401** on any other request: `POST {{baseUrl}}/api/auth/refresh` with `{{refreshToken}}`; on success update env; if refresh fails, unset tokens and stop.  
- **Logout**: unset `accessToken` and `refreshToken`.  
- Optional pre-request: if JWT `exp` is soon, refresh first.

## Response contract

Assert against the project envelope (see mern-init `04-api-contract.md` if present):

- 2xx → `success === true`, `data` present  
- 4xx/5xx → `success === false`, `error.code` + `error.message`  
- Status matches the route (200 / 201 / 204 / 400 / 401 / 403 / 404 / 409)

## Runner order

1. `health`  
2. `auth` (login before anything protected)  
3. Other module folders  
4. `realtime` (after tokens exist)  
5. `logout` last if they want a clean env  

## After sync

Tell the user: Postman workspace + collection + environment, what was created or added, what was left alone. They open the Postman app (refresh if it was already open). No import.
