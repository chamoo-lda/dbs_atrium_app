# Week 4 runtime evidence (Completed)

Your name: Chamoo Lakshitha | 20094106
Date: 2026-10-12
Atrium address and port: http://localhost:9090
ZAP run or class scan evidence: ZAP run

## 1. Coverage before sign-in

- Account state: loggedout
- Routes reached by the spider: `/login`, static assets (`/css/app.css`, `/js/...`), error and default fallback routes.
- My direct `GET /directory` request: status, redirect and final page: Status `302 Found`, redirecting to `/login` (final page: login authentication form).
- What this run could not test: All authenticated application routes and functionality, including `/dashboard`, `/directory`, `/documents`, `/profile`, and parameterised search states or administrative actions requiring an active session.

## 2. Coverage after sign-in

- Seeded account used: `alice.nolan` (password: `SpringRiver44`)
- Evidence that ZAP received the dashboard as this account: ZAP History showing `GET /dashboard` request returning status `200 OK` with dashboard HTML and authenticated session cookie (`connect.sid`).
- Evidence that ZAP received `/directory?q=Alice` as this account: ZAP History showing `GET /directory?q=Alice` returning status `200 OK` containing staff directory results for Alice.
- Routes reached as this user: `/dashboard`, `/directory`, `/directory?q=...`, `/documents`, `/profile`.
- Routes or states newly reached compared with section 1: Authenticated staff routes (`/dashboard`, `/directory`, `/documents`, `/profile`) and parameterised search states that were previously blocked by the `requireAuth` redirect to `/login`.
- Active scan target and how I kept login and logout out of it: Targeted scope limited to authenticated functional routes (`/dashboard`, `/directory`, `/documents`, `/profile`) while explicitly excluding `/login` and `/logout` URLs from active scanning to prevent terminating the authenticated session or triggering account lockouts.

## 3. Scanner alert

- Alert name: Cross Site Scripting (Reflected) (ZAP Rule 40012)
- Risk and confidence as ZAP reported them: Medium risk / Medium confidence
- Method and affected URL: `GET`, `http://localhost:9090/directory?q=...`
- The input ZAP sent: Test fuzz payloads / script tags / encoded query inputs (e.g., `<script>alert(1)</script>`)
- Response status and relevant evidence: Status `200 OK`, response body reflecting the input unescaped within the EJS template (`<strong><%- q %></strong>`).
- Where I saw the request and response: ZAP History tab / Alerts tree, and browser Developer Tools Network tab.

## 4. Manual comparison

**Ordinary search**

- Account, method and full URL: `alice.nolan`, `GET`, `http://localhost:9090/directory?q=Alice`
- Status and any redirect: Status `200 OK`, no redirect.
- Relevant response text or visible result: Displays staff directory search results matching "Alice", rendering `<strong>Alice</strong>`.

**Alert request repeated by me**

- Account, method and full URL: `alice.nolan`, `GET`, `http://localhost:9090/directory?q=%3Cb%3Ehello%3C%2Fb%3E`
- Status and any redirect: Status `200 OK`, no redirect.
- Relevant response text or visible result: Renders bold text `<b>hello</b>` in the search summary banner.
- Did the alert's observation repeat? If not, what happened?: Yes, the input was reflected directly in the HTML response without escaping because the EJS view uses unescaped output (`<%- q %>`).

## 5. Conclusion

**What this evidence proves.** Limit the answer to this route, account and response.
This evidence proves that when running Atrium locally as `alice.nolan`, query parameter input sent to `/directory?q=` is rendered back into the HTML response unescaped (`<%- q %>`). Additionally, session cookies lack hardening flags (`HttpOnly`, `Secure`, `SameSite`).

**What this evidence does not prove.** Name an impact or wider claim that would need another test.
This evidence does not prove that arbitrary malicious JavaScript execution or session hijacking has successfully targeted any user, that attackers have stolen session cookies, or that the live production system behaves identically.

## 6. Short finding
During dynamic security analysis of Atrium running locally as `alice.nolan`, testing the staff directory search (`/directory?q=...`) revealed that user input supplied via the `q` parameter is reflected into the HTML response without proper HTML escaping (`<strong><%- q %></strong>`). If exploited by an authenticated attacker via a crafted malicious link, this reflected Cross-Site Scripting (XSS) vulnerability could enable arbitrary script execution within a victim's browser session. Evidence was captured via OWASP ZAP and verified manually through browser testing. High-level repair: Replace unescaped EJS output tags (`<%- q %>`) with HTML-escaped output tags (`<%= q %>`) in template views and enforce proper input sanitization and output encoding.

## Optional, if you had time
`/dashboard` (and `/directory`, `/documents`, `/profile`). This demonstrates that an unauthenticated spider has zero coverage of authenticated application logic and cannot discover routes protected behind authentication middleware, emphasizing that dynamic scanners require valid session credentials and configuration to be effective against access-controlled web applications.
