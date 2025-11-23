# Lab: Reflected XSS into HTML context with all tags blocked except custom ones

**Difficulty:** Practitioner

**Vulnerability:** Reflected XSS with WAF Evasion & Browser Logic Abuse

**Lab Link:** https://portswigger.net/web-security/cross-site-scripting/contexts/lab-html-context-with-all-standard-tags-blocked

##### 1. The Goal

The objective of this lab was to execute a Cross-Site Scripting (XSS)
attack in a search function protected by a strict WAF that blocks all 
standard HTML tags.

My goal was to inject a custom HTML tag, make it interactive using 
specific attributes, and force the victim's browser to execute code 
automatically upon page load.

##### 2. Reconnaissance (Fuzzing)

I started by testing standard XSS vectos (<script>, <img>, <body>), 
but the server returned 400 Bad Request. This confirmed a strict
blocklist WAF. 

<img width="1440" height="791" alt="b1" src="https://github.com/user-attachments/assets/5a6175e4-321d-4f94-87f8-3c58b5f3faca" />
<img width="1440" height="876" alt="b2" src="https://github.com/user-attachments/assets/8eefd29c-0b95-470a-afed-44824751ac59" />

Phase 1: Tag Enumeration

I hypothesized that the WAF might only block known HTML tags. To test
this, I injected a non-existent custom tag:

- Payload: <test>

- Result: 200 OK. The server reflected the tag raw: <test>

<img width="1437" height="871" alt="b3" src="https://github.com/user-attachments/assets/55141aa4-6e29-44e7-a7d7-ab7091baa353" />

Phase 2: Event Enumeration 

Since custom tags do not load external resources, standard events like
onload or onerror do not fire. I needed an interaction event.

1. I fuzzed the custom tag for attributes: `<test §event§=1>`

2. I found that onfocus was allowed.

<img width="1440" height="874" alt="b4" src="https://github.com/user-attachments/assets/f19e2db2-b950-4af3-a66f-d1d399582d10" />

##### 3. The Exploit Chain

I had a working vector: `<xss onfocus=alert(1)>`.

However, a custom tag is just a text container. It cannot receive
"focus" by default (unlike an input field), and I could not rely on
the victim clicking it manually. I needed to chain three browser
behaviour to weaponise this: 

1. tabindex="1": This attribute forces the browser to treat the
custom tag as an interactive, focusable element (allowing it to
receive focus).

2. id="x": This gives the element a targetable identity.

3. URL Fragment (#x): Appending #x to the URL forces the browser
to automatically scroll to and focus on the element with id="x"
immediately upon page load.

The trigger logic: Load Page -> Browser reads #x -> Jumps to <xss 
id=x> -> Element receives Focus -> onfocus fires -> Alert executes.

<img width="1440" height="875" alt="b5" src="https://github.com/user-attachments/assets/40dedd6a-7fe6-46a1-bc3c-7d28b0cd807c" />

##### 4. The Delivery Strategy (iframe vs. Redirect)

I initially attempted to use an <iframe> to deliver the payload 
invisibly. 

- Result: Failed

- Root Cause: Modern browsers implement Focus Hijacking Prevention.
They block elements inside a cross-origin <iframe> from receiving
focus unless the user is already interacting with that frame.

To bypass this security control, I switched to a Top-Level Navigation
attack. I used window.location to redirect the victim's entire tab
to the malicious URL. This makes the lab the "Active Window," allowing
the focus event to fire automatically. 

##### 5. Tha Payload

I constructed the following JavaScript for the Exploit Server body:

```HTML
<script>
location = '[https://YOUR-LAB-ID.web-security-academy.net/?search=%3Cxss+id%3Dx+onfocus%3Dalert%28document.cookie%29%20tabindex=1%3E#x](https://YOUR-LAB-ID.web-security-academy.net/?search=%3Cxss+id%3Dx+onfocus%3Dalert%28document.cookie%29%20tabindex=1%3E#x)';
</script>
```

Breakdown:

- Location = ...: Triggers the top-level redirect.
- %3Cxss... : The URL-encoded payload injected into the search bar.
- #x: The hash fragment that triggers the automatic focus.

<img width="1424" height="789" alt="b6" src="https://github.com/user-attachments/assets/46137479-53df-4d22-bf9c-895527a5e867" />

##### 6. Execution

1. I pasted the script into the Exploit Server.

2. I clicked "Deliver exploit to victim".

3. The victim visited my page, was immediately redirected to the lab,
and the browser's focus behaviour triggered the alert(document.
cookie) popup.
