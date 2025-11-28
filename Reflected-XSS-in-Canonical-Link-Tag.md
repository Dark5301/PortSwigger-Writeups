# Lab: Reflected XSS in Canonical Link Tag

**Lab Difficulty:** Apprentice

**Topic:** Cross-Site Scripting (Reflected)

##### 1. Objective

The objective of this lab is to perform a Reflected Cross-Site Scripting
(XSS) attack. The vulnerability exists within the HTML `<link>` tag used
for canonical URL definition. The goal is to inject a payload that
executes the `alert` function. 

<img width="1186" height="746" alt="c1" src="https://github.com/user-attachments/assets/86b37cea-b306-4aad-9786-e46f5c6b1c89" />

##### 2. Reconnaissance & Analysis 

Identification of Reflection Point 

We started by navigating to the blog posts and observing how the URL 
parameters affect the HTML source. The application uses a "canonical" link
tag to help search engines understand the preffered URL for a page.

Upon inspecting the source code for a post (e.g., `/post?postId=4`), we
noticed that the entire URL, including query parameters, is reflected inside
the `href` attribute of the `<link rel="canonical">` tag.

<img width="1440" height="876" alt="c2" src="https://github.com/user-attachments/assets/ec5ed6e7-a449-4668-8e62-f765d0f97b07" />

Vulnerability Assessment 

The reflection occurs inside a single-quoted string:

```
<link rel="canonical" href='.../post?postId=4'/>
```

If the application fails to sanitise single quotes, we can break out of 
the `href` attribute. However, since this tag resides in the `<head>`
section of the HTML document and is not a visible element, standard event
handlers like `onmouseover` or `onfocus` will not trigger automatically or
via mouse interaction. 

To exploit this, we need an attribute that allows user interaction via the
keyboard. The `accesskey` attribute is suitable here, as it allows us to 
assign a keyboard shortcut to the element. 

##### 3. Exploitation 

Payload Construction 

We crafted a payload to:

1. Break Out: Use a single quote `'` to close the `href` attribute.

2. Add Interaction: Inject the `accesskey='x'` attribute. This allows us to
target the hidden element by pressing a key combination (e.g., `ALT+X` on
Windows, `CTRL+ALT+X` on macOS).

3. Add Trigger: Inject `onclick='alert(1)'`. When the access key is pressed,
the "click" event fires, executing our JavaScript.

Payload: 
```
' accesskey='x' onclick='alert(1)
```

Full URL:
```
/?postId=3&x=' accesskey='x' onclick='alert(1)
```

Execution 

We sent the request via Burp Suite to verify the injection structure.

<img width="1440" height="876" alt="c3" src="https://github.com/user-attachments/assets/45597426-c906-4c4e-b97d-3041d2a8825f" />
<img width="1200" height="794" alt="c4" src="https://github.com/user-attachments/assets/a322f608-3546-4845-8541-1f7281a03471" />

As seen in the response:

```
<link rel="canonical" href='...postId=3&x=' accesskey='x' onclick='alert(1)'/>
```
