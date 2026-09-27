# V3: Stored Cross-Site Scripting (OWASP A03:2021 Injection / NodeGoat A3)

**Owner:** Member 2 · **Branch:** `m2/threat-model-xss` · **Threat:** T3 in [`../02_THREAT_MODEL.md`](../02_THREAT_MODEL.md)

## 1. The weakness

NodeGoat renders pages with Swig templates, and `server.js` turns HTML auto-escaping **off**:

```js
swig.setDefaults({ autoescape: false });
```

Profile fields (first name, last name) are saved to MongoDB as typed and printed back with `{{firstName}}` in:

- `app/views/layout.html` line 75: the top-right name shown on **every** page for the logged-in user, and
- `app/views/benefits.html` lines 51-52: the employee list, which the **admin** views.

Because nothing is encoded, markup saved in a name is sent to the browser as live HTML. That makes it *stored* XSS: it is saved once and runs for every later viewer. There was also no Content-Security-Policy to stop an injected inline script from running.

## 2. How we demonstrate it (ethics form signed, local Docker only)

Follow NodeGoat's own tutorial for this issue at `http://localhost:4000/tutorial/a3` and use a **harmless** test script that only pops an alert box.

1. Log in as `user1`, open **Profile**, put the test script in **First Name**. Set Bank Routing # to digits ending in `#` (e.g. `123456#`), otherwise the form rejects the update, then Submit.
2. Reload any page: the alert fires from the name in the header (screenshot).
3. Log out, log in as `admin`, open **Benefits**: the alert fires in the admin's browser (screenshot). This is the "stored, hits another user" proof.
4. Right-click → View page source and find the unescaped `<script>` in the name cell (screenshot).

## 3. The fix (secure coding, defence in depth)

| Layer | Change | File |
|---|---|---|
| Output encoding (primary fix) | `autoescape: true`, so `<` `>` `"` `'` `&` become HTML entities on every template variable | `server.js` |
| Content-Security-Policy | `helmet.contentSecurityPolicy` with `script-src 'self'`, `object-src 'none'`, `frame-ancestors 'none'`. Even if some markup slips through, the browser refuses inline scripts. | `server.js` |
| CSP compatibility | The only inline script (cookie check on the login page) moved to `app/assets/js/cookie-check.js`, so the CSP does not break login | `app/views/login.html`, `app/assets/js/cookie-check.js` |
| Keep memos working | Memos are rendered by `marked` (with `sanitize: true`) and must output HTML, so that one call is marked `|safe` | `app/views/memos.html` |
| Session cookie | `cookie: { httpOnly: true }` set explicitly (express-session's default, now stated in code; clears a Semgrep warning) | `server.js` |

## 4. Re-test (after the fix)

1. `docker compose down -v && docker compose up --build` (the `-v` resets the DB so old payloads are gone).
2. Repeat steps 1-3 above with the same test script.
3. Expected: **no alert**. The name shows as literal text such as `<script>...` on the header and on the admin Benefits page (screenshot both).
4. View page source: the name now appears as `&lt;script&gt;...` (screenshot).
5. DevTools → Network → click the page request → Response Headers shows `Content-Security-Policy: default-src 'self'; script-src 'self'; ...` (screenshot).
6. Regression check: log in, post a memo with `**bold**`, confirm it still renders bold; confirm Dashboard charts and the login page still work.

## 5. Verification done before pushing

A test harness rendered the real `benefits.html` and `memos.html` templates through Express with a script payload in `firstName`:

| | Raw `<script>` in HTML | Escaped `&lt;script&gt;` | CSP header | Memo markdown renders |
|---|---|---|---|---|
| Before (autoescape off) | yes | no | none | yes |
| After (this branch) | **no** | **yes** | `default-src 'self'; script-src 'self'; ...` | yes |

## 6. SAST before/after

Re-run the same Semgrep command used for the baseline on this branch (`NEED.md` item E10). Expect `express-cookie-session-no-httponly` to disappear. Semgrep has no Swig rule, so the XSS fix itself is evidenced by the browser re-test and the CSP header, which the report should state.
