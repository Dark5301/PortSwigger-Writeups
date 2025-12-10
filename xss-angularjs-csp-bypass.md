# Lab: Reflected XSS via AngularJS Sandbox Escape & CSP Bypass

**Severity:** High

**Vulnerability Type:** Reflected XSS/CSP Bypass/Client-Side Template Injection

**Target:** Web Security Academy Blog-Search Functionality 

##### 1. Executive Summary

A Reflected Cross-Site Scripting (XSS) vulnerability was identified in the 
blog search functionality. 

The application uses AngularJS and implements a Content Security Policy (CSP)
to prevent inline script execution. However, the application reflects user 
input into the HTML body where AngularJS is active. An attacker can inject
specific HTML directives (`ng-focus`) that the AngularJS framework executes.
By combining this with a specific URL fragment to force element focus and an
AngularJS sandbox escape payload, an attacker can bypass the CSP and execute 
arbitrary JavaScript on the victim's browser immediately upon page load. 

##### 2. Technical Walkthrough 

2.1 Target identification & Reflection 

The search function reflects the user's input directly into the HTML page. The
response headers confirm that the application is protected by a CSP that 
disallows inline scripts (`script-src 'self'`).

<img width="1440" height="876" alt="g2" src="https://github.com/user-attachments/assets/f879fea7-d972-4a50-8427-ad54c625fb79" />

However, an analysis of the loaded scripts revealed the presence of AngularJS
(v1.4.4).

<img width="1440" height="876" alt="g3" src="https://github.com/user-attachments/assets/fb05b9ec-081e-4250-bf65-bcc615024928" />

2.2 Exploitation Strategy 

Standard XSS payloads (like `<script>`) are blocked by the CSP. However, 
AngularJS parses HTML attributes starting with `ng-` (directives) and 
executes them within its own context, which is "trusted" by the CSP because
the Angular library itself is loaded from a trusted source (`'self'`).

The Attack Chain:

1. Injection: Inject an HTML element (`<input>`) with an AngularJS event
listener (`ng-focus`).

2. Trigger: Use the URL fragment identifier (`#x`) to tell the browser to
automatically "focus" on our injected element (id=`x`) as soon as the page
loads.

3. Sandbox Escape: Inside the `ng-focus` event, use an expression that breaks
out of the AngularJS sandbox to call `alert()`.

2.3. Payload Construction 

The following payload was constructed to achieve execution:

```
<input id=x ng-focus=$event.composedPath()|orderBy:'(z=alert)(document.cookie)'>
```

- `id=x`: Sets the element ID so we can target it with the URL fragment.

- `ng-focus`: The directive that executes code when the element receives focus.

- `$event.composedPath() | orderBy:...`: This specific chain is a known
sandbox escape for this version of AngularJS. It manipulates an array (the
event path) and passes it to the `orderBy` filter, which allows function
calls (like `alert`) to be executed during the sorting process.

<img width="1440" height="876" alt="g4" src="https://github.com/user-attachments/assets/ffd90a33-06a6-4d29-b4c8-0386510c682d" />

2.4 Execution

When the victim visits the malicious URL constructed on the exploit server,
the browser loads the page, focuses the injected input field, and triggers
the XSS payload, displaying the session cookie. 

<img width="1186" height="746" alt="g5" src="https://github.com/user-attachments/assets/6d827681-5431-49e7-86ea-b98f5ebc73c3" />

##### 3. Impact Analysis 

The impact is rated as High.

- CSP Bypass: The attack demonstrates that the current CSP is ineffective
against library-based attacks (Client-Side Template Injection).

- Zero-Interaction: The use of the `autofocus` or URL fragment technique
means the victim does not need to interact with the page (other than
loading it) for the code to execute.

- Session Compromise: Arbitrary JavaScript execution allows for session
hijacking (cookie theft) or redirection to malicious sites.

##### 4. Remediation 

To mitigate this vulnerability: 

1. Update AngularJS: The application is using an outdated version of
AngularJS. Migrate to a modern, supported framework (Angular 2+, React,
Vue) that does not rely on this type of sandbox.

2. Context-Aware Encoding: Strictly encode all user-supplied data reflected
in the HTML body.

3. Disable `ng-app`: If possible, do not place the `ng-app` directive on the
`<body>` or `<html>` tag. Limit the scope of AngularJS to specific
containers that do not contain reflected user input.

4. CSP Refinement: Consider using CSP Level 3 features like `strict-dynamic`
with nonces, though this is difficult to implement with legacy AngularJS. 
