---
title: Code Injection
parent: Web Security
nav_order: 2
layout: page
header-includes:
  - \pagenumbering{gobble}
output:
  pdf_document:
    pandoc_args: ["--lua-filter=color-text-span.lua"]
---

# Code Injection

## What Is Code Injection?

**Code Injection** is a class of attacks in which an attacker tricks an application into executing **arbitrary code** the developer never intended.

In simple terms: the attacker supplies specially crafted input that the application mistakenly runs as instructions instead of treating it as plain data.

Example: A developer writes this innocent-looking code to let users ping an IP:

```python
import os
ip = input("Enter IP address to ping: ")
os.system("ping -c 4 " + ip)          # Looks harmless...
```

The developer _expects_ the user to enter a normal IP address like `8.8.8.8`.
But an attacker enters this instead:

```
8.8.8.8; rm -rf /
```

The command that actually runs on the server is:

```bash
ping -c 4 8.8.8.8; rm -rf /
```

The server pings Google’s DNS **and then silently deletes every file it can reach**.

This happens when untrusted user input is interpreted as **executable code** instead of being treated as plain data (**Code is Data and Data is Code**). The boundary between “user data” and “server code” has been completely erased.

### Why Code Injection Is Extremely Dangerous

Code injection attacks are among the most severe vulnerabilities because they often give the attacker full or near-full control over the application and even the underlying server. An attacker can steal, modify, or delete sensitive data such as databases, files, and user accounts; take over user accounts or escalate privileges; execute arbitrary commands on the server through remote code execution; install backdoors or malware; or completely compromise the application and server infrastructure.

Unlike XSS (which runs in the victim’s browser), many code injection attacks allow direct compromise of the _server-side_ application and infrastructure.

## SQL Injection (The Most Common Form)

**SQL Injection (SQLi)** is the most well-known and frequently exploited form of code injection. It occurs when an attacker manipulates a SQL query by injecting malicious SQL code through user input.
Most web applications rely on a database to store and retrieve data. Unfortunately, many developers, especially when working with legacy code or building quick prototypes, still concatenate user input directly into SQL query strings. This dangerous practice creates the perfect opening for attackers.

Here is a vulnerable example in PHP:

```php
$username = $_POST['username'];
$password = $_POST['password'];
$query = "SELECT * FROM users WHERE username = '$username' AND password = '$password'";
$result = mysqli_query($conn, $query);
```

If an attacker enters the following in the username field:

```
' OR '1'='1
```

The resulting query becomes:

```sql
SELECT * FROM users WHERE username = '' OR '1'='1' AND password = '...'
```

Because `'1'='1'` is always true, the query returns all users and allowing the attacker to bypass authentication entirely.

### A SQL Injection Example (TRU Course Feedback)

To understand exactly how this works, imagine a simple course feedback system for SENG courses at Thompson Rivers University. The system has a database table called `course_feedback`:

| id  | course    | rating | term        |
| --- | --------- | ------ | ----------- |
| 1   | SENG 4220 | 4.7    | Fall 2026   |
| 2   | SENG 4640 | 4.3    | Fall 2026   |
| 3   | SENG 4620 | 4.8    | Winter 2027 |

A user can view ratings by visiting a URL like:

```
https://www.tru.ca/feedback?course=SENG4220
```

On the server, the value of the `course` URL parameter is directly concatenated into an SQL query. The server builds and executes this SQL query:

```sql
SELECT rating FROM course_feedback WHERE course = 'SENG4220'
```

Now suppose an attacker enters this malicious value for the `course` parameter:

```sql
garbage'; SELECT password FROM users WHERE username = 'admin
```

The query the server actually executes becomes:

```sql
SELECT rating FROM course_feedback WHERE course = 'garbage';
SELECT password FROM users WHERE username = 'admin'
```

Because the input contained a closing quote (`'`) and a semicolon (`;`), the attacker was able to end the original query early and start a completely new query that steals the administrator’s password.

This is the power of SQL injection: with a single crafted input, an attacker can make the database execute arbitrary SQL statements.

Consider another common vulnerable pattern. Consider the web application has a `users` table containing usernames and passwords. When a user tries to log in, the server runs:

```sql
SELECT username FROM users
WHERE username = 'alice' AND password = 'password123'
```

