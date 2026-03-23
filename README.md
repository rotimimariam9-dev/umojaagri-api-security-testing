project
# UmojaAgri API Security Assessment

![Security Testing](https://img.shields.io/badge/Security-API%20Testing-red)
![Status](https://img.shields.io/badge/Status-Completed-green)
![Tools](https://img.shields.io/badge/Tools-Postman%20%7C%20DevTools%20%7C%20JWT.io-blue)

## Overview

This repository documents a black box API security assessment conducted on **UmojaAgri** — a Flutter-based agricultural marketplace platform connecting farmers, sellers, and buyers across Nigeria.

The assessment was conducted as part of the **Women Techsters Cybersecurity Fellowship Capstone Project** by Amusa Rotimi Mariam.

---

## Target

| Item | Detail |
|------|--------|
| Application | UmojaAgri |
| Frontend | https://umojaagri-org-1.onrender.com |
| Backend API | https://umojaagri-org.onrender.com |
| Stack | Flutter (frontend), Node.js + Prisma (backend) |
| Testing Date | March 2026 |

---

## Tools Used

- **Postman** — API request testing
- **Chrome DevTools** — Network traffic analysis and frontend JS inspection
- **JWT.io** — Token decoding and analysis
- **Linux Terminal** — Testing environment

---

## Findings Summary

| # | Vulnerability | Severity |
|---|--------------|----------|
| F-01 | Verbose Error Disclosure | 🔴 High |
| F-02 | User Enumeration via Login | 🔴 High |
| F-03 | No Rate Limiting on Login | 🔴 High |
| F-04 | Weak Password Policy | 🟡 Medium |
| F-05 | Role ID Exposed in Frontend JS | 🟡 Medium |
| P-01 | Password Hashing with bcrypt | ✅ Positive |

---

## Repository Structure
```
├── report/
│   └── security-report.md    # Full detailed findings report
├── testing-notes/
│   └── testing-log.md        # Step by step testing documentation
├── screenshots/              # Evidence from testing
└── endpoints/
    └── endpoints-tested.md   # All discovered API endpoints
```

---

## Key Findings

### 🔴 Verbose Error Disclosure
The registration endpoint exposes raw Prisma ORM errors revealing internal database schema, field names, and server architecture to anyone who sends a malformed request.

### 🔴 User Enumeration
The login endpoint returns different messages for registered vs unregistered emails, allowing attackers to confirm which email addresses have accounts on the platform.

### 🔴 No Rate Limiting
The login endpoint accepts unlimited requests with no throttling or account lockout, making the platform vulnerable to brute force attacks.

### 🟡 Weak Password Policy
The application accepts weak passwords such as "password123" with no complexity requirements enforced.

### 🟡 Role ID in Frontend JS
A valid role UUID was discovered hardcoded in the compiled Flutter JavaScript, undermining role-based access control.

---

## Methodology

1. **Reconnaissance** — Identified frontend and backend URLs, discovered API stack
2. **Endpoint Discovery** — Mapped all available API routes using Postman and DevTools
3. **Authentication Testing** — Tested registration, login, and token handling
4. **IDOR Testing** — Attempted cross-user data access using valid tokens
5. **Error Analysis** — Reviewed error responses for sensitive data exposure
6. **Frontend Analysis** — Inspected compiled JavaScript for hardcoded secrets

---

## Full Report

See [report/security-report.md](report/security-report.md) for the complete findings with evidence and recommendations.

---

## About the Tester

**Amusa Rotimi Mariam**  
Women Techsters Cybersecurity Fellow | SOC Analyst in Training  
📍 Lagos, Nigeria  

---

*This assessment was conducted ethically with full authorization from the UmojaAgri development team # umojaagri-api-security-testing
project
