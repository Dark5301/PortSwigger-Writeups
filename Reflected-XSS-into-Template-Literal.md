# Lab: Reflected XSS into a Template Literal with Angle Brackets, Single, Double Quotes, Backslash, and Backticks Unicode-Escaped

**Lab Difficulty:** Apprentice

**Topic:** Cross-Site Scripting (Reflected)

##### 1. Objective

The objective of this lab is to perform a Reflected Cross-Site Scripting (XSS)
attack. The injection point is inside a JavaScript Template Literal (denoted
by backticks `'`). The application implements robust defenses: angle brackets
(`<>`), single quotes (`'`), double quotes (`"`), backslashes (`\`), and
backticks (`'`) are all Unicode-escaped. The goal is to execute the `alert(1)`
function despite these restrictions.

<img width="1186" height="747" alt="i1" src="https://github.com/user-attachments/assets/5570dd7f-12a5-42e6-9f7c-48ebd1c07ce8" />

##### 2. Reconnaissance & Analysis

Identification of Reflection Point

We began by injecting the string `test123` into the search bar. The response
revealed that our input is reflected inside a JavaScript template
literal:

```
var message = `0 search results for 'test123'`;
```

<img width="1440" height="877" alt="i2" src="https://github.com/user-attachments/assets/9382e0fa-571c-4235-a573-42a059c6e11b" />

Vulnerability Assessment 

We are inside a string delimited by backticks: var message = `INPUT`;

Constraints:

- String Breakout: Attempting to close the string with a backtick(``) fails
because the server escapes it to `\u0060`.

- Standard Quotes: Single (``) and double (`"`) quotes are also escaped
(`\u0027, \u0022`), preventing us from writing standard strings.

- Tag Injection: Angle brackets (`<>`) are escaped (`\u003c, \u003e`),
preventing HTML tag injection.

The Loophole: While we cannot break out of the string, the context is a 
Template Literal. Template literals in JavaScript have a unique feature 
called String Interpolation. This allows embedded expressions to be evaluated
inside the string using the syntax `${expression}`.

If the application does not escape the dollar sign `$` or the curly braces
`{}`, we can execute JavaScript inside the string itself without needing
to break out of it.

##### 3. Exploitation 

Payload Construction

Our strategy is to use the interpolation syntax `${ ...}` to run JavaScript.

Attempt 1: `${alert(1)}`

- Result: The `alert(1)` function runs, but the `alert` function returns
`undefined`. The string becomes "0 search results for 'undefined'".

Crucial Check: The lab likely checks if the alert function executes, 
regardless of the return value.

Payload:

```
${alert(1)}
```

Execution 

We sent the payload via the search parameter in Burp Suite.

Request: `GET /?search=${alert(1)} HTTP/2`

Response:

```
var message = `0 search results for '${alert(1)}'`;
```

<img width="1440" height="877" alt="i3" src="https://github.com/user-attachments/assets/478a0c45-4833-4dcb-8302-771d8b65c44f" />
<img width="1200" height="791" alt="i4" src="https://github.com/user-attachments/assets/6b6f7b33-b0f9-477a-9ad8-e061aafa3ab3" />