If the query returns at least one row, login succeeds.

An attacker who does not know any valid username or password can still log in by carefully crafting the input. Here is how they do it:

In the username and password fields the attacker submits:

```sql
Username: `evil' OR 1=1--`
Password: `garbage`
```

This produces:

```sql
SELECT username FROM users
WHERE username = 'evil' OR 1=1--' AND password = 'garbage'
```

The extra quote closes the string early. And since the attacker has added `OR 1=1` so the `WHERE` clause will always be true, no matter what username is used. Everything after `--` is treated as a comment and ignored. Now, because `1=1` is a universal truth, the `WHERE` condition evaluates to True for every record. Therefore, the DBMS bundles up the entire users table and hands it back to the web server. When the web application's login routine processes this response, it typically only fetches the very first row, expecting a single matching account. In most databases, records are returned in the order they were originally created. Because the system administrator account is almost always the first account set up when a database is initialized, it occupies that first row. As a result, the application blindly grabs the first record it sees and logs the attacker in as the `admin`, granting them full control of the system.

This single technique (`' OR 1=1--`) is one of the most famous and effective SQL injection payloads.

### Common SQL Injection Techniques

Once an attacker has found a vulnerable input field, they can use several techniques depending on what kind of feedback the application gives them. Here are the most common approaches:

**1. Authentication Bypass**  
This is the most basic and well-known technique. By injecting something like `' OR 1=1--` or `admin'--`, the attacker modifies the query’s logic so that the condition always evaluates to true. This allows them to log in as any user (often the administrator) without knowing the password.

**2. UNION-based Attacks**  
When the application displays data from the database (for example, search results or user profiles), attackers can use the `UNION` operator to combine the original query with a second query. This lets them extract data from other tables in the database, such as usernames, passwords, credit card numbers, or email addresses.

**3. Boolean-based Blind SQL Injection**  
In many cases the application does not return database errors or raw data. The attacker must then rely on subtle differences in the application’s behavior. By injecting conditions that are either true or false (for example, `AND 1=1` vs `AND 1=2`), they can observe whether the page loads differently, returns different content, or shows different error messages. Bit by bit, they can reconstruct sensitive information.

**4. Time-based Blind SQL Injection**  
When even boolean differences are not visible, attackers turn to timing attacks. They inject database functions that cause deliberate delays, such as `AND SLEEP(5)`. By measuring how long the server takes to respond, they can determine whether a condition was true or false and gradually extract data one character at a time.

**5. Error-based SQL Injection**  
Some database systems leak valuable information through error messages when they encounter invalid SQL syntax. Attackers deliberately craft input that triggers these errors, forcing the database to reveal table names, column names, database version, or even parts of the query structure itself. This information is then used to craft more precise attacks.

### Why SQL Injection Is So Powerful

A successful SQL Injection attack can give an attacker devastating control. They can read, modify, or delete any data in the database, add new administrator accounts, execute operating system commands (in some configurations), and in many cases completely take over the database server.

This level of access is exactly why SQL Injection has been responsible for some of the largest data breaches in history. One of the most infamous examples is the _2008 Heartland Payment Systems breach_. Attackers used SQL Injection to steal more than 130 million credit and debit card numbers, one of the largest financial data breaches ever recorded at the time. The attack caused massive financial losses, destroyed customer trust, and nearly destroyed the company.

## Other Types of Code Injection

While SQL Injection is the most common, the same fundamental mistake, i.e., concatenating user input into executable contexts, can occur in many other places.

### 1. OS Command Injection

This occurs when user input is passed to operating system commands (for example, through functions that execute shell commands).
A vulnerable example in Python looks like this:

```python
import os
filename = input("Enter filename: ")
os.system("cat " + filename)  # Dangerous!
```

**Attack**  
If the user enters:

```
file.txt; rm -rf /
```

The actual command that runs on the server becomes:

```bash
cat file.txt; rm -rf /
```

The attacker can now execute any system command they want. In this case, deleting every file the application has permission to remove.

**Safe Alternative:** Never build shell commands using string concatenation. Instead, use APIs that pass arguments as separate lists so the shell cannot interpret them as code:

