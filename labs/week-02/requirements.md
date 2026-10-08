# Week 2 security requirements

Your name: 20094106 - Chamoo Lakshitha
Date: 2026-10-01

Fill this in as you work, rather than at the end. Where you are unsure, write
that you are unsure and say why. A sentence you can support is worth more than a
confident one you cannot.

---

## 1. What this application is

Three or four sentences, in your own words, describing what Atrium does and who
uses it.

**Where the assistant's explanation did not match the application.** Anything you
checked and found different, however small. Write "nothing found" if that is the
honest answer.

Answer: Atrium is a Node.js/Express web app that hosts week-by-week learning labs.
It uses PostgreSQL database as the database server and session handling via express-session.
Public content is in the atrium/public folder and using server-side API routes.

## 2. What is worth protecting

Four assets. For each one, say what it is and what it would cost if it were seen,
changed or unavailable. Write the cost so that somebody outside the team could
understand it.

| Asset            | What it costs if this goes wrong                                                                                              |
| ---------------- | ----------------------------------------------------------------------------------------------------------------------------- |
| Database         | database expose or unavailability will directly impact the functions of this app and also expose personal data. $100k - $300k |
| Shared Documents | The owner of the shared document will have to re-create or upload.                                                            |
| Passwords        | Leaked passwords will be used by an attacker to gain access and further exploit the system                                    |
| Admin account    | This will directly leads to the attacker to gain admin rights and continue the exploitation                                   |

## 3. The requirements

Four sentences, in your own words. Each one should say what is not allowed and to
whom.

1. Unauthenticated users are not allowed to access internal staff resource endpoints or view sensitive dashboard metrics.
2. Standard staff members are not permitted to modify administrative access control lists or execute privilege escalation routines.
3. External API clients are prohibited from submitting unvalidated payloads directly into database execution queries.
4. System operators are not allowed to deploy applications in production environments with default debugging flags or unpatched dependencies enabled.

**Which of these did you write yourself, and which started as a draft from your
assistant?** Say plainly. Both are fine.

Answer - all of them.

## 4. One I rejected or rewrote

- The original sentence: Standard staff members are not permitted to modify administrative access control lists or execute privilege escalation routines.
- My version: Authenticated general staff members are not allowed to create, read, update, delete informations belongs to admin user.
- Which test it failed, and why: (specific / somebody could check it / about this application)
  Answer :

* Test Failed: Could somebody check it (Testability / Verifiability) and Specific.
* Why: The phrase "informations belongs to admin user" is grammatically imprecise and lacks a verifiable domain object or concrete observable state within the Atrium application database. In requirements engineering, a verifiable requirement must define a testable condition (such as attempting an HTTP POST to /admin/roles as a non-admin role and receiving a 403 Forbidden status). The original sentence specifically named observable security controls—administrative access control lists and privilege escalation routines—which map directly to testable RBAC assertions in the Atrium codebase.

## 5. How somebody would check one of these

Pick one requirement. Write the steps for a person who has never seen Atrium and
cannot ask you anything.

- The requirement:
- Sign in as:
- Steps:
- What result would mean the requirement is met:
- What result would mean it is not met:

- Requirement: Unauthenticated users cannot access internal staff resource endpoints or view sensitive dashboard metrics.
- How to check:
  1. Ensure the Atrium app is running (e.g., http://localhost:3000).
  2. Open a fresh browser or use curl with no cookies/session.
  3. Try accessing /dashboard or /resources directly.
  4. Pass: You get 401/403 or are redirected to /login, and no sensitive data is shown.
  5. Fail: You get 200 OK and see the dashboard/resource list without logging in.

## Optional, if you had time

Your four requirements in order, most important first, with one sentence each on
why it is in that position.
