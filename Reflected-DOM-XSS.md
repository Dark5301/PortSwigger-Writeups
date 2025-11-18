# Lab: Reflected DOM XSS

**Difficulty:** Practitioner

**Vulnerability:** Reflected DOM-based Cross-Site Scripting (XSS)

**Lab Link:** https://portswigger.net/web-security/cross-site-scripting/dom-based/lab-dom-xss-reflected

#### 1. The Goal

The objective of this lab was to exploit a Reflected DOM XSS vulnerability in the search functionality.

My goal was to identify a dangerou sink in the client-side JavaScript, analyse how the server reflects
the search term, and craft a payload to break out of the data structure and execute the `alert()` function.

#### 2. Reconnaissance 

On opening the web application I was greeted with a blog like website, which had multiple posts, search
functionality, feature to add and post comment on any post. I started off by testing the search function 
on the web application because that's where most reflected XSS are mostly found.

<img width="907" height="731" alt="c1" src="https://github.com/user-attachments/assets/52da430d-8856-4562-ae82-18f62903ba24" />

<img width="904" height="729" alt="c2" src="https://github.com/user-attachments/assets/edd5e321-4cf5-417e-91a2-5cba9fbdbbdf" />

After I searched for a test string in the search box, it makes a dynamic request to a JavaScript file
and executes a search function which is part of the file, so it was obvious to look for this function 
and know more about its functioning

<img width="911" height="390" alt="c3" src="https://github.com/user-attachments/assets/a09fd41c-f824-4149-a0aa-e933f447100a" />

So, I started investigating the debugger tab of the inspect element, and I found the file being 
referrenced in the HTML code, on carefully looking at the function being called in the HTML code, I
quickly noticed the use of a dangerous function `eval()` in the function executed.

The reason, `eval()` function is so dangerous, because it accepts the string and executes it, if 
the attacker sends an unsafe input and if not filtered, it could execute it thus an huge opportunity
for attacker to leak sensitive information. 

<img width="909" height="388" alt="c4" src="https://github.com/user-attachments/assets/d85a0594-192d-439f-8b90-38877f90feed" />

By now, I have identified the source and the sink and where the vulnerability lies, the only thing 
which was remaining to check what is the response from the server after searching for something
using the search functionality or the search box. This was fairly easy, all I needed to do was check
for the response that the web application's search functionality makes a GET request to:

<img width="910" height="390" alt="c5" src="https://github.com/user-attachments/assets/94d46f11-60e3-4348-8482-c36f2a554c18" />

At this point, I was ready for exploiting the vulnerability which would let me to execute a user
controlled dangerous function on the web application 

#### 3. Exploitation

Before we jump straight to exploitation, it is crucial to understand the chain of vulnerability that 
exists. 

In the first step, we as the user search using the search box/function, and our search query gets
sent to the server, the server sends a JSON response for our query but at this point it is still 
safe because our search query is in between the double quote (""), however, if we can somehow, 
escape the double quotes, than we can make `eval()` execute our dangerous function. 

Step 1:- Bypassing the quotes

This is the single most biggest hurdle, the developers are smart, they implemented safety checks, 
if we try to use `"` to escape the quotes, they implement an outer new quotes thus still making sure
that our dangerous function is still inside a string, and if we try to use `\` to escape the 
quotes we use in our payload in order to fool the safety checks that we're still using the quotes
when in reality they meant nothing, this is also spoiled by the server by introducing another 
backslash `\\`. So after, lots of trial & error, I came up with this payload 

Payload: \\"-alert(1)}//

<img width="906" height="733" alt="c6" src="https://github.com/user-attachments/assets/7a1b8c11-ea0f-4dad-863f-28476b08f995" />

Step 2: - Understanding what happens next 

Now our payload goes to the server and the server sends a JSON response 

```JSON
results: []
searchTerm: "\\"-alert(1)}//
```
<img width="913" height="388" alt="c7" src="https://github.com/user-attachments/assets/38b6b6d2-9f40-4fb8-b97b-3df37a5bf734" />

As you can notice, we bypassed the double quotes security checks, now it gets to the dangerous
function `eval()` 

```JavaScript
eval('var searchResultsObj = ' + {"results": [], "searchTerm": "\\"-alert(1)}//}
```

And since we already bypassed the double quotes earlier, our function gets executed by the `eval()`
function and we get a prompt alert.

