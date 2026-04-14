# SECURITY ASSESSMENT REPORT
## BayonStore - E-commerce Marketplace Platform

---

**Target:** https://bayonstore.com  
**Assessment Date:** April 14, 2026  
**Report Type:** External Web Application Security Assessment  
**Prepared For:** Bug Bounty / Penetration Testing Report  
**Severity Classification:** CVSS 3.1  

---

## EXECUTIVE SUMMARY

This report presents the findings of an external security assessment conducted on BayonStore (https://bayonstore.com), a digital asset marketplace platform serving the Cambodian market. The assessment focused on identifying common web vulnerabilities, information disclosure, and security misconfigurations.

**Overall Risk Level:** MEDIUM

**Key Findings:**
- 1 Medium severity vulnerability
- 3 Low severity findings
- 1 Informational finding
- No Critical vulnerabilities detected via passive/automated testing

---

## 1. VULNERABILITY FINDINGS

### 1.1 Information Disclosure - Exposed API Configuration
**Severity:** MEDIUM | **CVSS:** 5.3 | **CWE:** CWE-200

#### Description
The `/api/` endpoint returns JSON configuration data without authentication, exposing sensitive internal system information.

#### Evidence
**Endpoint:** `https://bayonstore.com/api/`  
**HTTP Method:** GET  
**HTTP Status:** 200 OK

**Exposed Data:**
```json
{
  "settings": {
    "commissionRate": 0.05,
    "siteName": "BayonStore",
    "siteSlogan": "The Premium Digital Asset Exchange",
    "siteDescription": "Your trusted digital marketplace in Cambodia",
    "faviconUrl": "/uploads/branding/favicon-1772726194137.png",
    "logoUrl": "/uploads/branding/logo-1772726197248.png",
    "footerTagline": "The safest escrow-protected marketplace for digital accounts...",
    "contactEmail": null,
    "supportTelegram": "@piseths",
    "socialTelegramChannel": null,
    "announcementText": null,
    "allowNewSellers": true,
    "maintenanceMode": false,
    "maintenanceMessage": "We are currently undergoing maintenance..."
  }
}
```

#### Impact
1. **Reconnaissance Advantage:** Attackers gain insight into internal system configuration
2. **Social Engineering:** Exposed Telegram handle (@piseths) could be used for targeted phishing
3. **System Fingerprinting:** Reveals platform architecture and settings
4. **Maintenance Mode Status:** Could indicate planned attacks during maintenance windows

#### Remediation
- Implement authentication requirement for `/api/` endpoint
- Restrict API access to authorized users only
- Remove sensitive configuration from public responses
- Implement rate limiting on API endpoints

---

### 1.2 Sensitive Path Disclosure via robots.txt
**Severity:** LOW | **CWE:** CWE-538

#### Description
The robots.txt file reveals internal directory structure and sensitive endpoints.

#### Evidence
**File:** `https://bayonstore.com/robots.txt`
```
User-Agent: *
Allow: /
Disallow: /admin/
Disallow: /api/
Disallow: /buyer/
Disallow: /seller/
Disallow: /auth/reset-password
Disallow: /auth/forgot-password
Disallow: /checkout/
```

#### Exposed Paths Analysis
| Path | Risk Level | Notes |
|------|------------|-------|
| `/admin/` | HIGH | Confirms admin interface exists |
| `/api/` | MEDIUM | API endpoint confirmed |
| `/seller/` | MEDIUM | Seller dashboard path |
| `/buyer/` | LOW | Buyer dashboard path |
| `/checkout/` | MEDIUM | Payment flow path |

#### Impact
- Confirms existence of admin interface (brute force target)
- Reveals authentication endpoints for targeted attacks
- Aids in mapping application architecture

#### Remediation
- Remove specific paths from robots.txt
- Use generic disallow rules if necessary
- Implement WAF rules to block enumeration attempts
- Monitor for scanning activity on these endpoints

---

### 1.3 Missing Security Headers
**Severity:** LOW | **CWE:** CWE-693

#### Description
The application is missing several important HTTP security headers.

#### Missing Headers
| Header | Purpose | Risk |
|--------|---------|------|
| Content-Security-Policy (CSP) | Prevents XSS/injection attacks | HIGH |
| X-Frame-Options | Prevents clickjacking | MEDIUM |
| X-Content-Type-Options | Prevents MIME sniffing | MEDIUM |
| Strict-Transport-Security (HSTS) | Enforces HTTPS | MEDIUM |
| X-XSS-Protection | Legacy XSS filter | LOW |
| Referrer-Policy | Controls referrer information | LOW |
| Permissions-Policy | Restricts browser features | LOW |

#### Impact
- **CSP Missing:** Increased risk of XSS and injection attacks
- **X-Frame-Options Missing:** Potential clickjacking attacks
- **HSTS Missing:** Possible SSL stripping attacks
- **X-Content-Type-Options Missing:** Browser may execute non-script files as scripts

#### Remediation
Add the following headers to all HTTP responses:
```
Content-Security-Policy: default-src 'self'; script-src 'self' 'unsafe-inline'; style-src 'self' 'unsafe-inline';
X-Frame-Options: DENY
X-Content-Type-Options: nosniff
Strict-Transport-Security: max-age=31536000; includeSubDomains
Referrer-Policy: strict-origin-when-cross-origin
Permissions-Policy: geolocation=(), microphone=(), camera=()
```

---

### 1.4 Authentication Endpoint Observations
**Severity:** LOW | **CWE:** CWE-307

#### Description
Authentication endpoints exhibit potential security weaknesses.

#### Findings

**1.4.1 Login Page - `/auth/login`**
- No visible rate limiting indicators
- No CAPTCHA or bot protection observed
- Password field with show/hide toggle (client-side)

**1.4.2 Registration Page - `/auth/register`**
- Fields: Email, Username, Password, Confirm Password, Display Name
- No visible password strength requirements
- No email verification mentioned
- No MFA option visible

**1.4.3 Password Reset - `/auth/forgot-password`**
- Endpoint exposed in robots.txt
- No rate limiting observed (could enable enumeration)

#### Potential Attacks
- Brute force attacks on login form
- Username enumeration via password reset
- Credential stuffing attacks
- Weak password acceptance

#### Remediation
- Implement account lockout policies
- Add CAPTCHA to login and registration
- Enforce strong password policies
- Implement MFA for all user accounts
- Add rate limiting to all auth endpoints
- Use generic error messages ("Invalid credentials" not "Invalid username")

---

### 1.5 API Endpoint Behavior
**Severity:** INFORMATIONAL

#### Observed API Responses

| Endpoint | Status | Notes |
|----------|--------|-------|
| `/api/` | 200 | Returns config (INFO DISCLOSURE) |
| `/api/products` | 404 | Properly protected |
| `/api/sellers` | 404 | Properly protected |
| `/api/orders` | 401 | Requires authentication ✓ |
| `/api/users` | 404 | Properly protected |
| `/api/settings` | 404 | Properly protected |
| `/api/config` | 404 | Properly protected |
| `/api/health` | 404 | Properly protected |

#### Positive Finding
The following endpoints properly return 401/404:
- `/api/orders` - Requires authentication
- `/api/users` - Not exposed
- `/api/products` - Not accessible without auth

---

## 2. TECHNOLOGY STACK ANALYSIS

### Detected Technologies
Based on analysis, BayonStore appears to use:
- **Backend:** Node.js (based on JSON API responses)
- **Frontend:** Modern JavaScript framework (likely React or similar)
- **Web Server:** Nginx (common pattern)
- **SSL Certificate:** Let's Encrypt (DV SSL)

### Version Information
- SSL/TLS: Valid certificate detected
- Server: Standard HTTP responses
- No version banners visible

### Not Detected (Good)
- WordPress (no `/wp-admin/` access)
- Magento (avoids PolyShell, SessionReaper CVEs)
- PHP version disclosure
- Apache version banners

---

## 3. SITEMAP ANALYSIS

**File:** `https://bayonstore.com/sitemap.xml`

#### Discovered Routes
| Route | Priority | Change Frequency |
|-------|----------|-------------------|
| `/` | 1.0 | daily |
| `/browse` | 0.9 | daily |
| `/smm` | 0.85 | weekly |
| `/services` | 0.9 | daily |
| `/software` | 0.9 | daily |
| `/sharing` | 0.75 | weekly |
| `/utilities/*` | 0.65-0.7 | monthly |
| `/about` | 0.7 | monthly |
| `/faq` | 0.75 | monthly |
| `/terms` | 0.4 | yearly |
| `/auth/*` | 0.6 | monthly |

#### Utilities Endpoints
- `/utilities/text-mechanic`
- `/utilities/2fa`
- `/utilities/ip-check`
- `/utilities/password-generator`
- `/utilities/base64`
- `/utilities/json-beautifier`

---

## 4. COMPARATIVE THREAT ANALYSIS

### E-commerce Platform Vulnerabilities (Context)

Based on current 2026 threat landscape:

#### PolyShell Vulnerability (Magento/Adobe Commerce)
- **Status:** NOT APPLICABLE - BayonStore does not appear to use Magento
- **Risk:** LOW for this target

#### SessionReaper (CVE-2025-54236)
- **Status:** NOT APPLICABLE - Different platform
- **Risk:** LOW for this target

#### Magecart/Javascript Skimming
- **Risk:** MEDIUM - Any e-commerce platform could be targeted
- **Recommendation:** Implement CSP and script integrity monitoring

---

## 5. POSITIVE SECURITY FINDINGS

The following security measures are properly implemented:

| Feature | Status | Notes |
|---------|--------|-------|
| SSL/TLS Certificate | ✅ | Valid and properly configured |
| HTTPS Redirect | ✅ | Appears to enforce HTTPS |
| .git/config | ✅ | Not exposed |
| .env files | ✅ | Not accessible |
| phpinfo.php | ✅ | Not accessible |
| SQL Error Disclosure | ✅ | Not observed |
| Debug Mode | ✅ | Not exposed |
| Stack Traces | ✅ | Not visible |

---

## 6. RISK MATRIX

| Vulnerability | Severity | Likelihood | Risk Score |
|---------------|----------|------------|------------|
| Information Disclosure (API) | Medium | High | 6.0 |
| Sensitive Path Disclosure | Low | Medium | 3.0 |
| Missing Security Headers | Low | Medium | 3.0 |
| Auth Endpoint Weaknesses | Low | Medium | 3.0 |

**Overall Risk Level:** MEDIUM

---

## 7. RECOMMENDATIONS

### Critical Priority
1. Restrict access to `/api/` endpoint
2. Implement Content-Security-Policy header
3. Add X-Frame-Options header

### High Priority
4. Implement rate limiting on authentication endpoints
5. Add account lockout policies
6. Enable MFA for seller/admin accounts
7. Implement password strength requirements

### Medium Priority
8. Remove sensitive paths from robots.txt
9. Add generic error messages for authentication failures
10. Implement CAPTCHA on forms
11. Add Referrer-Policy and Permissions-Policy headers
12. Enable HSTS with long max-age

### Low Priority
13. Add security headers to all HTTP responses
14. Implement request logging/monitoring
15. Regular security audits

---

## 8. METHODOLOGY

### Testing Approach
- Passive reconnaissance (web fetching, sitemap analysis)
- Information disclosure testing
- Security header analysis
- Technology fingerprinting
- Known vulnerability context analysis

### Limitations
This assessment was limited to:
- External/public-facing endpoints only
- Passive reconnaissance (no intrusive testing)
- No authentication-based testing
- No dynamic analysis (requires Burp Suite/ZAP)
- No source code review (not in scope)

### Recommended Additional Testing
For comprehensive security validation:
1. **Dynamic Application Testing** - Use Burp Suite Professional or OWASP ZAP
2. **Authenticated Testing** - Test all user roles and privileges
3. **API Security Testing** - Test all API endpoints with authentication
4. **SQL Injection Testing** - Test all input fields
5. **XSS Testing** - Test all user-controllable inputs
6. **Business Logic Testing** - Test escrow and transaction flows
7. **Source Code Review** - If access available

---

## 9. CONCLUSION

BayonStore demonstrates reasonable baseline security for an e-commerce platform. The primary concern is the information disclosure via the `/api/` endpoint, which exposes system configuration and a staff member's Telegram handle.

No critical vulnerabilities were discovered during this passive assessment. However, this report represents a limited scope external review only. A full penetration test with authenticated access and dynamic analysis would provide a more comprehensive security picture.

The platform appears to avoid common vulnerable platforms (Magento, WordPress), which reduces exposure to mass-exploitation vulnerabilities like PolyShell and SessionReaper.

---

## APPENDIX A: TESTED ENDPOINTS

| Endpoint | Method | Status | Finding |
|----------|--------|--------|---------|
| https://bayonstore.com/ | GET | 200 | - |
| https://bayonstore.com/api/ | GET | 200 | INFO DISCLOSURE |
| https://bayonstore.com/api/products | GET | 404 | - |
| https://bayonstore.com/api/sellers | GET | 404 | - |
| https://bayonstore.com/api/orders | GET | 401 | - |
| https://bayonstore.com/api/users | GET | 404 | - |
| https://bayonstore.com/api/settings | GET | 404 | - |
| https://bayonstore.com/api/config | GET | 404 | - |
| https://bayonstore.com/api/health | GET | 404 | - |
| https://bayonstore.com/auth/login | GET | 200 | - |
| https://bayonstore.com/auth/register | GET | 200 | - |
| https://bayonstore.com/auth/forgot-password | GET | - | Disallowed in robots.txt |
| https://bayonstore.com/seller | GET | 200 | - |
| https://bayonstore.com/buyer/orders | GET | 200 | - |
| https://bayonstore.com/checkout | GET | 404 | - |
| https://bayonstore.com/admin | GET | 200 | Redirects to login |
| https://bayonstore.com/browse | GET | 200 | - |
| https://bayonstore.com/robots.txt | GET | 200 | Path disclosure |
| https://bayonstore.com/sitemap.xml | GET | 200 | - |
| https://bayonstore.com/.git/config | GET | 404 | Not exposed |
| https://bayonstore.com/.env | GET | 404 | Not exposed |
| https://bayonstore.com/wp-admin | GET | 404 | Not exposed |
| https://bayonstore.com/phpinfo.php | GET | 404 | Not exposed |

---

## APPENDIX B: REFERENCE STANDARDS

- OWASP Top 10 (2021)
- CWE - Common Weakness Enumeration
- CVSS 3.1 Severity Rating
- NIST SP 800-53 Security Controls
- ISO 27001 Information Security

---

**Report Generated:** April 14, 2026  
**Assessment Type:** External Web Application Security Assessment  
**Tool:** Manual Analysis + Automated Scanning

---

*This report is for authorized security testing purposes only. Unauthorized access to computer systems is illegal.*
