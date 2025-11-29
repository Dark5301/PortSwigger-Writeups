# Lab: Stored XSS into onclick Event with Angle Brackets and Double Quotes HTML-Encoded and Single Quotes and Backslash Escaped 

**Lab Difficulty:** Apprentice 

**Topic:** Cross-Site Scripting (Stored)

##### 1. Objective 

The objective of this lab is to perform a Stored Cross-Site Scripting (XSS)
attack via a blog comment. The injection point is inside the `onclick` event
handler of a user's name link. The application employs strict filtering:
angle brackets (`<>`) and double quotes (`"`) are HTML-encoded, while single
quotes (`'`) and backslashes (`\`) are escaped. The goal is to bypass these 
filters and execute the `alert(1)` function when the comment author's name
is clicked.

<img width="1186" height="745" alt="h1" src="https://github.com/user-attachments/assets/d79bca68-4baf-48cc-ad39-40ca407b9c9b" />

##### 2. Reconnaissance & Analysis

Identification of Injection Point 

We began by analysing the comment submission form. We submitted of a comment
with the website field set to `https://www.example.com`. The server reflects
the website URL into the `onclick` attribute of the author's name link:

```
<a id="author" href="http://..." onclick="var tracker={track(){}};tracker.track('[https://www.example.com](https://www.example.com)');">hacker</a>
```

<img width="1440" height="874" alt="h2" src="https://github.com/user-attachments/assets/f3df80e4-dd15-4d16-b105-26e2b400e3ce" />

Vulnerability Assessment 

The input lands inside a single-quoted JavaScript string:

`tracker.track('INPUT');`

Constraints:

- Angle Brackets (`<>`) & Double Quotes (`"`): HTML-encoded. We cannot break
out of the HTML tag or attribute.

- Single Quotes (`'`) & Backslashes (`\`): Escaped with a backslash.

  - Input: `'` -> Reflected: `\'`
 
  - Input: `\` -> Reflected: `\\`

This escaping mechanism prevents us from closing the JavaScript string
using a literal single quote. However, since we are inside an HTML
attribute (`onclick`) the browser performs HTML entity decoding before
executing the JavaScript.

Hypothesis: If we inject the HTML entity `&apos;`, the server's filter
(which looks for literal `'`) will ignore it. But the browser will decode
`&apos;` into a literal `'` at runtime, allowing us to break out of the 
string.

##### 3. Exploitation 

Payload Construction 

We crafted a payload that uses HTML entities to smuggle the single quotes
past the filter. 

Strategy:

1. Close String: `&apos;` (Decodes to `'`).

2. Separate: `-` (Minus operator forces the JS engine to evaluate the expression).

3. Command: `alert(1)`.

4. Re-open String: `-&apos;` (Decodes to `-'`). This creates a syntactically
valid subtraction operation with the trailing quote from the original code.

Payload: 

```
http:&apos;-alert(1)-&apos;
```

<img width="1440" height="876" alt="h3" src="https://github.com/user-attachments/assets/abd7715f-9201-467f-b7ce-26cd909da0b1" />
<img width="1201" height="745" alt="h4" src="https://github.com/user-attachments/assets/c988e029-ee76-40cb-a22c-22173e7e1450" />
