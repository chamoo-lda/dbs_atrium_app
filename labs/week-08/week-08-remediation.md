# Week 8 remediation record

Your name: Chamoo Lakshitha | 20094106
Date: 2026-11-05
Atrium version or copy used: Atrium v0.1.0 (local repo copy)

## 1. Original finding and before evidence

- Page and route: Directory Page, GET /directory
- Account used: Guest/Unauthenticated
- Earlier finding or source I started from: Week 6 SQL Injection finding
- Ordinary search text, URL, status and results-table rows: 'Alice', /directory?q=Alice, 200, returns users matching 'Alice'
- No-match search text, URL, status and results-table rows: 'XYZ123', /directory?q=XYZ123, 200, 0 rows
- Comparison search text, URL, status and results-table rows: '' OR '1'='1', /directory?q=' OR '1'='1, 200, returns all users in the database
- What the comparison shows, and what it does not show: It shows that the input is interpreted as SQL code, bypassing the filter. It does not show the internal database structure, only the output.

## 2. Root cause

- Source of the search text, with file and line: req.query.q in src/handlers/directory.js:21
- Path through the handler: showDirectory() -> searchDirectory(db, q)
- Database operation, with file and line: db.prepare(sql).all() in src/handlers/directory.js:12
- Control missing on that path: Lack of input parameterization; use of string concatenation to build the SQL query.
- Difference between the visible result and the root cause: The visible result is an over-privileged data dump; the root cause is the lack of separation between data and command.

## 3. Security property and useful behaviour

- One sentence stating the property the repair must enforce: User input must be treated strictly as data and never as part of the SQL command structure.
- Ordinary behaviour that must remain, including name and department search: The ability to perform partial matches using the LIKE operator for both display_name and department.

## 4. Candidate fixes

| Candidate             | Where the control acts | Effect on the property and ordinary search                                      | Decision |
| --------------------- | ---------------------- | ------------------------------------------------------------------------------- | -------- |
| Input Sanitization    | Handler                | May block malicious characters but prone to bypasses                            | Rejected |
| Parameterized Queries | DB Layer               | Ensures absolute separation of data and command; maintains search functionality | Chosen   |

- Assistant used, if any: Hermes Agent
- One suggestion I changed or rejected, and why: Rejected manual escaping of quotes because it is error-prone and less secure than native parameterization.
- Reason for my choice: Parameterized queries are the industry standard for preventing SQL Injection.

## 5. Change made

- File and function changed: src/handlers/directory.js -> searchDirectory()
- Changed lines:
  BEFORE: const sql = "... WHERE display_name LIKE '%" + q + "%' OR department LIKE '%" + q + "%'";
  AFTER: const sql = "SELECT id, display_name, email, department, role FROM users WHERE display_name LIKE ? OR department LIKE ?";
  EXECUTION: return db.prepare(sql).all(`%${q}%`, `%${q}%`);
- How the change keeps search text separate from the database instruction: The '?' placeholders tell the database engine to expect data values separately from the query logic.
- What I left outside this repair: Other potential injection points in the API (to be handled in separate remediation tasks).

## 6. First checks after the change

- Same ordinary search: 'Alice', /directory?q=Alice, 200, returns users matching 'Alice'.
- Same no-match search: 'XYZ123', /directory?q=XYZ123, 200, 0 rows.
- Same comparison search: '' OR '1'='1', /directory?q=' OR '1'='1, 200, 0 rows (searches for literal string).
- Empty search result: /directory?q=, 200, returns all users (as per original intended logic for empty search).
- npm test result and any failing test: All functional tests passed.
- npm run scan result, if available: Static scan no longer flags the concatenation in searchDirectory.

## 7. Handoff to Week 9

- What this evidence supports: Proof that the SQLi in the directory search is fully mitigated.
- Cases or related paths still to check: Check other search inputs in the administration panel.
- A focused automated test I would add: A test case in tests/functional that specifically tries the '' OR '1'='1 payload and asserts that the result count is not the total user count.
- Any known regression or remaining risk: None identified for this specific route.

## 8. Document description output
