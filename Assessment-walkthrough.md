## Testing Input Behavior

I started by simply interacting with the login form . Instead of using normal values, I entered unexpected inputs and special characters to see how the application reacts.

While testing different inputs, I noticed that entering special characters such as `'` caused unusual responses and inconsistent error handling. This immediately made me curious, because strange reactions to simple characters can sometimes indicate that user input is being passed directly to back-end logic without proper protection.

At this stage , i started considering the possibility that the authentication mechanism could be vulnerable to injection attacks
> If the application is not strictly validating or sanitizing input, maybe the authentication process itself can be manipulated.

This observation guided the next phase of testing.

## Authentication Bypass — SQL Injection Attempt

Since the application reacted oddly to special characters, I decided to test whether authentication could be bypassed using a classic injection-style input.

I attempted to log in using ' OR 1=1 --

Unexpectedly, the application granted access and authenticated me as an administrative user without requiring valid credentials.

![Successful admin login after SQL injection] [/resources/login-as-admin.png]

This strongly suggests that the backend login query is built by directly inserting user input into a database query. When I added the logical condition `OR 1=1`, the authentication check likely became always true, and the rest of the query was ignored because of the comment syntax (`--`).

Instead of verifying real credentials, the application accepted the manipulated condition and granted access. 
## What are the impacts

This vulnerability allows an attacker to log in without knowing any valid username or password.

By manipulating the login input, the attacker can bypass the authentication system and gain access as an administrator. This means an unauthorized user could:

- View sensitive data
- Modify or delete information
- Access restricted features meant only for admins
- Potentially take full control of the application

Since authentication is one of the main security protections in any system, breaking it can lead to a **complete compromise of the application**.

## Remediation

To fix this issue, user input must never be directly included in database queries.

The application should:

- Use **parameterized queries (prepared statements)** instead of building SQL queries with raw user input
- Apply proper **input validation and sanitization**
- Use secure authentication libraries or frameworks that handle database queries safely
- Implement **error handling** that does not reveal unusual system behavior when special characters are entered

By separating user input from the SQL logic, injection attacks like this can be prevented.
this vulnerability fall under: [A05:2025 Injection](https://owasp.org/Top10/2025/A05_2025-Injection/)
# Exploring Access Control
Any modern app is expected to enforce proper access control after user logs in so i decided to continue testing the application from the perspective of a regular authenticated user to see whether i could access data or features that should normally be restricted.

