# Lab: Reflected XSS via CSP Header Injection 

**Severity:** High 

**Vulnerability Type:** Reflected Cross-Site Scripting (XSS)/CSP Injection

**Target:** Web Security Academy Blog - Search Functionality 

##### 1. Executive Summary 

During the security assessment of the blog search functionality, a Reflected
Cross-Site Scripting (XSS) vulnerability was identified. 

The application attempts to mitigate XSS attacks using a Context Security 
Policy (CSP). However, the application insecurely reflects a user-controlled
parameter (`token`) directly into the CSP HTTP header. This allows an 
attacker to inject arbitrary CSP directives. By injecting the `script-src-
elem 'unsafe-inline'` directive, the attacker can override the existing 
safety controls and execute malicious JavaScript payloads reflected in the 
search results. 

##### 2. Technical Walkthrough 

2.1 Target Identification 

The target is the blog search function, which reflects the user's search 
query into the HTML response. Standard XSS payloads are blocked by the 
browser due to the presence of a CSP header. 

<img width="1186" height="746" alt="e1" src="https://github.com/user-attachments/assets/62ca1aec-b7b1-42c9-be75-f7b931d03d59" />

2.2 CSP Configuration Analysis 

An analysis of the HTTP response headers revealed a CSP configuration that 
restricts scripts to `'self'`. However, the policy also includes a `report-
uri` directive with a `token` parameter. 

```
Content-Security-Policy: default-src 'self'; ... report-uri /csp-report?token=
```

<img width="1440" height="876" alt="e3" src="https://github.com/user-attachments/assets/bcb20b3d-c22d-46ce-9750-ea4fc84b611d" />

2.3 Vulnerability Verification (Header Injection) 

To test for injection, a custom value (`123`) was passed to the `toekn` GET
parameter. The response confirmed that this input is reflected directly 
into the CSP header without sanitisation. 

<img width="1440" height="877" alt="e4" src="https://github.com/user-attachments/assets/fa2d16a2-252c-471e-85dc-ab7ec762335d" />

2.4 Exploitation 

Since we can modify the CSP header, we can inject a directive that allows 
inline scripts. The `script-src-elem` directive is more specific than 
`script-src` and controls the execution of `<script>` blocks. By setting this
to `unsafe-inline`, we tell the browser to ignore the previous restriction 
for script elements. 

The Attack Vector:

1. XSS Payload: Inject `<script>alert(1)</script>` into the `search` parameter.

2. CSP Bypass: Inject `;script-src-elem 'unsafe-inline'` into the `token` parameter.

Final Malicious URL:

```
/?search=<script>alert(1)</script>&token=;script-src-elem 'unsafe-inline'
```

The browser parses the reflected header, accepts the new "unsafe" directive, 
and executes the JavaScript found in the search results. 

<img width="1181" height="790" alt="e5" src="https://github.com/user-attachments/assets/b4a805e5-c570-49fa-a25a-481390f3dab1" />
<img width="1183" height="744" alt="e6" src="https://github.com/user-attachments/assets/253786d2-7d43-4ee3-a037-889156006eae" />

##### 3. Impact Analysis 

The impact is rated as High. 

- Arbitrary Code Execution: The attacker can execute arbitrary JavaScript in
the victim's browser.

- Security Bypass: The vulnerability completely negates the intended protection
of the Content Security Policy.

- Session Compromise: This can lead to cookie theft, session hijacking, or
redirection to malicious sites.

##### 4. Remediation

To mitigate this vulnerability: 

1. Strict Input Validation: The `token` parameter should be stricly validated
(e.g., allowed to contain only alphanumeric characters) before being
included in HTTP headers.

2. Avoid Reflection in Headers: Do not reflect user-supplied data directly
into critical security headers like CSP.

3. Static CSP: Ideally, the CSP should be static string configured on the server,
rather than dynamically constructed using query parameters. 
