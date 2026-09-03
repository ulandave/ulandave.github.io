---

layout: post
title: "EchoCTF: Apache 2.4.49 Path Traversal to Root"
date: 2026-09-03 20:00:00 +0000
categories: [CTF, Exploitation]
tags: [apache, path-traversal, ssh, privilege-escalation, linux, echocTF]
-------------------------------------------------------------------------

## Overview

This writeup documents the compromise of a vulnerable Apache HTTP Server 2.4.49 instance.

The challenge demonstrated how an attacker can chain multiple weaknesses together to move from remote file disclosure to SSH access and ultimately root privileges.

### Attack Chain

```text
Apache 2.4.49
      ↓
Path Traversal
      ↓
Arbitrary File Read
      ↓
SSH Private Key
      ↓
SSH Access
      ↓
sudo Enumeration
      ↓
/bin/cpio
      ↓
Root
```

## 1. Reconnaissance

The first stage was service enumeration.

The target was found to be running:

```text
Apache HTTP Server 2.4.49
```

The version immediately stood out because Apache 2.4.49 is associated with a well-known path traversal and file disclosure vulnerability.

## 2. Path Traversal

The vulnerable web server was tested for directory traversal.

The vulnerability allowed files outside the normal web directory to be accessed.

One of the initial files examined was:

```text
/etc/passwd
```

This provided information about users present on the system.

## 3. Sensitive File Discovery

After confirming arbitrary file disclosure, further enumeration was performed to identify potentially sensitive files.

The investigation eventually led to the discovery of an SSH private key associated with a local user.

The key was saved locally and its permissions were restricted:

```bash
chmod 600 id_rsa
```

## 4. SSH Access

The recovered private key was then used to authenticate to the target through SSH.

This provided the initial shell access required for local enumeration.

## 5. Local Enumeration

Once inside the system, the next objective was determining what privileges were available.

The following command was used:

```bash
sudo -l
```

The output revealed that `/bin/cpio` could be executed with elevated privileges.

## 6. Privilege Escalation

The privileged `cpio` binary was abused to obtain execution with root-level privileges.

This demonstrates why administrators should carefully review binaries permitted through `sudo`.

A single incorrectly configured privileged utility can turn a limited user account into complete system compromise.

## 7. Root

After successfully exploiting the sudo configuration, root-level access was obtained.

Privilege escalation was verified using:

```bash
id
```

The resulting privileges confirmed successful root access.

## Attack Summary

| Stage                  | Technique             |
| ---------------------- | --------------------- |
| Reconnaissance         | Service enumeration   |
| Initial vulnerability  | Apache path traversal |
| Information disclosure | Arbitrary file read   |
| Credential access      | SSH private key       |
| Initial access         | SSH                   |
| Enumeration            | `sudo -l`             |
| Privilege escalation   | `/bin/cpio`           |
| Final access           | Root                  |

## Lessons Learned

This challenge demonstrated the importance of:

* Keeping internet-facing software patched.
* Restricting access to sensitive files.
* Protecting SSH private keys.
* Reviewing `sudo` configurations.
* Applying the principle of least privilege.
* Performing thorough enumeration after obtaining initial access.

## Conclusion

The system was compromised by chaining several weaknesses together:

**Apache 2.4.49 → Path Traversal → File Disclosure → SSH Key → SSH → sudo → cpio → Root**

The challenge was a good demonstration of how an initial web vulnerability can eventually result in complete system compromise.
