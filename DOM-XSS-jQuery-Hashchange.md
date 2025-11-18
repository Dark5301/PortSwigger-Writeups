# Lab: DOM XSS in jQuery selector sink using a hashchange event

**Difficultly:** Apprentice

**Vulnerability:** DOM-based Cross-Site Scripting (XSS) via jQuery

**Lab Link:** https://portswigger.net/web-security/cross-site-scripting/dom-based/lab-jquery-selector-hash-change-event

#### 1. The Goal

The objective of this lab was to exploit a DOM XSS vulnerability on the home
page. The application uses the jQuery library to handle navigation via the URL
hash.

My goal was to deliver an exploit to a victim that forces their browser to 
execute the `print()` function.

#### 2. Reconnaissance

Upon starting the lab, I was greeted with a blog website which had different 
posts and offers the comment functionality and a search functionality. I started
off by testing the search functionality.

<img width="906" height="734" alt="a1" src="https://github.com/user-attachments/assets/0f2cdd84-e1a8-4207-97e6-ed7f5c6cfc19" />

After using the search feature available on the web application, I checked 
in the inspect element, in case it dynamically loads any specific script or 
feature that could help me to identify the vulnerability, and surprisingly 
enough, it does loads up a JavaScript script which is vulnerable.

<img width="908" height="388" alt="a2" src="https://github.com/user-attachments/assets/80541329-7114-4c9f-87f9-58ab9ad224dd" />

```JavaScript
$(window).on('hashchange', function(){
  var post = $('section.blog-list h2:contains(' + decodeURIComponent(window.
location.hash.slice(1)) + ')');
  if (post) post.get(0).scrollIntoView();
});
```

Let use try to understand the above vulnerable function:

The above function through its **window.location.hash** accepts the input from
the URL after the hash, so if we had an example URL, https://example.com?#has
than everything after '#' will be considered its value including the hash, the
slice(1) makes sure to only use anything after '#' as the input. 

This could almost be used for exploitation but there is still one thing to know
about, the 'hashchange' rule, this ensures that the function gets executed, 
only if there is a change in the hash, which means that if we have the same
above example URL, https://example.com?#hash than it must change to 
https://example.com?#hash2. This change in the hash will cause this function
to execute which could be dangerous. 

### 3. Exploitation

Now comes the fun and interesting part of exploiting the vulnerability, we can
divide the exploitation into two parts, the first part deals with bypassing 
the safety checks put in by the developers and the second part to change the 
hash to make the function execute. 

Step 1: Bypassing the sink 

The sink in our case here is 'window.location.hash.slice(1)' which accepts the
input after the '#'. In the above function, the sink is bounded between two
single quotes

var post = $('section.blog-list h2:contains(' + decodeURIComponent(window.
location.hash.slice(1)) + ')');

Let's start with the payload ')<img src=x onerror=print()>'

$('section.blog-list h2:contains(' + ')<img src=x onerror=print()>' + ')')

and it becomes 

$('section.blog-list h2:contains() <img src=x onerror=print()')')

So, this could be used to bypass this which solves the first problem. 

Step 2: Bypassing the hash change to get function to execute 

We can use **<iframe>** to load the website which is vulnerable to hash change
which in our case would be our web application's home page search feature with
an empty hash. 

Next, once it loads the website, we can use the onload tag to make it do 
what actions it needs to perform once the web applications loads, in this case
we can add our earlier payload from step 1

Now, if we combine both of them, we get this as the payload

payload = <iframe src="https://0a0900a4030c8b5880d8f07d002a0071.web-security-academy.net/#"
onload="this.src += ')<img src=x onerror=print()>'">
</iframe>

Using this payload, we can successfully exploit the vulnerability

<img width="903" height="731" alt="a4" src="https://github.com/user-attachments/assets/6a8e6e92-0891-4215-99e9-f61004c8ffa0" />


