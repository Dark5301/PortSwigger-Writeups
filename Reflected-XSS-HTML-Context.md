# Lab: Reflected XSS into HTML context with nothing encoded

**Difficulty:** Apprentice

**Vulnerability:** Reflected Cross-Site Scripting (XSS)

**Lab Link:** https://portswigger.net/web-security/cross-site-scripting/reflected/lab-html-context-nothing-encoded

#### 1. The Goal 

The objective of this lab was to exploit a classic Reflected
XSS vulnerability in the application's search functionality.

My goal was to identify a reflection point where user input is
echoed back to the browser without sanitization, and then craft
a URL that executes the `alert()` function when visited.

#### 2. Reconnaissance

I started by probing the search bar with a string: `can I hack
this`

I inspected the server's response (View Page Source) to see where
my input appeared.

<img width="806" height="805" alt="1" src="https://github.com/user-attachments/assets/20c3fd9a-10cc-4a7f-91ad-db6e1f6cd87a" />

<img width="804" height="731" alt="2" src="https://github.com/user-attachments/assets/7abdebf9-9492-47c0-90fc-c44a45b180eb" />

<img width="805" height="805" alt="3" src="https://github.com/user-attachments/assets/d0187454-b0c7-4488-b36f-efba0ded0a2d" />

The server reflected my input directly into the HTML body:

```HTML
<h1>0 search results for 'can I hack this'</h1>
```

#### 3. Exploitation

Since there is no validation checks implemented by the 
developers of the web application therefore we can use a very
common payload `<script>alert(1)</script>`.

I entered the payload `<script>alert(1)</script>` into the search 
bar.

I pressed Search.

The server responded with a page containing:

<img width="805" height="731" alt="4" src="https://github.com/user-attachments/assets/8c93b642-2405-40c9-a383-d3f7871ff484" />

<img width="1140" height="780" alt="8" src="https://github.com/user-attachments/assets/86fb1549-6bad-4ce8-a262-46f08999a3e4" />

```HTML
<h1>
  0 search results for '<script>
    alert(1)
  </script>
'
</h1>
```

The browser parsed the script tag and executed the alert.

The vulnerability is Reflected, meaning the payload is not stored
in the database. To exploit a victim, I would need to construct
a malicious link:

`https://LAB-ID.web-security-academy.net/?search=<script>alert(1)
</script>`

If I trick a victim into clicking this link (e.g., via a phishing
email), the script will execute in their browser session, 
allowing me to steal cookies or perform actions on their behalf.

<img width="802" height="727" alt="5" src="https://github.com/user-attachments/assets/0b81cc2a-30f5-47fc-a26f-c811c8f0ae22" />
