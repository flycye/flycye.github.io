---
title: ALl About Active Directory
date: 2025-06-27
categories: ['guide']
tags: ['active-directory']
author: nefeli
---

## Introduction

## Types of Attacks

### LLMNR (Link Local MulticastName Resolution) Poisoning

- identifies hosts when DNS can’t (ex. you type `gogle.com` instead of `google.com`)
- **the problem: uses the username and password hash that can be intercepted**

We can attack this service by impersonating the server, have the user send their hash to us, and then try and crack it!

### Mitigations

LLMNR can be disabled by going to:

```
Turn off multicast name resolution (Local Computer Policy)
-> Computer Config -> Admin Templates -> Network
-> DNS Client (Group Policy Editor)
```
