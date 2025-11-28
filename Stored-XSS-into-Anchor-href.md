# Lab: Stored XSS into Anchor href Attribute with Double Quotes HTML-Encoded

**Lab Difficulty:** Apprentice 

**Topic:** Cross-Site Scripting (Stored)

##### 1. Objective

The objective of this lab is to perform a Stored Cross-Site Scripting (XSS)
attack via the comment functionality of a blog post. The goal is to inject
a malicious payload that calls the `alert` function when a user interacts 
with the injected element. 

<img width="1184" height="741" alt="b1" src="https://github.com/user-attachments/assets/7d74876b-895f-408e-9044-918f2134a349" />

##### 2. Reconnaissance & Analysis 

Identification of Input Vectors

We began by analysing the comment submission form on a blog post. The form 
accepts four inputs:

1. Comment

2. Name

3. Email

4. Website

We mapped where each input is reflected in the HTML response. The 
"Comment" body is standard text, but the "Name" field is rendered as a 
hyperlink (`<a>` tag) pointing to the URL provided in the **Website** 
field.

Analysing the Reflection

To understand how the data is handled, we submitted a generic comment
with the website set to `https://www.example.com`.

Observation: Inspecting the server response in Burp Suite revealed that the
website input is placed directly into the `href` attribute of the 
author's anchor tag.

<img width="1440" height="876" alt="b2" src="https://github.com/user-attachments/assets/b2f0cefe-b5f5-4d4c-9f44-b77cb862f124" />

Vulnerability Assessment 

The lab descripting states that double quotes are HTML-encoded. This means
we cannot break out of the `href` attribute using a payload like `"> <script>`.

However, because we control the content of the `href` attribute, we do need
to break out of the tag to execute JavaScript. The `href` attribute supports
the `javascript:` pseudo-protocol. If the application fails to validate the 
protocol scheme (e.g., ensuring it starts with `http://` or `https://`),
we can inject executable JavaScript directly. 

##### 3. Exploitation 

Payload Construction

Since we are inside an `href` attribute and cannot use double quotes, our
strategy is to change the protocol of the link.

Payload:

```
javascript:alert(1)
```
