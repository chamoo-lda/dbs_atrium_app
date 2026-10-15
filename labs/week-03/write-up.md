# Week 3 record

Your name: Chamoo Lakshitha
Date: 2026-10-08

Fill this in as you work, rather than at the end. Where you are unsure, write
that you are unsure and say why. That is worth more than a confident sentence you
cannot support.

---

## 1. The finding I chose

State which of the three the tool reported.

- File: src/handlers/directory.js
- Line: 8-11
- What the tool said about it: This SQL statement is built by joining pieces of text together... if any pieces came from outside the application, the database will read it as part of the command rather than as a value

## 2. What an assistant told me

Say which assistant you asked, and what it said in a sentence or two.

- Assistant used: Hermes
- Its explanation, in your own words: The search query is directly inserted into the SQL statement so user input can change the logic of the SQL query.
- One thing it asserted that I had not verified at that point: whether the search input actually reaches the SQL in a dangerous way in the running app.

## 3. What the code shows

Answer all four. If you cannot answer one, say so.

**Where does the data come from?** - The input comes from req.query.q in the HTTP request for the directory page.

**What happens to it on the way?** - It is passed into searchDirectory(db, q), then concatenated into the SQL at the LIKE clauses.

**Where does it become dangerous?** - When the database executes the final SQL string, the attacker-controlled string becomes part of the SQL syntax instead of a literal value.

**What stands in the way?** - Nothing in this code prevents malicious input from being inserted into the query string. There is no parameter binding and no sanitization.

## 4. What the running application shows

Record both. A single result on its own proves nothing.

**Ordinary case**
- What I entered: Nolan
- What came back: 1 matching colleague (Alice Nolan)

**The case I was testing for**
- What I entered: Nolan%' OR '%'='
- What came back: All 3 colleagues in the staff directory (Alice Nolan, Bob Keane, Morgan Doyle)
- How this differs from the ordinary case: Instead of filtering the results for colleagues matching "Nolan", the injected SQL logic (`OR '%'='`) evaluates to true for all rows, returning the entire user directory.

## 5. My answer

Delete the two that do not apply.

**Real**

**Why, in two or three sentences.** Write for somebody who has not seen any of this.
The application concatenates user-supplied input (`req.query.q`) directly into the SQL query string without parameterization. By supplying crafted input (`Nolan%' OR '%'='`), an attacker alters the SQL query logic, bypassing the intended name and department filters to extract all staff records from the database.

**What would change my mind.** If new information would alter this answer, say what.
Evidence that the input is sanitized, escaped, or parameterized before reaching the database, or if the database driver strictly enforced parameterized execution.

**How far this answer reaches.** What you established applies to a particular page,
a particular set of data and this version of the application. Say what you have
shown, and be careful not to claim more.
This finding and proof apply exclusively to the staff directory search route (`/directory` in `src/handlers/directory.js`) within this specific version of the Atrium application using the SQLite database. It does not establish vulnerabilities in other pages or endpoints.

## 6. Back to the assistant

The thing you noted in section 2, that you had not verified at the time.

- Did I check it? Yes.
- Was it right? Yes, the assistant correctly diagnosed that direct string concatenation allowed user input to alter SQL query logic and compromise data filtering.

---

## Optional, if you had time

The other two findings matched the same rule. Why are they not the same situation?
Two sentences.
`src/db/reset.js` uses parameterized placeholders (`?`) for database initialization/seeding rather than untrusted runtime input, while `src/handlers/resources.js` uses parameterized queries (`WHERE resources.id = ?`) for resource retrieval.
