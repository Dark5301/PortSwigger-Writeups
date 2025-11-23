# Lab: Reflected XSS with event handlers and href attributes blocked

**Difficulty:** Exper

**Vulnerability:** Reflected XSS via SVG Attribute Smuggling 

**Lab Link:** https://portswigger.net/web-security/cross-site-scripting/contexts/lab-event-handlers-and-href-attributes-blocked

##### 1. The Goal 

The objective of this lab was to execute a Cross-Site Scripting
(XSS) attack in a search function protected by a comprehensive
blocklist WAF.

The WAF blocked all standard event handlers (onclick, onload, 
onmouseover) and the href attribute, making standard exploitation
impossible. My goal was to construct a malicious link vectore that
bypasses these filters and executes alert() when clicked.

##### 2. Reconnaissance (Fuzzing)

I started by testing standard tags and attributes. The application
blocked almost everything, returning 400 Bad Request for payloads
containing `href=` or `on[event]=`.

To find a foothold, I fuzzed for allowed HTML tags using Burp intruder.
I identified a specific set of allowed tags related to SVG (Scalable
Vectore Graphics):

- `<svg>`
- `<a> (SVG Anchor)`
- `<animate>`

<img width="1424" height="791" alt="c1" src="https://github.com/user-attachments/assets/a790a61c-ddb7-41c2-8586-382038f31c3e" />
<img width="1440" height="848" alt="c2" src="https://github.com/user-attachments/assets/615a0a32-306f-40fc-bbb8-a34018a86b2b" />
<img width="837" height="456" alt="c3" src="https://github.com/user-attachments/assets/0c1afdc2-464a-40fc-a933-56d72ada0dc1" />
<img width="901" height="518" alt="c4" src="https://github.com/user-attachments/assets/a10c3739-76a3-4b34-814a-55beecc09b19" />
<img width="991" height="518" alt="c5" src="https://github.com/user-attachments/assets/17ae488a-6e4b-486f-b573-52664f3d3593" />

##### 3. Vulnerability Analysis (The "Attribute Smuggling" Concept)

Since I could use the <svg> and <animate> tags, I recognised a 
potential SMIL injection vulnerability (Synchronised Multimedia
Integration Language). 

The <animate> tag is designed to animate the attributes of a 
parent element. Crucially, it can target any including functional
ones like href.

The Bypass Logic

The WAF is a static text analyser. It blocks the string href=. 
However, the <animate> tag doesn't use href=. It blocks the string
href=. However, the <animate> tag doesn't use href=. It uses
attributeName="href".

This discrepancy allows me to "smuggle" the forbidden attribute 
past the WAF. I don't send the attribute directly; I send an
"animation instruction" that forces the browser to create the 
attribute after the page loads.

##### 4. The Exploit Strategy 

I constructed a payload using three nested componenets:

1. <svg>: Acts as the container to enable SVG-specific behavior.

2. <a>: Creates a clickable link. I defined it "naked" (without
an href) to bypass the WAF.

3. <animate>: Placed inside the link. It instructs the browser to
immediately "animate" (set) the href attribute of its parent to
a malicious JavaScript URI.

##### 5. The Payload

I crafted the following XML payload:

```XML
<svg>
    <a>
        <animate attributeName="href" values="javascript:alert(1)" />
        <text x="20" y="20">Click me</text>
    </a>
</svg>
```

Breakdown:

- attributeName="href": Targets the parent link's destination.

- values="javascript:alert(1)": Sets the destination to the
JavaScript payload.

- <text>Click me</text>: Provides a visible label for the victim
to click (required by the lab).

<img width="1184" height="745" alt="c7" src="https://github.com/user-attachments/assets/4e2d9884-3d41-41a6-96c2-ea65ed0f56ab" />

##### 6. Execution 

1. I injected the payload into the search bar.

2. The WAF allowed the request because it did not see the forbidden
href= string.

3. The browser rendered the SVG.

4. The <animate> tag executed immediately, dynamically adding
href="javascript:alert(1)" to the link in the DOM.

5. I clicked the "Click me" text, and the alert executed.
