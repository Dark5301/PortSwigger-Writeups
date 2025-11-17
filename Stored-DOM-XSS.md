# Lab: Stored DOM XSS
**Difficulty:** Practitioner

**Vulnerability:** Stored DOM-based Cross-Site Scripting (XSS)

**Lab Link:** https://portswigger.net/web-security/cross-site-scripting/dom-based/lab-dom-xss-stored

#### 1. The Goal

The objective of this lab was to exploit a Stored DOM XSS vulnerability in the blog's comment functionality. 

My goal was to submit a malicious payload that would be stored by the server, bypass a weak client-side filter (`escapeHTML`), and execute an `alert()` function by leveraging a dangerous `innerHTML` sink.

#### 2. Reconnaissance

On starting the lab, I was greeted by a blog page which had comment features for each of the posts 

<img width="906" height="734" alt="d1" src="https://github.com/user-attachments/assets/3c1c47da-51ad-4696-b137-d16f38d8aa78" />

After I have added the comment on one of the post, I opened the 'inspect element' to check for all the resources
and files being requested by the web application, after adding the comment to the post, and I found an interesting
JavaScript file being requested by the comment feature of the web application.

<img width="909" height="384" alt="d2" src="https://github.com/user-attachments/assets/11fdf417-f390-4636-b9d4-38ae22adc8d8" />

As, it's clear from the screenshot, the function escapeHTML is a santisier whose job is to make sure that if it 
encounters dangerous expressions like '<' or '>' it will replace it with their encoded versions to keep the 
web application safe, but here's where the vulnerability lies, the function only gets executed once, which means 
that if it encounters a payload which has '<>' that's enough to bypass the safety check filters. This is the source
which takes the user input and tries its best to filter out dangerous expressions but the safety filters are not
strong enough and can be bypassed very easily. 

<img width="912" height="381" alt="d3" src="https://github.com/user-attachments/assets/53de0146-6c62-4a6a-bc8f-1a5b60fc3178" />

Inside the same file, I also found the dangerous sink, `innerHTML`, which can be used for executing the dangerous 
functions which is user controlled after the user bypasses the safety filters 

#### 3. Exploitation
First we need to bypass the safety checks deployed by the developers of the web application, especially this
function

```JavaScript
function escapeHTML(html) {
  return html.replace('<', '&lt;').replace('>', '&gt;');
};
```

To bypass this safety checks, all we need to do is include '<>' in the beginning of our payload. Since, the function
executes only once, it will check for dangerous expression and once it sees '<>' it converts it '&alt;&gt;' and 
after this we can include our payload. 

The next hurdle is this piece of code 

```JavaScript
commentBodyPElement.innerHTML = escapeHTML(comment.body)
```

Now, since the developers have used an unsafe method of writing into the sink, `innerHTML` we can use `<img>` or 
`<iframe>` to try and write our own payload into the sink. One of the most common payload which we can use for our
task is 

`<img src=x onerror=alert(1)>`

For our final payload,

`<><img src=x onerror=alert(1)>` 

<img width="906" height="731" alt="d4" src="https://github.com/user-attachments/assets/1dbd940a-67f4-47c5-9c47-40abd39403a5" />

Finally, we bypassed the safety checks and exploited the vulnerability 

<img width="906" height="733" alt="d5" src="https://github.com/user-attachments/assets/886a18ee-ad87-42d3-961c-dc582c62104f" />
