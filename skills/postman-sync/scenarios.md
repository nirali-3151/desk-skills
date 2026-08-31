# Postman scenarios

Use these when they fit the routes. Do not add requests for APIs that do not exist.

| Scenario | Collection / script |
|---|---|
| New module | New folder + CRUD; create saves `{{createdId}}` |
| List + pagination | Query `page`, `limit`; optional save `meta.total` |
| 400 validation | Extra example or request with bad body; assert `error.code` |
| 409 duplicate | Same unique field twice |
| 404 | GET fake id |
| 403 / two roles | `{{accessToken}}` vs `{{adminToken}}`; two login requests |
| Logout | Unset tokens |
| Refresh fail | Unset tokens; do not retry refresh |
| Upload | `form-data`; save `{{fileUrl}}` |
| Health | `GET {{baseUrl}}/health` first in runner |
| Idempotent pay | Header `Idempotency-Key` |
| Webhook | Own folder; signed header vars (local only) |
| Multi-env | Same collection; different `baseUrl` / `wsUrl` per env |
| Rate limit 429 | Optional wait + retry once |
| Socket | Folder `realtime`; WS connect after login with `{{accessToken}}` |
| Socket events | One WS item per event (join, message, disconnect) |
| Invite / magic link | Save `{{inviteToken}}` from the create response |
| Contract | Every 2xx/4xx uses the default envelope test |

**Socket**

- Add only if the server has a socket gateway.  
- `{{wsUrl}}` in env (often `ws://localhost:5000`).  
- Auth: query `?token={{accessToken}}` or first client message — match the server.  
- Do not open WS before login in a collection run.

**Do not regenerate**

Login, refresh, logout, and 401-refresh scripts once they exist. New routes only get defaults.
