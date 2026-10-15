# Week 2 security requirements (Completed)

Your name: 20094106 - Chamoo Lakshitha
Date: 2026-10-01

---

## 1. What this application is

Atrium is a Node.js and Express web application backed by a PostgreSQL database, designed for hosting and managing week-by-week learning labs and internal staff operations. It recognizes distinct user roles (such as standard staff/students and administrators) and provides features for session authentication, viewing and searching the staff directory, accessing shared documents/resources, and administrative management.

**Where the assistant's explanation did not match the application.** Nothing found. The assistant's structural overview of routes (`src/routes/`), handlers (`src/handlers/`), session configuration (`src/config/session.js`), and database schema matched the codebase upon verification.

## 2. What is worth protecting

Four assets. For each one, say what it is and what it would cost if it were seen, changed or unavailable. Write the cost so that somebody outside the team could understand it.

| Asset                                         | What it costs if this goes wrong                                                                                                                                                                                                            |
| --------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Staff Directory & Contact Records**         | If unauthorized external parties or lower-privileged users view or export full contact records, staff face targeted phishing, social engineering, and privacy violations, leading to reputational damage and potential regulatory scrutiny. |
| **Shared Resources & Lab Documents**          | If shared documents or lab materials are modified, deleted, or corrupted by unauthorized users, ongoing educational activities and lab delivery are disrupted, requiring hours of manual instructor time to restore and verify integrity.   |
| **User Passwords & Credential Hashes**        | If password hashes stored in the PostgreSQL database are leaked and cracked, attackers can successfully authenticate as legitimate staff or administrators, granting unauthorized access to internal tools and student data.                |
| **Administrator Privileges & Access Control** | If an attacker compromises an administrative account (or bypasses admin middleware checks on `/admin`), they gain full control over user role assignments, database records, and system settings, leading to total system compromise.       |

## 3. The requirements

Four sentences, in your own words. Each one should say what is not allowed and to whom.

1. Unauthenticated users and unverified sessions are not permitted to view or query the employee directory (`/directory`) or retrieve staff contact details.
2. Standard staff members are not allowed to upload, overwrite, or delete shared resource documents (`/resources`) designated for course instruction.
3. Users authenticated without administrator status are prohibited from accessing the administrative control panel (`/admin`) or modifying user role records in the PostgreSQL database.
4. The application must not transmit user session cookies over unencrypted HTTP connections or store plaintext passwords in application logs.

**Which of these did you write yourself, and which started as a draft from your assistant?**
Requirements 1 and 3 started as general drafts from the assistant and were rewritten by me to specifically reference Atrium routes (`/directory`, `/admin`) and database interactions. Requirements 2 and 4 were written entirely by myself to target Atrium's resource management and session handling.

## 4. One I rejected or rewrote

- The original sentence (Assistant draft): "External clients and unauthorized users must not be permitted to execute privilege escalation attacks against administrative endpoints."
- My rewritten version: "Authenticated users without the administrator role are strictly prohibited from accessing administrative routes (`/admin`) or executing POST requests to `/admin/users` to alter account permissions."
- Which test it failed, and why:
  - _Test Failed_: **Is it about Atrium?** and **Could somebody check it?**
  - _Why_: The original sentence used vague security jargon ("privilege escalation attacks") rather than referencing specific Atrium files, routes, or HTTP states. It did not name the exact page or request where an auditor would look (e.g., trying to GET `/admin` without admin rights).

## 5. How somebody would check one of these

Choose one of your four requirements and write out the steps somebody would follow to find out whether Atrium satisfies it. Write for a person who has never seen the application and cannot ask you questions.

- **Requirement**: Unauthenticated users and unverified sessions are not permitted to view or query the employee directory (`/directory`) or retrieve staff contact details.
- **Sign in as**: No user (unauthenticated / guest session).
- **Steps**:
  1. Open a browser or HTTP client (e.g., `curl`) with no existing cookies or active session.
  2. Attempt to navigate directly to `http://localhost:9090/directory`.
  3. Observe the server response and rendered page.
- **What result would mean the requirement is met**: The server responds with a 401 Unauthorized or 403 Forbidden status code (or redirects to the login page `/auth/login`), and no staff directory records or contact details are displayed.
- **What result would mean it is not met**: The server returns a 200 OK status code and displays the staff directory table containing names, departments, and contact information without requiring authentication.

## Optional, if you had time

**Priority order of the four requirements (most important first) and why**:

1. _Requirement 3 (Administrative Privilege Enforcement)_: This is top priority because compromise or unauthorized access to the `/admin` panel grants complete control over the application, user databases, and security posture.
2. _Requirement 1 (Directory Access Limitation)_: Second priority because protecting employee contact details prevents direct harvesting of personal/professional data that enables social engineering and phishing attacks.
3. _Requirement 2 (Resource Integrity & Authorization)_: Third priority because protecting course and lab documents ensures operational continuity and learning integrity.
4. _Requirement 4 (Session Security & Credential Isolation)_: Fourth priority because robust session management and credential handling underpin all higher-level authentication and authorization guarantees.

**Fifth requirement (unique choice)**: 5. "The application profile endpoint (`/profile`) must ensure that an authenticated user can only view and update their own user record, and any attempt to request or modify another user ID via parameter tampering must be rejected with a 403 Forbidden response."
