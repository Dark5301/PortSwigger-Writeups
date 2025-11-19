# Lab: DOM XSS in jQuery anchor href attribute sink using location.search source

**Difficulty:** Apprentice

**Vulnerability:** DOM-based Cross-Site Scripting (XSS) via jQuery

**Lab Link:** https://portswigger.net/web-security/cross-site-scripting/dom-based/lab-jquery-href-attribute-sink

#### 1. The Goal 

The objective of this lab was to execute a Cross-Site Scripting
attack by exploiting a vulnerability in how the application handles
navigation links.

My goal was to inject a malicious JavaScript protocol (`javascript:`)
into an anchor tag's `href` attribute, causing the `alert()` function
to execute when the link is clicked.

#### 2. Reconnaissance 

Upon starting the lab's web application, we are greeted by a blog 
website which has different blog posts and offers a feedback feature.

<img width="904" height="731" alt="z1" src="https://github.com/user-attachments/assets/f561d081-1b88-4d26-80f2-c12d2c1a6fee" />

I started exploring the different functionalities & features, this
web application offers, and one interesting feature was the feedback
feature. Upon inspecting the feeback request through Burp Suite's
Repeater, I noticed that it implements the .attr(href) function.

It is crucial to understand the dangers of this function, href tag,
doesn't only hold the URLs but it can also be used to hold JavaScript
commands and in this case, it tries to locate the JavaScript command
in its code and in case it doesn't find the exact match, it writes
the command given to it. This could be especially dangerous in the
the situation where the user input is not validated properly and could
be used to write dangerous functions or perform malicious actions.

<img width="1139" height="782" alt="z3" src="https://github.com/user-attachments/assets/8f2b81ec-6d12-4315-8bf4-5c7e4c74a672" />

### 3. Exploitation 

The vulnerability was very clear in this situation, there was no 
validation checks implemented to validate the input accepted from the
users, the goal of exploitation was to make href store our JavaScript
command which in our case is alert(1) and if it doesn't find any 
occurrence of this command, than write it and execute it. 

For this case, we can use a very simple payload, that denotes that
it is a JavaScript command that we wish href to store and get it
executed 

Payload: javascript:alert(1)

That successfully exploits the vulnerability present in the web
application

