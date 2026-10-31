# INE – Web Application Penetration Testing CTF 1

## Overview

This repository documents my solution to the **Web Application Penetration Testing CTF 1** Skill Check Lab from INE.

The lab focused on identifying and exploiting common web application vulnerabilities to retrieve four hidden flags.

The assessment involved file disclosure through a vulnerable file viewer, directory enumeration, HTTP authentication testing, and SQL injection.

## Lab Environment

- **Platform:** INE
- **Operating System:** Kali Linux
- **Target:** `http://target.ine.local`

## Flags and Objectives

| Flag | Objective | Technique |
|---|---|---|
| Flag 1 | Retrieve `/flag.txt` | Arbitrary file read |
| Flag 2 | Discover hidden files in server directories | Directory enumeration |
| Flag 3 | Identify weak login credentials | HTTP brute-forcing |
| Flag 4 | Access the admin account without its password | SQL injection |

## Tools Used

- DIRB
- Hydra
- Firefox
- Kali Linux

## Repository Structure

```text
INE-Web-Application-Penetration-Testing-CTF-1/
│
├── README.md
└── lab-notes.md
```

## Detailed Walkthrough

The complete command history, explanations, browser interactions, findings, and flag retrieval steps are documented in:

**[lab-notes.md](lab-notes.md)**

## Key Learning Outcomes

- Identifying arbitrary file-read vulnerabilities.
- Enumerating hidden web directories using DIRB.
- Testing HTTP login forms using Hydra.
- Understanding weak authentication mechanisms.
- Identifying SQL injection vulnerabilities.
- Understanding authentication bypass through SQL comments.
- Combining manual browser testing with automated security tools.

## Disclaimer

This repository is intended for educational purposes and documents activities performed in an authorized INE lab environment.

All techniques should only be used against systems for which explicit permission has been granted.