```python
import subprocess
subprocess.run(["cat", filename])  # Safe — arguments are not interpreted by the shell
```

### 2. Code Evaluation (eval / exec)

Many programming languages, especially scripting languages such as Python, JavaScript, PHP, and Ruby, provide functions that allow a program to execute code stored in a string at runtime. Functions like `eval()`, `exec()`, or similar are extremely powerful but also extremely dangerous when used with untrusted input.

While this vulnerability is most common in scripting languages (because they treat strings as executable code by design), it can also appear in lower-level languages. For example, in C and C++, functions like `system()` or the `exec()` family can be misused in similar ways, although exploiting them usually requires more effort than in scripting environments.

A vulnerable example in JavaScript (Node.js):

```js
const userInput = req.body.expression;
const result = eval(userInput); // Extremely dangerous
```

An attacker can send:

```js
process.mainModule.require("child_process").execSync("rm -rf /");
```

This gives the attacker full control over the Node.js process and potentially the entire server.

There is almost never a legitimate reason to use eval(), exec(), or any equivalent function with user-supplied input in production code. These functions should be avoided entirely when dealing with untrusted data.

This code does two things:

1. Loads Node.js’s built-in `child_process` module (the only way to run shell commands from JavaScript)
2. Tells it to execute the dangerous system command `rm -rf /`

When `eval()` runs this string, the server deletes files and gives the attacker full control over the Node.js process and potentially the entire server.

The attacker sends the following valid JavaScript code:

```js
process.mainModule.require("child_process").execSync("rm -rf /");
```

This code loads Node.js’s built-in child_process module (the standard way to run shell commands from JavaScript) and instructs it to execute rm -rf /. When eval() runs this string, the server deletes files, giving the attacker full control over the Node.js process and potentially the entire server.

### 3. Server-Side Template Injection (SSTI)

Modern web frameworks rely on template engines such as Jinja2 (Python), Twig (PHP), Freemarker (Java), and others to generate dynamic HTML. These engines are designed to safely insert data into templates. However, when developers pass user input directly into the template context without proper sandboxing or escaping, attackers can inject malicious template code that gets executed on the server.

For example, in Jinja2 a simple test payload like `{{7*7}}` will output `49` if the template engine is evaluating expressions. More dangerous payloads can access internal objects and even execute system commands:

```jinja
{{config}}
{{self.__init__.__globals__['os'].popen('id').read()}}
```

The second line above can execute operating system commands and return their output, leading to full remote code execution on the server.

> **Note**: XSS is technically a form of code injection, but because it targets the browser (client-side) rather than the server, it is usually covered separately.

## Historical Real-World Code Injection Attacks

Despite being a well-understood vulnerability for decades, Code injection, and particularly SQL Injection, remains one of the most common serious vulnerabilities in web applications.

One of the earliest high-profile cases occurred in 2011, when **Sony Pictures** was breached. Attackers used SQL Injection, among other techniques, to steal large amounts of sensitive data, including personal information of employees and their families, internal emails, executive salaries, and unreleased films. The breach caused massive reputational damage to the company.

In 2012, a hacker group used SQL Injection to breach **Yahoo! Voices**, stealing approximately 450,000 login credentials. The passwords were stored in plaintext, making the breach particularly damaging as the stolen credentials could be easily reused in credential stuffing attacks against other services.

In 2014, a hacking group called **NullCrew** exploited an SQL Injection vulnerability in a third-party IT supplier used by **Bell Canada**. The attackers extracted the usernames and passwords of over 22,000 Bell customers, along with a small number of valid credit card numbers.

In 2015, the UK telecommunications company **TalkTalk** suffered a major breach when attackers exploited SQL Injection vulnerabilities in legacy web pages. The attackers accessed the personal and banking details of approximately 157,000 customers. The company was later fined a record £400,000 by the UK Information Commissioner’s Office for failing to protect customer data.

A major shift in the scale of code injection attacks occurred in 2021 with **Log4Shell** (CVE-2021-44228). This remote code execution vulnerability in the widely used Apache Log4j logging library allowed attackers to execute arbitrary code on millions of servers worldwide simply by sending specially crafted log messages. It is considered one of the most severe vulnerabilities in recent history due to its ease of exploitation and massive impact.

