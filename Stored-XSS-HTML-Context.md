# Lab: Stored XSS into HTML context with nothing encoded

**Difficulty:** Apprentice

**Vulnerability:** Stored Cross-Site Scripting (XSS)

**Lab Link:** https://portswigger.net/web-security/cross-site-scripting/stored/lab-html-context-nothing-encoded

#### 1. The Goal 

The objective of this lab was to exploit a classic Stored XSS
vulnerability in the blog comment functionality.

My goal was to submit a malicious payload that the server would 
save to its database and later serve to other users, executing
the `alert()` function in their browsers.

#### 2. Reconnaissance

I navigated to a blog post and tested the comment section to 
understand how inputs were handled.

1. I submitted a benign comment: `<script>alert(1)</script>`

2. I reloaded the page to verify persistence.

3. I inspected the page source (`Ctrl+U`) to see how my input
was rendered.

<img width="804" height="727" alt="1 1" src="https://github.com/user-attachments/assets/bfd563cf-ce27-4814-9690-ae69c2c887d6" />

<img width="805" height="729" alt="2 2" src="https://github.com/user-attachments/assets/b5d47326-5ee3-4135-bb7a-73f90b8c2f90" />

#### 3. Exploitation

The server is not converting < and > into safe HTML entities
(&lt;, &gt;). My input is being rendered directly into the HTML
body, not inside an attribute or JavaScript string. Because there
is "nothing encoded," I can inject standard HTML tags.

Since the context is plain HTML and there are no filters, the 
most direct payload is the standard `<script>`
tag.

Payload: `<script>alert(1)</script>`

<img width="805" height="692" alt="3 3" src="https://github.com/user-attachments/assets/0ff925a7-5084-45cf-84a2-d77b6f0d717c" />
