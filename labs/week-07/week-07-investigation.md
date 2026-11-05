# Week 7 API Investigation

**Finding: Broken Object Level Authorization (BOLA / IDOR)**

**Vulnerable Endpoint:** GET /api/profile/:id
**Vulnerable Code:** src/handlers/api.js (lines 7-12) and src/routes/api.js

**Analysis:**
1. The route `/api/profile/:id` is protected by `requireAuth` middleware, meaning the user must be logged in.
2. However, the `profileJson` handler in `src/handlers/api.js` does not verify if the logged-in user is authorized to view the profile of the requested `:id`.
3. It simply executes `db.prepare(PROFILE_QUERY).get(req.params.id)` and returns the full user profile.

**Exploit Scenario:**
- Attacker logs in as a low-privileged user (e.g., ID 10).
- Attacker changes the URL from `/api/profile/10` to `/api/profile/1`.
- The system returns the full profile of the administrator (ID 1), including sensitive fields like email and bio.

**Remediation:**
Implement an authorization check to ensure the requested ID matches the session ID, or that the requester has an 'admin' role.
Example fix in `profileJson`:
if (req.user.id !== req.params.id && req.user.role !== 'admin') {
    return res.status(403).json({ error: "Unauthorized access to profile." });
}
