# Lab: Reflected XSS in a JavaScript URL with Some Characters Blocked

**Lab Difficulty:** Expert 

**Topic:** Cross-Site Scripting (Refined)

##### 1. Objective

The objective of this lab is to perform a Reflected Cross-Site Scripting (XSS)
attack. The injection point is inside a JavaScript object within a `javascript:`
URL scheme (specifically, a `fetch` request). The application employs a Web
Application Firewall (WAF) that blocks specific characters (such as parentheses (
) and spaces), making standard function calls impossible. The goal is to bypass
these filters and execute the `alert(1337)` function.

<img width="1186" height="746" alt="g1" src="https://github.com/user-attachments/assets/b56a3ed1-9c9f-4f54-aa7e-4775e3123aa3" />

##### 2. Reconnaissance & Analysis 

Identification of Reflection Point 

We began by analysing the "Back to Blog" link functionality on a post page.
The HTML source code revealed the following structure for the link:

```
<a href="javascript:fetch('/analytics', {method:'post',body:'/post?postId=2'}).finally(...)">Back to Blog</a>
```

The server reflects the current URL query parameters directly inside the
`body` property string of the `fetch` options object. 

<img width="1440" height="877" alt="g2" src="https://github.com/user-attachments/assets/7f66d76e-54e5-4077-919b-cbd3cce47a47" />

Vulnerability Assessment 

The input is reflected inside a single-quoted string within a JavaScript 
object literal: `body:'/post?postId=INPUT'`

To exploit this, we need to:

1. Break Out of String: Use a single quote `'` to close the `body` value.

2. Break Out of Object: Use a closing brace `}` to close the `fetch` options object.

3. Inject Code: Insert our payload as a new argument to the `fetch` function.

Constraints (WAF): Through manual testing, we determined that the 
application blocks:

- Parentheses `(` and `)`: This prevents standard calls like `alert(1337)`.

- Spaces `%20`: This prevents separation of keywords like `throw 1337`.

##### 3. Exploitation 

The Bypass Strategy 

To overcome the WAF restrictions, we employed a "Polyglot" strategy
utilising JavaScript error handling and arrow functions.

1. Statement vs. Expression: We cannot inject a statement like `throw`
directly as a function argument. We must wrap it in an Arrow Function
`x => { ... }` to make it a valid expression.

2. Space Bypass: We used block comments `/**/` as a separator instead
of spaces.

3. Parenthesis Bypass (Exception Handler): We used `onerror=alert; throw
1337`. By throwing an exception, the browser automatically passes the
error value (1337) to the assigned error handler (alert), triggering
the alert without using parentheses.

4. Execution Trigger (`toString`): We assigned our malicious arrow
function `x` to the `toString` property of the window. We then forced a
string conversion using `window + ''`. This implicitly calls
`window.toString()`, which executes our malicious function.

Payload Construction 

The Payload:

```
&'},x=x=>{throw/**/onerror=alert,1337},toString=x,window%2b'',{x:'
```

<img width="1440" height="877" alt="g3" src="https://github.com/user-attachments/assets/94fdfecb-f084-44c0-a112-bb170604ca0b" />
<img width="1440" height="876" alt="g4" src="https://github.com/user-attachments/assets/72d780c1-2c82-4d07-9e0a-d0bb9664edf0" />
<img width="1202" height="747" alt="g5" src="https://github.com/user-attachments/assets/063c9b5b-78c4-482d-b891-0250a265ce65" />
