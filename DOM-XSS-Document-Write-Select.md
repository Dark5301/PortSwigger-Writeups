# Lab: DOM XSS in document.write sink using source location.search inside a select element

**Difficulty:** Apprentice

**Vulnerability:** DOM-based Cross-Site Scripting (XSS) via document.
write

**Lab Link:** https://portswigger.net/web-security/cross-site-scripting/dom-based/lab-document-write-sink-inside-select-element

#### 1. The Goal 

The objective of this lab was to exploit a DOM XSS vulnerability 
where user input is written dynamically into a specific HTML 
element.

My goal was to identify the context of the injection (inside a 
`<select>` menu), break out of that context, and execute the `alert()`
function.

#### 2. Reconnaissance

I started by checking the "Check Stock" functionality on a product
page. I noticed that after checking stock in a specific city, the 
URL contained a `storeID` parameter.

<img width="906" height="734" alt="x1" src="https://github.com/user-attachments/assets/d59f95d0-c83c-40c0-a2f3-177a19bf3228" />

Upon carefully inspecting the request in Burp Suite, I found the
use of an unsafe way of accepting the input from the users without
validating it for any harmful expressions that might be there.

<img width="1139" height="780" alt="x4" src="https://github.com/user-attachments/assets/39bf81a7-cf95-44e5-b0b6-294f33d804d2" />

In the above screenshot, this piece of code 

```JavaScript
var store = (new URLSearchParams(window.location.search)).get('storeId');
document.write('<select name="storeId">');
if(store) {
  document.write('<option selected>'+store+'</option');
}
for(var i=0;
i<stores.length;
i++) {
  if(stores[i] === store) {
    continue;
  }
  document.write('<option>'+stores[i]+'</option>');
}
document.write('</select>');
```

This line in the above code 

```JavaScript
var store = (new URLSearchParams(window.location.search)).get('storeId');
```

Expects a parameter `storeId` in the URL which initially is not there
when checking for the stocks in the web application, so we can add
this parameter in the URL from ourselves using `&storeId=`.

The very next line 

```JavaScript
document.write('<select name="storeId">');
```

and the last line

```JavaScript
document.write('</select>');
```

This is what the lab also hinted, that this new parameter which accepts 
the user input is stuck between the `<select> ... </select>` tags,
and unless we escape these tags, we cannot get our alert(1) to get
executed.

#### 3. Exploitation

As we already mentioned in the reconnaissance step also how the lab
hinted that the dangerous write function is included between the 
`<select> ... </select>` tag and our primary job is to be able to 
escape this and once we escape this tag we can get our command to be
executed. 

And the next thing is that the user controlled input needs to be sent
via a parameter, `storeId` where we are going to send our payload.

Step 1: Add the new parameter

We will add the new parameter in the URL along with productId using
&storeId and add our payload

Step 2: Escape the <select> tag

We can escape this simply closing the tag </select> and after this
we can add our own payload that we want to get it executed. 

Based on the above two conditions, we can use a payload like this 
one

Payload: &storeId=</select><script>alert(1)</script>

We successfully exploited the vulnerability that existed in the
web application.

<img width="902" height="731" alt="x3" src="https://github.com/user-attachments/assets/b90bc21c-2025-471d-b607-382281bdc549" />
