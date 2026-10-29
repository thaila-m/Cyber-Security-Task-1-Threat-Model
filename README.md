# Cyber Security Task 1: Threat Model and Secure Configuration

**Track:** Cyber Security  
**Level:** Beginner  
**Project:** Student Portal Web Application  
**Purpose:** Threat modelling and defensive security hardening

## 1. Introduction

A Student Portal is a web application that allows students to log in, view their profiles, check marks, and access academic information.

The application stores important information such as student names, login credentials, and marks. Therefore, it must protect data from unauthorized access, modification, and loss.

The objective of this project is to identify possible security threats, assess their risks, and recommend security measures to protect the application.

## 2. System Components and Assets

### Users
- Student: Logs in and views personal academic information.
- Administrator: Manages student accounts and academic records.
- Web Server: Processes requests and provides application services.
- Database: Stores user accounts, profiles, and marks.
- External Identity Service: Optionally provides secure authentication.

### Important Assets
1. Student personal information.
2. Usernames and password hashes.
3. Academic records and marks.
4. Administrator privileges.
5. Application configuration and security logs.
6. Availability of the student portal.

### Trust Boundaries
- Internet to web application: Untrusted external requests enter the application.
- Web application to database: The application accesses stored information using restricted database permissions.
- Application to external identity service: Authentication requests cross an external service boundary.

## 3. Architecture Diagram

The following diagram represents the proposed application architecture.

```mermaid
flowchart TD
    S[Student] -->|HTTPS| WAF[Web Security Layer]
    A[Administrator] -->|HTTPS + MFA| WAF
    WAF -->|Validated Requests| APP[Web Application]
    APP -->|Restricted DB Connection| DB[(Student Database)]
    APP -->|Authentication Request| IDP[External Identity Service]
    APP -->|Security Events| LOG[Centralized Security Logs]

    subgraph TB1["Trust Boundary 1: Public Internet"]
        S
        A
    end

    subgraph TB2["Trust Boundary 2: Application Environment"]
        WAF
        APP
    end

    subgraph TB3["Trust Boundary 3: Protected Data"]
        DB
    end
```

**Diagram explanation:** Students and administrators access the portal through HTTPS. The web security layer filters incoming requests. The application validates requests and accesses the database through a restricted connection. An external identity service may assist with authentication, while security events are recorded in centralized logs.

The trust boundaries identify where data moves between systems with different levels of trust.

## 4. STRIDE Threat Modelling

STRIDE is a method used to identify six categories of security threats.

| STRIDE category | Possible threat | Security protection |
|---|---|---|
| Spoofing | An attacker uses stolen student credentials. | Strong password hashing, rate limiting, and multi-factor authentication for administrators. |
| Tampering | An attacker changes marks without permission. | Server-side authorization, input validation, and database permissions. |
| Repudiation | A user denies changing an academic record. | Timestamped audit logs recording important actions and user identities. |
| Information Disclosure | A student accesses another student's marks. | Object-level access checks, HTTPS, and restricted database access. |
| Denial of Service | Excessive requests make the portal unavailable. | Rate limiting, request-size limits, monitoring, and suitable infrastructure protections. |
| Elevation of Privilege | A student gains administrator privileges. | Role-based access control, least privilege, and server-side permission checks. |

These are potential threats for the proposed system, not vulnerabilities discovered through live testing.

## 5. Risk Assessment and Prioritization

Risk is assessed using two factors:

- Likelihood: 1 = Rare, 2 = Unlikely, 3 = Possible, 4 = Likely, 5 = Very likely.
- Impact: 1 = Negligible, 2 = Minor, 3 = Moderate, 4 = Major, 5 = Critical.

**Risk score = Likelihood × Impact**

For this assessment:
- 1–4: Low risk
- 5–9: Medium risk
- 10–15: High risk
- 16–25: Critical risk

