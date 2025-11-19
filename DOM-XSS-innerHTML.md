# Lab: DOM XSS in innerHTML sink using source location.search

**Difficulty:** Apprentice

**Vulnerability:** DOM-based Cross-Site Scripting (XSS) via innerHTML

**Lab Link:** https://portswigger.net/web-security/cross-site-scripting/dom-based/lab-innerhtml-sink

#### 1. The Goal

The objective of this lab was to exploit a DOM XSS vulnerability in
the search blog functionality.

My goal was to inject a payload into the `innerHTML` sink that triggers
the `alert()` function.

#### 2. Reconnaissance

Upon starting the lab, we see a blog web application which has a 
search functionality, differnt blog posts on the web application. As
the lab hinted that the source of the input was location.search, 
which is used for getting the input from the URL, I started off with
the search functionality presnt in the web application. 

<img width="904" height="734" alt="y1" src="https://github.com/user-attachments/assets/55dfd976-979f-4ad1-afba-1d13a8f099ba" />

For testing purpose, I started with capturing the search request
made by the web application using Burp suite and inside the repeater 
tab to continue to try & experiment with finding the vulnerability, if 
it exists. On careful investigation, I found a JavaScript function
in the server's response which was vulnerable. 

<img width="1138" height="779" alt="y3" src="https://github.com/user-attachments/assets/f58e706c-6bd7-4c03-8208-59b5a3768361" />

The function implements `innerHTML` which is a dangerous function
and could be used to write dangerous function into the HTML code which 
could do harmful things. 

#### 3. Exploitation

For exploiting the vulnerability which exists in the web application, 
we can break it into two steps, step one implements passing the 
payload to the source which accepts input from user controlled
source. Step two is the indirect step where the sink accepts the 
input from the source and if vulnerable, which in this case it is,
executes the dangerous function. 

Step 1: The source

```
var query = (new URLSearchParams(window.location.search)).get('search');
```

In this above piece of code which is the source, it accepts the 
input from the URL, especially from the `search` parameter's value,
since there was no other validation checks implemented by the developers,
we can go with a simple payload like <script>alert(1)</script>

Step 2: The sink

```
function doSearchQuery(query) {
  document.getElementById('searchMessage').innerHTML = query;
}
```

This is the sink in the web application, which is going to pass our
user controller input via the `query` variable and executes the 
dangerous function
