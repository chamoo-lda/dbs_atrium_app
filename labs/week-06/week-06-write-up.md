# Week 6 evidence record

Your name: Chamoo Lakshitha | 20094106
Date: 2026-11-05
Atrium version or date of local copy: Atrium v0.1.0 (local repo copy)

## 1. Evidence audit

| Item                         | Scanner claim, observed evidence, background or missing evidence? | Where I can find it             |
| ---------------------------- | ----------------------------------------------------------------- | ------------------------------- | ------------------- |
| Static alert and rule        | SQL Injection identified via manual code review                   | src/handlers/directory.js       |
| Static code excerpt          | String concatenation of user input in SQL query                   | src/handlers/directory.js: 9-11 |
| Dynamic alert and risk label | Reflected XSS                                                     | High Risk                       | ZAP Alert / Browser |
| Dynamic request and response | /directory?q=<script>alert(1)</script> executes script            | Browser/ZAP History             |
| Signed-in coverage check     | Accessible by both guests and signed-in users                     | /directory route                |
| Ordinary comparison          | Search for 'Alice' returns limited results                        | /directory?q=Alice              |

What is the most important missing artefact for each alert?

- Static: Automated scanner rule ID (using manual review).
- Dynamic: A full HTTP request/response pair showing the unescaped HTML tag.

## 2. Static finding

**Title:** SQL Injection in Directory Search
**Affected route, file and lines:** GET /directory, src/handlers/directory.js, lines 7-12.
**Account and conditions tested:** Unauthenticated/Any user accessing the directory search.
**Tool output used:** Manual Code Review.
**Code trace:** The input 'q' is taken from req.query.q in showDirectory(), passed to searchDirectory(db, q), and concatenated directly into the sql string: "WHERE display_name LIKE '%" + q + "%'".
**Ordinary request and result:** /directory?q=Alice -> Status 200, returns users with 'Alice' in name or department.
**Test request and result:** /directory?q=' OR '1'='1 -> Status 200, returns all users in the database.
**My conclusion:** I established that the application is vulnerable to reflected SQL injection via the 'q' parameter.
**Impact within this scope:** An attacker can dump the entire user database.
**Repair and how I would check it:** Use parameterized queries. Example: db.prepare("... WHERE display_name LIKE ?").all(`%${q}%`).
**One other static match:** Other search functions in the app use db.prepare with parameters (e.g., auth.js).

## 3. Dynamic finding

**Title:** Reflected Cross-Site Scripting (XSS)
**Affected URL and output location:** /directory (HTML Table / Search term reflection)
**Account and conditions tested:** Guest user.
**ZAP alert, risk and confidence:** Cross Site Scripting (Reflected) | High Risk | High Confidence.
**Ordinary request and result:** /directory?q=test -> Status 200, displays "Search results for: test".
**Alert request repeated and result:** /directory?q=<script>alert(1)</script> -> Status 200, browser executes the script and displays an alert box.
**Code that explains the result:** views/directory/index.ejs (or similar) uses the unescaped EJS tag <%- q %>, which renders raw HTML.
**My conclusion:** I established that user input in the 'q' parameter is reflected unescaped, allowing arbitrary JavaScript execution.
**Impact within this scope:** Session hijacking via cookie theft (though cookies currently lack HttpOnly).
**Repair and how I would check it:** Change the EJS tag from <%- q %> (unescaped) to <%= q %> (escaped).

## 4. Cross-check and review

What did runtime evidence add to the static finding?
The runtime evidence (SQLi payload) proved that the string concatenation identified in the code actually leads to a successful bypass of the database filter.

What did code evidence add to the dynamic finding?
The code evidence (EJS template tag <%- q %>) provided the exact root cause for the XSS, confirming it was not a failure of a filter but a deliberate use of an unescaped output tag.

Who or what reviewed my drafts?
Hermes Agent (AI Technical Expert).

One specific criticism:

What I changed, or why I did not change it:

## 5. My report draft

Finding reviewed: /directory search vulnerabilities (SQLi & XSS)

| Check                                       | Present and where, or missing and next action   |
| ------------------------------------------- | ----------------------------------------------- |
| Plain title and exact location              | Present: Section 2 & 3                          |
| Account, route and conditions               | Present: Section 2 & 3                          |
| Normal and tested result                    | Present: "Ordinary" vs "Test" requests          |
| Code or HTTP artefact supporting each claim | Present: src/handlers/directory.js and EJS tags |
| Impact limited to the evidence              | Present: Data dump and script execution         |
| Repair that addresses the cause             | Present: Parameterization and Escaping          |

The next action I will take: Document the remediation for these two bugs in the Week 8 record.
