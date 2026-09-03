---

layout: post
title: "Understanding Server-Side Request Forgery (SSRF)"
date: 2026-09-07 20:00:00 +0000
categories: [Web Security]
tags: [ssrf, web-security, burp-suite, portswigger]
---------------------------------------------------

## Introduction

**Server-Side Request Forgery (SSRF)** is a vulnerability that occurs when an attacker can influence a server into making requests to an unintended destination.

Instead of the attacker's machine making the request directly, the vulnerable application makes the request on the attacker's behalf.

## The Basic Concept

A vulnerable application may provide functionality such as:

```text
Fetch an image from this URL
```

An application might normally receive:

```text
https://example.com/image.jpg
```

If the server does not properly validate the destination, an attacker may attempt to influence the server into requesting an internal resource.

## SSRF Attack Flow

```text
Attacker
   ↓
Vulnerable Application
   ↓
Server Makes Request
   ↓
Internal / Restricted Resource
```

The key issue is that the server may have network access that the attacker does not.

## Why SSRF Is Dangerous

Depending on the environment, SSRF can potentially expose:

* Internal services
* Administrative interfaces
* Cloud metadata services
* Internal APIs
* Sensitive information

The impact depends heavily on the application's network environment and permissions.

## Testing Methodology

A controlled SSRF assessment generally involves:

1. Identifying functionality that makes server-side requests.
2. Determining whether the destination can be controlled.
3. Testing how the application validates URLs.
4. Observing differences in responses.
5. Determining what resources the server can reach.

Burp Suite can be used to intercept and modify requests during authorized testing.

## Mitigation

Developers can reduce SSRF risk through:

* Strict allowlists
* URL parsing and validation
* Blocking access to internal networks
* Restricting outbound network access
* Network segmentation
* Cloud metadata service protections
* Avoiding unnecessary server-side URL fetching

## Key Takeaways

SSRF demonstrates why applications should never blindly trust user-controlled URLs.

The security boundary is not only the application itself—the server's network position and permissions can significantly increase the impact of the vulnerability.

## Conclusion

Understanding SSRF is important for both penetration testers and defenders because the vulnerability sits at the intersection of application security and network security.
