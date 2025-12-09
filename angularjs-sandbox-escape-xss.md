# Lab: Reflected XSS via AngularJS Sandbox Escape

**Severity:** High

**Vulnerability Type:** Client-Side Template Injection (CSTI)/Reflected XSS

**Target:** Web Security Academy Blog - Search Functionality

##### 1. Executive Summary 

A Reflected Cross-Site Scripting (XSS) vulnerability was identified in the 
blog search functionality. The application utilises an outdated version of 
the AngularJS framework. 

While AngularJS attempts to "sandbox" user input to prevent the execution of 
arbitrary JavaScript, flawed logic in how the application parses search terms
allows an attacker to break out of this sandbox. Specifically, the 
vulnerability permits the injection of Angular JS expression that bypass the
sandbox restrictions (even without using string literals) to execute arbitrary
JavaScript code in the victim's browser. 

##### 2. Technical Walkthrough 

2.1 Target Identification 

The application was identified as using AngularJS via the `ng-app` and `ng-
controller` directives in the HTML source code. The search query is reflected
inside an HTML header tag within the AngularJS scope.

<img width="1185" height="744" alt="f1" src="https://github.com/user-attachments/assets/8174ec2c-2ea1-4d8b-9b09-28b8cb30a86f" />

2.2 Code Analysis (The Sink)

An analysis of the client-side code revealed a critical security flaw in the
controller logic. The application takes the user's input (`search` parameter)
and passes it directly into the `$parse` function. 

```
$scope.value = $parse(key)($scope.query);
```

The `$parse` service is designed to convert AngularJS expressions into 
functions. When user input is passed here, it is evaluated as code. 

<img width="1200" height="747" alt="f2" src="https://github.com/user-attachments/assets/615aec14-5afb-49d4-9e19-187801bc3ce2" />

2.3 The Sandbox Constraint 

AngularJS implements a "sandbox" that restricts expressions from accessing 
the `window` object or the `Function` constructor directly. Additionally, this
specific environment filtered out quote characters, meaning standard
string-based payloads (like `alert(1)`) would fail.

2.4 Exploitation (Sandbox Escape)

To bypass these restrictions, a "Sandbox Escape" payload was constructed.
The payload leverages the `toString()` method to access the `constructor`
property, eventually reaching the `String` constructor to generate code 
without using quotes.

The Payload Logic:

1. Access Prototype: `toString().constructor.prototype` grants access to
the root Object prototype.

2. Overwrite Function: We overwrite `charAt` with `[].join` to facilitate
the next step.

3. Generate String (No Quotes): We use `String.fromCharCode(...)` to generate
the malicious code string (`x=alert(1)`) using ASCII decimal values, bypassing
the need for quote marks.

4. Execute via Filter: We pipe the result into the `orderBy` filter, which
forces AngularJS to execute the constructed function.

The Final Payload: 

```
1&toString().constructor.prototype.charAt%3d[].join;[1]|orderBy:toString().constructor.fromCharCode(120,61,97,108,101,114,116,40,49,41)=1
```

<img width="1440" height="875" alt="f3" src="https://github.com/user-attachments/assets/d2e26724-b350-4e37-ae09-babbb86da684" />

2.5 Execution 

When the victim loads the URL containing this payload, the AngularJS engine
evaluates the expression, escapes the sandbox, and executes the `alert(1)`
function.

<img width="1186" height="745" alt="f4" src="https://github.com/user-attachments/assets/78dc21c4-ffc5-4445-811f-cb07899430c7" />

##### 3. Impact Analysis

The impact is rated as High. 

- Arbitrary Code Execution: The attacker can execute any JavaScript commands
within the victim's session.

- Data Exfiltration: Sensitive data (cookies, session tokens, personal
information) can be stolen.

- Widespread Impact: Because this is a framework-level vulnerability
triggered by reflection, it can potentially affect any user who clicks a
malicious link.

##### 4. Remediation 

To mitigate this vulnerability:

1. Avoid `$parse` with User InputL Never pass user-controlled data directly
to the `$parse`, `$eval`, or `$compile` services in AngularJS.

2. Upgrade Framework: The version of AngularJS used is likely End-of-Life
(EOL) and contains known sandbox escapes. Migrate to a modern framework
(Angular, React, Vue) that handles data binding more securely.

3. Content Security Policy (CSP): Implement a strict CSP to restrict where
scripts can be loaded from and prevent inline script execution. 
