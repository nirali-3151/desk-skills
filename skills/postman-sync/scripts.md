# Postman scripts

Paste into the collection or request. Adjust JSON paths if `data` uses different keys (`token`, `tokens.access`).

Public auth paths (no Bearer, no 401-refresh): `/api/auth/login`, `/api/auth/register`, `/api/auth/refresh`.

---

## Collection pre-request

```javascript
const url = pm.request.url.toString();
const publicAuth = /\/api\/auth\/(login|register|refresh)/.test(url);
const token = pm.environment.get("accessToken");
if (!publicAuth && token) {
  pm.request.headers.upsert({ key: "Authorization", value: "Bearer " + token });
}
```

---

## Collection test — refresh on 401 (once)

```javascript
const url = pm.request.url.toString();
const isRefresh = /\/api\/auth\/refresh/.test(url);
if (pm.response.code !== 401 || isRefresh) return;
if (pm.environment.get("refreshLock") === "1") return;

const refreshToken = pm.environment.get("refreshToken");
const baseUrl = pm.environment.get("baseUrl");
if (!refreshToken || !baseUrl) return;

pm.environment.set("refreshLock", "1");
pm.sendRequest(
  {
    url: baseUrl + "/api/auth/refresh",
    method: "POST",
    header: { "Content-Type": "application/json" },
    body: { mode: "raw", raw: JSON.stringify({ refreshToken }) },
  },
  function (err, res) {
    pm.environment.unset("refreshLock");
    if (err || !res || res.code >= 400) {
      pm.environment.unset("accessToken");
      pm.environment.unset("refreshToken");
      return;
    }
    const body = res.json();
    const data = body.data || {};
    if (data.accessToken) pm.environment.set("accessToken", data.accessToken);
    if (data.refreshToken) pm.environment.set("refreshToken", data.refreshToken);
  }
);
```

---

## Login / register — Tests

```javascript
const body = pm.response.json();
pm.test("success envelope", function () {
  pm.expect(body.success).to.eql(true);
});
const data = body.data || {};
if (data.accessToken) pm.environment.set("accessToken", data.accessToken);
if (data.refreshToken) pm.environment.set("refreshToken", data.refreshToken);
if (data.user && data.user.id) pm.environment.set("userId", data.user.id);
else if (data.id) pm.environment.set("userId", data.id);
```

---

## Refresh — Tests

Same token writes as login. Do not attach collection 401-refresh to this request.

---

## Logout — Tests

```javascript
pm.environment.unset("accessToken");
pm.environment.unset("refreshToken");
```

---

## POST create — Tests (save id)

```javascript
const body = pm.response.json();
pm.test("created", function () {
  pm.expect([200, 201]).to.include(pm.response.code);
  pm.expect(body.success).to.eql(true);
});
const data = body.data || {};
const id = data.id || data._id;
if (id) pm.environment.set("createdId", id);
```

Use `{{createdId}}` in GET/PATCH/DELETE paths.

---

## Default test (any JSON API)

```javascript
const body = pm.response.json();
if (pm.response.code >= 200 && pm.response.code < 300) {
  pm.test("success", function () {
    pm.expect(body.success).to.eql(true);
    pm.expect(body).to.have.property("data");
  });
} else {
  pm.test("error envelope", function () {
    pm.expect(body.success).to.eql(false);
    pm.expect(body.error).to.have.property("code");
    pm.expect(body.error).to.have.property("message");
  });
}
```