| ID | Risk scenario | Likelihood | Impact | Score | Priority |
|---|---|---:|---:|---:|---|
| R1 | Unauthorized access to another student's records | 4 | 5 | 20 | Critical |
| R2 | Compromised administrator account | 3 | 5 | 15 | High |
| R3 | Unauthorized modification of marks | 3 | 5 | 15 | High |
| R4 | Exposure of data during transmission | 3 | 4 | 12 | High |
| R5 | Excessive requests disrupt portal availability | 3 | 4 | 12 | High |
| R6 | Missing audit records prevent investigation | 3 | 3 | 9 | Medium |

These scores are illustrative estimates for a beginner threat model. They should be revised using actual system information and evidence.

## 6. Prioritized Security Hardening Plan

### Priority 1: Protect student records
- Verify authorization on the server for every request to access a student's records.
- Ensure students can access only their own information.
- Restrict database permissions to the minimum required.
- Use parameterized database queries to reduce SQL injection risk.

### Priority 2: Secure authentication and administrator access
- Store passwords using a suitable password-hashing algorithm such as Argon2id or bcrypt.
- Require multi-factor authentication for administrators.
- Apply rate limiting to login attempts.
- Use secure session cookies with HttpOnly, Secure, and appropriate SameSite settings.
- Expire sessions and invalidate them after logout.

### Priority 3: Protect data and communications
- Enforce HTTPS and redirect HTTP requests to HTTPS.
- Keep application secrets outside source code.
- Encrypt sensitive stored data where appropriate and protect encryption keys.
- Validate input on the server and encode output appropriately.
- Configure suitable security headers, including Content-Security-Policy.

### Priority 4: Improve availability and monitoring
- Apply request-size limits and rate limiting.
- Monitor repeated failed logins and unusual access patterns.
- Record important security events without logging passwords, session tokens, or unnecessary sensitive data.
- Back up the database regularly and test restoration.
- Apply security updates to the application, libraries, and server.

## 7. Hardening Checklist and Verification Steps

The following checklist can be used during a defensive security review.

- [ ] **HTTPS:** Confirm the site uses HTTPS and its TLS certificate is valid.
- [ ] **Authorization:** Using two test student accounts, verify that one cannot access the other's records.
- [ ] **Administrator access:** Confirm that administrative actions require the correct role and that administrator MFA is enabled.
- [ ] **Password storage:** Review the authentication configuration to confirm passwords are securely hashed, not stored as plain text.
- [ ] **Input handling:** Review database queries for parameterization and test input validation in an authorized test environment.
- [ ] **Session security:** Inspect cookie settings and confirm logout invalidates the session.
- [ ] **Security headers:** Inspect HTTP response headers and verify the intended security policies.
- [ ] **Logging:** Generate a test login failure and verify that the event is recorded without exposing credentials.
- [ ] **Backups:** Confirm backups exist and document the result of a test restoration.
- [ ] **Updates:** Review dependency and server versions for available security updates.

**Verification status:** These checks are proposed verification steps. They must be performed on an actual test environment before being marked as passed. No live application has been tested as part of this report.

## 8. Implementation Order

1. Fix authorization weaknesses that could expose student records.
2. Secure administrator accounts and authentication.
3. Enforce HTTPS and improve session security.
4. Review database access and input handling.
5. Configure logging, monitoring, backups, and updates.
6. Repeat the verification checklist after implementing changes.

This order prioritizes protection of sensitive academic records and privileged accounts before additional hardening tasks.

## 9. Conclusion

This project demonstrates how threat modelling can identify security risks in a student portal before they are exploited. The STRIDE method was used to identify potential threats, and likelihood and impact scores were used to prioritize them.

The proposed hardening plan focuses on authorization, authentication, secure communications, database protection, logging, and availability. The verification checklist provides practical steps for reviewing these controls.

Threat modelling is an ongoing process. The model should be updated whenever the application's architecture, features, or security requirements change.

