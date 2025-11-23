# Lab: Reflected XSS into HTML context with most tags and attributes blocked

**Difficulty:** Practioner

**Vulnerability:** Reflected XSS with WAF Evasion 

**Lab Link:** https://portswigger.net/web-security/cross-site-scripting/contexts/lab-html-context-with-most-tags-and-attributes-blocked

##### 1. The Goal

The objective of this lab was to execute a Cross-Site Scripting (XSS)
attack in a search function protected by a strict Web Application 
Firewall (WAF).

My goal was to identify a single allowed HTML tag and a compatible 
event handler, then bypass the need for user interaction to execute the 
print() function automatically.

##### 2. Reconnaissance (Fuzzing)

I started by testing standard XSS vectors (<script>, <img>), but the
application returned 400 Bad Request, indicating a blocklist WAF was 
in place.

<img width="773" height="788" alt="a1" src="https://github.com/user-attachments/assets/77093149-e022-44b0-b635-3ddb364c327d" />

To find a bypass, I used **Burp Suite Intruder** to fuzz for allowed
tags.

Phase 1: Tag Enumeration
1. I intercepted the search request: `GET /?search=<§test§>`
2. I loaded the standard PortSwigger XSS Cheat Sheet tag list.
3. I analysed the response. Most returned 400, but one tag returned
4. 200 OK and was reflected raw: `<body>`

<img width="1437" height="875" alt="a2" src="https://github.com/user-attachments/assets/0931de27-2e1f-49d0-8271-de5d6a7d3004" />
<img width="1339" height="799" alt="a3" src="https://github.com/user-attachments/assets/de61345c-2676-42b1-af66-b9274cc1bc1d" />
<img width="629" height="429" alt="a5" src="https://github.com/user-attachments/assets/a8c978c5-fd48-40f2-8a92-de06eac0faf6" />

Phase 2: Attribute Enumeration
Next, I needed an event handler compatible with the <body> tag.
1. I configured the payload position: `GET /?search=<body+§event§=1>`
2. I loaded the Events list.
3. I found that the `onresize` event returned 200 OK.

<img width="1440" height="874" alt="a6" src="https://github.com/user-attachments/assets/36dc8aec-401b-4249-9c72-a0583f13ebeb" />
<img width="810" height="428" alt="a7" src="https://github.com/user-attachments/assets/8ff62ccd-bbcc-4633-8cdb-380f3b51627b" />

##### 3. Code Analysis (The Constraint)

I had a working vector: `<body onresize=print()>`.

However, the onresize event only fires when the browser window changes
dimensions. I could not rely on a victim manually resizing their 
browser. I needed a delivery mechanism to force this interaction.

##### 4. The Exploit Strategy

I chose to use the Exploit Server to host an iframe attack.

The Logic:

1. Trap: Load the vulnerable lab page inside an iframe on my exploit
page.
2. Trigger: Use JavaScript on my page (onload) to programmatically
   change the width of the iframe.
3. Execution: The browser inside the iframe detects the size change,
fires the onresize event (from my injected <body> tag), and executes
the payload.

<img width="770" height="791" alt="a8" src="https://github.com/user-attachments/assets/e2a12477-64a6-4fdb-9a79-90cffc187d3e" />

##### 5. The Payload

I constructed the following HTML for the Exploit Server body:

```HTML
<iframe src="[https://YOUR-LAB-ID.web-security-academy.net/?search=%3Cbody%20onresize=print()%3E](https://YOUR-LAB-ID.web-security-academy.net/?search=%3Cbody%20onresize=print()%3E)"
        onload="this.style.width='100px'">
</iframe>
```
<img width="774" height="785" alt="a9" src="https://github.com/user-attachments/assets/dfe477c1-ac26-4f04-85d8-a2a35b95c70a" />

Note: I URL-encoded the payload (%3C, %3E) to ensure the iframe src
attribute remained valid. 

##### 6. Execution

1. I pasted the payload into the Exploit Server.
2. I clicked "Deliver exploit to victim."
3. The victim visited my page, the iframe loaded, resizing itself,
and the print() function executed automatically.
