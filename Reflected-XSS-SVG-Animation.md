# Lab: Reflected XSS with some SVG markup allowed

**Difficulty:** Practitioner 

**Vulnerability:** Reflected XSS with WAF Evasion & SVG 
Animation

**Lab Link:** https://portswigger.net/web-security/cross-site-scripting/contexts/lab-some-svg-markup-allowed

##### 1. The Goal

The objective of this lab was to execute a Cross-Site
Scripting (XSS) attack in a search function protected by
a comprehensive blocklist WAF.

My goal was to identify whitelisted SVG tags, find a 
compatible event handler that fires automatically, and 
bypass a specific WAF filter to execute the alert() function.

##### 2. Reconnaissance (Fuzzing)

I started by testing standard tags (<script>, <img>), but
the server returned 400 Bad Request.

To map the attack surface, I used Burp Suite Intruder to 
fuzz for allowed tags.

- Payload: `GET /?search=<§tag§>`

- Result: Most tags were blocked, but four specific SVG-related tags returned 200 OK:

 - `<svg>`
 
- `<animatetransform>`
 
- `<image>`
 
- `<title>`

<img width="1201" height="747" alt="d1" src="https://github.com/user-attachments/assets/8205b69d-65f1-47fc-9345-02d3c7fbbdab" />
<img width="1440" height="877" alt="d2" src="https://github.com/user-attachments/assets/2ed8041a-fb61-45d5-add4-c3a6bbb7e2cb" />
<img width="906" height="460" alt="d3" src="https://github.com/user-attachments/assets/43fae5a5-5abb-4715-8aea-ec05c11db5d3" />
<img width="886" height="465" alt="d7" src="https://github.com/user-attachments/assets/02fa1ddb-ff04-42dc-8a78-336c19ce4d0e" />

##### 3. Vulnerability Analysis 

I focused on <animatetransform> because it is an 
animation element. Animation elements have their own
lifecycle events, which can be abused to execute code
automatically without user interaction.

Event Enumeration

I fuzzed the <animatetransform> tag for allowed event
handlers.

- Standard events: onload and onerror were blocked or not applicable.

- Animation events: I identified that onbegin is a valid event for this tag. It fires immediately when the animation starts.

The WAF Bypass (The Slash Trick)

When I attempted to use the event normally (
<animatetransform onbegin=1>), the WAF blocked it. I
suspected the WAF was using a Regular Expression to 
look for the pattern: `[SPACE]on[a-z]+=`.

To bypass this, I used the Slash Trick. HTML parsers treat
a forward slash (/) as a valid separator between the 
tag name and the attribute, but many WAFs fail to 
detect it.

- Blocked: `<animatetransform onbegin=1>`

- Allowed: `<animatetransform/onbegin=1>`

##### 4. The Exploit Strategy 

I constructed a payload using the allowed componenets:

1. Container: <svg> (Required parent for SVG animations).

2. Trigger: <animatetransform> (The allowed animation tag).

3. Event: onbegin (Fires immediately).

4. Validation: I added attributeName=transform because the browser requires a target attribute to consider the animation "valid" and start the timeline.

##### 5. The Payload

I crafted the following payload: 

```HTML
<svg><animatetransform/onbegin=alert(1) attributeName=transform>
```

Note on Encoding: When injection this into the URL, I
encountered a "Protocol Error" because of the raw 
characters. I had to URL-encode the entire payload to 
ensure the server processed it correctly.

Final Encoded Payload:

`%3Csvg%3E%3Canimatetransform%2Fonbegin%3Dalert(1)%20attributeName%3Dtransform%3E`

<img width="1440" height="877" alt="d10" src="https://github.com/user-attachments/assets/ca51e079-9db9-4168-bc06-d07194bc5662" />

##### 6. Execution

1. I injected the encoded payload into the search bar.

2. The WAF passed the request (due to the slash trick
and allowed tags).

3. The browser rendered the SVG and validated the
animation instruction.

4. The animation timeline started, triggering the
onbegin event immediately.

5. The alert(1) function executed.

<img width="1184" height="790" alt="d11" src="https://github.com/user-attachments/assets/11b1a9ce-fa37-4fa8-b2bd-aba0c53a350f" />