In 2023, the **MOVEit Transfer** file transfer software was exploited through a critical SQL Injection vulnerability (CVE-2023-34362). Threat actors, notably the Cl0p ransomware group, used the flaw to steal sensitive data from thousands of organizations that used the software. The incident affected an estimated 60 million individuals and became one of the largest data breaches of the year.

In 2012, a hacker group used SQL Injection to breach **Yahoo! Voices**, stealing approximately 450,000 login credentials. The passwords were stored in plaintext, making the breach particularly damaging as the stolen credentials could be easily used in credential stuffing attacks against other services.

Another significant incident occurred in 2011 when **Sony Pictures** was breached. Attackers used SQL Injection, among other techniques, to steal large amounts of sensitive data, including personal information of employees and their families, internal emails, executive salaries, and unreleased films. The breach caused massive reputational damage.

In 2014, a hacking group called **NullCrew** exploited an SQL Injection vulnerability in a third-party IT supplier used by **Bell Canada**. The attackers were able to extract the usernames and passwords of over 22,000 Bell customers, along with a small number of valid credit card numbers. Although the breach was relatively small in scale compared to others, it highlighted how even well-established Canadian companies could be exposed through vulnerabilities in their supply chain and legacy systems.

In 2015, the UK telecommunications company **TalkTalk** suffered a major breach when attackers exploited SQL Injection vulnerabilities in legacy web pages. The attackers were able to access the personal and banking details of approximately 157,000 customers, including names, addresses, dates of birth, email addresses, and in some cases bank account details. The company was later fined a record £400,000 by the UK Information Commissioner’s Office for failing to protect customer data.

## Defending Against Code Injection

All forms of code injection, whether SQL Injection, OS Command Injection, Code Evaluation, or Server-Side Template Injection, share the same fundamental root cause. In every case, untrusted user input is being interpreted as executable code rather than as plain data.

The defense philosophy is therefore identical across all of them: _treat user input strictly as data, never as code_. Once this principle is understood and applied consistently, the vast majority of code injection vulnerabilities can be prevented at the source.

### The Primary Defense Against SQL Injection

The most reliable protection against SQL Injection is the use of **parameterized queries**, also known as **prepared statements**. This technique completely separates the structure of the SQL query from the data the user provides.

Instead of building the SQL query by concatenating strings, the developer:

1. Writes the SQL query with **placeholders** (e.g., `?` or named parameters like `:username`).
2. The database **compiles** (prepares) the query first.
3. The developer then **binds** the user input as parameters **after** the query is compiled.

Because the user input is never treated as part of the SQL syntax, injection becomes impossible even if the input contains quotes, semicolons, or SQL keywords.

Consider this common but dangerous pattern in Python, where user input is concatenated directly into a SQL query:

```python
query = f"SELECT * FROM users WHERE username = '{username}' AND password = '{password}'"
cursor.execute(query)
```

The proper and secure approach is to use a parameterized query instead:

```python
query = "SELECT * FROM users WHERE username = %s AND password = %s"
cursor.execute(query, (username, password))
```

This pattern works across virtually every programming language and database library. The exact syntax for placeholders varies slightly depending on the language and database driver, but the principle remains identical in every case. Modern object-relational mappers (ORMs) such as SQLAlchemy, Django ORM, Entity Framework, and Prisma automatically use parameterized queries when used correctly. However, the rule is simple: if a query includes any user input, it must use parameterized queries. There is no acceptable exception.

### Defending Against Other Forms of Code Injection

The same fundamental principle applies to every other type of code injection.

For **OS Command Injection**, never build shell commands by concatenating strings. Instead, use safe APIs that accept commands and arguments as separate lists. In Python, for example, use `subprocess.run(["cat", filename])` rather than `os.system("cat " + filename)`. The former prevents the shell from interpreting user input as command syntax.

For **Code Evaluation**, the solution is absolute: never use `eval()`, `exec()`, or any equivalent function with user-supplied input. These functions should be removed from production code entirely. There is almost never a legitimate need for them when handling untrusted data.

For **Server-Side Template Injection**, treat template engines as code execution environments rather than simple string substitution tools. Use sandboxed template configurations, disable dangerous globals and methods, and avoid passing raw user input directly into template contexts.
