# Lab: DOM XSS in AngularJS expression with angle brackets and double quotes HTML-encoded

**Difficulty:** Practitioner
**Vulnerability:** Client-Side Template Injection (CSTI)
**Lab Link:** https://portswigger.net/web-security/cross-site-scripting/dom-based/lab-angularjs-expression

#### 1. The Goal 

The objective of this lab was to exploit a Cross-Site Scripting (XSS) vulnerability in a search
function.

The challenge was that the application HTML-encodes angle brackets ('<>') and double quotes (`"`), 
preventing standard HTML injection. My goal was to leverage the **AngularJS** framework to 
execute JavaScript without using standard HTML tags.

#### 2. Reconnaissance

In this lab, it was given the web application implements AngularJS framework so as always, on 
opening the lab, I was greeted with a blog website which had the search function

<img width="905" height="731" alt="b1" src="https://github.com/user-attachments/assets/eadde648-f6f0-47f0-9364-fcddc6798a95" />

Since, I knew beforehand that it implements AngularJS, I tried to see if I passed this `{{ 7+7 }}`
would I get 14 as result or will it be printed to the screen of the web application as it is. The
`{{ ... }}` can be used for execution, if a function, or an expression is included between the curly-
braces, it gets executed, or solved and the final result is shown as the output. To confirm, if 
it really does exists, I tried a test payload and the web application was indeed had AngularJS 
framework implemented underneath 

<img width="906" height="733" alt="b2" src="https://github.com/user-attachments/assets/4d70a78e-3b20-4874-b514-2b4b36f1e56e" />

Since, it executed successfully, I just wanted to test the waters, so I tried to see if I can 
execute the alert(1) function directly, that would make things easier for me, but sadly it didn't 
happen and it didn't got executed as I expected it 

<img width="905" height="734" alt="b3" src="https://github.com/user-attachments/assets/fded5d4f-acc3-4877-9d98-dc97c0846e85" />

Since, the earlier function didn't work, which I had already expected to fail, since in AngularJS 
framework, window level functions like that of alert() are usually restricted, therefore we would 
need something else that could help us to escape this restriction. So I tried to see what is my current
status within this AngularJS environment, which can be checked easily by executing `**{{ this }}**`
which would tell us what we are in this environment.

<img width="905" height="728" alt="b4" src="https://github.com/user-attachments/assets/91bbf7b9-8625-4d41-b2c3-c573d8122679" />

On executing `**{{ this }}**`, I found that we are currently an object in the current environment. 
This is actually a great thing, since every object in the JavaScript has a constructor property.
This means that the current object created was done using a certain function, and to know about the
constructor properties of this function or the constructor properties that this object has, we can
check by executing `**{{ constructor }}**` which tells us about the properties of the function which 
was used to create this object.

<img width="907" height="732" alt="b5" src="https://github.com/user-attachments/assets/bcd001e2-de6e-479d-9008-21c2b68262f4" />

This is actually fantastic, since we can see the properties of the function used to create this object
our next goal is to try seeing what is the property of the function that created this function that
created the object we are now, or in other words, we are trying to see the properties of the ultimate
or master function, if we are able to find this master function, there is a probability that we can
use this master function to build our own function that could help us to escape the restrictions.
This is rather easy, to check the properties of the function of the function, all we need to do is 
``**{{ constructor.constructor }}**`` 

<img width="905" height="732" alt="b6" src="https://github.com/user-attachments/assets/ac8b61d8-a617-49d7-9e9e-03aa218f5e42" />

**BOOM!** We found the master function Function(), JavaScript has a default function which is used
for creating another functions which in turn is used for creating objects, and this is a dangerous
function, since it allows string input execution, all an attacker needs to do is, bypass the security
checks put in place by the developers and once done, it executes the string command, which is what
we're going to do now. 

#### 3. Exploitation

We have figured out the vulnerable function which is Function() which would execute the dangerous
functions and the second challenge is escaping the input validation checks implemented by the 
developers. 

Step 1: Bypassing the validation checks

The developers implemented a validation check, that '<' or '>' or '"' will get encoded by the 
server, so we need to come up with a payload that doesn't include either of these expressions. Our
best candidate payload for this could be a simple alert(1) function, it doesn't include any of the
expressions checked for validation plus that is also the objective of this lab 

Step 2: Now the payload alone even if bypassed the security/validation checks is not enough if there
is nothing that will execute it, this is where our Function() comes into play, using 
constructor.constructor, we will be asking the master function to build us a function that contains
only one expression, alert(1) 

Step 3: We bypassed the validation checks, we built our own custom function using the master function, 
but it is still incomplete if there is no one that calls this function and it gets executed. Therefore, 
this step can be easily completed by simply adding () after the function for function calling. 

Now, our final payload would look like 

Payload: `{{ constructor.constructor('alert(1)')() }}`

<img width="904" height="733" alt="b7" src="https://github.com/user-attachments/assets/89c40c74-088c-4ccc-a6d8-f6b63ab23f2c" />

Finally, we have exploited the vulnerable function and solved the lab.