**Project type:** Defensive cybersecurity assessment  
**Scope:** Hypothetical student portal  
**Testing status:** Proposed controls and verification steps; no live testing performed.
## 10. Defensive Lab Brief and Incident Analysis

### 10.1 Lab Environment

The assessment uses a fictional Student Portal Web Application running only on localhost or an isolated virtual machine. All testing must remain within the authorized lab environment. External systems must not be scanned.

IP addresses and authentication events used for analysis should be synthetic or appropriately anonymized. Secrets and personal information must be redacted from screenshots and reports.

### 10.2 Authentication-Log Analysis

**Objective:** Establish a normal authentication baseline, identify clusters of failed logins, distinguish likely user mistakes from suspicious activity, and recommend suitable defensive controls.

#### Analysis Procedure

1. **Establish a baseline:** Count successful and failed login events and examine their distribution over time.
2. **Identify failure clusters:** Group failed attempts by timestamp, source IP address, and account identifier.
3. **Classify patterns:** A small number of isolated failures may indicate a mistyped password. Repeated failures against one account or attempts against many accounts in a short period may indicate suspicious activity.
4. **Build a timeline:** Record the first and last event times, the number of failures, related successful logins, and any unusual changes.
5. **Recommend controls:** Introduce rate limiting, progressive delays, appropriate account protections, alerts, and centralized authentication logging.

These are analysis steps. Actual findings must be based on the supplied synthetic CSV rather than assumptions.

### 10.3 Incident Timeline Template

| Time | Event | Source | Observation | Assessment |
|---|---|---|---|---|
| To be determined | Failed login | Synthetic source IP | Record the number of failures | Pending analysis |
| To be determined | Successful login | Synthetic source IP | Check for preceding failures | Pending analysis |
| To be determined | Repeated failures | Synthetic source IP | Examine frequency and targeted accounts | Pending analysis |

### 10.4 Detection and Rate-Limit Recommendations

- Monitor repeated failed login attempts within a defined time window.
- Alert on repeated failures across multiple accounts from the same source.
- Investigate successful logins that follow unusual failure patterns.
- Apply server-side rate limiting and progressive delays.
- Use multi-factor authentication for administrator accounts.
- Avoid permanent account lockouts that could be abused to deny service.
- Record timestamps, outcomes, and relevant source information without logging passwords or session tokens.

Thresholds should be selected using the normal login baseline and adjusted to reduce false positives.

## 11. Threat-Model Assessment Template

| Field | Details to record |
|---|---|
| Asset and owner | Identify the protected asset and responsible owner. |
| Trust boundary and entry point | Identify where requests enter the system and cross trust boundaries. |
| Threat scenario and precondition | Describe the possible attack and what must happen for it to succeed. |
| Existing control and evidence | Record current safeguards and evidence that they work. |
| Risk rating with rationale | Rate likelihood and impact, explaining the reason for each rating. |
| Recommended mitigation and verification | Describe the fix and how it can be tested safely. |

### Example Threat Assessment

**Asset and owner:** Student academic records; portal administrator.

**Trust boundary and entry point:** Student-facing web application and its database access boundary.

**Threat scenario and precondition:** A student changes a record identifier in a request and attempts to view another student's marks. The issue could occur if the application does not check authorization for each record.

**Existing control and evidence:** Not yet verified. The application should enforce server-side authorization checks.

**Risk rating with rationale:** High potential impact because academic records may be disclosed. Likelihood requires assessment using the application's implementation and test evidence.

**Recommended mitigation and verification:** Enforce server-side ownership checks for every record request. In an authorized test environment, use two test student accounts and verify that neither account can access the other's records.

## 12. Conclusion

This defensive lab extends the threat model with authentication-log analysis and a structured incident assessment. Establishing a baseline, reviewing suspicious failure patterns, recording a timeline, and verifying defensive controls helps improve the security of the fictional Student Portal.

All findings must be supported by evidence from the authorized lab. No external systems should be scanned, and secrets or personal information must not appear in submitted evidence.
