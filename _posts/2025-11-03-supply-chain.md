---
title: Dependency Confusion
date: 2025-11-03
categories: ['guide']
tags: ['binary-exploitation']
author: nefeli
---

## Introduction

Supply-chain attacks don't have to be groundbreaking enterprise malware. Sometimes, attackers
can pry their way in from one misconfigured file.

Are you constantly updating your packages with pip, RubyGems, npm (the list goes on)?
All that commands like these do are:
- Look for the package with that name ie. Does the repository exist?
- Install it.

As you might be thinking, this does **not** sound very sophisticated. From a security
perspective, malicious code could be pulled from any repository with the same name
straight into your environment.

## What's dependency confusion?

Dependency confusion occurs when package managers prefer a public package of the same name
over your private, internal package... oops. An attacker finds your leaked package name
(this slips past often, even `package.json` files that are built can be embedded into your
public-facing `.js` files) and your tooling favours that repo and pulls it.

Imagine what scripts and malware you would be subject to installing.

> In 2021 Alex Birsan, a security researcher and bug bounty hunter, was able to publish malicious
packages with the same name as leaked, internal packages. 
{: .prompt-info}
