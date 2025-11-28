# Lab: Reflected XSS into a JavaScript string with Angle Brackets HTML Encoded

**Lab Difficulty:** Apprentice

**Topic:** Cross-Site Scripting (Reflected)

##### 1. Objective

The objective of this lab is to perform a Reflected Cross-Site Scripting (XSS)
attack. The injection point is inside a JavaScript string within a `<script>`
block. The application projects against breaking out of the script tag by
the HTML-encoding angle brackets (`<` and `>`), but we need to verify if it
properly escapes JavaScript string delimiters.

<img width="1202" height="746" alt="e1" src="https://github.com/user-attachments/assets/3a1eead9-e0cb-4ad3-b7c3-2f2d4baf5e31" />

##### 2. Reconnaissance & Analysis

Identification of Reflection Point 

We began by injecting a standard alphanumeric string `test123` into the 
search bar to trace where the input is reflected. The response showed that 
our input is reflected directly inside a JavaScript variable assignment:

```
var searchTerms = 'test123';
```

<img width="1440" height="877" alt="e2" src="https://github.com/user-attachments/assets/753af32d-652c-4de7-b32c-ba147bf4dc2f" />

Vulnerability Assessment 

The lab description states that angle brackets are HTML-encoded. This means 
inputting `</script>` to close the script block will fail because it will 
be converted to `&lt;/script&gt;`, which the JavaScript engine treats as a 
variable or strings, not a tag closer.

However, we need to check if single quotes (') are escaped. If the 
applicattion does not escape single quotes (e.g., converting `'` to `\'`),
we can terminate the `searchTerms` string and append our own JavaScript
commands within the existing script block.

##### 3. Exploitation 

Payload Construction 

To exploit this, we need to:

1. Break Out of the String: Use a single quote `'` to close the `searchTerms`
variable assignment.

2. Separate the Command: Use a semicolon `;` to end the current statement.

3. Inject Payload: Write `alert()` to trigger the XSS

4. Neutralise the Rest: Use a double forward slash `//` (JavaScript comment)
to comment out the trailing single quote and semicolon reflected by the server,
preventing syntax errors.

Payload:

```
';alert(1);//
```

Execution

We sent the payload via the search parameter in Burp Suite

Request: `GET /?search=';alert(1);// HTTP/2`

Response:

```
var searchTerms = '';alert(1);//';
```
