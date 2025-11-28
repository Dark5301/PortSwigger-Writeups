# Lab: Reflected XSS into a JavaScript string with Angle Brackets and Double Quotes HTML-Encoded and Single Quotes Escaped

**Lab Difficulty:** Apprentice

**Topic:** Cross-Site Scripting (Reflected)

##### 1. Objective

The objective of this lab is to perform a Reflected Cross-Site Scripting (XSS)
attack. The injection point is inside a JavaScript string within a `<script>`
block. The application implements multiple layers of defense: angle brackets (`<>`)
and double quotes (`"`) are HTML-encoded, and single quotes (`'`) are 
escaped with a backslash. The goal is to bypass these restrictions to execute
the `alert(1)` function. 

<img width="1185" height="744" alt="f1" src="https://github.com/user-attachments/assets/1900cf34-15a0-4c6f-a45b-24b011c0c9c7" />

##### 2. Reconnaissance & Analysis 

Identification of Reflection Point 

We began by injecting a standard alphanumeric string `test123` into the search
bar. The response revealed that our input is reflected inside a single-quoted
JavaScript string:

```
var searchTerms = 'test123';
```

<img width="1440" height="876" alt="f2" src="https://github.com/user-attachments/assets/3671ad79-5902-4fb1-a90a-352bf0750a2d" />

Vulnerability Assessment 

To test the sanitation, we attempted to break out of the string using a 
simple quote `'`.

- Input: `'`

- Reflected: `\'`

The application defends against string breakout by placing a backslash 
before the single quote. This treats the quote as data rather than a 
string delimiter. Additionally, angle brackets are encoded, so we cannot
close the `<script>` tag.

However, we need to verify if the backslash (`\`) character itself is 
escaped. If the application escapes `'` to `\'` but does not escape `\`
to `\\`, we can exploit this behavior. 

##### 3. Exploitation 

The "Escaping the Escape" Strategy 

We formulated a hypothesis: If we inject a backslash followed by a single
quote (`\'`), the server will process them as follows: 

1. The injected backslash `\` is not escaped and remains `\`.

2. The injected single quote `'` is escaped `\'`

Result: `\\'`

In JavaScript:

- `\\` is interpreted as a literal backslash.

- The following `'` is no longer escaped (because the server's escaping backslash was consumed by our backslash).

- Therefore, the `'` acts as a valid string delimiter, closing the variable assignment.

Payload Construction 

Based on this logic, we constructed the following payload:

1. Neutralise Escape: `\` (Turns the server's escape character into a literal backslash).

2. Close String: `'` (Now unescaped, closing the string).

3. Command: `;alert(1)` (Executes our code)

4. Comment: `\\` (Comments out the rest of the line).

Payload:

```
\';alert(1);//
```

Execution

We sent the payload via the search parameter in Burp Suite. 

Request: `GET /?search=\';alert(1);// HTTP/2`

Response: 

```
var searchTerms = '\\';alert(1);//';
```
