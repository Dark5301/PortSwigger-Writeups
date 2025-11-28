# Lab: Reflected XSS into a JavaScript String with Single Quote and Backslash Escaped

**Lab Difficulty:** Apprentice

**Topic:** Cross-Site Scripting (Reflected)

##### 1. Objective

The objective of this lab is to perform a Reflected Cross-Site Scripting (XSS)
attack. The injection point is inside a JavaScript string within a `<script>`
block. The application attempts to prevent XSS by escaping single quotes and
backslashes, preventing standard string breakouts. We need to find an alternative
way to execute JavaScript.

<img width="1185" height="747" alt="d1" src="https://github.com/user-attachments/assets/ade285e3-d925-4321-b27d-88b0538545ee" />

##### 2. Reconnaissance & Analysis

Identification of Reflection Point

We began by injecting a standard alphanumeric string `test123` into the search
bar. Inspecting the source code revealed that our input is reflected directly
inside a JavaScript variable assignment:

```
var searchTerms = 'test123';
```

<img width="1440" height="876" alt="d2" src="https://github.com/user-attachments/assets/2689811f-c202-4abc-b0dc-506f99cb9ea9" />

Vulnerability Assessment 

To test the sanitation, we attempted to break out of the string using a 
single quote `'`. Input: `test'123` Reflected: `var searchTerms='test\'123';`

The application places a backslash `\` before our single quote, neutrailising
it. This prevents us from simply closing the string and adding a command 
like `';alert(1)//`. Similarly, injecting a backslash `\` results in `\\`,
preventing us from escaping the escape character itself.

##### 3. Exploitation 

The "Script Breakout" Strategy 

Since we cannot break out of the string syntax due to character escaping, 
we must look at the higher-level context: the HTML `<script>` tag itself.

The browser's HTML parser runs before the JavaScript engine. If the HTML 
parser encounters a closing `</script>` tag, it terminates the script 
block immediately, regardless of whether that tag is inside a JavaScript
string or not.

Strategy:

1. Close the Script Block: Inject `</script>` to force the browser to exit
the current JavaScript context.

2. Start a New Script: Immediately open a new `<script>` tag.

3. Inject Payload: Write the malicious `alert(1)` command.

Payload Construction

We crafted the following payload:

```
</script><script>alert(1)</script>
```

We included some extra characters in our test (as seen in the screenshot)
to confirm the behaviour, but the core logic relies entirely on the tags.

Full Payload Used:

```
':;</script><script>alert(1)</script>;//
```
