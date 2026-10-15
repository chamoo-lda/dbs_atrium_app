# Week 5 cross-tool triage

Your name: Chamoo Lakshitha | 20094106
Date: 2026-10-14
Atrium address and port: http://localhost:9090
Atrium version or date of local copy: Atrium v0.1.0 (local repo copy)

Fill this in as you work. If a check could not be completed, record what stopped you and what evidence you used instead. Do not paste a full session cookie value.

## 1. Scan coverage and sources

- Static evidence: `npm run scan` output (`src/handlers/directory.js`, `src/handlers/resources.js`, `src/db/reset.js`) and Week 3 scan output.
- Runtime evidence: ZAP authenticated run (Week 4 record / ZAP run against Atrium).
- Signed-in account used for protected routes: `alice.nolan` (password: `SpringRiver44`).
- Evidence that a protected route returned its page rather than `/login`: ZAP History and browser developer tools showing `GET /dashboard` returning status `200 OK` with dashboard HTML and valid `connect.sid` session cookie.
- Active scan target and how login and logout stayed outside it, if I used ZAP: Targeted scope restricted to authenticated functional routes (`/dashboard`, `/directory`, `/documents`, `/profile`), explicitly excluding `/login` and `/logout.*`.
- An entry my scan did not reproduce, and how I checked it instead: Z3 (User Controllable HTML Element Attribute) and Z4 (CSP Header Not Set) were verified via response header/source code inspection and ZAP case material rather than live active fuzzing.

## 2. The queue

For each entry, use **supported**, **ruled out**, or **needs review**. Keep overlap separate from status. Name a file and line, or a request and response, in the evidence column. If evidence is missing, say what check would settle it.

| ID | Status | Evidence or missing check | Overlap with another ID? | Investigate now, later, or close? |
|---|---|---|---|---|
| S1 | supported | `src/handlers/directory.js:8` — SQL query built by string concatenation combining `q` without parameterization. | Overlaps Z1, Z2, Z3 | Investigate now |
| S2 | ruled out | `src/handlers/resources.js:5` — `RESOURCE_QUERY` uses parameterized `?` placeholder despite Semgrep static heuristic rule match. | None | Close |
| S3 | ruled out | `src/db/reset.js:12` — `INSERT_USER` uses parameterized placeholders `?` with `.run()`. | None | Close |
| Z1 | supported | ZAP alert for SQL Injection / HTTP 500 error returned when testing special characters in `GET /directory?q=`. | Overlaps S1 | Investigate now |
| Z2 | supported | ZAP alert for Reflected XSS / search parameter `q` reflected unescaped in response HTML. | Overlaps S1, Z3 | Investigate now |
| Z3 | supported | ZAP alert for User Controllable HTML Element Attribute in search input `value="<%= q %>"`. | Overlaps S1, Z2 | Investigate later |
| Z4 | supported | ZAP alert for Content Security Policy (CSP) Header Not Set on `GET /dashboard`. | None | Investigate later |
| Z5 | supported | ZAP alert for Cookie No HttpOnly Flag (`Set-Cookie: connect.sid` without HttpOnly in `src/config/session.js`). | None | Investigate later |

## 3. First three matters to investigate

Rank the matters. Combine entries only when they concern one underlying path. For each, record the relevant account, route or file, possible consequence, uncertainty and next check.

1. **First:**
   - Queue ID or IDs: S1, Z1
   - Account and place: `alice.nolan`, `GET /directory?q=` (`src/handlers/directory.js:8`)
   - Why first: High severity impact (remote database error disclosure or potential data extraction via unparameterized string concatenation).
   - Next check: Test directory search with SQL control characters (`' OR 1=1 --`) and analyze server error or response data.
2. **Second:**
   - Queue ID or IDs: Z2, Z3
   - Account and place: `alice.nolan`, `GET /directory?q=` (`src/handlers/directory.js` / template rendering)
   - Why second: Direct risk to user sessions and browser security via reflected input in HTML response and form attributes.
   - Next check: Inspect template rendering of `q` in directory view and test script injection in search query.
3. **Third:**
   - Queue ID or IDs: Z5
   - Account and place: `alice.nolan`, `POST /login` (`src/config/session.js`)
   - Why third: Exposes session identifier (`connect.sid`) to client-side scripts, increasing risk if client-side vulnerabilities (like XSS) are present.
   - Next check: Inspect `src/config/session.js` session cookie configuration settings.

## 4. One exclusion or combination

- Entry ID: S3 (`src/db/reset.js:12`)
- Close, exclude from active work, or combine with another entry?: Exclude / Close (ruled out)
- Evidence for this decision: Code inspection of `src/db/reset.js` lines 12-13 shows `INSERT INTO users ... VALUES (?, ?, ?, ?, ?, ?, ?, ?)` uses parameterized placeholders (`?`) executed via `db.prepare(INSERT_USER).run(...)`. The string concatenation is only joining SQL query structure strings, not user input.
- What change would make me reopen it: If `src/db/reset.js` were modified to concatenate user-supplied input directly into the SQL string instead of using parameterized placeholders.

## 5. One static and runtime comparison

- Semgrep entry and ZAP entry: S1 (`src/handlers/directory.js:8`) and Z1 (`GET /directory?q=`)
- What the code observation shows: Semgrep flags that the SQL query is constructed using string concatenation (`+`) combining static SQL fragments with the `q` parameter without parameterization.
- What the request and response show: ZAP runtime scan shows that sending a query with special characters (such as an apostrophe `'`) causes the server to return an HTTP 500 Internal Server Error, indicating a database syntax/execution failure resulting from raw input handling.
- Do they concern the same underlying matter? Why?: Yes, both point to the exact same vulnerability: unparameterized handling of the `q` query parameter in `searchDirectory` (`src/handlers/directory.js`). Semgrep identifies the unsafe code pattern at the sink, while ZAP triggers the database error at runtime via HTTP request input.
- One claim they do not yet prove: Neither observation alone proves whether arbitrary data can be exfiltrated or modified via SQL injection (data extraction/manipulation impact remains untested without a successful exploit verification).

## 6. One finding for a colleague

**Finding:**
An unparameterized SQL injection vulnerability exists in the staff directory search function (`src/handlers/directory.js:8`), where the user-supplied `q` parameter is concatenated directly into the SQL query string (`LIKE '%" + q + "%'`). When authenticated users (such as `alice.nolan`) submit search queries containing database control characters (e.g., `'`), the application fails with an HTTP 500 error. If exploited, this could allow unauthorized database data retrieval or manipulation.

**Evidence references:**
Semgrep finding S1 (`src/handlers/directory.js:8`), ZAP finding Z1 (`GET /directory?q=`), and code inspection of `searchDirectory` in `src/handlers/directory.js`.

---

## Optional, if you had time

What does `src/config/session.js` show about the session that the Z5 response cannot?
`src/config/session.js` explicitly defines the session configuration settings (`secret`, `resave`, `saveUninitialized`, and cookie defaults like `httpOnly: false`, `sameSite: false`, `secure: false`), revealing the intended development configuration and configuration source code.

What does the response confirm that source inspection alone cannot?
The response headers (`Set-Cookie: connect.sid=...`) confirm that the running server is actively issuing cookies with those exact runtime flags in practice (confirming the lack of `HttpOnly` on the live instance rather than just static configuration intent).
