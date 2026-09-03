---

layout: post
title: "PortSwigger Lab: Understanding Blind SQL Injection"
date: 2026-09-06 20:00:00 +0000
categories: [Web Security, PortSwigger]
tags: [sql-injection, blind-sql-injection, burp-suite, web-security]
--------------------------------------------------------------------

## Introduction

SQL Injection occurs when an application incorporates untrusted user input into SQL queries without properly separating data from SQL syntax.

One particularly interesting variant is **Blind SQL Injection**.

In a blind SQL injection scenario, the application does not directly return database information in its response.

Instead, the attacker has to infer information from the application's behavior.

## What Makes It "Blind"?

Consider an application that behaves differently depending on whether a SQL condition is true or false.

For example:

```text
Condition TRUE  → Page behaves normally
Condition FALSE → Page behaves differently
```

An attacker can use these differences to extract information indirectly.

## Testing for Blind SQL Injection

Burp Suite can be used to intercept and modify requests.

The general workflow is:

```text
Application
     ↓
Intercept Request
     ↓
Identify Input
     ↓
Modify Parameter
     ↓
Observe Response
     ↓
Determine TRUE/FALSE Behavior
```

## Boolean-Based Testing

A common technique is to test whether an injected condition evaluates to true or false.

Conceptually:

```sql
' AND 1=1 --
```

versus:

```sql
' AND 1=2 --
```

The exact syntax depends on the target database and application.

The important observation is whether the application's response changes between the two conditions.

## Using Burp Suite

Burp Suite's Repeater is particularly useful because it allows requests to be modified and repeatedly tested.

The process becomes:

1. Capture a request.
2. Send it to Repeater.
3. Identify a potentially injectable parameter.
4. Modify the parameter.
5. Compare responses.
6. Determine whether the input influences SQL execution.

## Why Blind SQL Injection Matters

Blind SQL injection can be more difficult to detect than traditional SQL injection because database results may never appear directly in the response.

However, differences in:

* Response content
* Response length
* Status codes
* Redirects
* Timing

can provide useful information.

## Defensive Measures

Developers should prevent SQL injection using:

* Parameterized queries
* Prepared statements
* Input validation
* Least-privileged database accounts
* Secure database configuration

## Key Takeaways

The PortSwigger lab helped reinforce several important concepts:

* SQL injection does not always expose database contents directly.
* Application behavior can leak information.
* Burp Suite is useful for controlled request manipulation.
* Boolean conditions can be used to test application behavior.
* Parameterized queries are a primary defense against SQL injection.

## Conclusion

Blind SQL Injection demonstrates that an attacker does not necessarily need a visible database error or returned query results to extract information.

Subtle differences in application behavior can be enough to reveal sensitive information.

