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
- Its explanation, in your own words: The search querry is directly inserted in to SQL statement so user input can change the logic of the SQL querry.
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

- What I entered:
- What came back:

**The case I was testing for**

- What I entered:
- What came back:
- How this differs from the ordinary case:

## 5. My answer

Delete the two that do not apply.

**Real** / **Not real** / **Not settled**

**Why, in two or three sentences.** Write for somebody who has not seen any of this.

**What would change my mind.** If new information would alter this answer, say what.

**How far this answer reaches.** What you established applies to a particular page,
a particular set of data and this version of the application. Say what you have
shown, and be careful not to claim more.

## 6. Back to the assistant

The thing you noted in section 2, that you had not verified at the time.

- Did I check it?
- Was it right?

---

## Optional, if you had time

The other two findings matched the same rule. Why are they not the same situation?
Two sentences.
