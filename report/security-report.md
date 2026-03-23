# UmojaAgri API Security Assessment Report

**Prepared by:** Amusa Rotimi Mariam  
**Role:** Security Tester | Women Techsters Cybersecurity Fellow  
**Target:** https://umojaagri-org.onrender.com  
**Date:** March 2026  
**Classification:** Confidential  

---

## 1. Executive Summary

This report presents the findings of a security assessment conducted on the UmojaAgri API as part of the Women Techsters Cybersecurity Capstone Project. UmojaAgri is a Flutter-based agricultural marketplace platform connecting farmers, sellers, and buyers across Nigeria.

Testing was conducted using Postman on Linux and browser-based developer tools. Five security vulnerabilities were identified ranging from High to Medium severity, along with one positive security control. Immediate remediation is recommended for all High severity findings.

---

## 2. Scope & Methodology

| Item | Detail |
|------|--------|
| Frontend URL | https://umojaagri-org-1.onrender.com |
| Backend API URL | https://umojaagri-org.onrender.com |
| Testing Period | March 2026 |
| Testing Type | Black Box / Grey Box API Security Testing |
| Tools Used | Postman, Browser DevTools, JWT.io |
| Tester | Amusa Rotimi Mariam |

### Methodology
- Endpoint Discovery — identifying exposed API routes through path probing and network traffic analysis
- Authentication Testing — testing registration, login, and token-based access controls
- IDOR Testing — attempting to access other users data using valid but unauthorized tokens
- Error Handling Analysis — reviewing API error responses for sensitive information disclosure
- Frontend Analysis — reviewing compiled Flutter JavaScript for exposed secrets

---

## 3. Findings Summary

| # | Finding | Severity | Endpoint |
|---|---------|----------|----------|
| F-01 | Verbose Error Disclosure | 🔴 High | POST /api/users/register |
| F-02 | User Enumeration via Login | 🔴 High | POST /api/users/login |
| F-03 | No Rate Limiting on Login | 🔴 High | POST /api/users/login |
| F-04 | Weak Password Policy | 🟡 Medium | POST /api/users/register |
| F-05 | Role ID Exposed in Frontend JS | 🟡 Medium | Frontend JavaScript |
| P-01 | Password Hashing with bcrypt | ✅ Positive | POST /api/users/register |

---

## 4. Detailed Findings

### F-01 | Verbose Error Disclosure — HIGH

| Field | Detail |
|-------|--------|
| Endpoint | POST /api/users/register |
| Severity | High |
| OWASP | A05:2021 Security Misconfiguration |

**Description:**  
The registration endpoint returns raw Prisma ORM error messages exposing internal database schema, field names, and server architecture.

**Evidence:**  
```
"Invalid `prisma.user.create()` invocation" — exposing fields: 
email, password, name, location, phone, roleId
```

**Impact:**  
An attacker can use this information to understand the database structure and craft targeted attacks.

**Recommendation:**  
- Implement a global error handler that returns generic messages
- Log detailed errors server-side only
- Return: `{"error": "Something went wrong. Please try again."}`

---

### F-02 | User Enumeration via Login — HIGH

| Field | Detail |
|-------|--------|
| Endpoint | POST /api/users/login |
| Severity | High |
| OWASP | A07:2021 Identification and Authentication Failures |

**Description:**  
The login endpoint returns different error messages depending on whether an email is registered, allowing attackers to enumerate valid accounts.

**Evidence:**  
```
Response for unregistered email:
{"error": "User is not registered. Please sign up first"}
```

**Impact:**  
Attackers can compile a list of valid registered emails for phishing or targeted password attacks against farmers and buyers.

**Recommendation:**  
- Return a single generic message for all failed logins
- `{"error": "Invalid email or password"}`
- Never distinguish between wrong email and wrong password

---

### F-03 | No Rate Limiting on Login — HIGH

| Field | Detail |
|-------|--------|
| Endpoint | POST /api/users/login |
| Severity | High |
| OWASP | A07:2021 Identification and Authentication Failures |

**Description:**  
The login endpoint does not implement rate limiting or account lockout. An attacker can make unlimited login attempts without being blocked.

**Evidence:**  
Multiple rapid requests were made to the login endpoint with no HTTP 429 response returned at any point.

**Impact:**  
Platform is vulnerable to brute force and password spraying attacks. Combined with F-02, an attacker can confirm valid emails then systematically guess passwords.

**Recommendation:**  
- Limit to 5 failed attempts per IP per minute
- Lock account after 10 consecutive failures
- Use express-rate-limit for Node.js

---

### F-04 | Weak Password Policy — MEDIUM

| Field | Detail |
|-------|--------|
| Endpoint | POST /api/users/register |
| Severity | Medium |
| OWASP | A07:2021 Identification and Authentication Failures |

**Description:**  
The application accepts extremely weak passwords. Test accounts were created using "password123".

**Impact:**  
User accounts are at risk of being compromised through dictionary attacks, especially combined with no rate limiting.

**Recommendation:**  
- Enforce minimum 8 characters
- Require uppercase, number, and special character
- Reject passwords on common password lists

---

### F-05 | Role ID Exposed in Frontend JavaScript — MEDIUM

| Field | Detail |
|-------|--------|
| Location | main.dart.js (compiled Flutter frontend) |
| Severity | Medium |
| OWASP | A02:2021 Cryptographic Failures |

**Description:**  
A valid role UUID was found hardcoded in the compiled Flutter JavaScript file used during user registration.

**Evidence:**  
```
Role UUID found in main.dart.js:
fa27a48e-1efa-4ce9-8643-7c06e75232a1
```

**Impact:**  
Anyone can extract this value and use it in API requests, undermining role-based access control.

**Recommendation:**  
- Never hardcode role IDs in frontend code
- Create a public endpoint to return available roles
- Validate all role assignments server-side

---

### P-01 | Password Hashing with bcrypt — POSITIVE FINDING ✅

The application correctly hashes passwords using bcrypt before storing them. This protects user passwords in the event of a database breach and is considered best practice.

---

## 5. Endpoints Discovered

| Method | Endpoint | Auth Required | Status |
|--------|----------|---------------|--------|
| POST | /api/users/register | No | Active |
| POST | /api/users/login | No | Active |
| GET | /api/users/profile | Bearer Token | Active |
| GET | /api/users/me | Bearer Token | Active |
| GET | /api/produces/me | Bearer Token | Active |
| GET | /api/shipments | Bearer Token | Active |
| GET | /api/finance/wallet | Bearer Token | Active |
| GET | /api/finance/transactions | Bearer Token | Active |

---

## 6. Conclusion

The UmojaAgri API security assessment identified five vulnerabilities across authentication and registration flows. Three findings are rated High severity and require immediate remediation before the platform is opened to real users.

The most critical issues are verbose error disclosure and the lack of rate limiting on the login endpoint, which together could enable an attacker to enumerate users and systematically compromise accounts.

Positively, the application correctly implements bcrypt password hashing. Addressing the identified findings will significantly improve the platform's security posture. A follow-up assessment is recommended after fixes are implemented.

---

## 7. References

- OWASP Top 10 2021 — https://owasp.org/Top10/
- OWASP API Security Top 10 — https://owasp.org/www-project-api-security/
- Testing Repo — https://github.com/rotimimariam9-dev/umojaagri-api-security-testing
- Women Techsters Fellowship — Cybersecurity Track Capstone Project
