# Lab: DOM XSS in document.write sink using source location.search

**Difficulty:** Apprentice

**Vulnerability:** DOM-based Cross-Site Scripting (XSS) via 
document.write

**Lab Link:** https://portswigger.net/web-security/cross-site-scripting/dom-based/lab-document-write-sink

#### 1. The Goal

The objective of this lab was to exploit a DOM XSS vulnerability
found in the search query tracking functionality.

My goal was to identify where user input was being written into
the DOM, understand the specific context (inside an HTML 
attribute), and craft a payload to break out of that attribute
to execute the `alert()` function. 

#### 2. Reconnaissance

I started by entering a generic string (`test123`) into the
search bar.

I inspected the page source to see where this string appeared. I
found it inside an `<img>` tag that looked like a tracking 
pixel.

<img width="854" height="733" alt="1 1 1" src="https://github.com/user-attachments/assets/aead1b26-afbb-4c33-a3fa-ac9bcf540e8b" />

<img width="854" height="734" alt="2 2 2" src="https://github.com/user-attachments/assets/b2ccd258-7d42-4f72-a95b-f101cd6f8140" />

<img width="913" height="358" alt="3 3 3" src="https://github.com/user-attachments/assets/da42cad3-44cd-4176-8d0c-d2a04e44824d" />

We can notice that this specific line of code directly accepts
the user input without doing any input validation checks which
could allow the malicious users to execute dangerous functions.

```JavaScript
<img src="/resources/images/tracker.gif?searchTerms=test123">
```

#### 3. Exploitation

The exploitation for this vulnerability is to be able to escape
from the above line of code and also escape the `img` tag,once
we escape from this, we can run our own dangerous functions or in 
this case execute the alert(1)

The complete function response from the server which was the 
vulnerable sink is as follows:

<img width="1139" height="778" alt="6 6 6" src="https://github.com/user-attachments/assets/297b8e6b-85dc-466d-b15e-6fc71e1204e5" />

```Javascript
function trackSearch(query) {
  document.write(
  '<img src="/resources/images/tracker.gif?searchTerms='+query
  +'">');
}
var query = (new URLSearchParams(window.location.search)).get('
search');
if(query) {
  trackSearch(query);
}
```

The web application's search functionality accepts the input 
directly from the user controlled input space.

```JavaScript
var query = (new URLSearchParams(window.location.search)).get('
search');
```

This accepts the input from the URL after the `search` parameter.
Here, we can input our payload.

And the sink where our input gets written is this specific line
of code 

```JavaScript
document.write('<img src="/resources/images/tracker.gif?searchTerms='
+query+'">');
```

We can try to bypass this by using a payload like this 

Payload: `"><script>alert(1)</script>`

And we successfully exploited the vulnerability which exists
in the web application

<img width="854" height="733" alt="4 4 4" src="https://github.com/user-attachments/assets/3af306ea-4e70-4721-a51b-0d9291b1380f" />
