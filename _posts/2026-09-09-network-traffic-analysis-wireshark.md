---

layout: post
title: "Network Traffic Analysis with Wireshark"
date: 2026-09-09 20:00:00 +0000
categories: [Blue Team, Network Security]
tags: [wireshark, pcap, network-analysis, soc, incident-response]
-----------------------------------------------------------------

## Introduction

Network traffic analysis is an important skill for security analysts.

When investigating a suspicious host or security incident, packet captures can provide evidence about what happened on the network.

**Wireshark** is one of the most useful tools for examining this traffic.

## What Is a PCAP?

A PCAP file contains captured network packets.

A security analyst can use a packet capture to investigate:

* Source IP addresses
* Destination IP addresses
* Ports
* Protocols
* DNS queries
* HTTP traffic
* TCP connections
* Suspicious communication patterns

## Investigation Workflow

My basic workflow is:

```text
PCAP
 ↓
Identify Hosts
 ↓
Identify Protocols
 ↓
Inspect Conversations
 ↓
Follow Suspicious Streams
 ↓
Extract Indicators
 ↓
Correlate Activity
 ↓
Determine Impact
```

## Step 1 — Identify Conversations

Wireshark provides useful statistics for understanding which hosts are communicating.

Important questions include:

* Which internal host generated the traffic?
* Which external IP did it contact?
* How frequently did communication occur?
* Which ports were involved?

## Step 2 — Analyze Protocols

Protocol analysis can reveal unexpected behavior.

For example:

```text
DNS
HTTP
HTTPS
TCP
UDP
ICMP
```

The analyst should investigate protocols that appear unusual for the affected host.

## Step 3 — Follow TCP Streams

When investigating suspicious TCP communication, following a stream can help reconstruct the conversation.

This can reveal:

* Requests
* Responses
* URLs
* Commands
* Headers
* Transmitted data

The visibility depends on whether the traffic is encrypted.

## Step 4 — Identify Indicators

Potential indicators include:

* Suspicious IP addresses
* Domains
* URLs
* User agents
* File hashes
* Unusual ports
* Repeated connection attempts

These indicators can then be correlated with other security telemetry.

## SOC Perspective

From a SOC analyst perspective, packet analysis should not be performed in isolation.

A suspicious connection should be correlated with:

* Endpoint activity
* Authentication logs
* DNS logs
* Firewall logs
* Proxy logs
* Process execution

This helps establish whether the traffic represents malicious activity or legitimate behavior.

## Key Takeaways

Wireshark is more than a packet viewer.

It can help analysts answer:

**Who communicated?**

**With whom?**

**Using what protocol?**

**When did it happen?**

**What was transmitted?**

**Does the activity indicate compromise?**

## Conclusion

Network traffic analysis is an essential skill for security analysts because network evidence can help reconstruct events and identify indicators of compromise.

Regular practice with PCAP files is an effective way to improve this skill.
