# Project 3: Web Fundamentals — Write-Up

**Author:** Suraj Pratap Singh
**Date:** August 2026
**Environment:** Same Kali Linux / Metasploitable2 lab from Projects 1 & 2
**Target app:** DVWA (Damn Vulnerable Web App) hosted on Metasploitable2
**Tools used:** Firefox Developer Tools (Storage, Network tabs), Wireshark

## Objective

Learn how HTTP, cookies, and sessions actually work by inspecting live traffic between Kali and a vulnerable web app, then apply that understanding to a hands-on SQL injection exploit.

## Part 1: Cookies & Sessions

- Logged into DVWA (`admin`/`password`) and inspected cookies via Firefox's Storage tab
- Found two cookies: `PHPSESSID` (session identifier) and `security` (app difficulty setting)
- Noted the `PHPSESSID` cookie had `HttpOnly: false` and `Secure: false` — meaning it's readable by JavaScript and would be sent over plain HTTP, both real weaknesses
- Deleted the `PHPSESSID` cookie and confirmed it immediately logged me out — proving the cookie alone represents the entire logged-in session, independent of the password

## Part 2: Inspecting Raw HTTP Traffic

- Used the Network tab in dev tools to capture the login POST request and viewed the username/password sent in plain text (no HTTPS on this test app)
- Used Wireshark to capture the same login at the packet level, filtering with `http.request.method == "POST"`, and confirmed the credentials were visible directly in the raw network capture
- This demonstrated two independent vantage points (browser-side and network-side) showing the same underlying issue: unencrypted HTTP exposes credentials to anyone who can observe network traffic

## Part 3: SQL Injection

- Set DVWA's security level to "Low" to expose the unprotected version of the vulnerability
- Confirmed normal behavior first: submitting User ID `1` returned one user's name
- Submitted the payload `1' OR '1'='1` and received **every user record in the database** instead of just one

**Why it worked:** the app likely builds its query as:
```sql
SELECT first_name, surname FROM users WHERE user_id = '$id'
```
The payload turns this into a condition that is always true (`'1'='1'`), so the `WHERE` clause matches every row instead of just one. User input was concatenated directly into the SQL query instead of being treated as pure data, letting injected code control the query's logic.

## Key Takeaways

- A session cookie is functionally equivalent to being logged in — cookie theft (e.g. via XSS) is as dangerous as password theft, sometimes more so since it needs no password at all
- `HttpOnly` and `Secure` cookie flags exist specifically to reduce this exposure, and their absence is a real, checkable vulnerability
- Plain HTTP transmits all form data, including passwords, as unencrypted plaintext — visible via browser dev tools or packet capture
- SQL injection stems from mixing user input with code (the query string) instead of keeping them separate; parameterized queries are the standard fix
- The same underlying vulnerability can be confirmed from multiple angles (application behavior, browser inspection, and raw packet capture), which is a valuable habit for verifying findings

## Next Steps

- Project 4: Cryptography practice set (Python + CyberChef)
